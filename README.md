# SUAVE Protocol Specifications

SUAVE is a platform for building MEV applications like OFAs and block builders in a decentralized and private way. SUAVE is looking to create a new paradigm for computation, where computation can be private and run at the speed of block building.

For the large part, the SUAVE protocol is still in a state where the code is the most up-to-date protocol spec, but this repository aims to be a starting places to gradually evolve into a non-programming-language based specification language.

---

Links:

- [Rigil Specs](./specs/rigil/)
- [Flashbots Forum](https://collective.flashbots.net/)
- [Flashbots Website](https://www.flashbots.net/)
- SUAVE Docs (coming soon)

todo?
- contribution guide
- Diagram folder

# Specs

Specifications for the SUAVE protocol can be found in [specs](specs/). Specs are currently organized by testnet during early protocol development.Some testnets will be developed in parallel.


### In-development Specifications

| Testnet | Phase | ChainID | Specs |
| - | - | - | - |
| [Rigil](/specs/rigil/) | **Big Bang** |`16813125` | <ul><li>Core</li><ul><li>[The suave chain](specs/rigil/suave-chain.md)</li><li>[mevm](specs/rigil/mevm.md)</li><li>[Confidential Data Store](specs/rigil/confidential-data-store.md)</li><li>[Bridge](specs/rigil/bridge.md)</li><li>[MEV Related Precompiles](specs/rigil/precompiles.md)</li></ul></ul></ul>|


### Contributing

* Feel free to open PRs and issues, and feel invited to discuss ideas in the [Flashbots Forum](https://collective.flashbots.net/)
* For creating tables of contents, we've started using this VS code extension: [xavierguarch.auto-markdown-toc](https://marketplace.visualstudio.com/items?itemName=xavierguarch.auto-markdown-toc) (recommend the following user setting: `"markdown-toc.depthFrom": 2`)