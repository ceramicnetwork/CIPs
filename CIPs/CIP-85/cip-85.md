---
cip: 85
title: AppendCollection
author: Paul Le Cam (@PaulLeCam)
discussions-to: <URL of the Github issue for this CIP (if this is a PR)>
status: Draft
category: Standards
type: RFC
created: 2021-02-16
# edited:
requires: [CIP-82](https://github.com/ceramicnetwork/CIP/issues/82)
# replaces: <EIP number(s)>
---

## Simple Summary

Provide schemas and associated tools to help interact with large lists in Ceramic.

## Abstract

This CIP presents a way to store a possibly large list of items split into multiple documents based on two complementary schemas: `AppendCollection` and `CollectionSlice`.
This allows to create a virtually infinite list of items such as a feed by leveraging multiple Ceramic documents rather than a single one.

## Motivation

Storing a list of items in a single Ceramic document can make the document grow large in size, possibly exceeding Ceramic's internal limits.
Providing a standard way to represent large lists such as feeds would be useful to provide reference schemas to solve this issue and associated tools to simplify interactions.

## Specification

### Concepts

#### Doc references

The schemas use document references defined in [CIP-82](https://github.com/ceramicnetwork/CIP/issues/82) to identify relations.

#### Cursors

A cursor is a unique identifier for an item in the collection, made of the tuple (CollectionSlice DocID, item index).
A collection slice should contain a maximum of 256 items, so the index of an item in the slice can be represented in a single byte.

### Schemas

Two schemas are needed to represent a collection: the `AppendCollection` schema represents an entry point to the collection, and the `CollectionSlice` schema represents a single slice of the full list of items.

A Collection "instance" would therefore be made of 1 `AppendCollection` document and any number of `CollectionSlice` documents with cross-references, as presented in the graphic below:

![Collection relations graphic](./assets/collection-graphic.png)

#### AppendCollection schema

The `AppendCollection` schema must be an `object` with a [`$id` field](https://json-schema.org/draft/2019-09/json-schema-core.html#rfc.section.8.2.2) value of `ceramic://appendCollection` and the following properties:

- `sliceSize`: the maximum number of items a single slice should contain
- `slicesCount`: the total number of slices the collection contains
- `first`: DocID of the first slice
- `last`: DocID of the last slice

```js
{
  $schema: 'http://json-schema.org/draft-07/schema#',
  title: 'MyCollection',
  type: 'object',
  $id: 'ceramic://appendCollection',
  properties: {
    sliceSize: { type: 'integer', minimum: 10, maximum: 256 },
    slicesCount: { type: 'integer', minimum: 1 },
    first: { type: 'string', $id: 'ceramic://doc', $ceramicSchema: '<slice schema ID>', maxLength: 150 },
    last: { type: 'string', $id: 'ceramic://doc', $ceramicSchema: '<slice schema ID>', maxLength: 150 },
  },
  required: ['sliceSize', 'slicesCount', 'first', 'last']
}
```

#### CollectionSlice schema

The `CollectionSlice` schema must be an `object` with a [`$id` field](https://json-schema.org/draft/2019-09/json-schema-core.html#rfc.section.8.2.2) value of `ceramic://collectionSlice` and the following properties:

- `contents`: array with a `maxItems` value matching the `AppendColelction` `sliceSize` property and defining the items schemas, that must include `{ type: 'null' }` in order to support removals
- `next`: DocID of the next slice (if the slice is not the last one)
- `prev`: DocID of the previous slice (if the slice is not the first one)

```js
{
  $schema: 'http://json-schema.org/draft-07/schema#',
  title: 'MyCollectionSLice',
  type: 'object',
  $id: 'ceramic://collectionSlice',
  properties: {
    contents: {
      type: 'array',
      maxItems: 256,
      minItems: 1,
      items: {
        oneOf: [{ type: 'object', ... }, { type: 'null' }],
      },
    },
    next: { type: 'string', $id: 'ceramic://doc', $ceramicSchema: '<slice schema ID>', maxLength: 150 },
    prev: { type: 'string', $id: 'ceramic://doc', $ceramicSchema: '<slice schema ID>', maxLength: 150 },
  },
  required: ['contents'],
}
```

### Algorithms

#### First insertion

> This flow assumes a prerequisite check that the `AppendCollection` document has not been created yet.

1. Create the `CollectionSlice` document with the `contents` array containing the item to insert.
1. Create the `AppendCollection` document with a `slicesCount` of `1`, and the `first` and `last` values containing the DocID of the created `CollectionSlice`.

#### Other insertions

1. Load the `AppendCollection` document.
1. Load the `CollectionSlice` document from the `last` field of the `AppendCollection`.
1. Check the length of the `contents` array of the `CollectionSlice`:

- If it is lower than the `sliceSize` value of the `AppendCollection`, add the item to the `contents` array.
- If it is equal to the `sliceSize` value:
  1. Create a new `CollectionSlice` document with the `contents` array containing the item to insert and the `prev` value containing the DocID of the previous slice.
  1. Update the previous slice to set the `next` value to the DocID of the newly created slice.
  1. Update the `AppendCollection` document with the incremented `slicesCount` and new `last` value.

#### Single item loading

Loading a single item is based on the item cursor (slice DocID + item index in slice):

1. Load the `CollectionSlice` document from its DocID.
2. Access the item at the given index in the `contents` array.

#### Multiple item loading

Loading multiple items can be done in order (from the `first` slice) or reverse order (from the `last` slice), with an optional `cursor` to start from, as described below:

- `first: N`

  1. Load the `AppendCollection` document.
  1. Load the `CollectionSlice` document from the `first` field of the `AppendCollection`.
  1. Iterate through `contents` filtering out `null` values until `N` items are collected.
  1. If `N` items are not collected and `next` is not set then return, otherwise load the `next` slice and continue from previous step.

- `first: N, after: cursor(slice ID, offset)`

  1. Load the `CollectionSlice` document from the `slice ID`.
  1. Skip the first `contents` items according to the `offset`.
  1. Iterate through `contents` filtering out `null` values until `N` items are collected.
  1. If `N` items are not collected and `next` is not set then return, otherwise load the `next` slice and continue from previous step.

- `last: N`

  1. Load the `AppendCollection` document.
  1. Load the `CollectionSlice` document from the `last` field of the `AppendCollection`.
  1. Iterate through `contents` in reverse order filtering out `null` values until `N` items are collected.
  1. If `N` items are not collected and `prev` is not set then return, otherwise load the `prev` slice and continue from previous step.

- `last: N, before: cursor(slice ID, offset)`

  1. Load the `CollectionSlice` document from the `slice ID`.
  1. Skip the first `contents` items according to the `offset`.
  1. Iterate through `contents` in reverse order filtering out `null` values until `N` items are collected.
  1. If `N` items are not collected and `prev` is not set then return, otherwise load the `prev` slice and continue from previous step.

#### Single item removal

Removing an item consists in replacing the item value by `null` in the `contents` array identified by the given `cursor(slice ID, offset)`:

1. Load the `CollectionSlice` document from the `slice ID`.
1. Access the item from the `contents` array at the given `offset`.
1. If the item exists, replace it by `null`.

## Rationale

This CIP implements a well-known data structure (a [doubly linked list](https://en.wikipedia.org/wiki/Doubly_linked_list)) in a similar way to the [ActivityStreams collections](https://www.w3.org/TR/activitystreams-core/#collections).

The implementation and algorithms are meant to support [GraphQL's connections](https://relay.dev/graphql/connections.htm) so this interface can easily be used on top of `AppendCollection`.

## Backwards Compatibility

None, as these are new schemas.

## Implementation

None yet.

## Security Considerations

The spec defines clear expectations about the number of items a slice can contain, and how cursors should be stable, but as with any other Ceramic document, it is up to the schema validation and correct spec implementations to ensure the correct behavior is applied.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
