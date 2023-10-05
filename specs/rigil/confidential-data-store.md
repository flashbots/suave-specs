# Confidential Data Store

<div class="toc">

**Table of Contents**

- [Introduction](#introduction)
- [Architecture Diagram](#architecture-diagram)
- [Core Components](#core-components)
    - [ConfidentialStore](#confidentialstore)
    - [SUAVE Mempool](#suave-mempool)
    - [Interface Definitions](#interface-definitions)
- [Data Management](#data-management)
    - [Initialization & Access Control](#initialization--access-control)
    - [Store & Retrieve Processes](#store--retrieve-processes)
- [Security and Confidentiality](#security-and-confidentiality)

---

</div>

## Introduction

This document provides an overview of the Confidential Data Store, an essential component of the SUAVE protocol. The Confidential Store serves as a secure and privacy-focused storage system, by exposing a key-value store for safeguarding confidential bid-related data. Only those with appropriate permissions (peekers) can access the stored data, thus ensuring privacy and control.

## Architecture Diagram

TODO: high-level diagram

## Core Components

### ConfidentialStore

From our [suave-geth](https://github.com/flashbots/suave-geth/tree/main)  reference implementation, `LocalConfidentialStore` serves as the foundation for the Confidential Store. Implemented as a thread-safe struct, it provides secured access to bid data.

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
