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