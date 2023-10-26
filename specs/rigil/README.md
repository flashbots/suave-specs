---
title: Rigil Testnet
description: Rigil is the first testnet for SUAVE. It is an experimental sandbox and foundation for building MEV applications in a decentralized and private manner.
---

<!-- omit from toc -->
# SUAVE Rigil Testnet

<div class="hideInDocs">

[![Docs at https://suave.flashbots.net/](https://img.shields.io/badge/read-SUAVE%20docs-blue.svg)](https://suave.flashbots.net/)
[![Join the discourse at https://collective.flashbots.net/](https://img.shields.io/badge/chat-on%20Flashbots%20forum-blue.svg)](https://collective.flashbots.net/)

This repository hosts the current SUAVE Rigil testnet specifications and design docs.

<div class="warning">

⚠️ The SUAVE protocol is still in a state where [the code](https://github.com/flashbots/suave-geth) is the most up-to-date protocol spec. The goal of these notes is to gradually evolve into an implementation agnostic specification. ⚠️

</div>

---

**Table of Contents**

<!-- TOC -->

- [Specs](#specs)
- [About SUAVE](#about-suave)
- [Rigil Overview](#rigil-overview)
  - [Users](#users)
  - [Rigil Design Goals](#rigil-design-goals)
  - [Design Decisions](#design-decisions)
  - [Architecture](#architecture)
  - [How To Use SUAVE?](#how-to-use-suave)
  - [Example Flows](#example-flows)
    - [High Level - OFA + Block Builder](#high-level---ofa--block-builder)
    - [Confindential Compute Request Flow](#confindential-compute-request-flow)
    - [OFA Example](#ofa-example)
    - [Block Building Example](#block-building-example)

<!-- /TOC -->

---

# Specs
- [Suave Chain](./suave-chain.md)
- [MEVM](./mevm.md)
- [Confidential Data Store](./confidential-data-store.md)
- [Precompiles](./precompiles.md)
- [Computor](./computor.md)
- [Bridge](./bridge.md)

---

# About SUAVE

SUAVE - Single Unifying Auction for Value Expression - is a platform for building MEV applications such as OFAs, block builders, and intent executors in a decentralized and private way. SUAVE does not replace other blockchains: it is intended to aggregate and coordinate all the things that ultimately change the state of other chains.

Read more about SUAVE:
- https://writings.flashbots.net/the-future-of-mev-is-suave
- https://writings.flashbots.net/mevm-suave-centauri-and-beyond

</div>

---

# Rigil Overview

This set of specs outlines the Rigil Testnet, a continuation of the star system theme (Centauri, Andromeda, Helios) laid out in [The Future of MEV](https://writings.flashbots.net/mevm-suave-centauri-and-beyond); and the first in a series of SUAVE testnets based on stars in the [(Alpha) Centauri system](https://en.wikipedia.org/wiki/Alpha_Centauri): Rigil Kentaurus (Alpha Centauri A), Toliman (B) and Proxima Centauri (C).

The Rigil Testnet is a sandbox for building MEV applications in a decentralized and private manner that is initially targetted towards developers. Developers are empowered with the MEVM, a modification of the EVM which gives them access to new MEV specific [precompiles](./precompiles.md) that allow them to write their applications as smart contracts in Solidity. Further, Rigil offers a live test network for rapid prototyping hosted by Flashbots that uses Goerli ETH for gas and a proof-of-authority consensus mechanism.

Rigil's architecture is composed of several parts:
* SUAVE "Computors" - a network of actors that provide private computation for MEV applications
* Confidential data storage: a private place to store sensitive data (e.g. user transactions)
* SUAVE Chain: to store MEV applications and public data
* MEVM: a modified EVM that exposes confidential execution and storage APIs to developers

The goal of the Rigil testnet is to gather feedback on developer experience and harden the overall SUAVE software stack. The testnet is not intended to be a long-lived network and may be reset in the future.

## Users

The Rigil testnet is initially focused on a specific set of actors:

1. **Developers** - create smart contracts on SUAVE Chain that define rules for MEV applications like orderflow auctions, block building, and intent executors.
2. **Users** - leverage unique applications on SUAVE, e.g. to send private transactions or execute your intents.
3. **Proposers** - outsource block building to SUAVE. Initially, this is focused only on Ethereum.
4. **Block Builders** - can be implemented as smart contracts inside Suave. In the Rigil Tesnet, Suave has the capability to submit bundles to several external builders to help transaction inclusion during early stages of development.
5. **Auction protocols** - can program their auctions as a smart contract.


## Rigil Design Goals

1. **Permissionless** - Allow anyone to deploy SUAVE smart contracts.
2. **Easy to use** - Create an environment that is as easy to use and test as possible, enabling rapid prototyping.
3. **SGX UX Closeness** - Get as close to SGX user experience as possible and minimize breaking changes down the road.


## Design Decisions

Here is a list of design decisions made for the Rigil testnet and associated reasoning:

- Decision *1*: **Proof-of-Authority Consensus**
    - *reason*: [SUAVE consensus](https://collective.flashbots.net/t/suave-consensus/2152) is an active open question, which whether or not answered does not drastically impact UX of users on **Rigil Testnet**.
- Decision *2*: **No SGX Nodes (yet)**
    - reason: SGX SUAVE computors are an active area of research and development and does not drastically impact UX of users on **Rigil Testnet**.
- Decision *3*: **Weak DA Layer Guarantees**
    - Reason: The Confidential Data Store currently only keeps private data available for one day. [Compute Output Validity and Heterogenous DA](https://collective.flashbots.net/t/suave-ensuring-output-validity-and-heterogenous-da/2184) are active open questions, which whether or not answered does not drastically impact UX on Rigil Testnet.
- Decision *4*: **Centralized Builder Interoperability**
    - reason: Blocks emitted from SUAVE computors will have unpredictable inclusion in early development so SUAVE rigil supports a precompile to send bundles to off-SUAVE block builders.


## Architecture

SUAVE Computors house all components necessary to perform confidential compute and are the main protocol actor in the SUAVE protocol. Below is a high level architectural overview followed by a brief descriptions of the main components.

![Rigil architecture](/assets/rigil-architecture.svg)

- **Confidential Compute Request (CCR) [**[🔗spec](./suave-chain.md#confidential-compute-request)**]**: A user request to Suave that contains:
    1. a wrapped transaction
    2. confidential data, and
    3. a list of programs (contracts) allowed to access and operate on confidential data.
- **Computor[**[🔗spec](./computor.md)**]**: accepts and processes confidential compute requests and maintains the SUAVE chain; the logical unit of the SUAVE network and main protocol actor.
- **Confidential Data Store (CDS) [**[🔗spec](./confidential-data-store.md)**]**: stores confidential data from MEV applications (L1 transactions, EIP 712 signed messages, userOps, privateKeys(?), etc).
- **SUAVE Chain [**[🔗spec](./suave-chain.md)**]**: a fork of Ethereum designed for usage alongside credible off-chain execution in MEV use cases. Running Clique PoA consensus for the Rigil Testnet.
- **MEVM [**[🔗spec](./mevm.md)**]**: modified EVM with MEV-specific precompiles that allow developers to define orderflow auctions and builder rules as smart contracts written in Solidity.
- **Precompiles [**[🔗spec](./precompiles.md)**]:** purpose-built functions with extended capabilities that can be called from Builder Solidity
- **MEV Application :** MEVM compatible Solidity which defines how orderflow should be processed and routed.


## How To Use SUAVE?
- **Deploy smart contracts** - Smart contracts on SUAVE function the same way as traditional EVM chains and can be developed with your favorite existing EVM toolset. But, they can leverage additional precompiles that give developers the ability to create MEV applications as smart contracts.

- **Use MEV applications** - Users can use applications on SUAVE by making a confidential computation request, which is a special SUAVE transaction. See [INSERT LINK HERE] for more details.

## Example Flows

The example flows in the following sections are used to illustrate some of the possibilities for MEV applications on SUAVE. We start at a high level showing how OFA and Block Builder smart contracts work together to emit a block from a SUAVE node. To understand more deeply the Confindential Compute Request Flow is detailed and from there a deeper dive into the specifics of how the OFA and Block Builder contracts work individually.

### High Level - OFA + Block Builder

Below we can see the journey of orderflow from transaction, to searcher back-run, to a block emitted from SUAVE.

![OFA + Block Builder flow](/assets/OFA_And_Block_Flow.svg)

1. A user sends their L1 transaction, EIP-712 message, UserOp, or Intent into a SUAVE computor.
2. MEVM processes this L1 transaction, extracts a hint, and emits it onchain.
3. Searchers listening to the chain see the hint, craft a backrun transactions, and send them to a SUAVE computor.
4. SUAVE computors will process the backrun, combine it into a bundle with the original transaction, include the bundle in a block, and then emit the block to an offchain relay.

*Optionally*, bundles can be sent straight to a centralized block builder, or the block can also be sent to an onchain relay instead of offchain.

### Confindential Compute Request Flow

The SUAVE-enabled node and the MEVM support multiple new data types, which are all specified in the [SUAVE Chain spec](./suave-chain.md#custom-types).

The diagram below showcases how these different types interact to enable confidential computation on SUAVE computors.

![Rigil transaction flow](/assets/rigil-tx-flow.svg)

Transaction Flow:
1. User sends a Confidential Compute Request to the RPC - Confidential Compute Requests are made up of two components, Compute Transaction and Confidential Inputs. The Compute request will reference data inside of the Confidential Inputs that the MEVM is able to use during computation.
2. Once the RPC receives the Confidential Compute Request, it will extract the confidential inputs and send them to the confidential data store. It will then send the Compute Request to the MEVM to process.
3. Confidential Compute Phase - During this phase the MEVM will process the compute transaction similar to an EVM transaction except it will also have access to the confidential inputs. After doing the initial computation with the confidential data, it will then grab the results and information from the Compute Transaction and put them into their final home a SUAVE transaction.
4. Block Inclusion - Once the SUAVE transaction has been created it will then quickly be included in a block by a SUAVE proposer.


### OFA Example

If we consider a specific use case, like an order flow auction, the high-level series of steps taken to complete the auction can represented as below:

![OFA example flow](/assets/OFA-example-flow.svg)

1. The user sends a Confidential Compute Request interacting with a SUAPP by calling it's `newBid` function. Included in this request is also the user's L1 transaction as a confidential Input.
2. The computor will receive the transaction and process it. To do so it first runs the offchain logic associated with `newBid` which will extract the transaction's data and then return a callback:
```go
return bytes.concat(this.emitHint.selector, abi.encode(hint));
```
which points to another function:
```go
function emitHint(Suave.Bid calldata bid, bytes memory hint) public {
    emit HintEvent(bid.id, hint);
}
```
3. The callback is inserted into the calldata of a SUAVE transaction and then shipped off to the SUAVE mempool.
4. The transaction will get picked up, inserted in a block, and propagated.
5. From here a searcher monitoring the chain and this specific OFA will see the log emitted and begin processing.
6. Once the searcher has a backrun crafted for the opportunity it will send it to the Computor as a Confidential Compute Request with the backrun transaction in the confidential inputs.
7. The MEVM node will receive and process the searcher's Confidential Compute Request based on the contracts logic. In this case it will:
    - Grab referenced User Transaction to be placed behind
    - Submit to domain specific service for simulation and validation
    - Construct bundle object with two transactions
8. From there, in this example, the MEVM will then forward the bundle to pre-configured off-SUAVE block builders, but could as easily also forward to onchain block builders.


The important thing to note here is that this Confidential Compute pattern makes it such that no sensitive data gets leaked onchain except for what the smart contract specifies and is thus creates progammable information leakage.

### Block Building Example

Blocks built from SUAVE will be unpredictable in the beginning, but adventurers are invited to hook into the block building flow already achievable today. Below is a walkthrough of a block being built via a solidity smart contract.

![Block Building Flow](/assets/block-building-flow.svg)

1. The user sends a Confidential Compute Request that specifies calling `buildBlock` on the onchain block builder contract.
2. The Computor receives and processes the transaction; specifically the logic will:
- grab all bundles that are stored in the block builders confidential data store
- simulate all bundles
- sort bundles via arbitrary logic but in this gas by effective gas price
- compute state root and package into a block
3. Optional: Similar to the above, the confidential compute result can be a callback which will emit a log of the block's bid value onchain as well as header which a validator can view.
4. Similar to sending to a centralzied block builder, the MEVM will then send the block to a centralized relay where it is free to access by validators.


