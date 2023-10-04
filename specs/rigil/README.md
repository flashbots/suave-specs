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

SUAVE is a platform for building MEV applications like OFAs and block builders in a decentralized and private way. SUAVE is looking to create a new paradigm for computation, where computation can be private and run at the speed of block building.

---

### Notes

- The types in these Rigil specs are in Golang (for ease of updating and correctness, for now). Over time they might become Python and we might include automated spec-tests.


---

### Design Goals

...

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