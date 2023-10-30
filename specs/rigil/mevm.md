---
title: MEVM
description: The MEVM modifies the EVM by adding a new runtime, interpreter, and execution backend so as to enable anyone to create MEV applications.
---

<div class="hideInDocs">

<!-- omit from toc -->
# MEVM

<!-- TOC -->

- [Overview](#overview)
- [Modified Interpreter](#modified-interpreter)
    - [Confidential Data Store APIs](#confidential-data-store-apis)
    - [`suavex` namespace](#suavex-namespace)
- [Precompiles](#precompiles)
    - [`IsConfidential`](#isconfidential)
    - [`ConfidentialInputs`](#confidentialinputs)
    - [`ConfidentialStore`](#confidentialstore)
    - [`ConfidentialRetrieve`](#confidentialretrieve)
    - [`NewBid`](#newbid)
    - [`FetchBids`](#fetchbids)
    - [`SimulateBundle`](#simulatebundle)
    - [`ExtractHint`](#extracthint)
    - [`SubmitBundleJsonRPC`](#submitbundlejsonrpc)
    - [`FillMevShareBundleAddress`](#fillmevsharebundleaddress)
    - [`BuildEthBlock`](#buildethblock)
    - [`SubmitEthBlockBidToRelay`](#submitethblockbidtorelay)
    - [SignEthTransaction](#signethtransaction)
- [Precompile Call Authorization](#precompile-call-authorization)
- [Precompiles Governance](#precompiles-governance)
- [Security and Confidentiality](#security-and-confidentiality)

<!-- /TOC -->

---

</div>

## Overview

This document provides the technical specification for the MEVM, a modified version of the Ethereum Virtual Machine (EVM). The MEVM is a set of precompiles to interact with APIs, two of these API services are the Confidential Data Store and the SUAVE Execution (SUAVEX) name space. There may be other API endpoints in the future.

## Modified Interpreter

Under the hood the MEVM is a modified EVM Interpreter which is able to use a new runtime called the `SuaveExecutionBackend`.

```mermaid
graph TB
    A[EVM]-->|1|B((StateDB))
    A-->|2|C((Context))
    A-->|3|D((chainConfig))
    A-->|4|E((Config))
    A-->|5|F((interpreter))
    D-->|6|R[ChainRules]
    E-->|7|S[Tracer]
    A-->|8|T[NewRuntime]
    T-->|9|Z((Runtime))
    Z-->|10|F
    A-->|11|U[NewRuntimeSuaveExecutionBackend]
    U-->|12|V((SuaveExecutionBackend))
    V-->|13|F
    class A,B,C,D,E,F yellow
    class G,H,I,J,K,L,M,N,O red
    class P,Q green
    class R blue
    class S orange
    class T,U purple
    class Z,V lightgreen
    classDef yellow fill:#f5cf58,stroke:#444,stroke-width:2px, color:#333;
    classDef red fill:#d98686,stroke:#444,stroke-width:2px, color:#333;
    classDef green fill:#82a682,stroke:#444,stroke-width:2px, color:#333;
    classDef blue fill:#9abedc,stroke:#444,stroke-width:2px, color:#333;
    classDef orange fill:#f3b983,stroke:#444,stroke-width:2px, color:#333;
    classDef purple fill:#ab92b5,stroke:#444,stroke-width:2px, color:#333;
    classDef lightgreen fill:#b3c69f,stroke:#444,stroke-width:2px, color:#333;
```

The `SuaveExecutionBackend` is responsible for exposing the APIs that precompiles are able to hook into.

```go
type SuaveExecutionBackend struct {
	ConfidentialStore      ConfidentialStore
	ConfidentialEthBackend suave.ConfidentialEthBackend
}
```

#### Confidential Data Store APIs

For more information on the capabilities exposed by the Confidential Data Store, see it's related [🔗 spec](./confidential-data-store.md). The interface exposed to precompiles:

```go
type ConfidentialStore interface {
	InitializeBid(bid types.Bid) (types.Bid, error)
	Store(bidId suave.BidId, caller common.Address, key string, value []byte) (suave.Bid, error)
	Retrieve(bid types.BidId, caller common.Address, key string) ([]byte, error)
	FetchBidById(suave.BidId) (suave.Bid, error)
	FetchBidsByProtocolAndBlock(blockNumber uint64, namespace string) []suave.Bid
}
```

#### `suavex` namespace

The `suavex` namespace is used internally by the MEVM to enable functionality like block building and external API calls via MEVM precompiles. We take this approach to make upstream updates and maintenance easier. Current endpoints include:

`suavex_buildEthBlockFromBundles` - takes an array of bundles and transactions, calculates state root and related fields, and returns a valid Ethereum L1 block.

```go
BuildEthBlockFromBundles(ctx context.Context, buildArgs *types.BuildBlockArgs, bundles []types.SBundle) (*engine.ExecutionPayloadEnvelope, error)
```

`suavex_buildEthBlock` - takes an array of transactions, calculates state root and related fields, and returns a valid Ethereum L1 block.

```go
BuildEthBlock(ctx context.Context, buildArgs *types.BuildBlockArgs, txs types.Transactions) (*engine.ExecutionPayloadEnvelope, error)
```

Domain specific services which seek to be used by SUAVE must implement the methods in this namespace. More details will be expanded in future iterations.

## Precompiles


Precompile are MEVM contracts that are implemented in native code instead of bytecode. Precompiles additonally can communicate with internal APIs. Currently the MEVM supports all existing Ethereum Precompiles up to Dencun, and introduces four new classes of precompiles:
- offchain computation that is too expensive in solidity
- calls to API methods to interact with the Confidential Data Store
- calls to `sauvex` API Methods to interact with Domain Specific Services 
- calls to retrieve context for the confidential compute requests

A list of available precompiles in Rigil are as follows:

#### `IsConfidential`

TODO: 🔗 Implementation 

Address: `0x0000000000000000000000000000000042010000`


Determines if the current execution mode is regular (onchain) or confidential. Outputs a boolean value.

```solidity
function isConfidential() internal view returns (bool b)
```

#### `ConfidentialInputs`

TODO: 🔗 Implementation 

Address: `0x0000000000000000000000000000000042010001`

Provides the confidential inputs associated with a confidential computation request. Outputs are in bytes format.

```solidity
function confidentialInputs() internal view returns (bytes memory)
```

#### `ConfidentialStore`

TODO: 🔗 Implementation 

Address: `0x0000000000000000000000000000000042020000`

Handles the storage of data in the confidential store. Requires the caller to be part of the `AllowedPeekers` for the associated bid.

```solidity
function confidentialStoreStore(BidId bidId, string memory key, bytes memory data1) internal view 
```

#### `ConfidentialRetrieve`

TODO: 🔗 Implementation 

Address: `0x0000000000000000000000000000000042020001`

Retrieves data from the confidential store. Also mandates the caller's presence in the `AllowedPeekers` list.

```solidity
function confidentialStoreRetrieve(BidId bidId, string memory key) internal view returns (bytes memory) 
```

#### `NewBid`

TODO: 🔗 Implementation 

Address: `0x0000000000000000000000000000000042030000`

Initializes bids within the ConfidentialStore. Prior to storing data, all bids should undergo initialization via this precompile.

```solidity
function newBid(uint64 decryptionCondition, address[] memory allowedPeekers, string memory bidType)
```

*Note: The name Bids are an artefact from early development. Bids represent a "Data Identifier" used when operating on confidential data and no longer have any relation to a bid in an auction. They useful for coordinating on confidential data with out revealing underlying data, for instance a SUAVE transaction can emit logs on chain which reference the `bidId` from a Confidential Compute Request which follow up transactions can now reference.*

#### `FetchBids`

TODO: 🔗 Implementation 

Address: `0x0000000000000000000000000000000042030001`

Retrieves all bids correlating with a specified decryption condition.

```solidity
function fetchBids(uint64 cond, string memory namespace) internal view returns (Bid[] memory)
```

#### `SimulateBundle`

TODO: 🔗 Implementation 

Address: `0x0000000000000000000000000000000042100000`

Performs a simulation of the bundle by building a block that includes it. Outputs indicate if the execution was successful and the Effective Gas Price of the resultant block.

```solidity
function simulateBundle(bytes memory bundleData) internal view returns (uint64)
```

#### `ExtractHint`

TODO: 🔗 Implementation 

Address: `0x0000000000000000000000000000000042100037`

Interprets the bundle data and extracts hints, such as the "To" address and calldata.

```solidity
function extractHint(bytes memory bundleData) internal view returns (bytes memory)
```

#### `SubmitBundleJsonRPC`

TODO: 🔗 Implementation 

Address: `0x0000000000000000000000000000000043000001`

Submits bytes as JSONRPC message to the specified URL with the specified method. As this call is intended for bundles, it also signs the params and adds `X-Flashbots-Signature` header, as usual with bundles.  
Regular eth bundles don't need any processing to be sent.  

```solidity
function submitBundleJsonRPC(string memory url, string memory method, bytes memory params) internal view returns (bytes memory)
```

#### `FillMevShareBundleAddress`

TODO: 🔗 Implementation 

Address: `0x0000000000000000000000000000000043200001`

Joins the user's transaction and with the backrun, and returns encoded mev-share bundle. The bundle is ready to be sent via `SubmitBundleJsonRPC`.  

```solidity
function fillMevShareBundle(BidId bidId) internal view returns (bytes memory)
```

#### `BuildEthBlock`

TODO: 🔗 Implementation 

Address: `0x0000000000000000000000000000000042100001`

Constructs an Ethereum block based on the provided `bidIds`. The construction follows the order of `bidId`s are given .

```solidity
function buildEthBlock(BuildBlockArgs memory blockArgs, BidId bidId, string memory namespace)
```

#### `SubmitEthBlockBidToRelay`

TODO: 🔗 Implementation 

Address: `0x0000000000000000000000000000000042100002`

Submits a given builderBid to a mev-boost relay. Outputs any errors that arise during submission.

```solidity
function submitEthBlockBidToRelay(string memory relayUrl, bytes memory builderBid)
```

#### `SignEthTransaction`

TODO: 🔗 Implementation 

Address: `0xTBD`	

Signs an Ethereum Transaction, 1559 or Legacy, and returns raw signed transaction bytes. `txn` is binary encoding of the transaction. `signingKey` is hex encoded string of the ECDSA private key *without the 0x prefix*. `chainId` is a hex encoded string *with 0x prefix*. 

```solidity
function signEthTransaction(bytes memory txn, string memory chainId, string memory signingKey) view returns (bytes memory)
```


##  Precompile Call Authorization
`checkIsPrecompileCallAllowed` implements the access control functionality. It validates whether a precompile and associated callers are authorized to access specific data. Key security functionality is as follows:

1. **Universal Access**: If data allows the "any peeker", the function searches the `CallerStack` for a caller different from the precompile, granting access upon finding one.

2. **Restricted Access and Validation**:
   - The function checks if the precompile is explicitly authorized to access bid data. 
   - It then validates each caller in the `CallerStack` against the `AllowedPeekers` list. If an authorized caller is found, access is granted.

3. **Error Handling**: 
   - Access is denied, with an error returned, if no authorized caller is found or if the precompile lacks authorization and is not a special case (like confStore addresses).


## Precompiles Governance

Governance process for adding precompiles is in it's early stages but is as follows:
- Discuss the idea in a [forum post](https://collective.flashbots.net/)
- Open a PR and provide implementation
- Feedback and review
- Possibly merge and deploy in the next network upgrade, or sooner, depending on the precompile


## Security and Confidentiality

`checkIsPrecompileCallAllowed` ensures that either the precompile or one of its callers must be authorized for access, enhancing security by strict validation of permissions. This approach is necessary for maintaining data integrity and preventing unauthorized access, but ultimately not entirely sufficient for total confidentiality. Future iterations will delve deeper into encryption methodologies, access controls, and auditing mechanisms to fortify data privacy further, as well as Trusted Execution Environment to ensure access control is maintained.
