# Suave Chain

## Introduction

This document outlines the specifications for the SUAVE Rigil testnet. The main purpose of these the suave chain in the suave protocol is to agree upon smart contract code for use cases such as OFAs, solvers, block builders, etc, in some cases store data onchain for better censorship guarantees, and in some cases, broadcast data. In the initial phases of SUAVE development, the chain is generated and maintained by a proof-of-authority consensus protocol clique over a network of permissioned nodes. The reason for this network configuration is to enable rapid iteration during architectural shift during protocol development.

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

## Consensus Mechanism: Proof-of-Authority (Clique)

Clique, an Ethereum-based Proof-of-Authority consensus protocol defined [here](https://eips.ethereum.org/EIPS/eip-225#:~:text=A%20PoA%20scheme%20is%20based,the%20list%20of%20trusted%20signers), restricts block minting to a predefined list of trusted signers. Because of this, every block header that a client sees can be checked against the list of trusted signers.

### MEVM Execution

### Geth Version

Suave-geth is based on geth v1.12.0 ([`e501b3`](https://github.com/flashbots/suave-geth/commit/e501b3b05db8e169f67dc78b7b59bc352b3c638d))

---

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

## 2.3. JSON-RPC

SUAVE JSON-RPC can be seen as a super set of Ethereum JSON-RPC, but for the most part, the entirety of the [Ethereum JSON-RPC standard](https://geth.ethereum.org/docs/interacting-with-geth/rpc) remains the same in interacting with the SUAVE chain.

1. New `IsConfidential` and `ExecutionNode` fields are added to TransactionArgs, used in `eth_sendTransaction` and `eth_call` methods.
If `IsConfidential` is set to true, the call will be performed as a confidential call, using the `ExecutionNode` passed in for constructing `ConfidentialComputeRequest`.
`SuaveTransaction` is the result of `eth_sendTransaction`! If `IsConfidential` is unset or false in the request, it will be processed as a regular Ethereum Legacy or EIP1559 transaction.

2. New optional argument - `confidential_data` is added to `eth_sendRawTransaction`, `eth_sendTransaction` and `eth_call` methods.
The confidential data is made available to the EVM in the confidential mode via a precompile, but does not become a part of the transaction that makes it to chain. This allows performing computation based on confidential data (like simulating a bundle, putting the data into confidential store).

3. The return type of all RPCs that get transaction or receipt objects back will be of type `SuaveTransaction`, a super set of regular Ethereum transactions.

Appendix C for an example of confideintial compute request and response objects.

### `suavex` namespace

`suavex_buildEthBlockFromBundles` - takes an array of bundles and transactions, calculates state root and related fields, and returns a valid Ethereum L1 block.

`suavex_buildEthBlock` - takes an array of transactions, calculates state root and related fields, and returns a valid Ethereum L1 block.

---

## Node Requirements and Setup

- **Hardware**:
    - Minimum 8GB RAM, four cores, 50GB SSD (??? How big do we expect the chain to grow(???)
    - These requirements will eventually incorporated Trusted Execution Environments (TEEs).
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

- **Appendix A**: Sample code snippets for DApp integration. (?) DM: this isn't very spec like, but I don't think we're trying to pretend too hard that this is a "formal spec"
- **Appendix B**: Diagrammatic representation of transaction flow.

![TransactionFlow Diagram](https://github.com/flashbots/suave-geth/assets/22778355/66210b57-dc97-4aa8-a335-a5ce4e6487a3)

- User -># RPC:
- RPC -> MEVM:
- MEVM -> Chain:

- **Appendix C**: Example confidential compute request and response