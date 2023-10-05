# SUAVE Protocol Specifications

[![Rigil specs ./specs/rigil/](https://img.shields.io/badge/jump%20into-Rigil%20Specs-blue.svg)](./specs/rigil/)
[![Docs at https://suave.flashbots.net/](https://img.shields.io/badge/SUAVE%20docs-blue.svg)](https://suave.flashbots.net/)
[![Join the chat at https://collective.flashbots.net/](https://img.shields.io/badge/chat-on%20Flashbots%20forum-blue.svg)](https://collective.flashbots.net/)

This repository hosts the current SUAVE specifications.

Discussions about design rationale and proposed changes can be brought up and discussed on the [Flashbots forum](https://collective.flashbots.net/). Solidified, agreed-upon changes to the spec can be made through pull requests.

⚠️ The SUAVE protocol is still in a state where [the code](https://github.com/flashbots/) is the most up-to-date protocol spec, but this repository aims to be a starting places to gradually evolve into an implementation agnostic specification.⚠️

---


# Specs

Specifications for the SUAVE protocol can be found in the [`specs`](specs/) subdirectory. Specs are currently organized by testnet during early protocol development.Some testnets will be developed in parallel.


## In-development Specifications

| Testnet | Phase | ChainID | Specs |
| - | - | - | - |
| [Rigil](/specs/rigil/) | [**Big Bang**](/assets/future_roadmap_draft.png) |`16813125` | <ul><li>Core</li><ul><li>[Suave Chain](specs/rigil/suave-chain.md)</li><li>[mevm](specs/rigil/mevm.md)</li><li>[Confidential Data Store](specs/rigil/confidential-data-store.md)</li><li>[Bridge](specs/rigil/bridge.md)</li><li>[MEV Related Precompiles](specs/rigil/precompiles.md)</li></ul></ul></ul>|

---

# Contributing

* Feel free to open PRs and issues, and feel invited to discuss ideas in the [Flashbots Forum](https://collective.flashbots.net/)
* For creating tables of contents, we've started using this VS code extension: [xavierguarch.auto-markdown-toc](https://marketplace.visualstudio.com/items?itemName=xavierguarch.auto-markdown-toc) (recommend the following user setting: `"markdown-toc.depthFrom": 2`)