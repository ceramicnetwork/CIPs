---
cip: 23
title: Also Known As Definition
author: Michael Sena (@michaelsena), Joel Thorstensson (@oed)
status: Withdrawn
category: RFC
created: 2020-07-21
edited: 2023-05-24
requires: 8
---

## Simple Summary

**Also Known As** is a data set that stores a list of accounts that are publicly linked to the users DID.


## Abstract

Users may want to publicly associate any number of social accounts, such as Twitter, GitHub, discord, etc. to their DID. Creating and storing public verifications for these various accounts in the Also Known As *record* document enables anyone to observe these associations and as a result infer trust about the given user.

Adding an account to the Also Known As document *does not* allow that account to control or authenticate the DID. Rather it is simply just a verifiable public mapping.


## Motivation

Publicly associating one or more accounts to a DID offers a number of benefits.

- Public association with a DID to any number of accounts
- Serves as a foundation for building aggregated reputation on an identity comprised of multiple accounts, not individual one-off accounts
- Provides a common interface for creating a list of linked social accounts, making discoverability and query predictable


## Specification

The Also Known As document is a *definition* that contains an array of *linked accounts* entries.

### Definition content

**Deployment:** `kjzl6cwe1jw146zfmqa10a5x1vry6au3t362p44uttz4l0k4hi88o41zplhmxnf`

```json
{
  "name": "Also Known As",
  "description": "Accounts linked to your DID",
  "schema": "<record-schema-DocID>"
}
```

### Record Schema

The record schema defines a document which maintains an array of JSON objects that represent a DID's linked accounts. Below find the fields that are contained in each object.

| Property       | Description                                                  | Value            | Max Size  | Required | Example                                      |
| -------------- | ------------------------------------------------------------ | ---------------- | --------- | -------- | -------------------------------------------- |
| `protocol`     | Protocol used to resolve the ID.                             | string           | 50 char   | required | https                                        |
| `host`         | Host used to resolve the ID.                                 | string           | 150 char  | optional | github.com                                   |
| `id`           | The unique ID of the user.                                   | string           | 450 char  | required | marysmith                                    |
| `claim`        | Unique location where the DID string is stored. Should be a place where only the given ID could have posted the DID. | string           | 450 char  | optional | https://gist.github.com/marysmith/5c48debdb7 |
| `attestations` | Attestations issued by third-party services which have verified the claim against the ID.<br />The key in this object should be "did-jwt-vc" in order to indicate a verifiable credential. <br />For backwards compatibility reasons "did-jwt" is also supported. | array of objects | 1000 char | optional | {did-jwt-vc: "eyJhbGciOiJIUzI1..."}          |

**Deployment:** `k3y52l7qbv1fryojt8n8cw2k04p9wp67ly59iwqs65dejso566fij5wsdrb871yio`

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "title": "AlsoKnownAs",
  "properties": {
    "accounts": {
      "type": "array",
      "items": {
        "$ref": "#/definitions/Account"
      }
    }
  },
  "additionalProperties": false,
  "required": [
    "accounts"
  ],
  "definitions": {
    "Attestation": {
      "type": "object",
      "properties": {
        "did-jwt": {
          "type": "string",
          "maxLength": 1000
        },
        "did-jwt-vc": {
          "type": "string",
          "maxLength": 1000
        }
      }
    },
    "Account": {
      "type": "object",
      "properties": {
        "protocol": {
          "type": "string",
          "maxLength": 50
        },
        "host": {
          "type": "string",
          "maxLength": 150
        },
        "id": {
          "type": "string",
          "maxLength": 450
        },
        "claim": {
          "type": "string",
          "maxLength": 450
        },
        "attestations": {
          "type": "array",
          "items": {
            "$ref": "#/definitions/Attestation"
          }
        }
      },
      "required": [
        "protocol",
        "id"
      ]
    }
  }
}
```

### Example

An example Also Known As *record* document that includes two accounts, Twitter and Github.

```js
const profile = await ceramic.createDocument('tile', {
  metadata: {
    schema: "<record-schema-DocID>"
    family: "<definition-DocID>"
  },
  content: {
    accounts: [{
      protocol: "https",
      host: "twitter.com",
      id: "marysmith",
      // ID of tweet containing the user's DID
      claim: "https://twitter.com/marysmith/status/1274020265417076736",
      attestations: [{ did-jwt-vc: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." }]
    }, {
      protocol: "https",
      host: "github.com",
      id: "marysmith",
      // ID of Gist containing the user's DID
      claim: "https://gist.github.com/marysmith/5c48debdb7089b3c8f86cca31739572c",
      attestations: [{ did-jwt-vc: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." }]
    }]
  }
})
```


## Suggested Usage

It is recomended to always include both the `claim` and the `attestation` property. The claim is usually a way to verify that the social account actually wants to be associated with the users DID. The attestation is a [Verifiable Credential](https://github.com/decentralized-identity/did-jwt-vc) from a third party that this is the case. In some scenarious it might be impossible to provide a publicly verifiable `claim`, for example; in some systems it may not be possible to create a public post which can be easily linked to. In these cases just an `attestation` will have to suffice. Note however that this means that there is full trust in the issuer of this attestation, where as when there is a public claim it's always possible to double check the truth of the attestation.


## Rationale

**Decentralization & Trust:** Linked account directories are data that needs to be globally-available, cross-chain, censorship-resistant, and live permissionlessly in the public domain (not on any single server). Additionally this information should be owned by a DID and will need to be updated from time to time. These requirements make Ceramic the most appropriate platform for publishing account links content.


## Implementation

- [**js-idx**](https://idx.xyz/): A javascript library for reading and writing data sets associated with a DID
- [**js-idx-constants**](https://github.com/ceramicstudio/js-idx-constants): A javascript library to be used with js-idx which contains the Also Known As definition.


## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
