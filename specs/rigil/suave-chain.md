# Suave Chain

<div class="hideInDocs">

**Table of Contents**

<!-- TOC -->

- [Introduction](#introduction)
- [Configuration](#configuration)
    - [Network Parameters](#network-parameters)
    - [Genesis Settings](#genesis-settings)
- [Consensus Mechanism: Proof-of-Authority (Clique)](#consensus-mechanism-proof-of-authority-clique)
    - [MEVM Execution](#mevm-execution)
    - [Geth Version](#geth-version)
- [Custom Types](#custom-types)
    - [Confidential Compute Record](#confidential-compute-record)
    - [Confidential Compute Request](#confidential-compute-request)
    - [Suave Transaction](#suave-transaction)
- [Suave JSON-RPC](#suave-json-rpc)
    - [`suavex` namespace](#suavex-namespace)
- [Node Requirements and Setup](#node-requirements-and-setup)
- [Gas and Transaction Fees](#gas-and-transaction-fees)
- [Security Considerations](#security-considerations)
- [Appendices](#appendices)

<!-- /TOC -->

---

</div>

## Introduction

This document outlines the specifications for the SUAVE Rigil testnet.

In the context of the SUAVE protocol, the main purpose of the SUAVE chain is to reach (and maintain) consensus about smart contract code for use cases such as order flow auctions, solvers, block builders, etc. 

The SUAVE chain can also be used to store and/or broadcast data for better censorship guarantees.

In the initial phases of development, the SUAVE chain runs a proof-of-authority consensus protocol called Clique, over a network of permissioned nodes. We do so in order to experiment and iterate quickly during protocol development. This will change in later testnets.

## Configuration

### Network Parameters

- **Network ID**: `16813125`
- **Chain ID**: `16813125`
- **RPC Endpoint**: `https://rpc.rigil.suave.flashbots.net`

### Genesis Settings

| Name | Value | Unit |
| - | - | - |
| `PERIOD` | 4 | `block`
| `EPOCH` | 30000 | `block`
| `BLOCK_TIME` | 3 | `second`
| `GAS_LIMIT`| 30000000 | `gwei`
| `NUM_VALIDATORS` | 3 | Nodes

## Consensus Mechanism 

**Proof-of-Authority**.

Clique, an Ethereum-based Proof-of-Authority consensus protocol defined [here](https://eips.ethereum.org/EIPS/eip-225#:~:text=A%20PoA%20scheme%20is%20based,the%20list%20of%20trusted%20signers), restricts block minting to a predefined list of trusted signers. Because of this, every block header that a client sees can be checked against the list of trusted signers.

### MEVM Execution

### Geth Version

Suave-geth is based on geth v1.12.0 ([`e501b3`](https://github.com/flashbots/suave-geth/commit/e501b3b05db8e169f67dc78b7b59bc352b3c638d)).

---

## Custom Types

The SUAVE protocol adds two main types to the base Ethereum protocol of which it is currently a fork. 

1. `ConfidentialComputeRequest`
2. `SuaveTransaction`

Taken together, these two form a `ConfidentialComputeRecord`, which is also specified below.

### Transaction-flow diagram

![Rigil transaction flow](/assets/rigil-tx-flow.svg)

Conceptually, transaction on SUAVE can be split into three distinct steps:

1. User -> RPC
2. RPC -> MEVM
3. MEVM -> Chain

### Confidential Compute Request

This type enables users to requests the MEVM to compute over their data via the `eth_sendRawTransaction` method. After processing, the request's `ConfidentialComputeRecord` is embedded into `SuaveTransaction.ConfidentialComputeRequest` and serves as an onchain record of computation.  

```go
type ConfidentialComputeRequest struct {
    ConfidentialComputeRecord
    ConfidentialInputs []byte
}
```

### Suave Transaction

A specialized transaction type that encapsulates the result of a confidential computation request. It includes the `ConfidentialComputeRequest`, signed by the user, which ensures that the result comes from the expected SUAVE computor, as the `SuaveTransaction`'s signer must match the `ExecutionNode`.  

```go
type SuaveTransaction struct {
    ExecutionNode              Address
    ConfidentialComputeRequest ConfidentialComputeRecord
    ConfidentialComputeResult  []byte

    // Signature fields
}
```
In the future the signature fields here will represent various different types of proof of computation and more.

### Confidential Compute Record

This type serves as an onchain record of computation. It's part of both the [Confidential Compute Request](#confidential-compute-request) and [Suave Transaction](#suave-transaction).  


```go
type ConfidentialComputeRecord struct {
    ExecutionNode          common.Address
    ConfidentialInputsHash common.Hash

    // LegacyTx fields
    Nonce    uint64
    GasPrice *big.Int
    Gas      uint64
    To       *common.Address `rlp:"nil"`
    Value    *big.Int
    Data     []byte

    // Signature fields
}
```

## Suave JSON-RPC

SUAVE JSON-RPC can be seen as a super set of Ethereum JSON-RPC. This means that the [Ethereum JSON-RPC standard](https://geth.ethereum.org/docs/interacting-with-geth/rpc) remains the same when interacting with the SUAVE chain, with the following exceptions:

1. New `IsConfidential` and `ExecutionNode` fields are added to the transaction arguments used in the `eth_sendTransaction` and `eth_call` methods.
    1. If `IsConfidential` is set to true, the call will be performed as a confidential call, using the SUAVE Computor passed in when constructing a `ConfidentialComputeRequest`.
    2. A `SuaveTransaction` is the result of `eth_sendTransaction`. If `IsConfidential` is unset or false in the request, this `SuaveTransaction` will be processed as a regular Ethereum Legacy or EIP1559 transaction.

2. New optional argument - `confidential_data` - is added to `eth_sendRawTransaction`, `eth_sendTransaction` and `eth_call` methods.
    1. Confidential data is made available to the MEVM via a precompile, but does not become a part of the transaction that makes it to chain.

3. All RPCs that return transaction or receipt objects will do so with type `SuaveTransaction`, a super set of regular Ethereum transactions.

See [Appendix B](#appendix-b) for an example of confidential compute request and response objects.

### `suavex` namespace

Any API endpoints defined in this namespace are used internally to connect the MEVM node and the SUAVE-enabled node. We take this approach to make upstream updates and maintenance easier. Current endpoints include:

* `suavex_buildEthBlockFromBundles` - takes an array of bundles and transactions, calculates state root and related fields, and returns a valid Ethereum L1 block.

* `suavex_buildEthBlock` - takes an array of transactions, calculates state root and related fields, and returns a valid Ethereum L1 block.

---

## Node Requirements and Setup

- **Hardware**:
    - Minimum 8GB RAM, four cores, 50GB SSD (How big do we expect the chain to grow?)
    - These requirements will eventually incorporate Trusted Execution Environments (TEEs).
- **Software**: [suave-geth](https://github.com/flashbots/suave-geth/)
- **Setup Steps**:
    1. Clone suave-geth
    2. Start the devnet: `make devnet-up`
    3. Create transactions: `go run suave/devenv/cmd/main.go`

---

## Gas and Transaction Fees

The SUAVE chain employs the same gas pricing mechanism as Ethereum pre-Cancun hardfork (no blob transactions) where gas prices adjust based on network demand. Nodes currently track Confidential Compute Request gas usage, but do not charge (??? I believe correct @ MM?).

todo: clarify usage of legacyTx + baseFee

---

## Security Considerations

- **Security Risk**: The protocol is unaudited. The protocol currently does not make any guarantees about the confidentiality of data in the network outside of a best effort.
- **DoS Risk**: Nodes have not yet been reviewed and there may be DoS vectors at this early stage.
- **Secure Key Management**: Storing private keys on Suave is experimental and should be considered unsecured.

If you find a security vulnerability in Suave, please let us know sending an email to security@flashbots.net.

---

## Appendices

### Appendix A

Sample code snippets for DApp integration. (?) DM: this isn't very spec like, but I don't think we're trying to pretend too hard that this is a "formal spec"

### Appendix B

Example of confidential compute request and response objects.
