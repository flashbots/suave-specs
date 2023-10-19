---
title: Bridge
description: A description of our current bridge implementation, which is meant to enable rapid prototyping.
---

<!-- no toc -->
<!-- omit from toc -->
# Bridge

SUAVE uses a simple and highly trusted bridge with the Goerli Ethereum network to transfer assets for gas and MEV applications.

The current bridge implementation is meant to enable rapid prototyping. It is not meant for production / mainnet usage.

## Components

The bridge code consists of two components:
1. on-chain `GasBridge` contract
2. off-chain `relayerClient`

An implementation of a `GasBridge` smart contract and the trusted `relayerClient` backend can be found [here](https://github.com/flashbots/suave-bridge/blob/master/contracts/GasBridge.sol) and [here](https://github.com/flashbots/suave-bridge/blob/master/internal/bridge.go) respectively.

The `GasBridge` solidity is responsible for emitting events that the `relayerClient` uses as a signal to mint on the SUAVE chain. The `relayerClient` backend initializes and manages a bridge between two Ethereum chains (rootchain and sidechain) for monitoring and relaying events. The  `relayerClient` code includes functions for tracking events, handling transactions, and managing event states.

```solidity
interface GasBridge {
    event GasEvent(address indexed sender, uint256 indexed id, uint256 amount);

    function transfer() external payable
}
```

```go
type relayerClient interface {
	SendTransaction(ctx context.Context, event *GasEvent) (string, error)
	TransactionReceipt(ctx context.Context, hash common.Hash) (*types.Receipt, error)
}
```
