---
cip: 71
title: MultiQueries
author: Joel Thorstensson (@oed)
discussions-to: https://github.com/ceramicnetwork/CIP/issues/72
status: Draft
category: Standards
type: Core
created: 2020-11-06
---

## Simple Summary

A specification for how to query multiple documents at once.


## Abstract

This CIP introduces a new way of querying documents in Ceramic. The *CeramicApi* gets an update to support this. The http daemon get's a new endpoint which supports the new query format, and the core gossip protocol get's a revamp with new features. The main feature that these changes allow for is to query multiple documents by specifying a a number of paths. If a path includes a Ceramic URL both the main document as well as the linked document will be synced.


## Motivation

Ceramic is a network of linked documents and currently you can only sync one document at a time. This means that if you have a data structure where many documents are linked and you want to sync all of them you would need to make multiple requests to the network. By introducing an api and a gossip specification that allows queries for multiple documents at once this problem is mitigated.


## Specification

There are three main changes outlined below. The most important one is **Pubsub queries** which describes the protocol change required for this entire CIP. 

### CeramicApi

The *CeramicApi* is updated to be able to sync mutliple documents (preform a Multiquery) that may be linked from the initial document thoughout the `paths`.

```typescript
function loadLinkedDocuments(id: DocID, paths: string[]): Promise<Record<DocID, Doctype>>
```

For example if *doc A* links to *doc B* on the `coolLink` property in its content `ceramic.loadDocuments(<DocID(doc A)>, ['/coolLink'])` would return:

```typescript
{
  <DocID(doc A)>: <content(doc A)>,
  <DocID(doc B)>: <content(doc B)>
}
```

If some of the given `paths` are not present in the document the method should just return the documents that where actually present. This will allow developers to optimistically query for multiple linked documents and only get the already created docs.

### MultiQuery on http-daemon

The http daemon should be extended with a `states` method. This method allows for multiple Multiqueries to be done at once. The reason to add the ability to make multiple queries at once is that the http-client often wants to maintain the state of multiple documents at once in the background, so this method can be used to reduce the number of requests made.

**Endpoint:** `POST /api/v0/states`

**Body:**

The body should conform to the `StatesQuery` interface.

```typescript
interface MultiQuery {
  docid: string
  paths?: Array<string>
}

interface StatesQuery {
  queries: Array<MultiQuery>
}
```

**Response:**

The body should conform to the `Response` interface.

```typescript
interface ResponseData {
  [docid: string]: DocState
}

interface Response {
  status: 'success'
  data: ResponseData
}
```

### Pubsub Queries

Replace the current message types with the ones defined below. 

```typescript
interface Update {
  typ: 0
  doc: string
  tip: string
  anchorService?: string
}

interface Query {
  typ: 1
  id: string
  doc: string
  paths?: Array<string>
}

interface TipMap {
  [docid: string]: string
}

interface Response {
  typ: 2
  id: string
  tips: TipMap
}
```

#### Update

The new update message merges the previous `UPDATE` and `ANCHOR_META` message types by adding an optional `anchorService` property to the message. This property will be set if an anchor has been requested but not yet returned. When an anchor does get returned a new update message will be sent with the CID of the *anchorRecord* as the `tip`, and `anchorService` not set.

**Properties:**

* `typ` - the message type, always `0`
* `doc` - the DocID that is being updated
* `tip` - the CID of the new Tip of the document
* `anchorService` - the url of the anchor service that was used to request an anchor

#### Query

The query message is a replacement of the old `REQUEST` message type. The most important aspect is the new format for the `id` and the new property `paths`.

**Properties:**

* `typ` - the message type, always `1`
* `id` - unique identifier of the query, multihash of the query object without the `id` property canonicalized using dag-cbor
* `doc` - the DocID that is being queried
* `paths` - the paths in the contents of the documents to explore

#### Response

The response message is a replacement of the previous `RESPONSE` message type. Note the use of the new format for the `id` property, and the map in the `tips` property as a way of adding multiple DocIDs and Tips to the same response.

There are two main reasons to include other DocIDs and tips than the main document requested:

* The given `paths` traverses other documents
* Any of the documents has a `schema` defined

**Properties:**

* `typ` - the message type, always `2`
* `id` - should be the same as the id of the query that is being responded to
* `tips` -  a map from DocIDs to tips

The `tips` property must contain at least one property where the key is the DocID of the requested document and the value is the  CID of the latest tip of the given document.


## Rationale

This CIP primarily defines new message types for the pubsub gossip protocol in ceramic. These new message type allows a node making a request to include an array of *paths* which describe paths in the content of the given document that is being queried. These paths may contain links to other documents. When another node that has the given document (and additional documents) it can respond with one message containing the latest tip of all of these documents.

The updates to the http-daemon and the *CeramicApi* are simply interface updates that sufraces this new feature.

## Future work

Currently the multiqueries are limited to only standard DocIDs and content of Tile doctypes. This means that you need to know the DocID of the first document and there needs to be a clear path to the other doucments. One example where this is not the case is when you want to resolve a `caip10-link`, find the DID and then find the IDX document from that DID. For these types of queries there needs to be some additional functionality to the query system. 


## Backwards Compatibility

The pubsub gossip message update is a breaking change to the protocol.


## Implementation

* [js-ceramic](https://github.com/ceramicnetwork/js-ceramic) - pending


## Security Considerations

When a node gets a query response with multiple documents it needs to validate that these documents are actually linked to by the first document that was referenced. Another consideration is that that if you have the following documents: *doc A*, *doc B*, and *doc C*, where *B* and *C* are linked from *A* and sync all of these documents in one go from *node 1*, it is possible that other nodes have more recent updates to these documents, or that *node 1* responds with an incorrect *tip* for document *B* or *C*. As a counter measure to this it is a good idea to also make requests for these documents individually. However, the multiquery approach still makes sense because in the optimistic case all document data can be loaded in one go.


## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
