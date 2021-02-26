---
cip: 82
title: DocID json-schema definition
author: Paul Le Cam (@PaulLeCam)
discussions-to: https://github.com/ceramicnetwork/CIP/issues/82
status: Draft
category: Standards
type: RFC
created: 2021-02-15
edited: 2021-02-22
---

## Simple Summary

Provide a static way to define a string in a JSON schema represents a Ceramic DocID, optionally with static references to the schema(s) that must be used by the referenced document.

## Abstract

This CIP defines a standard way to add a reference to an existing Ceramic document in a JSON schema and references to existing JSON schemas, so it is possible to access this information about Ceramic documents at build time rather than only at runtime on created documents.

## Motivation

It is sometimes necessary to reference Ceramic documents from other documents, such as a list containing the docIDs of individual Ceramic documents.
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
      $id: 'ceramic://doc',
      $ceramicSchema: '<Note schema docID>' ,
      maxLength: 150,
    },
  },
}
```

This way, by loading the `Notes` schema, it is possible by a tool/library to discover the `Note` schema the same way loading a IDX definition allows the discovery of the record's schema.

## Specification

References to Ceramic schema should use a `string` with the [`$id`](https://json-schema.org/draft/2019-09/json-schema-core.html#rfc.section.8.2.2) field of `ceramic://doc`, and optionally with a `$ceramicSchema` property containing either a string or array of strings containing the DocID (implicit reference to latest version) or CommitID (specific version) of the supported schema(s):

```js
{
  type: 'string',
  $id: 'ceramic://doc',
  $ceramicSchema: '<schema docID or commitID>' ,
  maxLength: 150,
}
```

## Rationale

Using the `$id` field on the object gives a unique namespace (`ceramic://`) and identifier (`doc`) so it creates a convention that can be used in similar CIPs.

Having this static way of identifying if a document references another document allows to simplify the associated logic, rather than relying on checking all string fields for the `ceramic://` prefix, or relying on `"$ref": "#/definitions/CeramicDocId"` matching the expected constraints.

This spec also allows to either define a single schema (using a string) or multiple ones (array of strings).
The use case would be to support different schemas for a single reference, for example a "media" schema could reference an "image" schema, but also the "audio" and "video" ones as acceptable document schemas: `$ceramicSchema: ['<image schema docID>', '<audio schema docID>', '<video schema docID>']`.

## Backwards Compatibility

Ideally we should replace the use of the `CeramicDocId` definition in IDX schemas we provide, as well as examples and tutorials.

## Implementation

None yet.

## Security Considerations

None I'm aware of.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
