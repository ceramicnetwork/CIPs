---
cip: 5
title: Streamtypes
author: Michael Sena (@michaelsena), Joel Thorstensson (@oed), Janko Simonovic
discussions-to: https://github.com/ceramicnetwork/CIP/issues/37
status: Review
category: RFC
created: 2020-06-23
edited: 2020-09-24
requires: 59
---

## Simple Summary

How to specify a doctype.

## Abstract

All documents stored on the Ceramic network must conform to a doctype. Doctypes define rules for document content and state transitions, enabling Ceramic nodes to validate updates to a given document.


## Motivation

With the guidance of this specification, anyone can specify their own doctype if the existing ones provided by the community do not suit their specific use case.


## Specification

Each Ceramic document consists of a series of linked records, which essentially form a "doc-chain." A doctype defines the rules for what data may be contained in these _records_ as well as the state transition function that determines how to process new records.

#### DocID code: `0xXX` (as per CIP-59)

### Record Formats
This section describes how the data contained in various record types has to be structured. The record schemas in the [Ceramic specs](https://github.com/ceramicnetwork/specs#document-records) define some basic properties that have to be specified by all records of that particular type. When a Doctype is defined in a CIP it needs to describe if the records differ in any way to the records defined in the specs, and in particular define what the content of the `data` property looks like.


### State Transitions
The state transition function is the heart of a doctype definition. It specifies how the doctype processes the records to update the state of the document. Below the `DocState` object is defined along with other important considerations. 

> Note that we use a typescript implementation as an example here to more easily describe how the state transitions should work. Implementations in other languages will look differently, but the core logic should be the same.

In general we refer to the function that computes the state transition as a **_DoctypeHandler_**. When creating a CIP for a doctype the state transitions below should be described in detail.

```ts
enum SignatureStatus {
  GENESIS,
  PARTIAL,
  SIGNED
}

enum AnchorStatus {
  NOT_REQUESTED,
  PENDING,
  PROCESSING,
  ANCHORED,
  FAILED
}

interface DocMetadata {
  owners: Array<string>;
  schema?: String;
  family?: String;
  tags?: Array<string>;
  isUnique?: boolean;

  [index: string]: any;
}

interface DocNext {
  content?: any;
  metadata?: DocMetadata;
}

interface DocState {
  doctype: string;
  metadata: DocMetadata;
  content: any;
  next?: DocNext;
  signature: SignatureStatus;
  anchorStatus: AnchorStatus;
  anchorScheduledFor?: number; // only present when anchor status is pending
  anchorProof?: AnchorProof; // the anchor proof of the latest anchor, only present when anchor status is anchored
  log: Array<CID>;
}
```

The `SignatureStatus` has three different values. 
* `GENESIS` means that the genesis object is the only record that has been applied, and that the genesis record does not contain a signature. 
* `PARTIAL` means that only a set out of multiple required signature (records) have been applied. 
* `SIGNED` means that the document is fully signed, only at this point can anchor records be applied.

The `AnchorStatus` describes if the latest update to the document has been anchored, or what state the anchor request is in.

Below the general functioning of the DoctypeHandler is described for each record type. When creating a CIP for a doctype each of these function need to be described in detail.

#### Applying a Genesis Record
When the DoctypeHandler receives a GenesisRecord it will also get an empty `DocState` and the CID of the record. Given the data in the record, the handler should return a `DocState` object which contains all the non optional properties. The CID should be added to the `log` property.

#### Applying a Signed Record
When the DoctypeHandler receives a SignedRecord it will get the current `DocState`, the record, and the CID of the record. Based on the content in the signed record the `nextContent` (and optionally the `nextMetadata`) properties should be populated. The `anchorStatus` and `signature` properties should be updated as well. The CID should be added to the `log` property.

> Note that this function may throw an error if the record is invalid for some reason.

#### Applying an Anchor Record
When the DoctypeHandler receives a AnchorRecord it will get the current `DocState`, the record, and the CID of the record. Based on the information in the anchor as well as the data in the `next*` properties the `content` and `metadata` properties should be updated. The `anchorProof` should be set to the data contained in the AnchorProof object, and the `anchorStatus` property should be updated. The CID should be added to the `log` property.


## Implementations
Each doctype CIP should include a reference to an implementation of the doctype. 

### Common Doctypes
A list of commonly used doctypes can be found in the Doctypes Table (to be included in PR). Since anyone can create and use their own doctype on Ceramic without creating a CIP, this list is non-exhaustive. However it provides a helpful starting point when deciding which doctype to use when publishing content to Ceramic.

If you would like to add a new doctype to the Doctypes Table:
1. Create an issue in the CIP repo for your doctype and gather community feedback.
2. Submit a Pull Request adding your doctype to the CIP repo.
3. Get your CIP finalized.
4. Submit a Pull Request to the Doctype Table adding your doctype.


## Security Considerations
Each doctype CIP should describe if there are any specific security considerations for the specified state transition logic. In general the Ceramic protocol should take care of most security issues, but there might be edge cases worth considering.


## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
