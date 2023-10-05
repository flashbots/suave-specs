# ☀️ SUAVE Protocol Specifications

<div class="toc">

[![Rigil specs ./specs/rigil/](https://img.shields.io/badge/jump%20into-Rigil%20Specs-purple.svg)](./specs/rigil/)
[![Docs at https://suave.flashbots.net/](https://img.shields.io/badge/read-SUAVE%20docs-blue.svg)](https://suave.flashbots.net/)
[![Join the chat at https://collective.flashbots.net/](https://img.shields.io/badge/chat-on%20Flashbots%20forum-blue.svg)](https://collective.flashbots.net/)

</div>

This repository hosts the current SUAVE specifications.

Discussions about design rationale and proposed changes can be brought up and discussed on the [Flashbots forum](https://collective.flashbots.net/). Solidified, agreed-upon changes to the spec can be made through pull requests.

⚠️ The SUAVE protocol is still in a state where [the code](https://github.com/flashbots/suave-geth) is the most up-to-date protocol spec, but this repository aims to be a starting places to gradually evolve into an implementation agnostic specification. ⚠️

---

# Specs

Specifications for the SUAVE protocol are currently organized by testnet during early protocol development. Several testnets might be developed in parallel.

## In-development Specifications

| Testnet | Phase | ChainID | Specs |
| - | - | - | - |
| [**Rigil**](/specs/rigil/) | [Big Bang](/assets/future_roadmap_draft.png) |`16813125` | <ul><li>[Suave Chain](specs/rigil/suave-chain.md)</li><li>[MEVM](specs/rigil/mevm.md)</li><li>[Confidential Data Store](specs/rigil/confidential-data-store.md)</li><li>[Precompiles](specs/rigil/precompiles.md)</li><li>[Bridge](specs/rigil/bridge.md)</li></ul></ul>|
| [**Sirrah**](/specs/sirrah/) | [Proto-Collision](/assets/future_roadmap_draft.png) | | |



---

# Contributing

You are welcome to open PRs and issues. We invite you to discuss your ideas in the [Flashbots Forum](https://collective.flashbots.net/) for greater visibility and to avoid doing work that is unlikely to be widely accepted and merged into the core specs.

For creating tables of contents, we use the [xavierguarch.auto-markdown-toc VS code extension](https://marketplace.visualstudio.com/items?itemName=xavierguarch.auto-markdown-toc) and recommend the following user setting: `"markdown-toc.depthFrom": 2`. However, we don't need these ToCs rendered in the documentation frontend. Therefore, please add the `<div class="toc"></div>` to any content that should not appear in the FE.