---
title: Confidential Data Store
description: This essential component of the SUAVE protocol serves as a secure and privacy-focused storage system.
---

<!-- TOC -->

- [Confidential Data Store](#confidential-data-store)
    - [Overview](#overview)
    - [Architecture](#architecture)
    - [Core Components](#core-components)
        - [Confidential Store Engine](#confidential-store-engine)
        - [Confidential Store](#confidential-store)
        - [Transport](#transport)
        - [Confidential APIs](#confidential-apis)
    - [Data Management](#data-management)
        - [Initialization & Access Control](#initialization--access-control)
        - [Security and Confidentiality](#security-and-confidentiality)

<!-- /TOC -->

This document provides an overview of the Confidential Data Store, an essential component of the SUAVE protocol. The Confidential Store serves as a secure and privacy-focused storage system by exposing a key-value store for safeguarding confidential data related to your MEV application. Examples of data that can be stored here are: transactions, UserOps, signed messages, bundles, simulation results and intermediate values, partial blocks, full blocks, netflix passwords and more. Only contracts with appropriate permissions (peekers) can access the stored data, allowing developers to define the entire data model of their application!

The Confidential Data Store can be seen as a subtrate for building private orderflow and routing layers, expect more customizability to come to this component in the near future.

## Architecture

The Confidential Data Store consists of three main components, Engine, Storage, and Transport. The engine is the main orchestrating component responisble for managing calls to storage, and as well sending and receiving synchronization messages over the transport. The MEVM is able to directly interact with the Confidential Store Engine through precompiles.

![Confidential Data Store Diagram](/assets/rigil_confidential_data_store.svg)

## Core Components

### Confidential Store Engine

The current, and certainly not final, implementation of the Confidential Store is managed by the ConfidentialStoreEngine. The engine consists of a storage backend, which holds the raw data, and a transport topic, which relays synchronization messages between nodes.

In our [suave-geth](https://github.com/flashbots/suave-geth/tree/main)  reference implementation we provide two storage backends to the confidential store engine: the LocalConfidentialStore, storing data in memory in a simple dictionary, and RedisStoreBackend, storing data in redis. For synchronization of confidential stores via transport we provide an implementation using a shared Redis PubSub in RedisPubSubTransport, as well as a crude synchronization protocol. Note that Redis transport only synchronizes current state, there is no initial synchronization - a newly connected node will not have access to old data, for now.

Redis as either storage backend or transport is temporary and will be removed once we have a well-tested p2p solution.

The `ConfidentialStoreEngine` provides the following key methods:

- *Initialize*: This method is used to initialize a bid. The method is trusted, meaning it is not directly accessible through precompiles. The method returns the initialized bid, importantly with the Id field set.
- *Store*: This method stores a given value under a specified key in a bid's dataMap. Access is restricted only to addresses listed in the bid's AllowedPeekers.
- *Retrieve*: This method retrieves data associated with a given key from a bid's dataMap. Similar to the Store method, access is restricted only to addresses listed in the bid's AllowedPeekers.

It is important to note that the actual implementation of the Confidential Store will vary depending on future requirements and the privacy mechanisms used.

```go
type ConfidentialStoreEngine struct {
	ctx    context.Context
	cancel context.CancelFunc

	storage        ConfidentialStorageBackend
	transportTopic StoreTransportTopic

	daSigner    DASigner
	chainSigner ChainSigner

	storeUUID      uuid.UUID
	localAddresses map[common.Address]struct{}
}
```

### Confidential Store

From our [suave-geth](https://github.com/flashbots/suave-geth/tree/main) reference implementation, `LocalConfidentialStore` serves as the foundation for the Confidential Store. Implemented as a thread-safe struct, it provides secure access to bid data.

TODO: This section is very implementation specific, worth it to generalzie a bit

```go
type LocalConfidentialStore struct {
  lock sync.Mutex
  bids map[suave.BidId]ACData
}
```

`ACData` struct holds the bid information and a `dataMap` that retains the confidential bid data.

```go
type ACData struct {
  bid     suave.Bid
  dataMap map[string][]byte
}
```

### Transport

Transport interface facilitates a bi-directional flow of information, ensuring that data remains consistent across different nodes. By using transport interfaces and protocols, we aim to foster rapid, reliable, and secure data transmission.
The Transport interface, as defined in our Golang reference:


```go
type StoreTransportTopic interface {
  node.Lifecycle
  Subscribe() (<-chan DAMessage, context.CancelFunc)
  Publish(DAMessage)
}
```

Subscribe and Publish:
* Subscribe(): Nodes call this method to subscribe to a particular topic or channel. Upon subscription, nodes can start receiving synchronization messages and other relevant data updates. The method returns a channel from which messages can be read, and a CancelFunc to stop the subscription when required.
* Publish(DAMessage): This method allows nodes to broadcast messages to all subscribers of a particular topic or channel. It's crucial for updating peers about new data or changes in the existing data.

Underlying Protocols:

While our current implementation leverages a shared Redis PubSub for synchronization, our roadmap envisions transitioning to a peer-to-peer (p2p) solution. Such a decentralized approach can offer improved resilience and reduced dependencies on third-party systems. The P2P solution will employ a gossip protocol to ensure that all nodes in the network receive the latest updates.
Future Implementations:
As the network grows, and more sophisticated needs arise, we might also consider implementing:
* Rate Limiting: To prevent any single node from overwhelming the system with too many messages.
* Batching: Sending multiple data pieces in a single transport packet to reduce overheads and improve throughput.
* Network Topology Awareness: Optimizing data routes based on the network's structure and conditions to ensure faster data delivery.


### Confidential APIs

Confidential precompiles have access to the following Confidential APIs during execution. This is subject to change!

```go
type ConfidentialStorageBackend interface {
	InitializeBid(bid suave.Bid) error
	Store(bid suave.Bid, caller common.Address, key string, value []byte) (suave.Bid, error)
	Retrieve(bid suave.Bid, caller common.Address, key string) ([]byte, error)
	FetchBidById(suave.BidId) (suave.Bid, error)
	FetchBidsByProtocolAndBlock(blockNumber uint64, namespace string) []suave.Bid
	Stop() error
}
```

## Data Management

Data Management is at the heart of the Confidential Data Store, ensuring that data is stored, retrieved, and maintained securely and efficiently. Effective data management is essential for optimizing performance, maintaining data integrity, and ensuring seamless user experiences.

### Initialization & Access Control

The Confidential Data Store ensures that only legitimate entities can store data, and only authorized parties can retrieve it.

- *Bid Registration*: When initializing, it's vital to ensure that only valid bids are registered in the system. A bid must satisfy specific criteria, such as proper formatting, authentication, and non-duplication.

- *Access Permissions*: A robust access control mechanism restricts data access. Only contracts with appropriate permissions (peekers) can access stored data. This mechanism can be role-based, attribute-based, or context-based, depending on the application's needs.

### Security and Confidentiality

Ensuring the confidentiality of the stored data is paramount. Current implementations, while solid, are still works in progress. Future iterations will delve deeper into encryption methodologies, access controls, and auditing mechanisms to fortify data privacy further.









