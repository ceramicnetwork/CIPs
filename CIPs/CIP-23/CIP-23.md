```
cip: 23
title: Social Accounts Definition
author: Michael Sena (@michaelsena), Joel Thorstensson (@oed)
status: Last Call
category: Standards
type: RFC
created: 2020-07-21
edited: 2020-11-03
requires: Tile Doctype (CIP-8)
```

## Simple Summary

**Social Accounts** is a data set that stores a list of a social accounts that are publicly linked to the users DID.


## Abstract

Users may want to publicly associate any number of social accounts, such as Twitter, GitHub, discord, etc. to their DID. Creating and storing public verifications for these various accounts in the Social Accounts *reference* document enables anyone to observe these associations and as a result infer trust about the given user. 

Adding an account to the Social Accounts document *does not* allow that account to control or authenticate the DID. Rather it is simply just a verifiable public mapping.


## Motivation

Publicly associating one or more social accounts to a DID offers a number of benefits.

- Public association with a DID to any number of accounts
- Serves as a foundation for building aggregated reputation on an identity comprised of multiple accounts, not individual one-off accounts
- Provides a common interface for creating a list of linked social accounts, making discoverability and query predictable


## Specification

The Social Accounts is a *definition* that contains an array of *linked accounts* entries.

### Definition content

**Deployment:** `<definition-DocID>`

```json
{
  "name": "Social Accounts",
  "description": "Social accounts linked to your DID",
  "schema": "<reference-schema-DocID>",
  "config": {
    "tags": ["SocialAccounts"]
  }
}
```

### Reference Schema

The reference schema defines a document which maintains an array of JSON objects that represent a DID's linked accounts. Below find the fields that are contained in each object.

| Property       | Description                                                  | Value            | Max Size  | Required | Example                                      |
| -------------- | ------------------------------------------------------------ | ---------------- | --------- | -------- | -------------------------------------------- |
| `protocol`     | Network protocol used to resolve the host                    | string           | 50 char   | required | https                                        |
| `host`         | Host used to resolve the ID                                  | string           | 150 char  | required | https://github.com                           |
| `id`           | Path of the unique ID within the host. Can be the same as the host to represent the root. | string           | 450 char  | required | https://github.com/marysmith                 |
| `claim`        | Unique location where the DID string is stored. Should be a place where only the given ID could have posted the DID. | string           | 450 char  | optional | https://gist.github.com/marysmith/5c48debdb7 |
| `attestations` | Attestations issued by third-party services which have verified the claim against the ID. | array of strings | 1000 char | optional | ceramic://bafy...123                         |

**Deployment:** `<reference-schema-DocID>`

```jsx
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "array",
  "title": "SocialAccounts",
  "items": {
    "$ref": "#/definitions/Account"
  },
  "definitions": {
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
          "type": "array"
          "items": {
            "type": "string",
		      	"maxLength": 1000
          }
        }
      },
      "required": [
        "protocol",
        "host",
        "id"
      ]
    }
  }
}
```

### Tags

In the config of the *definition* defined above we can see a `tags` property. We simply apply the tags in the config to the `tags` property in the *metadata* of the *reference* Tile.

### Example

An example Crypto Accounts *reference* document that includes two Ethereum accounts and one Bitcoin account.

```js
const profile = await ceramic.createDocument('tile', {
  metadata: {
    schema: "<reference-schema-DocID>"
    tags: ["SocialAccounts"]
  },
  content: [{
		protocol: "https",
		host: "https://twitter.com",
		id: "https://twitter.com/marysmith",
    // ID of tweet containing the user's DID
		claim: "https://twitter.com/marysmith/status/1274020265417076736",
		attestations: ["eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."]
	}, {
  	protocol: "https"
		host: "https://github.com"
		id: "https://github.com/marysmith"
    // ID of Gist containing the user's DID
		claim: "https://gist.github.com/marysmith/5c48debdb7089b3c8f86cca31739572c"
		attestations: ["eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."]
	}]
})
```


## Suggested Usage

It is recomended to always include both the `claim` and the `attestation` property. The claim is usually a way to verify that the social account actually want's to be associated with the users DID. The attestation is a [Verifiable Credential](https://github.com/decentralized-identity/did-jwt-vc) from a third party that this is the case. In some scenarious it might be impossible to provide a publicly verifiable `claim`. In these cases just an `attestation` will have to suffice.


## Rationale

**Decentralization & Trust:** Linked social account directories are data that needs to be globally-available, cross-chain, censorship-resistant, and live permissionlessly in the public domain (not on any single server). Additionally this information should be owned by a DID and will need to be updated from time to time. These requirements make Ceramic the most appropriate platform for publishing account links content.


## Implementation

- [**js-idx**](https://idx.xyz/): A javascript library for reading and writing data sets associated with a DID
- [**js-idx-constants**](https://github.com/ceramicstudio/js-idx-constants): A javascript library to be used with js-idx which contains the Basic Profile definition.


## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).