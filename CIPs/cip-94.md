---
cip: 94
title: NFT DID Method Specification
author: Joel Thorstensson (@oed)
discussions-to: https://github.com/ceramicnetwork/CIP/issues/95
status: Draft
category: RFC
created: 2021-02-12
updated: 2023-04-21
---

# NFT DID Method Specification

The NFT DID Method converts any non-fungible token on any blockchain into a decentralized identifier where the owner of the NFT is the controller of the DID. This is achieved by using the *Chain Agnostic Improvement Proposals* to describe NFT assets. The NFT DID method is a purely generative DID method. It's meant to be used together with [ChainProofs](https://github.com/ChainAgnostic/CAIPs/pull/218) in order to create an object capability that delegates control from the NFT to the current owner of the NFT.

## DID Method Name

The name string that shall identify this DID method is: `nft`.

A DID that uses this method MUST begin with the following prefix: `did:nft`. Per the [DID specification](https://w3c.github.io/did-core/), this string MUST be in lowercase. The remainder of the DID, after the prefix, is specified below.

## Method Specific Identifier

The method specific identifier is simply a [CAIP-19 Asset ID](https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-19.md) where all `/` characters are replaced by `-`, in order to comply with the DID spec.

### Example

In the example below we see an [ENS name](https://ens.domains/), specifically *jthor.eth*, being used as a DID. ENS is an ERC721 token on Ethereum and we can refer to it using the ERC721 asset namespace as defined in [CAIP-22](https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-22.md).

```sh
did:nft:eip155:1-erc721:0x57f1887a8bf19b14fc0df6fd9b2acc9af147ea85-96161218729996353997094449792539040731415990220743397894295225315809852060977
```

## CRUD Operation Definitions

In this section the CRUD operations for an NFT DID are defined.

### Create

Mint an NFT on any blockchain.

### Read/Verify

Extract the Asset ID from the method specific identifier. 

#### Ethereum ERC721

Given a DID with the `erc721` token namespace,

`<did> = did:nft:eip155:1-erc721:<nft-contract>-<token-id>`

the resulting DID document should be created as follows,

```jsonc
{
  "@context": "https://w3id.org/did/v1",
  "id": "<did>",
  "verificationMethod": [{
    "id": "<did>#0",
    "type": "ChainProofCAIP218",
    "controller": "<did>",
    "evmCall": "ownerOf(<token-id>): address"
  }],
  "authentication": ["<did>#0"],
  "assertionMethod": ["<did>#0"],
  "capabilityDelegation": ["<did>#0"],
  "capabilityInvocation": ["<did>#0"]
}
```

#### Ethereum ERC1155

Given a DID with the `erc1155` token namespace,

`<did> = did:nft:eip155:1-erc1155:<nft-contract>-<token-id>`

the resulting DID document should be created as follows,

```jsonc
{
  "@context": "https://w3id.org/did/v1",
  "id": "<did>",
  "verificationMethod": [{
    "id": "<did>#0",
    "type": "ChainProofCAIP218",
    "controller": "<did>",
    "evmCall": "balanceOf(address, <token-id>): uint256"
  }],
  "authentication": ["<did>#0"],
  "assertionMethod": ["<did>#0"],
  "capabilityDelegation": ["<did>#0"],
  "capabilityInvocation": ["<did>#0"]
}
```

### Update

Transfer the NFT to another account.

### Deactivate

Transfer the NFT to an account that no one controls, i.e. burn it.

## Security Requirements

The NFT DID derives most of its security properties from the blockchain which the given NFT lives on. The security of different blockchains may vary so implementers may choose to limit their implementations to specific NFT assets.

## Privacy Requirements

NFT DID utilizes a few components. First, blockchains provide a public, immutable audit trail of all previous owners of an NFT asset; these owners are blockchain accounts. Since NFTs are tradable, resources controlled by an NFT DID should not be seen as having been granted to one individual. Instead, access granted to an NFT DID becomes more fluid and changes as on-chain ownership changes.


## Extensibility

The NFT DID Method could currently supports ERC721 and ERC1155 tokens, but could be easily extended to support any other NFT token givent that the token is registered as a [CAIP asset namespace](https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-19.md).

## Implementations

* [nft-did-resolver](https://github.com/ceramicnetwork/nft-did-resolver/pull/1) (needs update) - javascript implementation supporting ERC721 and ERC1155 NFTs

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
