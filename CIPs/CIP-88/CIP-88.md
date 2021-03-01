---
cip: 88
title: Ceramic namespace in JSON schemas
author: Paul Le Cam (@PaulLeCam)
status: Draft
category: Standards
type: RFC
created: 2021-03-01
---

## Simple Summary

Define a standard extension point for Ceramic-specific metadata in a JSON schema.

## Abstract

This CIP defines a reserved namespace for Ceramic-specific metadata in a JSON schema, along with a reference table for standard uses of this namespace.

## Motivation

As commented in https://github.com/ceramicnetwork/CIP/issues/82#issuecomment-787449788 the `$id` cannot be used to define Ceramic-specific extensions as intended in [CIP-82](https://github.com/ceramicnetwork/CIP/blob/main/CIPs/CIP-82/CIP-82.md), so creating a custom namespace for Ceramic-specific metadata should be a safer option to enable further extensions.

## Specification

### Namespace

A JSON schema property can contain a `$ceramic` field, that must be an object with a unique `type` defined in the following reference table, for example:

```js
{
  type: 'string',
  maxLength: 150,
  $ceramic: {
    type: 'tile',
    schema: '<schema docID or commitID>' ,
  },
}
```

### Reference table

| Type               | CIP                                                                                                   | Status |
| ------------------ | ----------------------------------------------------------------------------------------------------- | ------ |
| `tile`             | [DocID json-schema definition](https://github.com/ceramicnetwork/CIP/blob/main/CIPs/CIP-82/CIP-82.md) | Draft  |
| `appendCollection` | [AppendCollection schemas](https://github.com/ceramicnetwork/CIP/blob/main/CIPs/CIP-85/CIP-85.md)     | Draft  |
| `collectionSlice`  | [AppendCollection schemas](https://github.com/ceramicnetwork/CIP/blob/main/CIPs/CIP-85/CIP-85.md)     | Draft  |

## Rationale

Using the `$ceramic` property should be consistent with other `$`-prefixed metadata properties in JSON schemas, avoiding possible conflicts with other property names.

A unique `type`, along with possible type-specific additional properties, should make it easy to add custom extensions and build tools (simple checks for existence of `$ceramic` property and type-specific logic, TypeScript interfaces and inference, etc.).

Finally, providing a reference table in this CIP should allow for easy discovery and avoid conflicts between extensions.

## Backwards Compatibility

[CIP-82](https://github.com/ceramicnetwork/CIP/blob/main/CIPs/CIP-82/CIP-82.md) and [CIP-85](https://github.com/ceramicnetwork/CIP/pull/85) are updated to this new format.

## Implementation

None yet.

## Security Considerations

None I'm aware of.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
