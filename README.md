<!-- no toc -->
# ☀️ SUAVE Protocol Specifications

<div className="hide-in-docs">

[![Rigil specs ./specs/rigil/](https://img.shields.io/badge/jump%20into-Rigil%20Specs-purple.svg)](./specs/rigil/)
[![Docs at https://suave.flashbots.net/](https://img.shields.io/badge/read-SUAVE%20docs-blue.svg)](https://suave.flashbots.net/)
[![Join the discourse at https://collective.flashbots.net/](https://img.shields.io/badge/chat-on%20Flashbots%20forum-blue.svg)](https://collective.flashbots.net/)

This repository hosts the current SUAVE protocol specifications. For SUAPP developers please check out our [developer docs](https://suave.flashbots.net/).

Discussions about design rationale and proposed changes can be brought up and discussed on the [Flashbots forum](https://collective.flashbots.net/). Solidified, agreed-upon changes to the spec can be made through pull requests.

<div class="warning">

⚠️ The SUAVE protocol is still in alpha state, and [the code](https://github.com/flashbots/suave-geth) is the most up-to-date protocol spec. The goal of these notes is to gradually evolve into an implementation-agnostic specification. ⚠️

</div>

</div>

---

# About Suave

SUAVE - *Single Unifying Auction for Value Expression* - is a platform for building MEV applications such as OFAs and block builders in a decentralized and private way.

Read more about SUAVE:
- https://writings.flashbots.net/the-future-of-mev-is-suave
- https://writings.flashbots.net/mevm-suave-centauri-and-beyond

See also: [suave-geth](https://github.com/flashbots/suave-geth), [suapp-examples](https://github.com/flashbots/suapp-examples)

---

# Specs - In-development

Specifications for the SUAVE protocol are split into two main tracks, Centauri and Andromeda.

| Testnet                       | Phase                                               | ChainID    | Specs                                                                                                                                                                                                                                                                                                   |
| ----------------------------- | --------------------------------------------------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [**Rigil**](./specs/rigil/)   | [Big Bang](/assets/future_roadmap_draft.png)        | `16813125` | • [SUAVE chain](./specs/rigil/suave-chain.md) <br/> • [MEVM](./specs/rigil/mevm.md) <br/> • [Precompiles](./specs/rigil/precompiles.md) <br/> • [Confidential Data Store](./specs/rigil/confidential-data-store.md) <br/> • [Bridge](./specs/rigil/bridge.md) <br/> • [Kettle](./specs/rigil/kettle.md) |
| [**Toliman**](./specs/toliman/) | [Big Bang](/assets/future_roadmap_draft.png) |  `33626250`          | • [SUAVE chain](./specs/toliman/suave-chain.md) <br/> • [MEVM](./specs/toliman/mevm.md) <br/> • [Precompiles](./specs/toliman/precompiles.md) <br/> • [Confidential Data Store](./specs/toliman/confidential-data-store.md) <br/> • [Bridge](./specs/toliman/bridge.md) <br/> • [Kettle](./specs/toliman/kettle.md) |
| [**Sirrah**](./specs/sirrah/) | [Proto-Collision](/assets/future_roadmap_draft.png) |            |                                                                                                                                                                                                                                                                                                         |

## Roadmap

**Centauri* is focused on exploration and developer experience, serving as a spec for the secure instantiation. The plan is a progression of three testnets in the direction of increasing functionality and scalability.*

**Andromeda* is the secure instantiation using SGX and cryptography, focusing on privacy with trust minimization. The plan is a progression of three milestones in the direction of better hardening and trust minimization.*

**Proto-Collisions* are plans to sync up between these two development tracks. They represents points where parts of Centauri testnets can be run in secure and trust minimized fashion. Permissioned components from Proto-Collisions, if self-contained, could be used with Ethereum mainnet.*

**Collision* is where the entirety of the SUAVE protocol can be safely deployed for use with Ethereum mainnet in a trust minimized and permissionless fashion.*

![SUAVE Roadmap](/assets/future_roadmap_draft.png)

---

# Contributing

You are welcome to open PRs and issues. We invite you to discuss your ideas in the [Flashbots Forum](https://collective.flashbots.net/) for greater visibility and to avoid doing work that is unlikely to be widely accepted and merged into the core specs.

<div className="hide-in-docs">

For creating tables of contents, we use the ["Markdown All in One" VS code extension](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one). This extension comes with a lot of keyboard shortcuts, which may override the ones you are already using. You can disable keyboard shortcuts for this extension by going to "Settings -> Keyboard Shortcuts" (cmd+K cmd+S) and searching for `markdown.ext` (see [#22](https://github.com/flashbots/suave-specs/pull/22) for more details).

However, we don't need the ToCs and some other content (like this paragraph) rendered in the documentation frontend. Therefore, please add the `<div className="hide-in-docs"></div>` to any content that should not appear in the FE.

</div>

---

Made with ☀️ by the ⚡🤖 collective.
