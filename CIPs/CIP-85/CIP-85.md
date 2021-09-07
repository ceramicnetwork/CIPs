---
cip: 85
title: AppendCollection schemas
author: Paul Le Cam (@PaulLeCam)
discussions-to: https://github.com/ceramicnetwork/CIP/issues/90
status: Draft
category: Standards
type: RFC
created: 2021-02-16
edited: 2021-07-02
requires: [CIP-82](https://github.com/ceramicnetwork/CIP/issues/82), [CIP-88](https://github.com/ceramicnetwork/CIP/issues/88)
---

## Simple Summary

Provide schemas and associated tools to help interact with large lists in Ceramic.

## Abstract

This CIP presents a way to store a possibly large list of items split into multiple streams based on two complementary schemas: `AppendCollection` and `CollectionSlice`.
This allows to create a virtually infinite list of items such as a feed by leveraging multiple Ceramic streams rather than a single one.

## Motivation

Storing a list of items in a single Ceramic Tile stream can make the stream grow large in size, possibly exceeding Ceramic's internal limits for this stream stype.
Providing a standard way to represent large lists such as feeds would be useful to provide reference schemas to solve this issue and associated tools to simplify interactions.

## Specification

### Concepts

#### Stream references

The schemas use stream references defined in [CIP-82](https://github.com/ceramicnetwork/CIP/issues/82) to identify relations.

#### Cursors

A cursor is a unique identifier for an item in the collection, made of the tuple (CollectionSlice StreamID, item index).
A collection slice should contain a maximum of 256 items, so the index of an item in the slice can be represented in a single byte.

#### Slice tag

A slice tag is a string identifying a unique slice part of a collection based on the index of the slice, as: `<collection StreamID string>:<slice index>`.

### Schemas

Two schemas are needed to represent a collection: the `AppendCollection` schema represents an entry point to the collection, and the `CollectionSlice` schema represents a single slice of the full list of items.

A Collection "instance" would therefore be made of 1 `AppendCollection` stream and any number of `CollectionSlice` streams with cross-references, as presented in the graphic below:

![Collection relations graphic](./assets/collection-graphic.png)

#### AppendCollection schema

The `AppendCollection` schema must be an `object` with a `$comment` field from [CIP-88](../CIP-88/CIP-88.md) using the `cip88:appendCollection` prefix and pointing to the slice's `schema`, along with the following properties:

- `sliceMaxItems`: the maximum number of items a single slice should contain
- `slicesCount`: the total number of slices the collection contains

```js
{
  $schema: 'http://json-schema.org/draft-07/schema#',
  $comment: 'cip88:appendCollection:<slice schema ID>',
  title: 'MyCollection',
  type: 'object',
  properties: {
    sliceMaxItems: { type: 'integer', minimum: 10, maximum: 256 },
    slicesCount: { type: 'integer', minimum: 1 },
  },
  required: ['sliceMaxItems', 'slicesCount']
}
```

#### CollectionSlice schema

The `CollectionSlice` schema must be an `object` with a `$comment` field from [CIP-88](../CIP-88/CIP-88.md) having the value `cip88:collectionSlice`, along with the following properties:

- `collection`: the StreamID of the collection the slice is part of
- `sliceIndex`: index of the slice in the collection, between `0` and the collection's `slicesCount` minus `1`
- `contents`: array with a `maxItems` value matching the `AppendCollection` `sliceMaxItems` property and defining the items schemas, that must include `{ type: 'null' }` in order to support removals

```js
{
  $schema: 'http://json-schema.org/draft-07/schema#',
  $comment: 'cip88:collectionSlice',
  title: 'MyCollectionSlice',
  type: 'object',
  properties: {
    collection: { type: 'string', maxLength: 150 },
    sliceIndex: { type: 'integer', minimum: 0 },
    contents: {
      type: 'array',
      maxItems: 256,
      minItems: 0,
      items: {
        oneOf: [{ type: 'object', ... }, { type: 'null' }],
      },
    },
  },
  required: ['collection', 'sliceIndex', 'contents'],
}
```

### Algorithms

#### Deterministic slice access

Accessing slices in the collection relies on Ceramic's ability to load streams deterministically based on their metadata by using `{ deterministic: true, tags: ['<slice tag>'] }`, where `slice tag` is a string of `<collection StreamID string>:<slice index>`.

#### First insertion

> This flow assumes a prerequisite check that the `AppendCollection` stream has not been created yet.

1. Create the `AppendCollection` stream with a `slicesCount` of `1`.
1. Create a deterministic `CollectionSlice` stream with a tag using the `index` value `0`.
1. Update the created `CollectionSlice` stream with the `collection` string, `sliceIndex` number and `contents` array containing the item to insert.

#### Other insertions

1. Load the `AppendCollection` stream.
1. Load the most recent `CollectionSlice` stream based on its deterministic content, using the `slicesCount` from the collection.
1. Check the length of the `contents` array of the `CollectionSlice`:

- If it is lower than the `sliceMaxItems` value of the `AppendCollection`, add the item to the `contents` array.
- If it is equal to the `sliceMaxItems` value:
  1. Create a new deterministic `CollectionSlice` stream with `index` value of the previous slice plus `1`.
  1. Update the created `CollectionSlice` stream with the `collection` string, `sliceIndex` number and `contents` array containing the item to insert.
  1. Update the `AppendCollection` stream with the incremented `slicesCount`.

#### Single item loading

Loading a single item is based on the item cursor (slice StreamID + item index in slice):

1. Load the `CollectionSlice` stream from its StreamID.
1. Access the item at the given index in the `contents` array.

#### Multiple item loading

Loading multiple items can be done in order (from the `first` slice) or reverse order (from the `last` slice), with an optional `cursor` to start from, as described below:

- `first: N`

  1. Load the `CollectionSlice` stream based on its determistic content with a `sliceIndex` of `0`.
  1. Iterate through `contents` filtering out `null` values until `N` items are collected.
  1. If `N` items are not collected, load the next slice deterministically and continue from previous step.

- `first: N, after: cursor(slice ID, offset)`

  1. Load the `CollectionSlice` stream from the `slice ID`.
  1. Skip the first `contents` items according to the `offset`.
  1. Iterate through `contents` filtering out `null` values until `N` items are collected.
  1. If `N` items are not collected, load the next slice deterministically and continue from previous step.

- `last: N`

  1. Load the `AppendCollection` stream.
  1. Load the `CollectionSlice` stream based on its determistic content, using the `slicesCount` from the collection.
  1. Iterate through `contents` in reverse order filtering out `null` values until `N` items are collected.
  1. If `N` items are not collected and the first slice is not reached, load the previous slice deterministically and continue from previous step

- `last: N, before: cursor(slice ID, offset)`

  1. Load the `CollectionSlice` stream from the `slice ID`.
  1. Skip the first `contents` items according to the `offset`.
  1. Iterate through `contents` in reverse order filtering out `null` values until `N` items are collected.
  1. If `N` items are not collected and the first slice is not reached, load the previous slice deterministically and continue from previous step

#### Single item removal

Removing an item consists in replacing the item value by `null` in the `contents` array identified by the given `cursor(slice ID, offset)`:

1. Load the `CollectionSlice` stream from the `slice ID`.
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

The spec defines clear expectations about the number of items a slice can contain, and how cursors should be stable, but as with any other Ceramic stream, it is up to the schema validation and correct spec implementations to ensure the correct behavior is applied.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
