---
title: Toliman Testnet
description: Toliman is the second testnet for SUAVE. It is a sandbox and foundation for building MEV applications in a decentralized and private manner, focused on developers.
custom_edit_url: "https://github.com/flashbots/suave-specs/edit/main/specs/toliman/README.md"
---


<!-- omit from toc -->
# SUAVE Toliman Testnet

<div className="hide-in-docs">

[![Docs at https://suave.flashbots.net/](https://img.shields.io/badge/read-SUAVE%20docs-blue.svg)](https://suave.flashbots.net/)
[![Join the discourse at https://collective.flashbots.net/](https://img.shields.io/badge/chat-on%20Flashbots%20forum-blue.svg)](https://collective.flashbots.net/)

This repository hosts the current SUAVE Toliman testnet specifications and design docs.

<div class="warning">

⚠️ The SUAVE protocol is still in a state where [the code](https://github.com/flashbots/suave-geth) is the most up-to-date protocol spec and [open questions](#open-questions) are being researched. The goal of these notes is to gradually evolve into an implementation-agnostic specification. ⚠️

</div>

---

**Table of Contents**

<!-- TOC -->

- [Specs](#specs)
- [About SUAVE](#about-suave)
- [Toliman Overview](#toliman-overview)
- [Users](#users)
- [Toliman Design Goals](#toliman-design-goals)
- [Design Decisions](#design-decisions)
- [Glossary](#glossary)
- [Architecture](#architecture)
- [Open Questions](#open-questions)
- [Example Flows](#example-flows)
  - [High Level - OFA + Block Builder](#high-level---ofa--block-builder)
  - [Confidential Compute Request Flow](#confidential-compute-request-flow)
  - [OFA Example](#ofa-example)
  - [Block Building Example](#block-building-example)

<!-- /TOC -->

---

## Specs

We recommend that you read the specs in the following order:

- [Suave Chain](./suave-chain.md)
- [MEVM](./mevm.md)
- [Precompiles](./precompiles.md)
- [Confidential Data Store](./confidential-data-store.md)
- [Kettle](./kettle.md)
- [Bridge](./bridge.md)

---

## About SUAVE

SUAVE - Single Unifying Auction for Value Expression - is a platform for building MEV applications such as OFAs, block builders, and intent executors in a decentralized and private way. SUAVE does not replace other blockchains: it is intended to aggregate and coordinate all the things that ultimately change the state of other chains.

Read more about SUAVE:
- https://writings.flashbots.net/the-future-of-mev-is-suave
- https://writings.flashbots.net/mevm-suave-centauri-and-beyond


---

## Toliman Overview

</div>

This set of specs outlines the Toliman Testnet, a continuation of the star system theme (Centauri, Andromeda, Helios) laid out in [The Future of MEV](https://writings.flashbots.net/mevm-suave-centauri-and-beyond); and the second in a series of SUAVE testnets based on stars in the [(Alpha) Centauri system](https://en.wikipedia.org/wiki/Alpha_Centauri): Toliman Kentaurus (Alpha Centauri A), Toliman (B) and Proxima Centauri (C).

The Toliman Testnet is a developer focused sandbox for creating SUAPPs (MEV applications) in a way that's both decentralized and confidential. It features the MEVM, a variant of the EVM, which equips developers with the ability to write SUAPPs as smart contracts by giving them access to unique MEV-specific precompiles. SUAPPs can send transactions and intents confidentially to a network of searchers, solvers, block builders, and more.

Toliman provides a live, Flashbots-hosted test network for rapid prototyping that uses ETH for gas and operates with a proof-of-authority consensus mechanism.

Toliman's architecture is composed of several parts:
* SUAVE Kettles: a network of actors that provide confidential computation for SUAPPs. These all operate in Trusted Execution Environments (TEEs).
* Confidential Data Storage: a private place to store data (e.g. user transactions).
* SUAVE Chain: a public place to store data (e.g. intentionally leaked information) and SUAPP logic (e.g. deployed smart contracts).
* MEVM: a modified EVM that exposes confidential computation and storage APIs to developers

The goal of the Toliman testnet is to gather feedback on developer experience and harden the overall SUAVE software stack. The testnet is not intended to be a long-lived network and will be decommissioned after the launch of the next testnet: Proxima.

You can learn more about TEEs and SUAVE via the links below:

* [Speedrunning a TEE Coprocessor](https://writings.flashbots.net/suave-tee-coprocessor)
* [Flashwares session: What are TEEs?](https://collective.flashbots.net/t/flashwares-i-tees-feat-intel-sgx/3405#what-are-tees-1)

## Users

The Toliman testnet is initially focused on a specific set of actors:

1. **Developers** - create smart contracts on SUAVE Chain that define rules for SUAPPs like order flow auctions, block building, and intent executors.
2. **Transaction Originators** - leverage unique applications on SUAVE, e.g. to send private transactions or execute your intents.
3. **Proposers** - outsource block building to SUAVE. Initially, SUAVE is focused only on Ethereum.
4. **Block Builders** - can be implemented as smart contracts inside Suave. In the Toliman Testnet, SUAPPs can submit bundles to external builders to help with transaction inclusion during the early stages of development.
5. **Auction Protocols** - can program their auctions as a smart contract.

## Toliman Design Goals

1. **Permissionless** - Allow anyone to deploy smart contracts on SUAVE.
2. **Easy to use** - Create an environment that is as easy to use and test as possible, enabling rapid prototyping.

## Design Decisions

Here is a list of design decisions made for the Toliman testnet and associated reasoning:

- Decision *1*: **Proof-of-Authority Consensus**
    - *reason*: [SUAVE consensus](https://collective.flashbots.net/t/suave-consensus/2152) is an active open question which, whether answered or not, does not drastically impact the UX of users on **Toliman Testnet**.
- Decision *2*: **Weak Data Availability Guarantees**
    - Reason: The Confidential Data Store on the first SUAVE testnet - Rigil - only kept private data available for one day. Toliman keeps it available for two weeks. [Compute Output Validity and Heterogenous DA](https://collective.flashbots.net/t/suave-ensuring-output-validity-and-heterogenous-da/2184) are still active open questions which, whether answered or not, do not drastically impact the UX on Toliman Testnet.

## Glossary

See [`glossary.md`](./glossary.md).

## Architecture

SUAVE Kettles house all components necessary to perform confidential compute and are the main protocol actor in the SUAVE protocol. Below is a high-level architectural overview.

![Toliman architecture](/assets/toliman-architecture.svg)

A broad-level view of how a SUAPP gets onchain and utilizes SUAVE core components is as follows:

1. **Developers** create contracts, which contain the logic for their SUAPP. A typical flow might look like: intake and validate user L1 transaction, simulate it on L1 state, then do something based on the simulation results. These contracts are deployed to the SUAVE chain by sending to a **Kettle**.
2. **Users** send *Confidential Compute Requests* directed to a **Kettle** or multiple.
3. Inside the **Kettle**:
   - Requests are routed using the **RPC** to the MEVM or regular EVM, depending on the context.
   - The **MEVM** processes the confidential computation or smart contract call.
   - Data might be stored in or retrieved from the **Suave Chain State** or **Confidential Data Store** based on SUAPP needs.
   - **Precompiles** aid in the efficient execution of certain functions.
   - **Domain-Specific Services** handle execution on different domains (such as simulation against Ethereum state) and return results to **MEVM**.
4. The **Suave PoA Chain** stores results of confidential computation.

## Open Questions

The are multiple open questions that need to be solved in the long term about SUAVE architecture. A non-exhaustive list of questions are:

- [Consensus Requirements](https://collective.flashbots.net/t/current-thinking-for-consensus/2695?u=quintus)
- [Output Validity and Heterogenous DA](https://collective.flashbots.net/t/suave-ensuring-output-validity-and-heterogenous-da/2184)
- [SUAVE Economic Security Models](https://collective.flashbots.net/t/suave-economic-security-models/1070)
- [Bridge Request For Comments](https://collective.flashbots.net/t/suave-bridge/2696)

## Example Flows

The example flows in the following sections are used to illustrate some of the possibilities for SUAPPs on SUAVE. Flows start at a high level showing how an OFA and Block Builder SUAPPs work together to emit a block from a SUAVE Kettle. Then to understand the Confidential Compute Request Flow more deeply, there is a detailed data flow diagram, followed by two deeper dives into the specifics of how the OFA and Block Builder flows work individually.

### High Level - OFA + Block Builder

Below we can see the journey of order flow from transaction, to searcher back-run, to a block emitted from SUAVE.

![OFA + Block Builder flow](/assets/OFA_And_Block_Flow.svg)

1. A user sends their L1 transaction, EIP-712 message, UserOp, or Intent into a SUAVE Kettle.
2. MEVM processes this L1 transaction, extracts a hint (data intentionally leaked by the contract), and does two things:
    - Stores the L1 transaction in the confidential data store
    - Sends a SUAVE transaction to the mempool which when executed emits the hint as a log
3. Searchers will be listening on two different lanes for hints:
    - The fast lane which is the mempool
    - The global lane which is the SUAVE chain, is slower but will surface any hints that may have been censored by your specific peer in the mempool
4. Once a hint is received, searchers craft a backrun transaction and send it to a SUAVE Kettle.
5. SUAVE Kettles will process the backrun, combine it into a bundle with the original transaction, include the bundle in a block, and then submit the block to a relay.

*Optionally*, bundles can be sent straight to a centralized block builder.

### Confidential Compute Request Flow

The SUAVE Kettle and the MEVM support multiple new data types, which are all specified in the [Kettle spec](./kettle.md#containers).

The diagram below showcases how these different types interact to enable confidential computation on SUAVE Kettles.

![Toliman transaction flow](/assets/toliman-tx-flow.svg)

Transaction Flow:
1. User sends a Confidential Compute Request to the RPC - Confidential Compute Requests are made up of two components, Compute Transaction and Confidential Inputs. The Compute request will reference data inside of the Confidential Inputs that the MEVM is able to use during computation.
2. Once the RPC receives the Confidential Compute Request, it will extract the confidential inputs and send them to the confidential data store. It will then send the Compute Request to the MEVM to process.
3. Confidential Compute Phase - During this phase, the MEVM will process the compute transaction similar to an EVM transaction except it will also have access to the confidential inputs. After doing the initial computation with the confidential data, it will then grab the results and information from the Compute Transaction and put them into their final home a SUAVE transaction.
4. Block Inclusion - Once the SUAVE transaction has been created it will then quickly be included in a block by a SUAVE proposer.

### OFA Example

If we consider a specific use case, like an order flow auction, the high-level series of steps taken to complete the auction can represented as below:

![OFA example flow](/assets/OFA-example-flow.svg)

1. The user sends a Confidential Compute Request interacting with a SUAPP by calling its `newTransaction` function. Included in this request is also the user's L1 transaction as a confidential Input.
2. The Kettle will receive the transaction and process it. To do so it first runs the offchain logic associated with `newTransaction` which will extract the transaction's data and then return a callback:
```go
return bytes.concat(this.emitDataRecord.selector, abi.encode(dataRecord));
```
which points to another function:
```go
function emitDataRecord(Suave.DataRecord calldata dataRecord) public {
    emit DataRecordEvent(dataRecord.id, dataRecord.decryptionCondition, dataRecord.allowedPeekers);
}
```
3. The callback is inserted into the calldata of a SUAVE transaction and then shipped off to the SUAVE mempool.
4. The transaction will get picked up, inserted in a SUAVE block, and propagated to SUAVE Kettles.
5. From here a searcher monitoring the chain and this specific OFA will see the log emitted and begin processing.
6. Once the searcher has a backrun crafted for the opportunity it will send it to the Kettle as a Confidential Compute Request with the backrun transaction in the confidential inputs.
7. The Kettle will receive and process the searcher's Confidential Compute Request based on the contract's logic. In this case, it will:
    - Grab the referenced User Transaction to be placed behind
    - Construct a bundle object with the two transactions
    - Submit to domain-specific service for simulation and validation
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


