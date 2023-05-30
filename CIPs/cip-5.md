---
cip: 5
title: Streamtypes
author: Michael Sena (@michaelsena), Joel Thorstensson (@oed), Janko Simonovic
discussions-to: https://github.com/ceramicnetwork/CIP/issues/37
status: Final
category: RFC
created: 2020-06-23
edited: 2020-09-24
requires: 59
---

## Simple Summary

How to specify a streamtype.

## Abstract

All streams stored on the Ceramic network must conform to a streamtype. Streamtypes define rules for the content of events and state transitions, enabling consumers to validate updates to a given stream.


## Motivation

With the guidance of this specification, anyone can specify their own streamtype if the existing ones provided by the community do not suit their specific use case.


## Specification

Each Ceramic stream consists of a series of linked events, which essentially form "micro ledger". A streamtype defines the rules for what data may be contained in these _events_ as well as the state transition function that determines how to process the payload of new events.

#### StreamID code: `0xXX` (as per CIP-59)

### Event Formats
This section describes how the data contained in events has to be structured. The event schemas in the [Ceramic specs](https://developers.ceramic.network/protocol/streams/event-log/) define some basic properties that have to be specified by all events of that particular type. When a streamtype is defined in a CIP, it needs to describe if the events include any additional fields from the events defined in the specs, and in particular define what the content of the `data` field looks like.


### State Transitions
The state transition function is the heart of a streamtype definition. It specifies how the streamtype processes the records to update the state of the document. Below the `State` object is defined along with other important considerations. 

> Note that we use a typescript implementation as an example here to more easily describe how the state transitions should work. Implementations in other languages will look differently, but the core logic should be the same.

In general we refer to the function that computes the state transition as a **_StreamtypeHandler_**. When creating a CIP for a streamtype the state transitions below should be described in detail.

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

interface Metadata {
  owners: Array<string>;
  schema?: String;
  family?: String;
  tags?: Array<string>;
  isUnique?: boolean;

  [index: string]: any;
}

interface Next {
  content?: any;
  metadata?: Metadata;
}

interface State {
  type: string;
  metadata: Metadata;
  content: any;
  next?: Next;
  signature: SignatureStatus;
  anchorStatus: AnchorStatus;
  anchorScheduledFor?: number; // only present when anchor status is pending
  anchorProof?: AnchorProof; // the anchor proof of the latest anchor, only present when anchor status is anchored
  log: Array<CID>;
}
```

The `SignatureStatus` has three different values. 
* `GENESIS` means that the InitEvent is the only event that has been applied, and that the InitEvent does not contain a signature. 
* `PARTIAL` means that only a set out of multiple required signed (events) have been applied. 
* `SIGNED` means that the stream is fully signed, only at this point could a TimeEvent be applied.

The `AnchorStatus` describes if the latest update to the stream has been anchored, or what state the anchor request is in.

Below the general functioning of the StreamtypeHandler is described for each record type. When creating a CIP for a streamtype each of these function need to be described in detail.

#### Applying an InitEvent
When the StreamtypeHandler receives a InitEvent it will also get an empty `State` and the CID of the event. Given the data in the event, the handler should return a `State` object which contains all the non optional properties. The CID should be added to the `log` property.

#### Applying a DataEvent
When the StreamtypeHandler receives a DataEvent it will get the current `State`, the event, and the CID of the event. Based on the content in the DataEvent the `nextContent` (and optionally the `nextMetadata`) properties should be populated. The `anchorStatus` and `signature` properties should be updated as well. The CID should be added to the `log` property.

> Note that this function may throw an error if the record is invalid for some reason.

#### Applying an TimeEvent
When the StreamtypeHandler receives a TimeEvent it will get the current `State`, the event, and the CID of the event. Based on the information in the anchor as well as the data in the `next*` properties the `content` and `metadata` properties should be updated. The `anchorProof` should be set to the data contained in the AnchorProof object, and the `anchorStatus` property should be updated. The CID should be added to the `log` property.


## Implementations
Each streamtype CIP should include a reference to an implementation of the streamtype. 

### Common Streamtypes
A list of commonly used streamtypes can be found in the [Streamtypes Table](../tables/streamtypes.csv). Since anyone can create and use their own streamtype on Ceramic without creating a CIP, this list is non-exhaustive. However it provides a helpful starting point when deciding which streamtype to use when publishing content to Ceramic.

If you would like to add a new streamtype to the Streamtypes Table:
1. Create an issue in the CIP repo for your streamtype and gather community feedback.
2. Submit a Pull Request adding your streamtype to the CIP repo.
3. Get your CIP finalized.
4. Submit a Pull Request to the streamtype Table adding your streamtype.


## Security Considerations
Each streamtype CIP should describe if there are any specific security considerations for the specified state transition logic. In general the Ceramic protocol should take care of most security issues, but there might be edge cases worth considering.


## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
