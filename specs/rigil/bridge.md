# Bridge

# Standard Bridge

This bridge is capable of transferring assets from a single domain to SUAVE chains via offchain trusted minter and is not a design intended for production. 

## Components

The standard bridge consists of two components, an onchain `GasBridge` contract and an offchain `relayerClient`. An implementation of a `GasBridge` smart contract and the trusted `relayerClient` backend can be found [here](https://github.com/flashbots/suave-bridge/blob/master/contracts/GasBridge.sol) and [here](https://github.com/flashbots/suave-bridge/blob/master/internal/bridge.go) respectively. . The `GasBridge` solidity is responsible for emitting events that the `relayerClient` uses as a signal to mint on the SUAVE chain. The `relayerClient` backend initializes and manages a bridge between two Ethereum chains (rootchain and sidechain) for monitoring and relaying events. The  `relayerClient`code includes functions for tracking events, handling transactions, and managing event states. 

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