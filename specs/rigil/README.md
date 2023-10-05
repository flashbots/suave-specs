# SUAVE Rigil Testnet

### Specs
- [Suave Chain](./suave-chain.md)
- [MEVM](./mevm.md)
- [Confidential Data Store](./confidential-data-store.md)
- [Precompiles](./precompiles.md)
- [Computor](./computor.md)
- [Bridge](./bridge.md)

---

### About Suave

SUAVE is a platform for building MEV applications such as OFAs and block builders in a decentralized and private way. SUAVE is looking to create a new paradigm for computation, where computation can be private and run at the speed of block building.

---

### Notes

- The types in these Rigil specs are in Golang (for ease of updating and correctness, for now). Over time they might become Python and we might include automated spec-tests.

---

### Design Goals

1. Launch the Rigil testnet as a step toward [SUAVE Centauri](https://writings.flashbots.net/mevm-suave-centauri-and-beyond) and .
2. Permissionless deployment & interaction with contracts on SUAVE chain.
3. Scaffolding to build related ecosystem tooling.
4. Enable a viable upgrade path to SGX.
5. SGX UX Closeness - Get as close to SGX user experience as possible and minimize breaking changes down the road.

#### **🔒 Design Decisions**
- Decision *1*: ***Weak DA Layer Guarantees***
    - *reason*: **[Compute Output Validity and Heterogenous DA](https://collective.flashbots.net/t/suave-ensuring-output-validity-and-heterogenous-da/2184)** is an active open question, which whether or not answered does not drastically impact UX of users on **Rigil Testnet**.
- Decision *2*: ***Proposer*** ***Centralization***
    - *reason*: [SUAVE consensus](https://collective.flashbots.net/t/suave-consensus/2152) is an active open question, which whether or not answered does not drastically impact UX of users on **Rigil Testnet**.
- Decision *3*: ***Permissionless Smart Contract Interaction***
    - reason: Users deploying and interacting with smart contracts on **Rigil Testnet** is our main goal.
- Decision *4*: **No SGX Nodes (yet)**
    - reason: not feasible or required right now
- Decision *5*: ***SGX UX Closeness***
    - reason: if it impacts DevEx for building an application, we should try to get close to SGX experience as possible to not introduce unnexpected complexity.

---

### High-level Rigil architecture diagram

![Rigil architecture](/assets/rigil-architecture.svg)

### Transaction-flow diagram

![Rigil transaction flow](/assets/rigil-tx-flow.svg)

Steps:

- User -> RPC:
- RPC -> MEVM:
- MEVM -> Chain:

---

TODO:

- OFA diagram