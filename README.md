# ☀️ SUAVE Protocol Specifications

<div class="hideInDocs">

[![Rigil specs ./specs/rigil/](https://img.shields.io/badge/jump%20into-Rigil%20Specs-purple.svg)](./specs/rigil/)
[![Docs at https://suave.flashbots.net/](https://img.shields.io/badge/read-SUAVE%20docs-blue.svg)](https://suave.flashbots.net/)
[![Join the discourse at https://collective.flashbots.net/](https://img.shields.io/badge/chat-on%20Flashbots%20forum-blue.svg)](https://collective.flashbots.net/)

This repository hosts the current SUAVE specifications.

</div>

Discussions about design rationale and proposed changes can be brought up and discussed on the [Flashbots forum](https://collective.flashbots.net/). Solidified, agreed-upon changes to the spec can be made through pull requests.

<div class="warning">

⚠️ The SUAVE protocol is still in a state where [the code](https://github.com/flashbots/suave-geth) is the most up-to-date protocol spec. The goal of these notes is to gradually evolve into an implementation agnostic specification. ⚠️

</div>

<!-- TOC depthFrom:10 -->

<!-- /TOC -->


---

# About Suave

SUAVE - Single Unifying Auction for Value Expression - is a platform for building MEV applications such as OFAs and block builders in a decentralized and private way.

Read more about SUAVE:
- https://writings.flashbots.net/the-future-of-mev-is-suave
- https://writings.flashbots.net/mevm-suave-centauri-and-beyond

---

# Specs

Specifications for the SUAVE protocol are currently organized by testnet during early protocol development. Several testnets might be developed in parallel.

## In-development Specifications

| Testnet | Phase | ChainID | Specs |
| - | - | - | - |
| [**Rigil**](./specs/rigil/)  | [Big Bang](/assets/future_roadmap_draft.png) | `16813125` | • [SUAVE chain](./specs/rigil/suave-chain.md) <br/> • [MEVM](./specs/rigil/mevm.md) <br/> • [Confidential Data Store](./specs/rigil/confidential-data-store.md) <br/> • [Precompiles](./specs/rigil/precompiles.md) <br/> • [Bridge](./specs/rigil/bridge.md)  |
| [**Sirrah**](./specs/sirrah/) | [Proto-Collision](/assets/future_roadmap_draft.png) | | |

---

# Contributing

You are welcome to open PRs and issues. We invite you to discuss your ideas in the [Flashbots Forum](https://collective.flashbots.net/) for greater visibility and to avoid doing work that is unlikely to be widely accepted and merged into the core specs.

<div class="hideInDocs">

For creating tables of contents, we use the [huntertran.auto-markdown-toc VS code extension](https://marketplace.visualstudio.com/items?itemName=huntertran.auto-markdown-toc).

However, we don't need the ToCs and some other content (like this paragraph) rendered in the documentation frontend. Therefore, please add the `<div class="hideInDocs"></div>` to any content that should not appear in the FE.

</div>