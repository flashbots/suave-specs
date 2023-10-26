---
title: Confidential Data Store
description: This essential component of the SUAVE protocol serves as a secure and privacy-focused storage system.
---

<!-- TOC -->

- [Overview](#overview)
- [Core Functionality](#core-functionality)
- [Architecture](#architecture)
    - [Engine](#engine)
    - [Store](#store)
    - [Transport](#transport)
- [Data Management](#data-management)
    - [Initialization & Access Control](#initialization--access-control)
- [Security and Confidentiality](#security-and-confidentiality)

<!-- /TOC -->

## Overview
This document provides the technical specification for the the Confidential Data Store, a privacy-centric networked storage system, specifically tailored to enable programmable privacy in SUAPPs. 

## Core Functionality

1. **Confidential Data Input**:  
   SUAPPs define both the shape and operations performed on user data which is put into the confidential data store. Users send data to SUAPPs, and ultimately the confidential data store, through confidential compute requests.

2. **Confidential Data Transfer**:  
   Once Computors successfully carry out confidential compute requests, the results are disseminated to other Computors using a dedicated transport protocol.

3. **Data Modeling**:  
   The confidential data store adopts a straightforward key-value storage paradigm. This allows flexibility in accommodating various data types:
   - Transactions, UserOps, and EIP 712 signed messages.
   - Bundles, simulation results, and intermediate values.
   - Partial and full block.
   - And other extensible data entities.

4. **Access Control**:  
   To enable programmable privacy, only contracts bearing the right permissions, termed 'peekers', are permitted to get and put data.


## Architecture

The Confidential Data Store consists of three main components Engine, Storage, and Transport. The Engine is the main orchestrating component responisble for managing calls to Storage, and as well sending and receiving synchronization messages over the Transport. The MEVM is able to directly interact with the Confidential Store Engine through precompiles.

![Confidential Data Store Diagram](/assets/rigil_confidential_data_store.svg)

### Engine

The `ConfidentialStoreEngine` is the central component in the Confidential Data Store's architecture, responsible for coordinating data storage operations and data synchronization across the network. Integrated with the MEVM, it facilitates storage interactions via precompiled contracts. The code snippet below outlines its composition in our current [suave-geth](https://github.com/flashbots/suave-geth/tree/main)  reference implementation:

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

### Store
`ConfidentialStorageBackend` is the interface that must be implemented by a storage backend for the Confidential Storage Engine.

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

*Implementation Note: in our [suave-geth](https://github.com/flashbots/suave-geth/tree/main) reference implementation we provide two `ConfidentialStorageBackend` the LocalConfidentialStore, storing data in memory in a simple dictionary, and RedisStoreBackend, storing data in redis. Redis was chosen as a way to allow for fast iteration but is not meant to be a long term solution.*

### Transport

`StoreTransportTopic` is the interface that must be implemented by a transport engine for the Confidential Storage Engine.

```go
type StoreTransportTopic interface {
	node.Lifecycle
	Subscribe() (<-chan DAMessage, context.CancelFunc)
	Publish(DAMessage)
}
```

*Implementation Note: in our [suave-geth](https://github.com/flashbots/suave-geth/tree/main) reference implementation we provide an implementation using a shared Redis PubSub in RedisPubSubTransport, as well as a crude synchronization protocol. Note that Redis transport only synchronizes current state, there is no initial synchronization - a newly connected node will not have access to old data, for now. This synchronization protocol will be defined in coming specifications.*


## Data Management

Data Management is at the heart of the Confidential Data Store, ensuring that data is stored, retrieved, and maintained securely and efficiently.

### Initialization & Access Control

The Confidential Data Store ensures that only legitimate entities can store data, and only authorized parties can retrieve it.

- *Data Registration*: When initializing, it's vital to ensure that only valid data is registered in the system. Data must satisfy specific criteria, such as proper formatting, authentication, and non-duplication, but these requirements do not places restrictions on *what* type of data gets put in, merely that it should follow a high level format.

- *Access Permissions*: A robust access control mechanism restricts data access. Only contracts with appropriate permissions (peekers) can access stored data. This mechanism is currently context-based and the specific access control mechansim is laid out in the [MEVM](./mevm.md) 

## Security and Confidentiality

Ensuring the confidentiality of the stored data is paramount. Current implementations are still works in progress and data should not be considered secure. Future iterations will delve deeper into encryption methodologies, access controls, and auditing mechanisms to fortify data privacy further.











