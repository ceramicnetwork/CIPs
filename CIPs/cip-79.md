---
cip: 79
title: 3 DID Method Specification
author: Joel Thorstensson (@oed)
discussions-to: https://github.com/ceramicnetwork/CIP/issues/80
status: Draft
category: Standards
type: Core
created: 2021-02-12
updated: 2023-01-13
---

## Simple Summary

<!--Provide a simplified and layman-accessible explanation of the CIP.-->
The 3 DID method enables users to compose multiple accounts into a signle identifier.


## Abstract

<!--A short (~200 word) description of the technical issue being addressed.-->
With a special type of Ceramic stream 3ID enables a revocation registry that can be used to add and remove verification methods from the 3 DID method as well as revoke any capability issued by the identifier. The revocation registry is based on hashes of object-capabilities encoded as CACAO.


## Motivation

<!--Motivation is critical for CIPs that want to change the Ceramic protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the CIP solves. CIP submissions without sufficient motivation may be rejected outright.-->
Currently in Ceramic the main accounts types are PKH DID. These are great because they enable existing wallets to be used directly with Ceramic. However, these accounts are inherently tied to a specific account and there is no way to rotate keys in a simple manner. The 3 DID method changes this by defining a DID method which works with object-capabilities and supports all DID CRUD operations.

Furthermore, the revocation regsitry of the 3 DID method enables the DID subject to revoke any object-capability issued by the 3ID. This can be useful in case of a key compromise, e.g. for a temporary session key.

## Specification

<!--The technical specification should describe the syntax and semantics of any new feature.-->
Specification goes here.

### The did:3 identifier

> ```
> did:3:<cacao-mh>
> ```

The `<cacao-mh>` value is a `keccak-256` multihash over the CID of a CACAO object-capability, e.g. ReCap or UCAN. The value MUST be encoded as a multibase string, using `base58btc`.

***Simple example:***

> ```
> did:3:z9CEZ1N1gbFK8J3rxVw3o6M5wygjoNFRSaEtkoGZw5fmbj
> ```

### CRUD Operation Definitions

3IDs are created, updated, and deactivated by creating, notarizing, and revoking object-capabilities in the form of CACAOs.

#### Create

Create and sign a [SIWx](https://chainagnostic.org/CAIPs/caip-122) message where:

- `address` - a blockchain address (e.g. `0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2`)
- `uri` - the PKH DID for the same address (e.g. `did:pkh:eip155:1:0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2`)

And resources contains the following ReCap object:

```java
{
  "att": {
    "did:3:new": { "3id/control": [] }
  }
}
```

- *When signing this would look something like this for the user:*

  ```
  example.com wants you to sign in with your Ethereum account:
  0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2
  
  I further authorize https://example.com to perform the following 
  actions on my behalf: (1) "3id/control" for "did:3:new".
  
  URI: did:pkh:eip155:1:0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2
  Version: 1
  Chain ID: 1
  Nonce: n-0S6_WzA2Mj
  Issued At: 2022-06-21T12:00:00.000Z
  Resources:
  - urn:recap:example:eyJkZWYiOlsicmVhZCJdLCJ0YXIiOnsibXkucmVzb3VyY2UuMSI6WyJhcHBlbmQiLCJkZWxldGUiXSwibXkucmVzb3VyY2UuMiI6WyJhcHBlbmQiXSwibXkucmVzb3VyY2UuMyI6WyJhcHBlbmQiXX19
  ```

##### Create the DID

```jsx
cacaoCID = ipld.put(CACAO(SIWx-message, signature))
did = 'did:3:' + multihash(cacaoCID)
```

#### Read

To read you must load a deterministic Ceramic stream to find any updates made to the 3ID.

1. Load the revocation registry Ceramic stream

   1. If the `?versionTime=<timestamp>` is specified load only the events `≤ timestamp`

2. Read all entries where `isRevoked = false`

3. For every CACAO multihash in the registry, as well as the 3ID identifier,

   1. Add the following object to the `verificationMethod`  property of the DID document,
   
      ```json
      {
        "id": "#<cacao-mh>",
        "type": "capabilityHash",
        "controller": "did:3:<cacao-mh>",
        "multihash": "<cacao-mh>"
      }
      ```
   
   2. Add `"#<cacao-mh>"` to the following fields,
   
      1. `authentication`
      2. `assertionMethod`
      3. `capabilityDelegation`
      4. `capabilityInvocation`

##### Example DID Document

For `did:3:z9CEZ1N1gbFK8J3rxVw3o6M5wygjoNFRSaEtkoGZw5fmbj` with no entires in the Revocation registry,

```json
{
  "@context": [
    "<https://www.w3.org/ns/did/v1>",
  ]
  "id": "did:3:z9CEZ1N1gbFK8J3rxVw3o6M5wygjoNFRSaEtkoGZw5fmbj",
  "verificationMethod": [{
    "id": "#z9CEZ1N1gbFK8J3rxVw3o6M5wygjoNFRSaEtkoGZw5fmbj",
    "type": "capabilityHash",
    "controller": "did:3:z9CEZ1N1gbFK8J3rxVw3o6M5wygjoNFRSaEtkoGZw5fmbj",
    "multihash": "z9CEZ1N1gbFK8J3rxVw3o6M5wygjoNFRSaEtkoGZw5fmbj"
  }],
  "authentication": [ "#z9CEZ1N1gbFK8J3rxVw3o6M5wygjoNFRSaEtkoGZw5fmbj" ],
  "assertionMethod": [ "#z9CEZ1N1gbFK8J3rxVw3o6M5wygjoNFRSaEtkoGZw5fmbj" ],
  "capabilityDelegation": [ "#z9CEZ1N1gbFK8J3rxVw3o6M5wygjoNFRSaEtkoGZw5fmbj" ],
  "capabilityInvocation": [ "#z9CEZ1N1gbFK8J3rxVw3o6M5wygjoNFRSaEtkoGZw5fmbj" ],
}
```

#### Update

##### To add a verification method:

1. Create a new ReCap capability with an existing VM as the issuer, delegating `"3id/control"` to another DID (PKH or Key DID), and encode it as a CACAO.

   ***ReCap***

   ```json
   {
     "att": {
       "did:3:z9CEZ1N1gbFK8J3rxVw3o6M5wygjoNFRSaEtkoGZw5fmbj": { "3id/control": [] }
     }
   }
   ```

   - *When signing this would look something like this for the user:*

     ```
     example.com wants you to sign in with your Ethereum account:
     0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2
     
     I further authorize https://example.com to perform the following 
     actions on my behalf: (1) "3id/control" for "did:3:z9CEZ1N1gbFK8J3rxVw3o6M5wygjoNFRSaEtkoGZw5fmbj".
     
     URI: did:pkh:eip155:1:0xa05bba39b223fe8d0a0e5c4f27ead9083cb22ee3
     Version: 1
     Chain ID: 1
     Nonce: n-0S6_WzA2Mj
     Issued At: 2022-06-21T12:00:00.000Z
     Resources:
     - urn:recap:example:eyJkZWYiOlsicmVhZCJdLCJ0YXIiOnsibXkucmVzb3VyY2UuMSI6WyJhcHBlbmQiLCJkZWxldGUiXSwibXkucmVzb3VyY2UuMiI6WyJhcHBlbmQiXSwibXkucmVzb3VyY2UuMyI6WyJhcHBlbmQiXX19
     ```

2. Load the *Revocation Registry* stream

3. Update (2) by adding the `cacao-mh` from (1)

***To remove a verification method:***

1. Load the *Revocation Registry* stream
2. Update (1) by removing the `cacao-mh` that is being revoked

#### Deactivate

In order to deactivate a 3ID all capabilities will need to be revoked.

### Revocation Registry

3ID is based on a self-certifying revocation registry represented as a special type of Ceramic stream. In it any verification method belonging to the 3ID can be registered and revoked. Every update to the registry is recorded as a *Data Event*. When a new event is created is needs to include a valid delegation chain back to previous version of the registry.

```verilog
type Prinicipal Bytes // multidid
type CACAOHash Bytes // multihash
type Varsig Bytes // varsig

type RevocationEntry struct {
  key CACAOHash
  isRevoked Boolean
} representation tuple
// The registry should eventually be a HAMT or similar data structure
type Regsitry { CACAOHash : RevocationEntry }
type Snapshot struct {
  registry &Regsitry
  actions [RevocationEntry]
}
```

#### Ceramic Stream

```verilog
type Event InitEvent | DataEvent | TimeEvent

type InitEvent &Prinicipal // an inline CID containing raw principal bytes

type DataEvent struct {
  id &InitEvent
  prv [&Event] // is prv needed if it's the first event after genesis?
  prf [&CACAO] // capabilities used to emit this event
  data &Snapshot
  sig Varsig
}

type EthereumTx // <https://ipld.io/specs/codecs/dag-eth/chain/#transaction-ipld>

type BlockchainTimestamp struct {
  root Link
  chainID String
  txType String
  txHash &EthereumTx
}

type TimeEvent struct {
  id &InitEvent
  prv [&DataEvent] // should always be one CID
  proof &BlockchainTimestamp
  path String
}
```

#### Streamid

Generating the Ceramic streamid can be done in three steps,

1. Generate the multidid representation of the 3ID,

   ```solidity
   	<multidid> := <varint multidid><cacao-mh><varint 0x00>
   ```

2. Encode the multidid as an inline CID,

   ```solidity
   	<genesis-cid> := <varint cidv1><varint 0x55><varint 0x00><varint multidid-length><multidid>
   ```

3. Encode the Streamid

   ```solidity
   	<streamid> := <varint 0xce><varint stream-type><genesis-cid>
   ```

#### Create a `DataEvent`

New data events

1. Attain a key with a valid capability chain to the most recent *previous data event(s)*

2. Create one or more `RevocationEntry` objects and update the `State` from the 

   previous data event(s):

   1. Write the objects to a new array and store them on the `actions` field
   2. Also append them to the `registry` map

3. Create the `DataEvent` struct,

   1. Set `id` to the principal (an inline CID containing a multidid encoded 3ID),
   2. Add the capability chain used to the `prf` field
   3. Add the previous events to the `prv` field
   4. Add the updated state to the `data` field
   5. Create a `Varsig` over the `DataEvent` with the key from (1) and add the `sig` field

#### Validating a `DataEvent`

The certification of a `DataEvent` can be validated using the following algorithm,

1. The varsig validates agains the *aud* `Principal` of the referenced CACAO

2. The multihash of the CACAO CID (caphash) is one of:

   1. CapHash is in the `Registry` and `isRevoked` is false
   2. CapHash is in the `Principal` and **not** in the `Registry`

3. The CACAO *ability* is one or both of:

   1. `"3id/add"` - the `Registry` is only allowed to grow and, new values must have `isRevoked = false`
2. `"3id/remove"` - the `Registry` can either grow or stay the same size, all modified values must have `isRevoked = true`

#### Consensus

In case of two conflicting events (two events share the same `prv` value) the event with the earliest `TimestampEvent` should be processed first. Note that this might lead to the latter event being invalid due to its delegation chain being revoked. Also, a new event emitted after the conflict must reference both branches in its `prv` and resolve any conflict of the `Registry`.

If there is no anchor for either event yet, the `DataEvent` with the lowest binary value of its CID will win. Note that if a `TimeEvent` appears this order might change.

#### Verified timestamps

For convenience, once a `TimeEvent` has been verified the data used to verify it can be stored as a receipt. This is helpful when resolving the registry at a particular point in time using the `?versionTime=<iso-time>` DID URL parameter.

```verilog
type EthereumHeader // <https://ipld.io/specs/codecs/dag-eth/chain/>

type TimestampRecipt struct {
  unixtime Integer // same as block.Time
  event &TimestampEvent
  block &EthereumHeader
  path String // ipld path in the block to find event.txHash
}
```

#### Dereferencing the Registry

The 3 DID resolver can [dereference](https://wiki.trustoverip.org/display/HOME/DID+URL+Resource+Parameter+Specification) the registry to a [CAR file](https://ipld.io/specs/transport/car/carv1/) which can be independently verified. This is achieved by using  the `did:3:<cap-hash>?resource=vnd.ipld.car` DID URL. Note that the `versionTime` may be used to resolve an earlier snapshot of the registry.

### Object Capabilities

The 3 DID method relies heavily on object-capabilities as they way to add and remove verification methods, as well as delegating permissions. The root capability is `did/control`, any verification method with this capability has [independent control](https://www.w3.org/TR/did-core/#independent-control) over the DID. This means that they can take any action on behalf of this DID, `*/*` is thus a equivalent of `did/control`. The `did/vm-add` and `did/vm-remove` are actions that can be taken to update the 3ID stream and thus the DID document. In the future other more specific `did/...` capabilities my be specified to define more granular actions on the DID document.

```
                ┌─────────────┐
                │ did/control │
                └──▲───▲───▲──┘
      ┌────────────┘   │   └────────┐
┌─────┴──────┐ ┌───────┴───────┐ ┌──┴──┐
│ did/vm-add │ │ did/vm-remove │ │ */* │
└────────────┘ └───────────────┘ └─────┘
```

#### Updating the 3ID stream

In order to make an update to the 3ID stream, one of the verification methods could delegate the following permission to a session key:

```json
{
   "att":{
      "did:3:z9CEZ1N1gbFK8J3rxVw3o6M5wygjoNFRSaEtkoGZw5fmbj": {
        "did/vm-add": [],
        "did/vm-remove": []
      }
			// CID of the event to append to the 3ID stream
   },
   "prf": [
       { "/": "bafybeigk7ly3pog6uupxku3b6bubirr434ib6tfaymvox6gotaaaaaaaaa" }
       // The CID of the capability that grants the key control over the 3ID
       // From this the 3ID can be found, resolve revreg to see if self cap is revoked
   ]
}
```

#### Using a 3ID in Ceramic

In order to act on behalf of a 3ID that controls a Ceramic stream, a delegation can be created by one of the verification methods in the 3ID. Note that the `prf` field must contain a link to the capability for the verification method.

```json
{
   "tar":{
      "ceramic://*?model=zkfif...": {
        "crud/mutate": [],
        "crud/read": []
      }
   },
   "prf": [
       { "/": "bafybeigk7ly3pog6uupxku3b6bubirr434ib6tfaymvox6gotaaaaaaaaa" }
       // The CID of the capability that grants the key control over the 3ID
   ]
}
```

## Rationale

<!--The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->

While most DID methods include full public keys, or other DIDs as verification methods 3ID chooses to use hashes of object-capabilities as verification methods. One of the main use cases for 3ID is to connect multiple PKH DIDs into a single identifier on Ceramic. While doing so publically is fine for some use cases, the ability to do so privately is quite important for many. Using *capability hashes* enables more privacy since the DIDs that are delegated to doesn't strictly need to be revealed. It is worth noting that revealing the CACAO object when used is the simplest way to prove a capability chain. However, it is possible to create zero-knowledge proofs that only reveal the hash of the capability used and which session key was delegated to.

### Cryptographic Agility

The 3 DID method itself doesn't really limit what cryptography can be used. It boils down to what the system that interprets the actual object capabilities is capable of. In Ceramic this includes the following, but is not limited to, and can be extended in the future:

* `ed25519`
* `secp256k1`
* `secp256r1`

Once good post quantum cryptography becomes more widely available extending Ceramic to support that will also be fairly straight forward.


## Backwards Compatibility

<!--All CIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The CIP must explain how the author proposes to deal with these incompatibilities. CIP submissions without a sufficient backwards compatibility section may be rejected outright.-->
Previous iterations of 3ID relied on the TileDocument stream type and an interpretation layer above that translated the tile contents to a DID document. That approach had numerious flaws the main one being the impossibility of privacy. This version of the spec deviates from the previous experiment in that there is a special stream type for the 3ID and that it fully relies on object-capabilities as verification methods. This is a great improvement but unfortunately it is not possible to make an upgrade in a backwards compatible manner. Instead all implementers are recommended to follow this specification only.

## Privacy Requirements

The 3 DID method provides a unique privacy enhancement over most other DID methods in that subject only need to reveal hashes of object-capabilities in order to use it. While the simplest way of implementing usage of the system would imply pulicly revealing these object-capabilities, it is indeed possible to create zero-knowledge proofs of the valididty of an object-capability without revealing its content.

## Security Considerations

3ID derives most of its security properties from the Ceramic protocol. Most notably *censorship resistance*, *decentralization*, and requiring a minimal amount of data to be synced to completely verify the integrity of a 3ID. For more details see the Ceramic [specification](https://github.com/ceramicnetwork/ceramic/blob/master/SPECIFICATION.md).

## Reference Implementations

Currently no reference implementation for 3ID exists.

## Appendix A: Registrations

This appendix contains all regsitrations necessary for the 3 DID method.

### Verification method property

**Property name:** `multihash`

The multihash property is used to specify a multibase-encoded multihash. Usually this is a hash over an object-capability.

* [Multihash specification](https://github.com/multiformats/multihash)
* [Multibase specification](https://github.com/multiformats/multibase)

### Verification Method Type

**Type name:** `capabilityHash`

The capabilityHash verification method type indicates that the verification method is the hash of a CACAO. For a signature to be valid under a specific capabilityHash verification method, the proof must show that there is a valid delegation chain from the CACAO that corresponds with the given hash. In the most trivial case this can be achived by revealing the CACAO itself as part of the proof, a verifier can then check the hashed value of the CACAO. However, zero-knowledge proofs may be used to remove the need to reveal the CACAO, improving privacy for the DID subject.

* [CAIP-74: CACAO - Chain Agnostic CApability Object](https://chainagnostic.org/CAIPs/caip-74) 

### Stream type code

**Code:** `5`

### Multidid code

**Code:** `0x1b` - keccak-256

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
