---
cip: 122
title: CapReg - Notary for object-capabilities
author: Joel Thorstensson (@oed)
discussions-to: https://forum.ceramic.network/t/cip-1xx-capreg
status: Draft
category: Standards
type: Core
created: 2023-04-11
updated: 2023-04-11
---

## Simple Summary

<!--Provide a simplified and layman-accessible explanation of the CIP.-->
The CapReg capability registry enables users to notarize and revoke any object-apability associated with their DID.


## Abstract

<!--A short (~200 word) description of the technical issue being addressed.-->
With a special type of Ceramic stream the capability registry enables users to notarize and revoke object-capabilities issued by their DID. The registry is based on hashes of object-capabilities encoded as CACAO.


## Motivation

<!--Motivation is critical for CIPs that want to change the Ceramic protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the CIP solves. CIP submissions without sufficient motivation may be rejected outright.-->

Currently in Ceramic the main accounts types are PKH DID. These are great because they enable existing wallets to be used directly with Ceramic. Once a PKH DID is used to delegate permissions to a session key. That delegation will remain valid until the capability expires. This could be a problem in the case of a stolen session key or a malicious application. By introducing a capability registry these capabilities can be revoked at any time by the main DID (PKH DID in the case above, but this would work for any DID method). A system that uses these object capabilities could refer to the registry to verify that the capability  has not been revoked before it was used.

## Specification

<!--The technical specification should describe the syntax and semantics of any new feature.-->
This specification describes the data structure of the capability registry, its validation and consensus logic, as well as how wallet UX would look like for someone using the registry.


### Capability Registry

The capability registry is based on a self-certifying data structure represented as a special type of Ceramic stream. Each DID has uniquely *one* capability registry. Any object capability issued by this DID can be notarized and revoked in the registry. Every update to the registry is recorded as a *Data Event*.

```verilog
type Prinicipal Bytes // multidid
type CACAOHash Bytes // multihash
type Varsig Bytes // varsig

type Entry struct {
  key CACAOHash 
  revoked Boolean // true if the capability has been revoked
  ocap optional Link // optionally include the CID of the capability
} representation tuple
// The registry should eventually be a HAMT or verkle-tree data structure
type Regsitry { CACAOHash : Entry }
type Snapshot struct {
  registry &Regsitry
  actions [Entry]
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

type BlockchainTimestamp struct { // https://chainagnostic.org/CAIPs/caip-168
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

Generating the streamid can be done in three steps,

1. Generate the multidid representation of the DID (see [Multidid specification]()).

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

2. Create one or more `Entry` objects and update the `State` from the 

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

   1. CapHash is in the `Registry` and `revoked` is false
   2. CapHash is in the `Principal` and **not** in the `Registry`

3. The CACAO *ability* is one or both of:

   1. `"3id/add"` - the `Registry` is only allowed to grow and, new values must have `revoked = false`
2. `"3id/remove"` - the `Registry` can either grow or stay the same size, all modified values must have `revoked = true`

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

### Object Capabilities

The 3 DID method relies heavily on object-capabilities as they way to add and remove verification methods, as well as delegating permissions. The root capability is `did/control`, any verification method with this capability has [independent control](https://www.w3.org/TR/did-core/#independent-control) over the DID. This means that they can take any action on behalf of this DID, `*/*` is thus a equivalent of `did/control`. The `did/vm-add` and `did/vm-remove` are actions that can be taken to update the 3ID stream and thus the DID document. In the future other more specific `did/...` capabilities my be specified to define more granular actions on the DID document.

```
                ┌─────────────┐
                │ crud/* │
                └──▲──▲───▲──┘
      ┌────────────┘  │   
┌─────┴─────┐ ┌───────┴──────┐
│  crud/update  │ │  curd/remove  │
└───────────┘ └──────────────┘
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

### Stream type code

**Code:** `5`

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
