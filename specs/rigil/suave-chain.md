# Suave Chain

## Table of Contents
1. Introduction
2. Custom types
   - 2.1. `ConfidentialComputeRequest`
   - 2.2. `SuaveTransaction`
   - 2.3. JSON-RPC
3. Configuration
   - 3.1 Clique Proof-of-Authority
   - 3.2 MEVM Execution
   - 3.3 Network Parameters
   - 3.4 Genesis Settings
4. Consensus Mechanism
5. Node Requirements and Setup
6. Gas and Transaction Fees
7. Security Considerations
8. Appendices

## Introduction

This document outlines the specifications for the SUAVE Rigil testnet. The main purpose of these the suave chain in the suave protocol is to agree upon smart contract code for use cases such as OFAs, solvers, block builders, etc, in some cases store data onchain for better censorship gurantees, and in some cases, broadcast data. In the initial phases of SUAVE development, the chain is generated and maintained by a proof-of-authority consensus protocol clique over a network of permissioned nodes. The reason for this network configuration is to enable rapid iteration during architectural shift during protocol development.

## Custom Types

There are two main additional types that the SUAVE protocol adds onto the base ethereum protocol that it's currently a fork of. Those two types are seen below. `ConfidentialComputeRequest` is how a user requests compute over their data and interacting with SUAVE computors. `SuaveTransaction` is the resulting data type from confidential compute.

### Confidential Compute Request

This type primarily facilitates users in expressing their preferences for order flow processing in the SUAVE system.

```go
type ConfidentialComputeRequest struct {
	ExecutionNode: address,
	Wrapped       Transaction,
}
```

### Suave Transaction

A specialized transaction type that encapsulates the result of a confidential computation request.

```go
type SuaveTransaction struct {
    ExecutionNode: address,
    ConfidentialComputeRequest: transaction,
    ConfidentialComputeResult: bytes,
    // node's signature fields
}
```
In the future the signature fields here will represent various different types of proof of computation and more.

### 2.3. JSON-RPC

SUAVE JSON-RPC can be seen as a super set of Ethereum JSON-RPC, but for the most part, the entirety of the [Ethereum JSON-RPC standard](https://geth.ethereum.org/docs/interacting-with-geth/rpc) remains the same in interacting with the SUAVE chain.

1. New `IsConfidential` and `ExecutionNode` fields are added to TransactionArgs, used in `eth_sendTransaction` and `eth_call` methods.
If `IsConfidential` is set to true, the call will be performed as a confidential call, using the `ExecutionNode` passed in for constructing `ConfidentialComputeRequest`.
`SuaveTransaction` is the result of `eth_sendTransaction`! If `IsConfidential` is unset or false in the request, it will be processed as a regular Ethereum Legacy or EIP1559 transaction.

2. New optional argument - `confidential_data` is added to `eth_sendRawTransaction`, `eth_sendTransaction` and `eth_call` methods.
The confidential data is made available to the EVM in the confidential mode via a precompile, but does not become a part of the transaction that makes it to chain. This allows performing computation based on confidential data (like simulating a bundle, putting the data into confidential store).

3. The return type of all RPCs that get transaction or receipt objects back will be of type `SuaveTransaction`, a super set of regular Ethereum transactions.

Appendix C for an example of confideintial compute request and response objects.

## Configuration

### Proof-of-Authority (Clique)

Utilizing Ethereum's Clique PoA mechanism, the SUAVE Rigil testnet ensures rapid block generation while maintaining data integrity and security.

### MEVM Execution


### Network Parameters

- **Network ID**: `16813125`
- **Chain ID**: `16813125`
- **RPC Endpoint**: `https://rpc.suave.io` (?? Do we post our RPC here?? )

### Genesis Settings

| Name | Value | Unit |
| - | - | - |
| `PERIOD` | 4 | `block`
| `EPOCH` | 30000 | `block`
| `BLOCK_TIME` | 3 | `second`
| `GAS_LIMIT`| 30000000 | `gwei`
| `NUM_VALIDATORS` | 3 | Nodes

## Consensus Mechanism - Clique (PoA)

Clique, an Ethereum-based Proof-of-Authority consensus protocol defined [here](https://eips.ethereum.org/EIPS/eip-225#:~:text=A%20PoA%20scheme%20is%20based,the%20list%20of%20trusted%20signers), restricts block minting to a predefined list of trusted signers. Because of this, every block header that a client sees can be checked against the list of trusted signers.


## Node Requirements and Setup

- **Hardware**: Minimum 8GB RAM, Quad-core CPU, 50GB SSD (??? How big do we expect the chain to grow(???) These requirements will eventually incorporated Trusted Execution Environments (TEEs).
- **Software**: suave-geth, version X.X or later
- **Setup Steps**:
  1. Install Geth
  2. Run `make devnet-up'

## Gas and Transaction Fees

The SUAVE chain employs the same gas pricing mechanism as Ethereum pre-Cancun hardfork (no blob transactions) where gas prices adjust based on network demand. Nodes currently track Confidential Compute Request gas usage, but do not charge (??? I believe correct @ MM?).

## Security Considerations

- **Security Risk**: The protocol is unaudited. The protocol currently does not make any gurantees about the confidentiality of data in the network outside of a best effort.
- **DoS Risk**: Nodes have not yet been audited for DoS.
- **Secure Key Management**: Store private keys with mininmal to know  on securely, preferably in hardware wallets. Private keys on SUAVE should be considered not secured for the time being so please keep minimal amounts of testnet eth on them.

## Appendices

- **Appendix A**: Sample code snippets for DApp integration. (?) DM: this isn't very spec like, but I don't think we're trying to pretend too hard that this is a "formal spec"
- **Appendix B**: Diagrammatic representation of transaction flow.

- **Appendix C**: Example confidential compute request and response