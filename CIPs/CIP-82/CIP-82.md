---
cip: 82
title: StreamID json-schema definition
author: Paul Le Cam (@PaulLeCam)
discussions-to: https://github.com/ceramicnetwork/CIP/issues/82
status: Draft
category: Standards
type: RFC
created: 2021-02-15
edited: 2021-07-02
requires: [CIP-88](https://github.com/ceramicnetwork/CIP/issues/88)
---

## Simple Summary

Provide a static way to define a string in a JSON schema represents a Ceramic StreamID, optionally with static references to the schema(s) that must be used by the referenced stream.

## Abstract

This CIP defines a standard way to add a reference to an existing Ceramic stream in a JSON schema and references to existing JSON schemas, so it is possible to access this information about Ceramic streams at build time rather than only at runtime on created streams.

## Motivation

It is sometimes necessary to reference Ceramic streams from other streams, such as a list containing the streamIDs of individual Ceramic streams.
Currently, we are sometimes using definitions that can be referenced in a schema using `"$ref": "#/definitions/CeramicDocId"` for example, but this has not be defined as a standard.
Using local definitions to a schema also has the downside of providing no guaranty of being unique or having the definition matching any standard.

Furthermore, IDX definitions contain a `schema` property that references an existing schema, guarantying the IDX record associated to the definition matches this schema.
This CIP provides a way to implement similar logic for any schema independently of IDX.

Using this CIP, a `NotesList` schema could explicitly reference a `Note` schema with the following example:

```
{
  $schema: 'http://json-schema.org/draft-07/schema#',
  title: 'Notes',
  type: 'object',
  properties: {
    notes: {
      type: 'array',
      title: 'list',
      items: {
        type: 'object',
        title: 'item',
        properties: {
          note: { $ref: '#/definitions/NoteDocID' },
          title: {
            type: 'string',
            maxLength: 100,
          },
        },
        required: ['note'],
      },
    },
  },
  definitions: {
    NoteDocID: {
      type: 'string',
      maxLength: 150,
      $comment: 'cip88:ref:<Note schema streamID>',
    },
  },
}
```

This way, by loading the `Notes` schema, it is possible by a tool/library to discover the `Note` schema the same way loading a IDX definition allows the discovery of the record's schema.

## Specification

References to Ceramic schema should use a `string` with the `$comment` field using the `cip88:ref` type, and optionally with a schema string of the StreamID (implicit reference to latest version) or CommitID (specific version) of the supported schema(s).
Multiple schemas can be provided, using the `|` character as separator.

```js
{
  type: 'string',
  maxLength: 150,
  $comment: 'cip88:ref:<Note schema streamID>',
}
```

## Rationale

This CIP uses the `$comment` field as defined in [CIP-88](https://github.com/ceramicnetwork/CIP/blob/main/CIPs/CIP-88/CIP-88.md).

This spec allows to either define a single schema or multiple ones sparated by the `|` character.
The use case would be to support different schemas for a single reference, for example a "media" schema could reference an "image" schema, but also the "audio" and "video" ones as acceptable document schemas: `$comment: 'cip88:ref:<image schema docID>|<audio schema docID>|<video schema docID>'`.

## Backwards Compatibility

Ideally we should replace the use of the `CeramicDocId` definition in IDX schemas we provide, as well as examples and tutorials.

## Implementation

None yet.

## Security Considerations

None I'm aware of.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
