<!-- TOC -->

- [Overview](#overview)
- [Architecture Diagram](#architecture-diagram)
- [Core Components](#core-components)
    - [ConfidentialStore](#confidentialstore)
    - [SUAVE Mempool](#suave-mempool)
    - [Interface Definitions](#interface-definitions)
- [Data Management](#data-management)
    - [Initialization & Access Control](#initialization--access-control)
    - [Store & Retrieve Processes](#store--retrieve-processes)
- [Security and Confidentiality](#security-and-confidentiality)

<!-- /TOC -->

---

</div>

## Overview

This document provides an overview of the Confidential Data Store, an essential component of the SUAVE protocol. The Confidential Store serves as a secure and privacy-focused storage system, by exposing a key-value store for safeguarding confidential bid-related data. Only those with appropriate permissions (peekers) can access the stored data, thus ensuring privacy and control.

The Confidential Store is an integral part of the SUAVE chain, designed to facilitate secure and privacy-preserving transactions and smart contract interactions. It functions as a key-value store where users can safely store and retrieve confidential data related to their bids. The Confidential Store restricts access (both reading and writing) only to the allowed peekers of each bid, allowing developers to define the entire data model of their application!

The current, and certainly not final, implementation of the Confidential Store is managed by the ConfidentialStoreEngine. The engine consists of a storage backend, which holds the raw data, and a transport topic, which relays synchronization messages between nodes.
We provide two storage backends to the confidential store engine: the LocalConfidentialStore, storing data in memory in a simple dictionary, and RedisStoreBackend, storing data in redis. To enable redis as the storage backed, pass redis endpoint via --suave.confidential.redis-store-endpoint.
For synchronization of confidential stores via transport we provide an implementation using a shared Redis PubSub in RedisPubSubTransport, as well as a crude synchronization protocol. To enable redis transport, pass redis endpoint via --suave.confidential.redis-transport-endpoint. Note that Redis transport only synchronizes current state, there is no initial synchronization - a newly connected node will not have access to old data.
Redis as either storage backend or transport is temporary and will be removed once we have a well-tested p2p solution.


## Architecture Diagram

TODO: high-level diagram

![Confidential Data Store Diagram](/assets/rigil_confidential_data_store.svg)

## Core Components

### ConfidentialStore

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

### SUAVE Mempool

`MempoolOnConfidentialStore` operates directly with the Confidential Store, ensuring private handling of bid transactions. It serves as an interim storage for transactions that await blockchain inclusion.

```go
type MempoolOnConfidentialStore struct {
	cs suave.ConfidentialStoreBackend
}
```

### Interface Definitions

TODO: Elaborate on other necessary interfaces

## Data Management

### Initialization & Access Control

To maintain data integrity, the initialization process ensures that only valid bids are registered. Data access is strictly regulated, allowing only authorized peekers to interact with the data.

### Store & Retrieve Processes

Confidential data can be safely stored using the `Store` method and later retrieved with the `Retrieve` method. Both operations are tightly controlled, ensuring privacy.

## Security and Confidentiality

TODO: Document security concerns (confidential data isn't actually confidential right now)




