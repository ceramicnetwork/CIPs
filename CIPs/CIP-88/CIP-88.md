---
cip: 88
title: Ceramic namespace in JSON schemas
author: Paul Le Cam (@PaulLeCam)
status: Draft
category: Standards
type: RFC
created: 2021-03-01
edited: 2021-07-02
---

## Simple Summary

Define a standard extension point for Ceramic-specific metadata in a JSON schema.

## Abstract

This CIP defines a reserved namespace for Ceramic-specific metadata in a JSON schema, along with a reference table for standard uses of this namespace.

## Motivation

As commented in https://github.com/ceramicnetwork/CIP/issues/82#issuecomment-787449788 the `$id` cannot be used to define Ceramic-specific extensions as intended in [CIP-82](https://github.com/ceramicnetwork/CIP/blob/main/CIPs/CIP-82/CIP-82.md).

A previous version of this CIP relied on creating a custom namespace for Ceramic-specific metadata using the non-standard `$ceramic` key, but it was not compatible with the "strict mode" of the JSON validation library used in Ceramic (AJV).
Instead, this CIP relies on the `$comment` field that is supported by AJV's strict mode.

## Specification

### Namespace

A JSON schema property can contain a `$comment` field, that must be a string starting with `cip88:`:

```js
{
  type: 'string',
  maxLength: 150,
  $comment: 'ceramic:tile:<schema streamID or commitID>',
}
```

### Reference table

| Type               | CIP                                                                                                      | Status |
| ------------------ | -------------------------------------------------------------------------------------------------------- | ------ |
| `doc`              | [StreamID json-schema definition](https://github.com/ceramicnetwork/CIP/blob/main/CIPs/CIP-82/CIP-82.md) | Draft  |
| `appendCollection` | [AppendCollection schemas](https://github.com/ceramicnetwork/CIP/blob/main/CIPs/CIP-85/CIP-85.md)        | Draft  |
| `collectionSlice`  | [AppendCollection schemas](https://github.com/ceramicnetwork/CIP/blob/main/CIPs/CIP-85/CIP-85.md)        | Draft  |

## Rationale

Using the `$comment` property allows to add a metadata string to a schema in a spec-compliant way (compatible with AJV's strict mode).

The string value must match the following pattern: `cip88:<type>[type-specific string]`.

A unique `type`, possibly followed by an additional type-specific string, should make it easy to add custom extensions and build tools based on having the `$comment` value start with `cip88:`.

Finally, providing a reference table in this CIP should allow for easy discovery and avoid conflicts between extensions.

## Backwards Compatibility

[CIP-82](https://github.com/ceramicnetwork/CIP/blob/main/CIPs/CIP-82/CIP-82.md) and [CIP-85](https://github.com/ceramicnetwork/CIP/pull/85) are updated to this new format.

## Implementation

None yet.

## Security Considerations

None I'm aware of.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
