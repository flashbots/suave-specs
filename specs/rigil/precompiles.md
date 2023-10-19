---
title: Precompiles
description: Precompiles are a convenient tool within the EVM that enable direct execution of predefined functions, simplifying development of complex applications.
---

<!-- omit from toc -->
# Precompiles

<div class="hideInDocs">

**Table of Contents**

<!-- TOC -->

- [Overview](#overview)
- [Precompiles Governance](#precompiles-governance)
- [Precompiles](#precompiles-1)
  - [`IsConfidential`](#isconfidential)
  - [`ConfidentialInputs`](#confidentialinputs)
  - [`ConfidentialStore`](#confidentialstore)
  - [`ConfidentialRetrieve`](#confidentialretrieve)
  - [`NewBid`](#newbid)
  - [`FetchBids`](#fetchbids)
  - [`SimulateBundle`](#simulatebundle)
  - [`ExtractHint`](#extracthint)
  - [`BuildEthBlock`](#buildethblock)
  - [`SubmitEthBlockBidToRelay`](#submitethblockbidtorelay)

<!-- /TOC -->

---

</div>

## Overview

Precompiles are a convenient tool within the Ethereum Virtual Machine (EVM) that enable direct execution of predefined functions, bypassing the need for an actual contract. Each precompile is associated with a fixed address defined within the EVM. Unlike regular contracts, there is no associated bytecode at that address.

In the context of the SUAVE protocol, the concept of "precompiles" extends to the MEVM. The precompiles available in the MEVM are purpose-built to cater to specific Maximal Extractable Value (MEV) use cases.

For a comprehensive reference, consult the [suave-geth](https://github.com/flashbots/suave-geth/) reference implementation.

## Precompiles Governance

Here is a rough outline of the initial governance process for adding precompiles:

- Discuss the idea in a [forum post](https://collective.flashbots.net/)
- Open a PR and provide implementation
- Feedback and review
- Possibly merge and deploy in the next network upgrade

## Precompiles

### `IsConfidential`

Implementation (link-to-github-or-other-source)

Address: `0x42010000`

Determines if the current execution mode is regular (on-chain) or confidential. Outputs a boolean value.

### `ConfidentialInputs`

Implementation (link-to-github-or-other-source)

Address: `0x42010001`

Provides the confidential inputs associated with a confidential computation request. Outputs are in bytes format.

### `ConfidentialStore`

Implementation (link-to-github-or-other-source)

Address: `0x42020000`

Handles the storage of values in the confidential store. Requires the caller to be part of the `AllowedPeekers` for the associated bid.

### `ConfidentialRetrieve`

Implementation (link-to-github-or-other-source)

Address: `0x42020001`

Retrieves values from the confidential store. Also mandates the caller's presence in the `AllowedPeekers` list for the bid.

### `NewBid`

Implementation (link-to-github-or-other-source)

Address: `0x42030000`

Initializes bids within the ConfidentialStore. Prior to storing data, all bids should undergo initialization via this precompile.

### `FetchBids`

Implementation (link-to-github-or-other-source)

Address: `0x42030001`

Retrieves all bids correlating with a specified decryption condition.

### `SimulateBundle`

Implementation (link-to-github-or-other-source)

Address: `0x42100000`

Conducts a simulation of the bundle, building a block that includes it. Outputs indicate if the apply was successful and the EGP of the resultant block.

### `ExtractHint`

Implementation (link-to-github-or-other-source)

Address: `0x42100037`

Interprets the bundle data and extracts hints, such as the "To" address and calldata.

### `BuildEthBlock`

Implementation (link-to-github-or-other-source)

Address: `0x42100001`

Constructs an Ethereum block based on the provided bid. The construction follows a specified order.

### `SubmitEthBlockBidToRelay`

Implementation (link-to-github-or-other-source)

Address: `0x42100002`

Submits a given builderBid to a boost relay. Outputs any errors that arise during submission.

