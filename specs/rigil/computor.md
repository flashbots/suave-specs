---
title: Computor
description: SUAVE computors contain all necessary components to accept, process, and route confidential compute requests and results.
---

<!-- TOC -->

- [Computor](#computor)
    - [Overview](#overview)
    - [Prerequisites](#prerequisites)
    - [Computor Responsibilities](#computor-responsibilities)
    - [Becoming a Computor](#becoming-a-computor)
        - [Computor Identification](#computor-identification)
    - [Computor Architecture](#computor-architecture)
        - [RPC](#rpc)
        - [SUAVE PoA Chain](#suave-poa-chain)
        - [MEVM](#mevm)
        - [Confidential Data Store](#confidential-data-store)
        - [Domain Specific Services](#domain-specific-services)
        - [Precompiles](#precompiles)
    - [Containers](#containers)
        - [Confidential Compute Record](#confidential-compute-record)
        - [ConfidentialComputeRequest](#confidentialcomputerequest)
        - [Suave Transaction](#suave-transaction)
    - [Confidential Computation](#confidential-computation)
        - [Confidential Compute Process](#confidential-compute-process)
    - [Honest Computor](#honest-computor)
    - [Suave JSON-RPC](#suave-json-rpc)

<!-- /TOC -->

</div>

## Overview

This document provides the technical specification for the SUAVE computor: one of the main protocol actors in the SUAVE protocol. The computor contains all necessary components to accept, process, and route confidential compute requests and results. With these basic primitives computors can self-organize their topology and functionality to conform to specific portions of the MEV supply network.

This document includes the expected behavior of an "honest computor" with respect to Rigil testnet version of the SUAVE protocol. This document does not distinguish between a "SUAVE computor" and a "SUAVE validator client". The separation of concerns between these two pieces of software is left as a design decision that is out of scope.

## Prerequisites

All terminology, functions, and protocol mechanics defined in [SUAVE chain](./suave-chain.md), [Confidential Data Store](./confidential-data-store.md), and [MEVM](./mevm.md) are prerequisites for this document and used throughout. Please see the Rigil overview before continuing and use as a reference throughout.

## Computor Responsibilities

A computor has several primary responsibilities within the SUAVE network:

1. **Handling Confidential Compute Requests:**
    - Process `ConfidentialComputeRequest` received from users or other network nodes.
    - Execute transactions in confidential mode, providing access to the usual confidential APIs.
    - Create a `SuaveTransaction` using the confidential computation request and the result of its execution.
    - Sign and submit the transaction into the SUAVE mempool.

2. **Maintaining the Confidential Data Store:**
    - Store and retrieve confidential data related to bids.
    - Restrict access to stored data based on the allowed peekers and stores of each bid.
    - Ensure synchronization of confidential data across the network.

3. **Propagating Confidential Compute Requests and Results:**
    - Broadcast the results of confidential computations to the appropriate parties within the SUAVE network along with any bundles or blocks emitted as result of confidential computation.
    - Ensure the propagation of results adheres to the confidentiality and privacy requirements of the SUAVE protocol.

## Becoming a Computor

Computors are currently permissioned in the SUAVE network, to become one you need your computor's ECDSA pubkey to be contained within the clique PoA genesis settings.

### Computor Identification

A unique identifier in the form of Ethereum compatible public key must be generated for each computor to ensure proper tracking and verification of Confidential Computation, as well as responsibility assignment within the network.

## Computor Architecture

SUAVE Computors house all components necessary to perform confidential computation and routing. Below is a high level architectural diagram followed by descriptions of the main components.

![Rigil architecture](/assets/rigil-architecture.svg)

### RPC

This component is similar to RPCs in any ethereum or blockchain client and are responsible for mapping user's requests to their appropirate functionality.

### SUAVE PoA Chain

The SUAVE chain is maintains consensus about smart contract code for use cases such as order flow auctions, solvers, block builders, etc. Additionaly it can also be used to store and/or broadcast data for better censorship guarantees.

Computors are required to keep a lively copy of the SUAVE chain state in order to process Confidential Compute Requests. Liveliness requirements are not defined in the Rigil Computor but will in the future.

For more details see the [🔗 SUAVE chain](./suave-chain.md) spec.

### MEVM

MEVM is a modified version of the EVM which includes a new runtime, interpreter, and execution backend. Confidential Compute Requests are ultimately handled by the MEVM which has an interface into the confidential data store and leverage a set of new precompiles tailored for MEV applications.

For more details see the [🔗 MEVM](./mevm.md) spec.

### Confidential Data Store

The Confidential Store serves as a secure and privacy-focused storage system, exposing a key-value store for safeguarding confidential compute and orderflow related data. Only those with appropriate permissions (peekers) can access the stored data, thus ensuring privacy and control.

For more details see the [🔗 Confidential Data Store](./confidential-data-store.md) spec.

### Domain Specific Services

SUAVE computors have the potential to interconnect with numerous domains, facilitating a more extensive and diverse network of interactions. This connectivity underscores the extensible nature of SUAVE computors, promoting interoperability across various domains, focusing on EVM based domains at first.

Domain Specific Services allow computors to scale horizontally by hosting Nodes for other domains in a separate process that can be queried at confidential compute time. Computors currently are not responsible for publicly committing to their domain specific services so support for a specific domain is done on computor-by-computor basis. In the event a computor attempts to process your confidential compute request for a domain it does not support it will simply return a failure on the computation and will not propagate the failure to the rest of the network so other computors can still attempt to process.

Eventually domain specific services may be recorded onto the SUAVE chain and the network topology will route your request to a computor offering such domain specific services.

For more details on how to support the needed APIs to enable a domain see [🔗 SUAVE Execution API](./confidential-data-store.md) spec.

### Precompiles

For more details see the [🔗 Precompiles](./precompiles.md) spec.

## Containers

The core data types used inside the computor.

### Confidential Compute Record

This type serves as a record of computation. It's part of both the [Confidential Compute Request](#confidential-compute-request) and [Suave Transaction](#suave-transaction).

```go
type ConfidentialComputeRecord struct {
	Nonce    uint64
	GasPrice *big.Int
	Gas      uint64
	To       *common.Address
	Value    *big.Int
	Data     []byte

	ExecutionNode          common.Address
	ConfidentialInputsHash common.Hash

	ChainID *big.Int
	V, R, S *big.Int
}
```


### ConfidentialComputeRequest

This type enables users to requests the MEVM to compute over their data via the `eth_sendRawTransaction` method. After processing, the request's `ConfidentialComputeRecord` is embedded into `SuaveTransaction.ConfidentialComputeRequest` and serves as an onchain record of computation.


```go
type ConfidentialComputeRequest struct {
    ConfidentialComputeRecord
    ConfidentialInputs []byte
}
```

A computor's signature is used as the integrity gurantee for the computation's results. Eventually this can include arbitrary proofs such zero-knowledge proofs.

### Suave Transaction

The final home of compute results and intentionally leaked data from confidential compute requests is a SUAVE transaciton, see [🔗 SUAVE chain](./suave-chain.md) specs for more details

```go
type SuaveTransaction struct {
	ExecutionNode              common.Address
	ConfidentialComputeRequest ConfidentialComputeRecord
	ConfidentialComputeResult  []byte

	// ExecutionNode's signature
	ChainID *big.Int
	V       *big.Int
	R       *big.Int
	S       *big.Int
}
```

## Confidential Computation

To successfuly process a request for confidential computation Computors must engage the the Confidential Compute Process.

### Confidential Compute Process

Confidential compute is defined by use of an [offchain function call](https://docs.soliditylang.org/en/latest/contracts.html#view-functions), signified in solidity via the `view` modifier.

These view functions are used in conjunction with plaintext access to decrypted data to perform confidential computation on the decrypted data. From there results and a signature of integrity are propagated along with the initial request. The confidential data is propagated separately via the confidential data store, but other Computors do not need the underlying data as they will trust any valid signature attesting to the computation's integrity.

## Honest Computor

At present, the protocol relies on the honesty of computors, akin to the reliance on honest block builders and relays. An honest computor performs the aforementioned duties of:

- **Handling Confidential Compute Requests**
- **Maintaining Confidential Data Privacy**
- **Maintaining the Confidential Data Store**
- **Propagating Confidential Compute Results**

On the Rigil testnet Computors do not live inside of Trusted Execution Environments, and because of this, a malicious computor could alter it's source code to censor Confidential Compute Requests and their results. It is for this reason the initial Computor set is strictly maintained to trusted actors.





## Suave JSON-RPC

SUAVE JSON-RPC can be seen as a super set of Ethereum JSON-RPC. This means that the [Ethereum JSON-RPC standard](https://geth.ethereum.org/docs/interacting-with-geth/rpc) remains the same when interacting with the SUAVE chain, with the following exceptions:

1. Suave JSON-RPC has two modes of operation: regular and confidential determined by the truth value of `IsConfidential` in the Confidential Compute Request.
- *Regular mode* is equivalent to the usual Ethereum virtual machine environment, with all computation occurring onchain and requests are made with SUAVE transactions instead of Confidential Compute Requests.
- *Confidential mode* accesses additional precompiles, both directly and through a convenient [library](https://github.com/flashbots/suave-geth/blob/main/suave/sol/libraries/Suave.sol). Confidential execution is *not* verifiable during on-chain state transition. The result of the confidential execution is instead cached in the `SuaveTransaction`.

2. New optional argument - `confidential_data` - is added to `eth_sendRawTransaction`, `eth_sendTransaction` and `eth_call` methods.
- Confidential data is made available to the MEVM via a precompile, but does not become a part of the transaction that makes it to chain.

3. All RPCs that return transaction or receipt objects will do so with type `SuaveTransaction`, a super set of regular Ethereum transactions.
