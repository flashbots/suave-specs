# ☀️ SUAVE Protocol Specifications

<div class="toc">

[![Rigil specs ./specs/rigil/](https://img.shields.io/badge/jump%20into-Rigil%20Specs-purple.svg)](./specs/rigil/)
[![Docs at https://suave.flashbots.net/](https://img.shields.io/badge/read-SUAVE%20docs-blue.svg)](https://suave.flashbots.net/)
[![Join the chat at https://collective.flashbots.net/](https://img.shields.io/badge/chat-on%20Flashbots%20forum-blue.svg)](https://collective.flashbots.net/)

</div>

This repository hosts the current SUAVE specifications.

Discussions about design rationale and proposed changes can be brought up and discussed on the [Flashbots forum](https://collective.flashbots.net/). Solidified, agreed-upon changes to the spec can be made through pull requests.

⚠️ The SUAVE protocol is still in a state where [the code](https://github.com/flashbots/suave-geth) is the most up-to-date protocol spec. However, the goal of these notes is to gradually evolve into a non-programming-language based specification language. ⚠️

---

# Specs

Our specs are currently organized by testnet during early protocol development. Some testnets will be developed in parallel.

## In-development Specifications

| Testnet | Phase | ChainID | Specs |
| - | - | - | - |
| **Rigil**| [Big Bang](/assets/future_roadmap_draft.png) |`16813125` | **Core** <br/> - [The suave chain](./specs/rigil/suave-chain.md) <br/> - [mevm](./specs/rigil/mevm.md) <br/> - [Confidential Data Store](./specs/rigil/confidential-data-store.md) <br/> - [Bridge](./specs/rigil/bridge.md) <br/> - [MEV Related Precompiles](./specs/rigil/precompiles.md)|
| **Sirrah** | [Proto-Collision](/assets/future_roadmap_draft.png) | | |

---

# Contributing

You are welcome to open PRs and issues. We invite you to discuss your ideas in the [Flashbots Forum](https://collective.flashbots.net/) for greater visibility and to avoid doing work that is unlikely to be widely accepted and merged into the core specs.

<div class="toc">

For creating tables of contents, we use the [xavierguarch.auto-markdown-toc VS code extension](https://marketplace.visualstudio.com/items?itemName=xavierguarch.auto-markdown-toc) and recommend the following user setting: `"markdown-toc.depthFrom": 2`. However, we don't need these ToCs rendered in the documentation frontend. Therefore, please add the `<div class="toc"></div>` to any content that should not appear in the FE.

</div>