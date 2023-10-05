# SUAVE Rigil Testnet

<div class="wipSection">

⚠️ The SUAVE protocol is still in a state where [the code](https://github.com/flashbots/suave-geth) is the most up-to-date protocol spec. The goal of these notes is to gradually evolve into an implementation agnostic specification. ⚠️

</div>

<div class="hideInDocs">

## Specs
- [Suave Chain](./suave-chain.md)
- [MEVM](./mevm.md)
- [Confidential Data Store](./confidential-data-store.md)
- [Precompiles](./precompiles.md)
- [Computor](./computor.md)
- [Bridge](./bridge.md)

---

## About Suave

SUAVE is a platform for building MEV applications such as OFAs and block builders in a decentralized and private way. SUAVE is looking to create a new paradigm for computation, where computation can be private and run at the speed of block building.

---

</div>

## 🥅 Design Goals

1. **Market for Mechanisms** - create a robust environment which offers basic programmable privacy and useful MEV precompiles.
2. **Permissionless SUApp Deployment & Interaction** - Enable permissionless innovation by allowing anyone to deploy and interact with contracts.
3. **SGX UX Closeness** - Get as close to SGX user experience as possible and minimize breaking changes down the road.

---

## 🔒 Design Decisions

Here is a list of design decisions and tradeoffs:

- Decision 1: **Weak DA Layer Guarantees**
    - Reason: [Compute Output Validity and Heterogenous DA](https://collective.flashbots.net/t/suave-ensuring-output-validity-and-heterogenous-da/2184) is an active open question, which whether or not answered does not drastically impact UX on Rigil Testnet.
- Decision 2: **Proposer Centralization**
    - Reason: [SUAVE consensus](https://collective.flashbots.net/t/suave-consensus/2152) is an active open question, which whether or not answered does not drastically impact UX on Rigil Testnet.
- Decision 3: **No SGX Nodes (yet)**
    - Reason: Research continues on the best wa to implement this, which will be detailed in a later spec.

---

## Overview

### Users

The Rigil testnet is initially focused on a specific set of actors:

1. **Developers** - create smart contracts on SUAVE Chain that define rules for MEV applications like orderflow auctions and block building. Smart contracts deployed by developers can call special *precompiles* to request confidential computation.
2.  **Users** - send requests/transactions to SUAVE. Such transaction can include confidential data.
3. **Proposers/Sequencer** - extend another blockchain with a new block.
4. **Builders** - can be implemented as smart contracts inside Suave. In the Rigil Tesnet, Suave submits bundles to several external builders.

### Architecture

SUAVE Computors house all components necessary to perform confidential compute and are the main protocol actor in the SUAVE protocol. Below is a high level architectural overview followed by a brief descriptions of the main components.

![Rigil architecture](/assets/rigil-architecture.svg)

- **Confidential Compute Request (CCR) [**[🔗spec](./specs/rigil/suave-chain.md#confidential-compute-request)**]**: A user request to Suave that contains:
    1. a wrapped transaction
    2. confidential inputs, and 
    3. a list of programs (contracts) (← list of Exec nodes) allowed to operate on confidential inputs.
- **Computor[**[🔗spec](./specs/rigil/computor.md)**]**: accepts and processes confidential compute requests and maintains the SUAVE chain; the logical unit of the SUAVE network and main protocol actor.
- **Confidential Data Store (CDS) [**[🔗spec](./specs/rigil/confidential-data-store.md)**]**: stores confidential data from MEV applications (L1 transactions, EIP 712 signed messages, userOps, privateKeys(?), etc).
- **SUAVE Chain [**[🔗spec](./specs/rigil/suave-chain.md#suave-chain)**]**: a fork of Ethereum designed for usage alongside credible off-chain execution in MEV use cases. Running Clique PoA consensus for the Rigil Testnet.
- **MEVM [**[🔗spec](./specs/rigil/mevm.md)**]**: modified EVM with MEV-specific precompiles that allow developers to define orderflow auctions and builder rules as smart contracts written in Solidity.
- **Precompiles [**[🔗spec](./specs/rigil/precompiles.md)**]:** purpose-built functions with extended capabilities that can be called from Builder Solidity


### Transaction-flow

The SUAVE-enabled node and the MEVM support multiple new data types, which are all specified in the [SUAVE Chain spec](./specs/rigil/suave-chain.md#custom-types). 

The diagram below showcases how these different types interact to enable confidential computation on SUAVE computors.

![Rigil transaction flow](/assets/rigil-tx-flow.svg)

Conceptually, transactions on SUAVE can be split into three steps:

1. User -> RPC
2. RPC -> MEVM
3. MEVM -> Confidential compute result to SUAVE Chain

---

### OFA Example

If we consider a specific use case, like an order flow auction, the high-level series of steps taken to complete the auction can represented as below:

![OFA example flow](/assets/OFA-example-flow.svg)

Conceptually, such auction mechanisms can be split into seven steps:

1. User -> RPC
2. RPC -> MEVM
3. MEVM -> Confidential compute result to SUAVE Chain 
4. Searcher, listening to events on SUAVE -> RPC
5. RPC -> MEVM
6. MEVM -> internal simulation (based on contract logic)
7. MEVM -> send bundle to builder via precompile