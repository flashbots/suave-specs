# SUAVE Rigil Testnet

⚠️ The SUAVE protocol is still in a state where [the code](https://github.com/flashbots/) is the most up-to-date protocol spec, but this repository aims to be a starting places to gradually evolve into an implementation agnostic specification.⚠️

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

## 🥅 Design Goals

1. **Market for Mechanisms** - create an environment that is robust, offers basic programmable privacy and MEV primitives.
2. **Permissionless SUApp Deployment & Interaction** - Enable permissionless innovation by allowing anyone to deploy and interact with contracts.
3. **SGX UX Closeness** - Get as close to SGX user experience as possible and minimize breaking changes down the road.


---

## 🔒 Design Decisions

Here is a list of design decisions and tradeoffs:
- Decision *1*: ***Weak DA Layer Guarantees***
    - *reason*: [Compute Output Validity and Heterogenous DA](https://collective.flashbots.net/t/suave-ensuring-output-validity-and-heterogenous-da/2184) is an active open question, which whether or not answered does not drastically impact UX of users on **Rigil Testnet**.
- Decision *2*: ***Proposer*** ***Centralization***
    - *reason*: [SUAVE consensus](https://collective.flashbots.net/t/suave-consensus/2152) is an active open question, which whether or not answered does not drastically impact UX of users on **Rigil Testnet**.
- Decision *3*: **No SGX Nodes (yet)**
    - reason: not feasible or required right now

---

## Overview

### Users
The Rigil testnet is initially focused on a specific set of users:xs
- **Developers** - create smart contracts on SUAVE Chain that define rules for MEV applications like orderflow auctions and block building. These can call special *precompiles* to request computation on *confidential data* using the *confidential data store.*
- **Users** - send requests/transactions to SUAVE. These are composed of *confidential data* and a set of programs the user allows to interact with its confidential data.
- **Proposers/Sequencer** - extend a domain chain with a new block.
- **Builders** - can be implemented as smart contracts inside Suave. In the Rigil release, Suave submits bundles to several external builders.

### Architecture

SUAVE Computors house all components necessary to perform confidential compute and are the main protocol actor in the SUAVE protocol. Below is a high level architectural overview followed by brief descriptions of the main components.

![Rigil architecture](/assets/rigil-architecture.svg)

- **Confidential Compute Request (CCR) [**[🔗spec](https://github.com/flashbots/suave-specs/blob/main/specs/rigil/suave-chain.md#confidential-compute-request)**]**: A user request to Suave that contains (1) a wrapped transaction, (2) confidential inputs, and (3) a list of programs (contracts) (← list of Exec nodes) allowed to operate on confidential inputs.
- **Computor[[🔗spec](https://github.com/flashbots/suave-specs/blob/main/specs/rigil/computor.md)]**: accepts and processes confidential compute requests and maintains the SUAVE chain; the logical unit of the SUAVE network and main protocol actor.
- **Confidential Data Store (CDS) [[🔗spec](https://github.com/flashbots/suave-specs/blob/main/specs/rigil/confidential-data-store.md)]**: stores confidential data from MEV applications (L1 transactions, EIP 712 signed messages, userOps, privateKeys(?), etc).
- **SUAVE PoA Chain [**[🔗spec](https://github.com/flashbots/suave-specs/blob/main/specs/rigil/suave-chain.md#suave-chain)**]**: a fork of Ethereum designed for usage alongside credible off-chain execution in MEV use cases. In Rigil running in PoA mode.
- **MEVM [[🔗spec](https://github.com/flashbots/suave-specs/blob/main/specs/rigil/mevm.md)]**: modified EVM with MEV-specific precompiles that allow developers to define orderflow auctions and builder rules as smart contracts written in Solidity.
- **Precompiles [[🔗spec](https://github.com/flashbots/suave-specs/blob/main/specs/rigil/precompiles.md)]:** purpose-built functions with extended capabilities that can be called from Builder Solidity


### Transaction-flow
SUAVE chain and the MEVM support multiple new data types: Confidential Compute Requests, Compute Compute Record, and SUAVE Transactions. The diagram below showcases how these different formats interact to enable confidential computation on SUAVE computors.
![Rigil transaction flow](/assets/rigil-tx-flow.svg)

- User -> RPC:
- RPC -> MEVM:
- MEVM -> Chain:

---

### OFA Example


![OFA example flow](/assets/OFA-example-flow.svg)

