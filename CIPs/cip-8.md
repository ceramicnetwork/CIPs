---
cip: 8
title: Tile Document
author: Joel Thorstensson (@oed)
discussions-to: https://github.com/ceramicnetwork/CIP/issues/56
status: Final
category: RFC
created: 2020-07-22
edited: 2020-07-22
requires: 5
---

## Simple Summary

This CIP describes the Tile document which is a very general stream type that has JSON as its content and is controlled by a [DID](https://w3c.github.io/did-core/).


## Abstract

The Tile document allows DIDs to create simple JSON documents that use [json-patch](http://jsonpatch.com/) for updating the content of the document. Each Tile is owned by a DID, which needs to sign the DataEvent. The DataEvents are encoded using [dag-jose](https://github.com/ceramicnetwork/js-dag-jose) which allows the signatures to be natively encoded directly in [IPLD](ipld.io). Tiles also support adding schemas to the document which means that the json data within the content of the document can be enforced to have a specific format.


## Motivation

The Tile document is a very generic stream type that gives developers flexibility in how they structure the data that they put into Ceramic. While other stream types might have certain restrictions around what type of data is allowed to be stored and what the structure of that data should be, tiles allows any type of JSON data to be stored and updated. Tiles can be owned and controlled by any DID that implements the [DID Provider](https://github.com/ceramicnetwork/CIP/issues/4) specification. This provides the entire Decentralized Identity ecosystem with an interoperable verifiable document network.


## Specification

Below the event formats and state transitions of the Tile document are specified. Together they should include all information needed to implement this stream type in Ceramic.

#### StreamID code: `0x00`

### Event formats

The **InitEvent** has the following required properties:

* `data` - the initial JSON content of the document
* `owners` - the owners array in the header needs to contain only DID strings

In addition to the required properties above the InitEvent may inlcude `schema`, `tags`, and `unique`. The `unique` property should include randomness as a string to ensure that there is no other document that is exactly the same.

Finally the InitEvent is signed and encoded using [dag-jose](https://github.com/ceramicnetwork/js-dag-jose).

The InitEvent can also be stored without a signature using *dag-cbor*. However in this case the `data` property MUST be set to `null`.

**DataEvents** of a Tile document are encoded using [dag-jose](https://github.com/ceramicnetwork/js-dag-jose). Other than that they conform to the standard event format. The content of the `data` property should be a [json-patch](http://jsonpatch.com/) object which contains the difference from the previous document content state.

### State transitions

Below the state transition logic for the three different supported events is described.

#### Applying the InitEvent

When the InitEvent is applied a `StreamState` object is created that looks like this:

```js
{
  doctype: 'tile',
  metadata: {
    owners: initEvent.owners,
    schema: initEvent.schema,
    family: initEvent.family,
    tags: initEvent.tags
  },
  content: initEvent.data,
  signature: SignatureStatus.SIGNED,
  anchorStatus: AnchorStatus.NOT_REQUESTED,
  log: [new CID(initEvent)]
}
```

In addition to this, the following should be verified:

* If the event uses the *dag-cbor* format (not signed), `initEvent.data` MUST equal `null`
* `owners` is an array of DID strings
* If `schema` is defined it should be Ceramic StreamId string
* If `tags` is defined it should be an array of strings
* `content` should be JSON
* If `schema` is defined the data in the `content` should be valid according to the schema
* The **dag-jose** signature should be valid and signed by the DID(s) in the `owners` array


#### Applying a DataEvent

When a DataEvent is applied the `StreamState` object should be modified in the following way:

```js
state.next = {
  content: applyJSONPatch(state.content, dataEvent.data),
  metadata: {
    schema: dataEvent.schema,
    family: initEvent.family,
    tags: dataEvent.tags
  }
}
state.signature = SignatureStatus.SIGNED
state.anchorStatus = AnchorStatus.NOT_REQUESTED
state.log.push(new CID(dataEvent))
```

Where `applyJSONPatch` is a function that applies a json-patch and returns a new json object. In addition the following should be validated:

* If `schema` is defined it should be Ceramic StreamId string
* If `tags` is defined it should be an array of strings
* If `schema` is defined the data in the `content` should be valid according to the schema
* The **dag-jose** signature should be valid and signed by the DID(s) in the `owners` array in the `metadata` property (not the `next.metadata` property of the state)
* The **dag-jose** signature should include a `kid` property in it's header which specifies the `keyFragment` as well as the `version-id` of the public key that was used to sign the event

#### Applying an TimeEvent

When an TimeEvent is applied the `StreamState` object should be modified in the following way:

```js
state.anchorStatus = AnchorStatus.ANCHORED
state.anchorProof = timeEvent.proof
state.content = state.next.content
state.metadata = Object.assign(state.metadata, state.next.metadata)
delete state.next
state.log.push(new CID(timeEvent))
```

No additional validation needs to happen at this point in the state transition function.


## Rationale

The Tile specification provides a fairly straightforward way of implementing a general JSON document in Ceramic. By allowing a schema to be defined, the content of the document can be constrained. This gives developers both flexibility and control. There should not really be anything in this specification that is controversial.


## Implementation

A reference implementation of the Tile document is provided in [js-ceramic](https://github.com/ceramicnetwork/js-ceramic) (see the `stream-tile` package).


## Security Considerations

Because of the double spend problem, a change of `owners` is a trusted operation. This means that the new owner has to trust that the previous owner is not withholding the data of another owner change event that was anchored earlier. If the first owner publishes this event at a later point the document would change owner away from the new owner. In future versions of the Ceramic protocol it will be possible to specify an NFT as an owner of a document which means that the ownership transfer can happen on-chain and thus sidestep this problem.


## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
