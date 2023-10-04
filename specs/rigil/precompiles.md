# Precompiles
**Table of Contents**

1. [Overview](#overview)
2. [Precompile Governance](#precompiles-governance)
3. [Precompiles](#overview)
    - 3.1. [IsConfidential](#isconfidential)
    - 3.2. [ConfidentialInputs](#confidentialinputs)
    - 3.3. [ConfidentialStore](#confidentialstore)
    - 3.4. [ConfidentialRetrieve](#confidentialretrieve)
    - 3.5. [NewBid](#newbid)
    - 3.6. [FetchBids](#fetchbids)
    - 3.7. [SimulateBundle](#simulatebundle)
    - 3.8. [ExtractHint](#extracthint)
    - 3.9. [BuildEthBlock](#buildethblock)
    - 3.10. [SubmitEthBlockBidToRelay](#submitethblockbidtorelay)

## Overview

Precompiles are a convenient tool within the Ethereum Virtual Machine (EVM) that allows for the execution of predefined functions directly by the EVM, bypassing the need for an actual contract. Each precompile is associated with a fixed address defined within the EVM, and unlike regular contracts, there is no associated bytecode at that address.

Within the context of the SUAVE protocol, the concept of "precompiles" extends to the MEVM. The MEVM serves as a modified version of the EVM and introduces a collection of specialized and optimized functions, often referred to as precompiles. These precompiles are purpose-built to cater to specific Maximal Extractable Value (MEV) use cases.

For a comprehensive reference, consult the [`suave-geth` reference implementation](link).

## Precompiles Governance

Here whould describe the governance process for precompiles, roughly:
- Forum post
- open PR and provide implementation 
- Feedback and Review
- merge
- deploy in network upgrade

Here is where you can find a precompile upgrade process request template

## Precompiles

### IsConfidential

[Implementation](link-to-github-or-other-source)

Address: `0x42010000`

Determines if the current execution mode is regular (on-chain) or confidential. Outputs a boolean value.

### ConfidentialInputs

[Implementation](link-to-github-or-other-source)

Address: `0x42010001`

Provides the confidential inputs associated with a confidential computation request. Outputs are in bytes format.

### ConfidentialStore

[Implementation](link-to-github-or-other-source)

Address: `0x42020000`

Handles the storage of values in the confidential store. Requires the caller to be part of the `AllowedPeekers` for the associated bid.

### ConfidentialRetrieve

[Implementation](link-to-github-or-other-source)

Address: `0x42020001`

Retrieves values from the confidential store. Also mandates the caller's presence in the `AllowedPeekers` list for the bid.

### NewBid

[Implementation](link-to-github-or-other-source)

Address: `0x42030000`

Initializes bids within the ConfidentialStore. Prior to storing data, all bids should undergo initialization via this precompile.

### FetchBids

[Implementation](link-to-github-or-other-source)

Address: `0x42030001`

Retrieves all bids correlating with a specified decryption condition.

## SimulateBundle

[Implementation](link-to-github-or-other-source)

Address: `0x42100000`

Conducts a simulation of the bundle, building a block that includes it. Outputs indicate if the apply was successful and the EGP of the resultant block.

### ExtractHint

[Implementation](link-to-github-or-other-source)

Address: `0x42100037`

Interprets the bundle data and extracts hints, such as the "To" address and calldata.

### BuildEthBlock

[Implementation](link-to-github-or-other-source)

Address: `0x42100001`

Constructs an Ethereum block based on the provided bid. The construction follows a specified order.

### SubmitEthBlockBidToRelay

[Implementation](link-to-github-or-other-source)

Address: `0x42100002`

Submits a given builderBid to a boost relay. Outputs any errors that arise during submission.

