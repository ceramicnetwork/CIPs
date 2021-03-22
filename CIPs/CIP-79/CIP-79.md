---
cip: 79
title: 3ID DID Method Specification
author: Joel Thorstensson (@oed)
discussions-to: https://github.com/ceramicnetwork/CIP/issues/80
status: Draft
category: Standards
type: RFC
created: 2021-02-12
---

# 3ID DID Method Specification

3ID is a DID method that is implemented natively on Ceramic. It uses a Ceramic document as a source of truth of the information which makes up the DID document for the 3ID. Documents in Ceramic achieves secure key rotation though *proof-of-publication* by anchoring document updates into a blockchain. This means that 3ID inherits this property.

## DID Method Name

The name string that shall identify this DID method is: `3`.

A DID that uses this method MUST begin with the following prefix: `did:3`. Per the [DID specification](https://w3c.github.io/did-core/), this string MUST be in lowercase. The remainder of the DID, after the prefix, is specified below.

## Method Specific Identifier

There are two versions of 3IDs, both versions use a Ceramic document as a way to update the DID document. 3IDv1 is the most recent version, 3IDv0 is supported for legacy reasons. If you are creating new 3IDs you should always use 3IDv1. Both versions are always encoded using multibase. To determine if it's v1 or v0 first convert the multibase string into a byte array, if the first varint is `0x01` we have a 3IDv0, and if it's a `0xce` we have a 3IDv1.

### 3IDv1

The method specific identifier for 3IDv1 is simply a Ceramic DocID. The DocID used refers to the Ceramic document which contains the information needed to construct the DID Document upon resolution. DocIDs are encoded according to [CIP-59](https://github.com/ceramicnetwork/CIP/blob/master/CIPs/CIP-59/CIP-59.md).

```
3idv1 = "did:3:<docId>"
```

#### Example

```sh
did:3:kjzl6cwe1jw149tlplc4bgnpn1v4uwk9rg9jkvijx0u0zmfa97t69dnqibqa2as
```

### 3IDv0

The method specific identifier for 3IDv0 is a [CID](https://github.com/multiformats/cid) as produced by the [IPLD](https://github.com/ipld/specs) codec [dag-cbor](https://github.com/ipld/specs/blob/master/block-layer/codecs/dag-cbor.md).

```
3idv0 = "did:3:<cid>"
```

#### Example

```sh
did:3:bafyreiffkeeq4wq2htejqla2is5ognligi4lvjhwrpqpl2kazjdoecmugi
```

## CRUD Operation Definitions

In this section the CRUD operations for a 3ID DID are defined.

### Create

A `3` DID is created by simply creating a Ceramic [`tile`](https://github.com/ceramicnetwork/CIP/blob/master/CIPs/CIP-8/CIP-8.md) document. The [`tile`](https://github.com/ceramicnetwork/CIP/blob/master/CIPs/CIP-8/CIP-8.md) document type takes a DID as the *controller*, and it's recommended that a `did:key` is used. The *controller* is the DID which is allowed to update the document. The *family* of the document is set to `3id`, and the *deterministic* flag is set to `true`.

Now the content of the document should consist of a JSON object with one property *publicKeys*. These public keys will be allowed to sign messages on behalf of the 3ID, or decrypt messages encrypted to the 3ID. The value of this property should be an object where the value is a [multicodec](https://github.com/multiformats/multicodec/) + multibase(*base58btc*) encoded public key, the key for any given key should be the last *15* characters of the encoded public key.

The 3ID DID method supports any type of public key that can be encoded using multicodec which can easily be extended to support quantum resistant signature and encryption schemes in the future.

#### Example

Below you can see an example of how to create a 3ID using the Ceramic javascript api.

```js
const doc = await ceramic.createDocument({
  metadata: {
    controllers: ['did:key:zQ3shQNcackrTByiYaPGso1Nt7b6r1gSMg4XXBmavzvTMqX1h'], // secp256k1 did:key
    family: '3id'
  },
  content: {
    publicKeys: {
      "XQCT2xJHsdY6iJH": "z6LSqKWh3XQ7AfsJuE2KR23cozEut8D5CXQCT2xJHsdY6iJH",  // x22519 public key
      "yh27jTt7Ny2Pwdy": "zQ3shrMGEKAjUTMmvDkcZ7Y3x9XnVjTH3myh27jTt7Ny2Pwdy", // secp256k1 public key
    }
  },
  deterministic: true
})

const didString = `did:3:${doc.id}`
```

### Read/Verify

Resolving a 3ID is quite straight forward. It is done by taking the DocID from the method specific identifier, looking up the referred document using Ceramic, and converting the content of the Ceramic document into a DID document.

#### Loading 3IDv1 document

To load a 3IDv1 document simply take the DocID from the method specific identifier and load the document from ceramic.

#### Loading 3IDv0 document

To load a 3IDv0 document, first take the CID from the method specific identifier and load the corresponding object from the IPFS network. The resolved object will contain a `publicKey` property which has an array of public key objects (see [Appendix A](#appendix-a)).

Now find the key with an id that ends in `#signingKey` and convert it to a multicodec encoded key (in compressed format), then use it as a `did:key` and look up the ceramic document which has this key as the *controller* and *family* set to `3id`, using the *deterministic* flag. Below you can see the code to do this with the javascript Ceramic api.



```js
const doc = await ceramic.createDocument({
  metadata: {
    controllers: [didKey], // secp256k1 did:key
    family: '3id'
  },
  deterministic: true
})
```

#### Converting the loaded Ceramic document to the DID document

Once we have the Ceramic document loaded, load the latest *AnchorCommit* of the document (the latest commit that was anchored to a blockchain). Then get the content of the document at this `AnchorCommit`, which will look something like this:

```json
{
  "publicKeys": {
    "j9uh8iKHSDSLat4": "zQ3shaLgXkpEXt7He5mogBxFfib5cE2kv9j9uh8iKHSDSLat4",
    "qYfZN2QNDgjJpaL": "z6LSnTyV1nSWfMmfV1PoRmo3enhDnfqfFqYfZN2QNDgjJpaL"
  }
}
```

To convert this into a DID document first create an empty DID document:

```json
{
  "@context": "https://w3id.org/did/v1",
  "id": "<did>",
  "verificationMethod": [],
  "authentication": [],
  "keyAgreement": []
}
```

Now iterate though the entires in the `publicKeys` object in the Ceramic document and do the following:

* If it's a `secp256k1` key:

  * Add a `Secp256k1VerificationKey2018` to the *verificationMethod* array:

    ```js
    {
      id: "<did>#<entry-key>",
      type: "EcdsaSecp256k1Signature2019",
      controller: "<did>",
      publicKeyBase58: "<entry-value-public-key-base58btc-encoded>"
    }
    ```

  * Add a `Secp256k1SignatureAuthentication2018` to the *authentication* array:

    ```js
    {
      id: "<did>#<entry-key>",
      type: "EcdsaSecp256k1Signature2019",
      controller: "<did>",
      publicKeyBase58: "<entry-value-public-key-base58btc-encoded>"
    }
    ```

* If it's a `x25519` key:

  * Add a `Curve25519EncryptionPublicKey` to the *verificationMethod* array:

    ```js
    {
      id: "<did>#<entry-key>",
      type: "X25519KeyAgreementKey2019",
      controller: "<did>",
      publicKeyBase58: "<entry-value-public-key-base58btc-encoded>"
    }
    ```

  * Add a `X25519KeyAgreementKey2019` to the *keyAgreement* array:

    ```js
    {
      id: "<did>#<entry-key>",
      type: "X25519KeyAgreementKey2019",
      controller: "<did>",
      publicKeyBase58: "<entry-value-public-key-base58btc-encoded>"
    }
    ```

##### 3IDv0 genesis

Since the content of the Ceramic document for a 3IDv0 is empty at the *GenesisCommit* we create the DID document by taking the public keys in the 3IDv0 genesis object (see [Appendix A](#appendix-a)) and convert them to multicodec public keys (one `secp256k1` and one `x25519`). The key properties in the DID document should be constructed as above, with the *entry-key* being the last *15* characters of the multicodec encoded keys.

##### DID Document Metadata

When resolving the DID document [DID Document Metadata](https://w3c.github.io/did-core/#did-document-metadata) should be provided. When resolving a 3ID we should populate the following fields:

* `create` - should be populated using the blockchain timestamp from the first *AnchorCommit*
* `updated` - should be populated using the blockchain timestamp from the most recent *AnchorCommit*
* `versionId` - should be equal to the commit CID from the most recent *AnchorCommit*

#### Resolving using the `versionId` parameter

When the `versionId` query parameter is given as a DID is resolved it means that we should try to resolve a specific version of the DID document. The resolution process is the same except that the *AnchorCommit* we use to get the content of the document should be equal to the DocID + CID from `versionId`. In addition we should construct the *DID Document Metadata* differently.

##### DID Document Metadata

* `create` - should be populated using the blockchain timestamp from the first *AnchorCommit*
* `updated` - should be populated using the blockchain timestamp from the resolved *AnchorCommit*
* `versionId` - should be equal to the commit CID from the resolved *AnchorCommit*
* `nextUpdate` - should be populated using the blockchain timestamp from the next *AnchorCommit* (if present)
* `nextVersionId` - should be equal to the commit CID from the next *AnchorCommit* (if present)

### Update

The 3ID DID can be updated by changing the content of the Ceramic document corresponding the particular 3ID. Any number of public key can be added or removed from the document content. Note that the *controller* of the Ceramic document can be changed as well. This does not have any effect on the state of the DID document, but changes the DID which is in control of the 3ID document.

### Deactivate

The 3ID can be deactivated by removing all content in the Ceramic document of the 3ID and replacing it with one property `deactivated` set to `true`.

## Security Requirements

3ID derives most of its security properties from the Ceramic protocol. Most notably *censorship resistance*, *decentralization*, and requiring a minimal amount of data to be synced to completely verify the integrity of a 3ID. For more details see the Ceramic [specification](https://github.com/ceramicnetwork/ceramic/blob/master/SPECIFICATION.md).

### Cryptographic Agility

As can be seen in the CRUD section, currently only `secp256k1` and `x25519` public keys are supported. This can be easily extended by using other multicodec encoded keys. The [multicodec table](https://github.com/multiformats/multicodec/blob/master/table.csv) already has support for BLS keys for example, so adding support for it would be trivial. Once good post quantum cryptography becomes more widely available extending 3ID to support that will also be fairly straight forward.

## Privacy Requirements

See [§ 10. Privacy Considerations](https://www.w3.org/TR/did-core/#privacy-considerations) in `did-core`.

## Extensibility

 The 3ID DID Method could also easily be extended to support other features specified in  [did-core](https://w3c.github.io/did-core/), e.g. service endpoints.

## Reference Implementations

* [3id-did-provider](https://github.com/ceramicstudio/js-3id-did-provider) - wallet side implementation of the 3ID DID method
* [3id-did-resolver](https://github.com/ceramicnetwork/js-ceramic/tree/develop/packages/3id-did-resolver) - 3ID DID method resolver

## Appendix A

An example 3IDv0 genesis object


```json
{
  "value": {
    "id": "did:3:GENESIS",
    "@context": "https://w3id.org/did/v1",
    "publicKey": [
      {
        "id": "did:3:GENESIS#signingKey",
        "type": "Secp256k1VerificationKey2018",
        "publicKeyHex": "0452fbcde75f7ddd7cff18767e2b5536211f500ad474c15da8e74577a573e7a346f2192ef49a5aa0552c41f181a7950af3afdb93cafcbff18156943e3ba312e5b2"
      },
      {
        "id": "did:3:GENESIS#encryptionKey",
        "type": "Curve25519EncryptionPublicKey",
        "publicKeyBase64": "DFxR24MNHVxEDAdL2f6pPEwNDJ2p0Ldyjoo7y/ItLDc="
      },
      {
        "id": "did:3:GENESIS#managementKey",
        "type": "Secp256k1VerificationKey2018",
        "ethereumAddress": "0x3f0bb6247d647a30f310025662b29e6fa382b61d"
      }
    ],
    "authentication": [
      {
        "type": "Secp256k1SignatureAuthentication2018",
        "publicKey": "did:3:GENESIS#signingKey"
      }
    ]
  }
}
```


## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
