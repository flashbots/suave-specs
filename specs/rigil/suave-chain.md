---
title: Suave Chain
description: The main purpose of the SUAVE chain is to reach (and maintain) consensus about smart contract code for SUAPPs, as well as be a global information leak broadcast system.
---

<div class="hideInDocs">

<!-- omit from toc -->
# Suave Chain

**Table of Contents**

<!-- TOC -->

- [Overview](#overview)
- [Configuration](#configuration)
  - [Network Parameters](#network-parameters)
  - [Genesis Settings](#genesis-settings)
- [Consensus Mechanism: Proof-of-Authority (Clique)](#consensus-mechanism-proof-of-authority-clique)
  - [Geth Version](#geth-version)
- [Suave Transaction](#suave-transaction)
- [Node Requirements and Setup](#node-requirements-and-setup)
- [Gas and Transaction Fees](#gas-and-transaction-fees)
- [Security Considerations](#security-considerations)

<!-- /TOC -->

---

</div>

## Overview

This document outlines the specifications for the SUAVE Rigil chain.

In the context of the SUAVE protocol, the main purpose of the SUAVE chain is to reach (and maintain) consensus about smart contract code for use cases such as order flow auctions, solvers, block builders, etc. Additionally the SUAVE chain can also be used to store and/or broadcast data for better censorship guarantees.

In the initial phases of development, the SUAVE chain runs a proof-of-authority consensus protocol called Clique, over a network of permissioned nodes. We do so in order to experiment and iterate quickly during protocol development. This will change in later testnets.

## Configuration

### Network Parameters

- **Network ID**: `16813125`
- **Chain ID**: `16813125`
- **RPC Endpoint**: `https://rpc.rigil.suave.flashbots.net`

### Genesis Settings

| Name             | Value    | Unit     |
| ---------------- | -------- | -------- |
| `PERIOD`         | 4        | `block`  |
| `EPOCH`          | 30000    | `block`  |
| `BLOCK_TIME`     | 3        | `second` |
| `GAS_LIMIT`      | 30000000 | `gas`    |
| `NUM_VALIDATORS` | 5        | Nodes    |


## Consensus Mechanism: Proof-of-Authority (Clique)

Clique, an Ethereum-based Proof-of-Authority consensus protocol defined [here](https://eips.ethereum.org/EIPS/eip-225#:~:text=A%20PoA%20scheme%20is%20based,the%20list%20of%20trusted%20signers), restricts block minting to a predefined list of trusted signers. Because of this, every block header that a client sees can be checked against the list of trusted signers.


### Geth Version

Suave-geth is based on geth v1.12.0 ([`e501b3`](https://github.com/flashbots/suave-geth/commit/e501b3b05db8e169f67dc78b7b59bc352b3c638d)).

---

## Suave Transaction

The SUAVE protocol adds a new transaction type to the base Ethereum protocol of which it is currently a fork of called a `SuaveTransaction`. The main purpose of this new transaction type is to process fees for offchain computation and to support the new data primitives associated with confidential compute. Blocks on the SUAVE chain consist of lists of SUAVE transactions. This new transaction type is meant to facilitate and capture key information involved in Confidential Compute Requests which are detailed in the [Kettle](./kettle.md) spec. These fields encapsulates the result and record of a confidential computation request. It includes the `ConfidentialComputeRequest`, signed by the user, which ensures that the result comes from the expected SUAVE Kettle, as the `SuaveTransaction`'s signer must match the `ExecutionNode`. Additionally it includes the original request as the `ConfidentialComputeRecord`.

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

The SUAVE chain employs the same gas pricing mechanism as Ethereum pre-Cancun hardfork (no blob transactions) where gas prices adjust based on network demand. Nodes currently track Confidential Compute Request gas usage, but only charge a small flat fee for it and there is no cap for offchain compute.

Currently SUAVE transactions can only be expressed as Legacy transaction types, but they will get converted into EIP-1559 base fee model under the hood.

---

## Security Considerations

- **Security Risk**: The protocol is unaudited. The protocol currently does not make any guarantees about the confidentiality of data in the network outside of a best effort.
- **DoS Risk**: Nodes have not yet been reviewed and there may be DoS vectors at this early stage.
- **Secure Key Management**: Storing private keys on Suave is experimental and should be considered unsecured.

If you find a security vulnerability in SUAVE, please let us know sending an email to security@flashbots.net.

