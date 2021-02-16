---
cip: 19
title: Basic Profile Definition
author: Michael Sena (http://github.com/michaelsena), Joel Thorstensson (http://github.com/oed)
status: Last Call
category: Standards
type: RFC
created: 2020-05-22
Edited: 2020-09-28
requires: IdentityIndex (IDX) (CIP-11)
---

## Simple Summary

The **Basic Profile** contains a DID's basic profile information.

## Abstract

Decentralized identifiers (DIDs) are simply abstract strings that uniquely represent an entity on the web. These entities can be people, businesses, organizations, applications, services, DAOs, devices, etc. In order for other users to have a better sense of the entity they are interacting with online, additional context is required. Profile information is usually the first and most common type of information that is added to an identifier to provide context and make them more usable.

The Basic Profile defines a standard for storing a DID's basic profile information (such as name, image, description, etc) using an [IDX Definition (CIP-11)](../CIP-11/CIP-11.md). By standardizing profile information for DIDs, the Basic Profile simplifies how applications can view and display the profiles of their users.

## Motivation

**A profile is one of the most important aspects to identity**: Profiles are often most associated with identity, and the Basic Profile serves as the standard profile for a DID that can be used everywhere across the web.

**DID-agnostic support:** Since Ceramic documents can be created and controlled by any DID, the Basic Profile can be used in conjunction with any DID method.


## Specification

The Basic Profile is simply a *definition* where the record holds the data of the user. Therefore the schma defined in the *definition* describes the profile itself.

### Definition content

**Deployment:** `<definition-DocID>`

```json
{
  "name": "Basic Profile",
  "description": "Basic profile information for a DID",
  "schema": "<record-schema-DocID>",
}
```

### Record Schema

The Basic Profile schema defines the format of a document that contains the properties listed below. Properties not defined in the schema *cannot* be included in the Basic Profile, however the schema can always be updated via a new CIP.

| Property           | Description                    | Value                                                        | Max Size | Required | Example                      |
| ------------------ | ------------------------------ | ------------------------------------------------------------ | -------- | -------- | ---------------------------- |
| `name`             | a name                         | string                                                       | 150 char | optional | Mary Smith                   |
| `image`            | an image                       | Image sources                                                |          | optional |                              |
| `description`      | a short description            | string                                                       | 420 char | optional | This is my cool description. |
| `emoji`            | an emoji                       | unicode                                                      | 2 char   | optional | 🔢                            |
| `background`       | a background image (3:1 ratio) | Image sources                                                |          | optional |                              |
| `birthDate`        | a date of birth                | [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601)           | 10 char  | optional | 1990-04-24                   |
| `url`              | a url                          | string                                                       | 240 char | optional | http://ceramic.network       |
| `gender`           | a gender                       | string                                                       | 42 char  | optional | female                       |
| `residenceCity`    | a city of residence            | string                                                       | 140 char | optional | Berlin                       |
| `residenceCountry` | a country of residence         | [ISO 3166-1 alpha-2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) | 2 char   | optional | DE                           |
| `nationalities`    | nationalities                  | array of [ISO 3166-1 alpha-2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) values | 2 char   | optional | CN                           |
| `affiliations`     | affiliations                   | array of strings                                             | 240 char | optional | Ceramic Ecosystem Alliance   |

**Deployment:** `<record-schema-DocID>`

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "BasicProfile",
  "type": "object",
  "definitions": {
    "IPFSUrl": {
      "type": "string",
      "pattern": "^ipfs://.+",
      "maxLength": 150
    },
    "positiveInteger": {
      "type": "integer",
      "minimum": 1
    },
    "imageMetadata": {
      "type": "object",
      "properties": {
        "src": {
          "$ref": "#/definitions/IPFSUrl"
        },
        "mimeType": {
          "type": "string",
          "maxLength": 50
        },
        "width": {
          "$ref": "#/definitions/positiveInteger"
        },
        "height": {
          "$ref": "#/definitions/positiveInteger"
        },
        "size": {
          "$ref": "#/definitions/positiveInteger"
        }
      },
      "required": ["src", "mimeType", "width", "height"]
    },
    "imageSources": {
      "type": "object",
      "properties": {
        "original": {
          "$ref": "#/definitions/imageMetadata"
        },
        "alternatives": {
          "type": "array",
          "items": {
            "$ref": "#/definitions/imageMetadata"
          }
        }
      },
      "required": ["original"]
    }
  },
  "properties": {
    "name": {
      "type": "string",
      "maxLength": 150
    },
    "image": {
      "$ref": "#/definitions/imageSources"
    },
    "description": {
      "type": "string",
      "maxLength": 420
    },
    "emoji": {
      "type": "string",
      "maxLength": 2
    },
    "background": {
      "$ref": "#/definitions/imageSources"
    },
    "birthDate": {
      "type": "string",
      "format": "date"
    },
    "url": {
      "type": "string",
      "maxLength": 240
    },
    "gender": {
      "type": "string",
      "maxLength": 42
    },
    "homeLocation": {
      "type": "string",
      "maxLength": 140
    },
    "residenceCountry": {
      "type": "string",
      "pattern": "^[A-Z]{2}$"
    },
    "nationalities": {
      "type": "array",
      "minItems": 1,
      "items": {
        "type": "string",
        "pattern": "^[A-Z]{2}$",
        "maxItems": 5
      }
    },
    "affiliations": {
      "type": "array",
      "items": {
        "type": "string",
        "maxLength": 240
      }
    }
  }
}
```

## **Example**

An example of how to create a Basic Profile document with js-ceramic.

```js
const profile = await ceramic.createDocument('tile', {
  metadata: {
    schema: "<record-schema-DocID>"
    family: "<definition-DocID>"
  },
  content: {
    name: "Samantha Smith",
    image: {
      original: {
        src: "ipfs://bafy...",
        mimeType: "image/png",
        width: 500,
        height: 200
      }
    },
    description: "This is my funny description.",
    emoji: "🚀",
    url: "http://ceramic.network"
  }
})
```


## Rationale

**Decentralization & Trust:** In order for profile information to be universally resolved and publicly available across all platforms., it should be stored on a globally-available, permissionless, and censorship-resistant network (not on any single server). Additionally this information should be owned by a DID and will need to be updated from time to time. These requirements make Ceramic the most appropriate platform for publishing profile content.


## Implementation

* [**js-idx**](https://idx.xyz/): A javascript library for reading and writing data sets associated with a DID

* [**js-idx-constants**](https://github.com/ceramicstudio/js-idx-constants): A javascript library to be used with js-idx which contains the Basic Profile definition.


## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
