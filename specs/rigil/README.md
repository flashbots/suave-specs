---
title: Rigil Testnet
description: Rigil is the first testnet for SUAVE. It is a sandbox and foundation for building MEV applications in a decentralized and private manner, focused on developers.
---

<div class="hideInDocs">

<!-- omit from toc -->
# SUAVE Rigil Testnet

[![Docs at https://suave.flashbots.net/](https://img.shields.io/badge/read-SUAVE%20docs-blue.svg)](https://suave.flashbots.net/)
[![Join the discourse at https://collective.flashbots.net/](https://img.shields.io/badge/chat-on%20Flashbots%20forum-blue.svg)](https://collective.flashbots.net/)

This repository hosts the current SUAVE Rigil testnet specifications and design docs.

<div class="warning">

⚠️ The SUAVE protocol is still in a state where [the code](https://github.com/flashbots/suave-geth) is the most up-to-date protocol spec. The goal of these notes is to gradually evolve into an implementation-agnostic specification. ⚠️

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
  - [Glossary](#glossary)
  - [Architecture](#architecture)
  - [Example Flows](#example-flows)
    - [High Level - OFA + Block Builder](#high-level---ofa--block-builder)
    - [Confidential Compute Request Flow](#confidential-compute-request-flow)
    - [OFA Example](#ofa-example)
    - [Block Building Example](#block-building-example)

<!-- /TOC -->

---

# Specs
Specs are recommended to be read in the following order:
- [Suave Chain](./suave-chain.md)
- [MEVM](./mevm.md)
- [Confidential Data Store](./confidential-data-store.md)
- [Kettle](./kettle.md)
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

The Rigil Testnet, targeted towards developers, serves as a dedicated sandbox for creating SUAPPs (MEV applications) in a way that's both decentralized and confidential. It features the MEVM, a variant of the EVM, which equips developers with the ability to write SUAPPs as smart contracts by giving them access to unique MEV-specific precompiles. Integrating a SUAPP into your dApp allows transactions and intents to confidentially interface with a network of searchers, solvers, block builders, and more. Rigil provides a live, Flashbots-hosted test network for rapid prototyping that uses Goerli ETH for gas and operates with a proof-of-authority consensus mechanism.

Rigil's architecture is composed of several parts:
* SUAVE "Kettles": a network of actors that provide confidential computation for SUAPPs.
* Confidential Data Storage: a private place to store data (e.g. user transactions).
* SUAVE Chain: a public place to store data (e.g. intentionally leaked information) and SUAPP logic.
* MEVM: a modified EVM that exposes confidential computation and storage APIs to developers

The goal of the Rigil testnet is to gather feedback on developer experience and harden the overall SUAVE software stack. The testnet is not intended to be a long-lived network and will be decommissioned after the launch of the next testnet Toliman.

## Users

The Rigil testnet is initially focused on a specific set of actors:

1. **Developers** - create smart contracts on SUAVE Chain that define rules for SUAPPs like order flow auctions, block building, and intent executors.
2. **Transaction Originators** - leverage unique applications on SUAVE, e.g. to send private transactions or execute your intents.
3. **Proposers** - outsource block building to SUAVE. Initially, SUAVE is focused only on Ethereum.
4. **Block Builders** - can be implemented as smart contracts inside Suave. In the Rigil Tesnet, SUAVE Rigil has the ability to submit bundles to external builders to help transaction inclusion during the early stages of development.
5. **Auction Protocols** - can program their auctions as a smart contract.

## Rigil Design Goals

1. **Permissionless** - Allow anyone to deploy smart contracts on SUAVE.
2. **Easy to use** - Create an environment that is as easy to use and test as possible, enabling rapid prototyping.
3. **SGX UX Closeness** - Get as close to the SGX user experience as possible and minimize breaking changes down the road.

## Design Decisions

Here is a list of design decisions made for the Rigil testnet and associated reasoning:

- Decision *1*: **Proof-of-Authority Consensus**
    - *reason*: [SUAVE consensus](https://collective.flashbots.net/t/suave-consensus/2152) is an active open question, which whether or not answered do not drastically impact the UX of users on **Rigil Testnet**.
- Decision *2*: **No SGX Nodes (yet)**
    - reason: SGX SUAVE Kettles are an active area of research and development and do not drastically impact the UX of users on **Rigil Testnet**.
- Decision *3*: **Weak Data Availability Guarantees**
    - Reason: The Confidential Data Store currently only keeps private data available for one day. [Compute Output Validity and Heterogenous DA](https://collective.flashbots.net/t/suave-ensuring-output-validity-and-heterogenous-da/2184) are active open questions, which whether or not answered does not drastically impact the UX on Rigil Testnet.
- Decision *4*: **Centralized Builder Interoperability**
    - reason: Blocks emitted from SUAVE Kettles will have unpredictable inclusion in early development so SUAVE rigil supports a precompile to send bundles to off-SUAVE block builders.

## Glossary 

- **User**: humans or computers interacting with SUAPPs, primarily through sending confidential compute requests (CCR) to Kettles.
- **SUAPP**: SUAVE application, smart contracts on SUAVE chain with rules for confidential computation and functions to submit to target domains (i.e. chains).
- **Developer:** creates smart contracts on SUAVE Chain that define rules for SUAPPs.
- **Confidential Compute Request (CCR) [**[🔗spec](https://github.com/flashbots/suave-specs/blob/initial/specs/rigil/suave-chain.md#confidential-compute-request)**]**: A user request to Suave that contains (1) a wrapped transaction, (2) confidential inputs, and (3) a list of programs (contracts) (← list of Exec nodes) allowed to operate on confidential inputs.
- **Builder solidity**: solidity with access to precompiles that help facilitate the processing of order flow.
- **Precompiles [[🔗spec](https://github.com/flashbots/suave-specs/blob/initial/specs/rigil/precompiles.md)]:** purpose-built functions with extended capabilities that can be called from Builder Solidity
- **Kettle[[🔗spec](https://github.com/flashbots/suave-specs/blob/initial/specs/rigil/kettle.md)]**: accepts and processes confidential compute requests and maintains the SUAVE chain; the logical unit of the SUAVE network and main protocol actor.
- **Confidential Data Store [[🔗spec](https://github.com/flashbots/suave-specs/blob/initial/specs/rigil/confidential-data-store.md)]**: stores confidential data from MEV applications (L1 transactions, EIP 712 signed messages, userOps, privateKeys(?), etc).
- **SUAVE Chain [**[🔗spec](https://github.com/flashbots/suave-specs/blob/initial/specs/rigil/suave-chain.md#suave-chain)**]**: a fork of Ethereum designed for usage alongside credible offchain execution in MEV use cases. In Rigil running in PoA mode.
- **MEVM [[🔗spec](https://github.com/flashbots/suave-specs/blob/initial/specs/rigil/mevm.md)]**: modified EVM with MEV-specific precompiles that allow developers to define order flow auctions and builder rules as smart contracts written in Solidity.
- **Domain-Specific Services (Execution Node?):** provide functionality to interact with target domains (i.e. for Goerli or Arbitrum, simulate transactions, build bundles, build blocks, …)
- **RPC** - receives user transactions, moves confidential input to the Confidential Data Store, and passes the compute request to MEVM.
- **OFA** - an application that receives transactions and either facilitates an auction on top of it or routes it elsewhere.
- **Solver** - actor who takes many user token trades as input and competes to provide a solution to the mathematically optimal way to route all trades.
- **Intent** - refers to “what” the desired outcome of an action on a blockchain should be as opposed to transactions which specify “how” an action should be performed.
- **Intent Executor** - actor who is responsible for taking consolidated user intents and executing them on a domain.
- **Relay** - actor in the [mev-boost protocol](https://github.com/flashbots/mev-boost) that is responsible for validating blocks and offering them to validators upon request.
- **Domains** - [a system with a globally shared state that is mutated by various players through actions](https://arxiv.org/pdf/2112.01472.pdf) (e.g. “transactions”) that execute in a shared execution environment.

## Architecture

SUAVE Kettles house all components necessary to perform confidential compute and are the main protocol actor in the SUAVE protocol. Below is a high-level architectural overview.

![Rigil architecture](/assets/rigil-architecture.svg)

A broad-level view of how a SUAPP gets onchain and utilizes SUAVE core components is as follows:

1. **Developers** create contracts, which contain the logic for their SUAPP. A typical flow might look like: intake and validate user L1 transaction, simulate it on L1 state, then do something based on the simulation results. These contracts are deployed to the SUAVE chain by sending to a **Kettle**.
2. **Users** send *Confidential Compute Requests* directed to a **Kettle** or multiple.
3. Inside the **Kettle**:
   - Requests are routed using the **RPC** to the MEVM or regular EVM, depending on the context.
   - The **MEVM** processes the confidential computation or smart contract call.
   - Data might be stored in or retrieved from the **Suave Chain State** or **Confidential Data Store** based on SUAPP needs.
   - **Precompiles** aid in the efficient execution of certain functions.
   - **Domain-Specific Services** handle execution on different domains and return results to **MEVM**.
4. The **Suave PoA Chain** stores results of confidential computation.

## Example Flows

The example flows in the following sections are used to illustrate some of the possibilities for SUAPPs on SUAVE. Flows start at a high level showing how an OFA and Block Builder SUAPPs work together to emit a block from a SUAVE Kettle. Then to understand the Confidential Compute Request Flow more deeply, there is a detailed data flow diagram, followed by two deeper dives into the specifics of how the OFA and Block Builder flows work individually.

### High Level - OFA + Block Builder

Below we can see the journey of order flow from transaction, to searcher back-run, to a block emitted from SUAVE.

![OFA + Block Builder flow](/assets/OFA_And_Block_Flow.svg)

1. A user sends their L1 transaction, EIP-712 message, UserOp, or Intent into a SUAVE Kettle.
2. MEVM processes this L1 transaction, extracts a hint, and does two things:
    - Stores the L1 transaction in the confidential data store
    - Sends a SUAVE transaction to the mempool which when executed emits the hint as a log
3. Searchers will be listening on two different lanes for hints:
    - The fast lane which is the mempool
    - The global lane which is the SUAVE chain, is slower but will surface any hints that may have been censored by your specific peer in the mempool
4. Once a hint is received, searchers craft a backrun transaction and send it to a SUAVE Kettle.
5. SUAVE Kettles will process the backrun, combine it into a bundle with the original transaction, include the bundle in a block, and then submit the block to a relay.

*Optionally*, bundles can be sent straight to a centralized block builder.

### Confidential Compute Request Flow

The SUAVE-enabled node and the MEVM support multiple new data types, which are all specified in the [SUAVE Chain spec](./suave-chain.md#custom-types).

The diagram below showcases how these different types interact to enable confidential computation on SUAVE Kettles.

![Rigil transaction flow](/assets/rigil-tx-flow.svg)

Transaction Flow:
1. User sends a Confidential Compute Request to the RPC - Confidential Compute Requests are made up of two components, Compute Transaction and Confidential Inputs. The Compute request will reference data inside of the Confidential Inputs that the MEVM is able to use during computation.
2. Once the RPC receives the Confidential Compute Request, it will extract the confidential inputs and send them to the confidential data store. It will then send the Compute Request to the MEVM to process.
3. Confidential Compute Phase - During this phase, the MEVM will process the compute transaction similar to an EVM transaction except it will also have access to the confidential inputs. After doing the initial computation with the confidential data, it will then grab the results and information from the Compute Transaction and put them into their final home a SUAVE transaction.
4. Block Inclusion - Once the SUAVE transaction has been created it will then quickly be included in a block by a SUAVE proposer.


### OFA Example

If we consider a specific use case, like an order flow auction, the high-level series of steps taken to complete the auction can represented as below:

![OFA example flow](/assets/OFA-example-flow.svg)

1. The user sends a Confidential Compute Request interacting with a SUAPP by calling its `newBid` function. Included in this request is also the user's L1 transaction as a confidential Input.
2. The Kettle will receive the transaction and process it. To do so it first runs the offchain logic associated with `newBid` which will extract the transaction's data and then return a callback:
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
6. Once the searcher has a backrun crafted for the opportunity it will send it to the Kettle as a Confidential Compute Request with the backrun transaction in the confidential inputs.
7. The MEVM node will receive and process the searcher's Confidential Compute Request based on the contracts logic. In this case, it will:
    - Grab referenced User Transaction to be placed behind
    - Submit to domain specific service for simulation and validation
    - Construct a bundle object with two transactions
8. From there, in this example, the MEVM will then forward the bundle to pre-configured off-SUAVE block builders, but could as easily also forward to onchain block builders.


The important thing to note here is that this Confidential Compute pattern makes it such that no sensitive data gets leaked onchain except for what the smart contract specifies, and thus creates programmable information leakage.

### Block Building Example

Blocks built from SUAVE will have unpredictable inclusion in the beginning, but adventurers are invited to hook into the block building flow already achievable today. Below is a walkthrough of a block being built via a solidity smart contract.

![Block Building Flow](/assets/block-building-flow.svg)

1. The user sends a Confidential Compute Request that specifies calling `buildBlock` on the onchain block builder contract.
2. The Kettle receives and processes the transaction; specifically, the logic will:
- grab all bundles that are stored in the block builders' confidential data store
- simulate all bundles
- sort bundles via arbitrary logic but in this gas by effective gas price
- compute state root and package into a block
3. Optional: Similar to the above, the confidential compute result can be a callback which will emit a log of the block's bid value onchain as well as a header that a validator can view.
4. Similar to sending to a centralized block builder, the MEVM will then send the block to a centralized relay where it is free to access by validators.


