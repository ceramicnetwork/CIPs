---
cip: 7
title: CAIP-10 Link
author: Joel Thorstensson (@oed), Michael Sena (@michaelsena)
discussions-to: https://github.com/ceramicnetwork/CIP/issues/15
status: Last Call
category: Standards
type: RFC
created: 2020-07-24
requires: Doctypes (CIP-5)
---

## Simple Summary

This CIP describes the CAIP-10 Link stream type which is used to create a public link between a blockchain account (formatted as a [CAIP-10](https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-10.md) accountId) and a DID. This enables the owner of an account to associate it with a DID. 


## Abstract

The CAIP-10 Link stream type enables the linking of a blockchain account to a DID. Blockchain accounts are represented using [CAIP-10](https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-10.md) accountIds, which is a cross-chain standard for referencing blockchain accounts. The stream type allows anyone with knowledge of a particular CAIP-10 accountId to look up which DID it is associated with using the Ceramic network. To make such a lookup, anyone can simply recreate the *genesis commit*, which only contains the CAIP-10 accountId as unique data, to get the *streamId* of the particular stream and then look it up in the network. 


## Motivation

While it's always possible to associate a blockchain account with some external identifier (such as a DID) by submitting a transaction on-chain, doing so can be expensive due to transaction costs and likely needs to be done in a single registry so lookups can occur. Ceramic provides a better infrastructure for making these public associations. 

In many cases, blockchain accounts are just simple cryptographic key pairs and can create an association with a DID only using a signature. The CAIP-10 Link stream type provides this with the addition of strict ordering, which allows an account to be securely relinked to a different DID at a later point in time. 

For smart contract based accounts, the association can happen in different ways depending on the implementation of the contract. At times this can happen on-chain or off-chain. A particular example is [ERC1271](https://eips.ethereum.org/EIPS/eip-1271) which is a standard for contract based "signatures". CAIP-10 Link can be extended to support any number of these use cases. 


## Specification

Below the commit formats and state transitions of the CAIP-10 Link stream type are specified. Together they should include all information needed to implement this stream type in Ceramic.

#### StreamID code: `0x01`

### Commit formats

The **genesis commit** is stored in IPLD using the *dag-cbor* codec. It has the following required properties:

* `owners` - one blockchain account encoded as CAIP-10

No other properties are allowed to be defined.

**Signed commits** of a CAIP-10 Link stream are also encoded using *dag-cbor*. Other than that they conform to the standard commit format. The content of the `data` property should be a *proof* as returned by the [3id-blockchain-utils](https://github.com/3box/js-3id-blockchain-utils) library. This library has a general structure for creating and verifying "link proofs" for any blockchain. 

Note that because the signature in included in the *proof* object, the rest of the *signed commit* is not authenticated. This could pose a potential problem, but can be mitigated if the proof timestamp is verified as described below.

### State transitions

Below the state transition logic for the three different supported commits is described. 

#### Applying the Genesis Commit

When the genesis commit is applied a `DocState` object is created that looks like this:

```js
{
  metadata: {
    owners: genesisCommit.owners
  },
  content: null,
  signature: SignatureStatus.GENESIS,
  anchorStatus: AnchorStatus.NOT_REQUESTED,
  log: [new CID(genesisCommit)]
}
```

In addition to this, the following should be verified:

* `owners` is a CAIP-10 AccountId

#### Applying a Signed Commit

When a signed commit is applied the `DocState` object should be modified in the following way:

```js
state.next = {
  content: verifiedProof.DID
}
state.signature = SignatureStatus.SIGNED
state.anchorStatus = AnchorStatus.NOT_REQUESTED
state.log.push(new CID(signedCommit))
```

Where `verifiedProof.DID` is the DID contained in the proof object from `signedCommit.data`. In addition it needs to be verified that this proof was signed by the CAIP-10 accountId that is represented in `owners`. If the variable `state.anchorProof` is present the implementation MUST also verify that `proof.timestamp` is larger than `state.anchorProof.blockTimestamp`.
For an example of how this verification can take place have a look in the [3id-blockchain-utils](https://github.com/3box/js-3id-blockchain-utils) library. 

#### Applying an Anchor Commit

When an anchor commit is applied the `DocState` object should be modified in the following way:

```js
state.anchorStatus = AnchorStatus.ANCHORED
state.anchorProof = anchorCommit.proof
state.content = state.next.content
state.metadata = Object.assign(state.metadata, state.next.metadata)
delete state.next
state.log.push(new CID(anchorCommit))
```

No additional validation needs to happen at this point in the state transition function.

### Examples

When resolving an caip10-link stream that is linked to a 3ID, the result looks like the examples below.

#### Ethereum account-link

```js
{
  metadata: {
    owners: ["0xab16a96d359ec26a11e2c2b3d8f8b8942d5bfcdb@eip155:1"]
  },
  content: "did:3:bafyreiecedg6ipyvwdwycdjiakhj5hiuuutxlvywtkvckwvsnu6pjbwxae"
}
```

#### Polkadot account-link

```JS
{
  metadata: {
    owners: ["5hmuyxw9xdgbpptgypokw4thfyoe3ryenebr381z9iaegmfy@polkadot:b0a8d493285c2df73290dfb7e61f870f"],
  },
  content: "did:3:bafyreiecedg6ipyvwdwycdjiakhj5hiuuutxlvywtkvckwvsnu6pjbwxae"
}
```

#### Cosmos account-link

```JS
{
  metadata: {
  	owners: ["cosmos1t2uflqwqe0fsj0shcfkrvpukewcw40yjj6hdc0@cosmos:cosmoshub-3"],
  },
  content: "did:3:bafyreiecedg6ipyvwdwycdjiakhj5hiuuutxlvywtkvckwvsnu6pjbwxae"
}
```

#### Bitcoin account-link

```JS
{
  metadata: {
  	owners: ["128Lkh3S7CkDTBZ8W7BbpsN3YYizJMp8p6@bip122:000000000019d6689c085ae165831e93"],
  },
  content: "did:3:bafyreiecedg6ipyvwdwycdjiakhj5hiuuutxlvywtkvckwvsnu6pjbwxae"
}
```


## Rationale

The CAIP-10 Link allows blockchain accounts to be linked to DIDs by signing a simple message. The signature is only over this message itself and not the entire *signed commit*. This provides less strong security guarantees and instead opts for user experience, since the user will be able to read the message and have a better understanding of what is going on. The use of CAIP-10 future proofs this stream type by allowing any blockchain account to be linked to a DID by extending the implementation of the stream type with additional verification methods. 


## Implementation

A reference implementation of the CAIP-10 Link stream type is provided in [js-ceramic](https://github.com/ceramicnetwork/js-ceramic). (see the `doctype-caip10-link` package).


## Security Considerations

Since the entire payload of the *signed commit* is not included in the payload of the signature used for this stream type, care needs to be taken so that the stream can not be attacked by a replay attack. For example, lets imagine that account `0xabc123@eip155:1` is first linked to `did:3:A`, then after that is updated to link to `did:3:B`. A malicious attacker could now take the proof from the first linkage and update the stream with a new *signed commit* which points to the latest update but contains the first proof link object. The account would now be linked to `did:3:A` once again which is not desirable. 

To circumvent this, each proof contains a `timestamp` which is the unix timestamp of when the message was signed. If we simply verify that this timestamp is larger than the timestamp of the most recent *anchor commit* we can be sure that this is not a reply attack. Further attacks could possibly include modification of the `refs` property, but this could in the worst case just make syncing of the stream slower.


## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).