```
cip: 20
title: 3ID Keychain
author: Michael Sena (@michaelsena), Joel Thorstensson (@oed)
status: Last Call
category: Standards
type: RFC
created: 2020-05-26
edited: 2020-10-14
requires: Tile Doctype (CIP-8), CAIP-10 Link Doctype (CIP-7)

```

## Simple Summary

**3ID Keychain** uses public-key cryptography to store a series of encrypted seeds that can be used to control a 3ID DID. This allows multiple  different wallets to have the ability to authenticate to the same DID,  by having control over just one of the *auth secrets* in the keychain.


## Abstract

The 3ID Keychain stores data which allows the DID to be authenticated using any number of external wallet keys. As a result, this document  allows the DID, and by extension its associated resources, to be  controllable and accessible using many different wallet key pairs. Each  key pair added to the document is considered an equal owner, and has the ability to add or remove any other key pairs found in the document by performing a key rotation in the users 3ID document.

Adding a new key pair to the 3ID Keychain does not disclose a public link between that account and the DID, which serves the purpose of providing privacy to the user. If you would like to create a public link between a DID and a given cryptographic account, it would need to be  achieved by additionally adding that account to a [Crypto Account Links (CIP-21)](../CIP-21/CIP-21.md) document.


## Motivation

A few reasons why having a multi-key, cross-chain, technology-agnostic keychain is beneficial for DIDs:

- Simplified, seamless user experience across platforms
- Interoperability of resources across contexts that may rely on different keys or key types
- Integration with existing wallets already possessed by a user
- Integration with and interoperability across many different blockchain networks, web3 technologies, and wallet providers
- Delegate the DID "key management problem" to wallet providers
- Resilience against DID loss, since a user could only lose control of a DID if they simultaneously lose control of all auth keys at the same  time


## Specification

The 3ID Keychain is a *definition* that contains a map from Ceramic DocIDs to auth methods, as well as an array of past seeds. This data is stored in the `authMap` and `pastSeeds` properties respectively.

#### `authMap`

The *authMap* is a json map from DocIDs to an *AuthData* object. The DocID refers to a [Crypto Account Links (CIP-21)](https://github.com/ceramicnetwork/CIP/issues/44) document which given this CAIP-10 account we can find the DID that uses this keychain. The auth data contains three properties `id`, `pub`, and `data`. The content of each should be the following.

* `id` - a name of this authentication method, stored as a JWE encrypted to the public key of the 3ID
  Once decrypted it should contain an object: `{ id: <string> }`
* `pub` - the public key of the auth method, a multicodec key (usually `x25519`)
* `data` - the seed of the 3ID, stored as a JWE encrypted to the `pub`.
  Once decrypted it should contain an object: `{ seed: <string> }`, this object may also contain an additional property `v03ID` if this keychain is used for a legacy 3ID.

#### `pastSeeds`

The past seeds array contain JWEs that includes previous seeds, which since have been rotated away from. This array is ordered where the most recently rotated seed is at the end of the array and the oldest seed is at the first position. Each seed is encrypted to the encryption key of the seed which came after it. For example `seec0` would be encrypted to the encryption key of `seed1`, which in turn would be encrypted to the encryption key of `seed2` which is the seed being currently used by the 3ID.

### Definition content

**Deployment:** `<definition-DocID>`

```json
{
  "name": "3ID Keychain",
  "description": "Key data for 3ID",
  "schema": "<reference-schema-DocID>"
}
```

### Reference Schema

**Deployment:** `<reference-schema-DocID>`

```jsx
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "definitions": {
    "JWE": {
      "type": "object",
      "properties": {
        "protected": {
          "type": "string"
        },
        "iv": {
          "type": "string"
        },
        "ciphertext": {
          "type": "string"
        },
        "tag": {
          "type": "string"
        },
        "recipients": {
          "type": "array"
        }
      },
      "required": [
        "protected",
        "tag",
        "ciphertext",
        "iv"
      ]
    },
    "WrappedJWE": {
      "type": "object",
      "additionalProperties": false,
      "properties": {
        "jwe": {
          "$ref": "#/definitions/JWE"
        }
      }
    },
    "AuthData": {
      "type": "object",
      "additionalProperties": false,
      "properties": {
        "id": {
          "$ref": "#/definitions/WrappedJWE"
        },
        "pub": {
          "type": "string"
        },
        "data": {
          "$ref": "#/definitions/WrappedJWE"
        }
      }
    }
  },
  "type": "object",
  "additionalProperties": false,
  "properties": {
    "authMap": {
      "type": "object",
      "additionalProperties": {
        "$ref": "#/definitions/AuthData"
      }
    },
    "pastSeeds": {
      "type": "array",
      "items": {
        "$ref": "#/definitions/JWE"
      }
    }
  }
}
```

### Example

An example Crypto Accounts *reference* document that includes two Ethereum accounts and one Bitcoin account.

```js
const profile = await ceramic.createDocument('tile', {
  metadata: {
    schema: "<reference-schema-DocID>",
    family: "<definition-DocID>"
  },
  content: {
    "authMap": {
      "ceramic://bafyreidr3l6p3o4diwxcjaju52qdstuoqzx6ab3difzefindn3z2gopyxm": {
        "pub": "z6LSgKUwxA84iwKFXLH5mtmdXP1Mo7dwTS35zPocLtdsTJEA",
        "data": {
          "jwe": {
            "protected": "eyJlbmMiOiJYQzIwUCJ9",
            "iv": "PmS1kPQPP1C-oBbMKwe1tjCo3fb2_HBo",
            "ciphertext": "SBQO8C52ohlhXoZ2PBoGmLCby6iCm_9NH1Cma4pLW0xeASGuSHlcAF_CuCO7skmX",
            "tag": "73N7eTO2OmA0b4WQcMlgiQ",
            "recipients": [
              {
                "encrypted_key": "H794nV-bAq5dVdprrzaJsf-sWtLsZsgwDnUmP3Ph_fo",
                "header": {
                  "alg": "ECDH-ES+XC20PKW",
                  "iv": "AIanqbmZFJguYNb6tr8btUXB_10R0TWH",
                  "tag": "eDmTfAiRCpdyLm6u_CfwSw",
                  "epk": {
                    "kty": "OKP",
                    "crv": "X25519",
                    "x": "sOxylUk7RaONasqY2dt8AQdUVyf8KpHe7SI7bLitLmw"
                  }
                }
              }
            ]
          }
        },
        "id": {
          "jwe": {
            "protected": "eyJlbmMiOiJYQzIwUCJ9",
            "iv": "r6Zo44cJ3qwV-8rZxqm1Dz3FgkU6QxAO",
            "ciphertext": "h69kUkEF_WJfrTkyCjId8K7Xd39jx7gf",
            "tag": "GceIifD_PwmIP28275KisA",
            "recipients": [{
              "encrypted_key": "FVr6Z4kzlaaW_BS9u5MzDlihKZXtzrMZVLCfP0VSfEY",
              "header": {
                "alg": "ECDH-ES+XC20PKW",
                "iv": "TNYT2GfWIrIteIm0GUkWbP1b4F1NFAHR",
                "tag": "lYJbVdMa5BV3CpW2wWa1OQ",
                "epk": {
                  "kty": "OKP",
                  "crv": "X25519",
                  "x": "Ralo1CZNsyvr3kZetcwoC0FIWoVlmJEsB6ooSMm2GGw"
                },
                "kid": "did:3:bagcqcerawvlfwwxgksmmyekzevnalt6qamc3os4is5uhiejlf5gyz6mmlyma#VeMYfDH9g9vsgr3"
              }
            }]
          }
        }
      }
    },
    "pastSeeds": []
  }
})
```


## Suggested Usage

**Crypto Account Links**: If you want to make a public association between a key pair account and a DID, the account should also be added to the [Crypto Account Links (CIP-21)](https://github.com/ceramicnetwork/CIP/issues/44) document.

**Key Rotation**: Deprovisioning auth keys, often called *key rotations*, is a sensitive operation  that relies on *strict ordering* to guarantee which keys are allowed to make updates to the DID or its resources at any given point in time. The *3ID Keychain* can play a key part in this process. Described below is an example of how a wallet that uses the 3ID DID method can use the auth keychain to securely have multiple keys that control the DID.

#### Example: 3ID multi-auth key revocation:

##### Setup

Let's set up a 3ID that has two authentication methods.

1. Generate a seed `s1`, derive keys `s1-managementKey`, `s1-signingKey`, and `s1-encryptionKey`
2. Create a 3ID document with the keys derived in **(1)**
3. Create two secret keys that can be used to authenticate to this 3ID, `sk1`, `sk2`
4. Encrypt the seed `s1` to the public keys of they keys from **(3)** and store them in the *3ID Keychain*

Both `sk1` and `sk2` can now be used to decrypt authData contained in the *Auth Keychain* and thus get read an write access to any data associated with the DID.

##### Revocation

Now suppose we want to revoke `sk2` to no longer have access and control over the 3ID.

1. Generate a new seed `s2`, derive keys `s2-managementKey`, `s2-signingKey`, and `s2-encryptionKey`
2. Make a key rotation in the 3ID document to the new keys derived in **(1)**
3. Remove the old authData from the *3ID Keychain*
4. Encrypt the seed `s2` to the public key `s1`, and add this authData to the *AuthKeychain*

No only `sk1` will be able to authenticate to the 3ID.


## Rationale

**Decentralization & Trust:** In order for keychain data to be controlled by users and publicly available across all  platforms, it should be stored on a globally-available, permissionless, and censorship-resistant network (not on any single server).  Additionally this information should be owned by a DID and will need to  be updated from time to time. These requirements make Ceramic the most  appropriate platform for publishing keychain content.


## Implementation

- [**js-idx**](https://idx.xyz/): A javascript library for reading and writing data sets associated with a DID
- [**js-idx-constants**](https://github.com/ceramicstudio/js-idx-constants): A javascript library to be used with js-idx which contains the Basic Profile definition.


## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
