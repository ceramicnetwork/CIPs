---
cip: 21
title: Crypto Accounts Definition
author: Michael Sena (@michaelsena), Joel Thorstensson (@oed)
status: Withdrawn
category: RFC
created: 2020-06-15
edited: 2023-05-24
requires: 7, 8
---

## Simple Summary

**Crypto Accounts** is a data set that stores a list of a DID's publicly linked crypto accounts.


## Abstract

Users may want to publicly associate any number of cryptographic key pairs or accounts, such as Ethereum accounts, Flow accounts, smart contracts, etc. to their DID. Creating and storing public links to these various accounts in the Crypto Accounts *record* document enables anyone to observe these associations and as a result resolve a DID and its resources by looking up a specific crypto account or vice versa. 

Adding an account to the Crypto Accounts document *does not* allow that account to control or authenticate the DID. Rather it is simply just a verifiable public mapping.


## Motivation

Publicly associating one or more public key pairs, accounts, or contracts to a DID offers a number of benefits.

- Public, two-way resolution between a DID and any number of accounts
- Users and developers can utilize DID-based resources (i.e. profiles, data, connections, etc.) in the context of any blockchain or other Web3 app
- Provides a better user experience for users in a multi-key, multi-chain world by eliminating the need to redundantly create new DIDs and resources for each new key and network
- Provides interoperability of DIDs and resources across applications, networks, and wallets where users have different keys
- Serves as a foundation for building aggregated reputation on an identity comprised of multiple accounts, not individual one-off accounts
- Provides a common interface for creating a list of linked crypto accounts, making discoverability and query predictable


## Specification

The Crypto Accounts is a *definition* that contains a map from CAIP-10 account-ids as keys, and DocIDs of CAIP-10 Link documents as values.

### Definition content

**Deployment:** `kjzl6cwe1jw149z4rvwzi56mjjukafta30kojzktd9dsrgqdgz4wlnceu59f95f`

```json
{
  "name": "Crypto Accounts",
  "description": "Crypto accounts linked to your DID",
  "schema": "<record-schema-DocID>"
}
```

### Record Schema

**Deployment:** `k3y52l7qbv1frypussjburqg4fykyyycfu0p9znc75lv2t5cg4xaslhagkd7h7mkg`

```jsx
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "title": "CryptoAccounts",
  "patternProperties": {
    "^[a-zA-Z0-9]{1,63}@[-a-zA-Z0-9]{3,16}:[-a-zA-Z0-9]{1,47}": {
      "type": "string",
      "pattern": "^ceramic://.+"
    }
  },
  "additionalProperties": false
}
```

### Example

An example Crypto Accounts *record* document that includes two Ethereum accounts and one Bitcoin account.

```js
const profile = await ceramic.createDocument('tile', {
  metadata: {
    schema: "<record-schema-DocID>"
    family: "<definition-DocID>"
  },
  content: {
    "0xab16a96d359ec26a11e2c2b3d8nmn8942d5bxzmn@eip155:1": "ceramic://bafyljsdf1...",
    "0xzx45a83d123ec26a11e2c2b3d8f8b8942d5bfcdb@eip155:1": "ceramic://bafyljsdf2...",
    "128Lkh3S7CkDTBZ8W7BbpsN3YYizJMp8p6@bip122:000000000019d6689c085ae165831e93": "ceramic://bafysdfoijwe3..."
  }
})
```


## Suggested Usage

**CAIP-10 Links**: The Crypto Accounts *record* document stores records to [CAIP-10 Links (CIP-7)](https://github.com/ceramicnetwork/CIP/issues/15), which are separate documents that cryptographically prove that the DID owns a given linked blockchain account. At the time a CAIP-10 Link is created, it should be added to this document.


## Rationale

**Decentralization & Trust:** Linked account directories are data that needs to be globally-available, cross-chain, censorship-resistant, and live permissionlessly in the public domain (not on any single server). Additionally this information should be owned by a DID and will need to be updated from time to time. These requirements make Ceramic the most appropriate platform for publishing account links content.


## Implementation

- [**js-idx**](https://idx.xyz/): A javascript library for reading and writing data sets associated with a DID
- [**js-idx-constants**](https://github.com/ceramicstudio/js-idx-constants): A javascript library to be used with js-idx which contains the Basic Profile definition.


## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
