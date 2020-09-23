---
cip: 59
title: DocID encoding
author: Joel Thorstensson (@oed)
discussions-to: https://github.com/ceramicnetwork/CIP/issues/59
status: Last call
category: Standards
type: Core
created: 2020-08-12
edited: 2020-09-24
---

## Simple Summary

Specification for how to encode a Document Id (DocID)


## Abstract
A DocID is composed of a docid-multicodec, a subnet varint, a doctype varint, and a CID.


## Motivation
A specific encoding for DocIDs allows us to distinguish them from CIDs as well as provide more information about the given document. 


## Specification
DocIDs are defined as:

```html
<docid> ::= <multibase-prefix><multicodec-docid><subnet><doctype><genesis-cid-bytes>

# e.g. using CIDv1
<docid> ::= <multibase-prefix><multicodec-docid><subnet><doctype><multicodec-cidv1><multicodec-content-type><multihash-content-address>
```

Where

- `<multibase-prefix>` is a [multibase](https://github.com/multiformats/multibase) code (1 or 2 bytes), to ease encoding DocIDs into various bases. **NOTE:** *Binary* (not text-based) protocols and formats may omit the multibase prefix when the encoding is unambiguous.
- `<multicodec-docid>` is a [multicodec](https://github.com/multiformats/multicodec) used to indicate that it's a DocID.
- `<subnet>` is a [varint](https://github.com/multiformats/unsigned-varint) representing subnet of the document, used to determine which sub network the document belongs to.
- `<doctype>` is a [varint](https://github.com/multiformats/unsigned-varint) representing the doctype of the document.
- `<genesis-cid-bytes>` is the bytes from the [CID](https://github.com/multiformats/cid) of the *genesis record*,  stripped of the multibase prefix.

### DocID multicodec
The multicodec for DocID is `0xce`

### Recommendations 
For compatibility with browser urls it's recommended to encode the DocID using [`base32`](https://github.com/multiformats/multibase).

### Registered values
Using the tables linked below doctypes and subnets can be be publicly registered by submitting a new CIP that adds it to one of the tables.

* [Doctypes table](./tables/doctypes.csv)
* [Subnets table](./tables/subnets.csv)

## Rationale
A Ceramic document can be identified using the CID of the *genesis record* of the document log, as well as the subnet and doctype of the document. Using the DocID a Ceramic node can query the network for the latest *tip* of the document. 

The subnet is used to uniquely identify a subnet on the Ceramic network. Initially this is used to determine if a document has been created on the testnet (`0`) or the livenet (`1`). In the future more subnets may be defined.

By having the doctype in the DocID, Ceramic implementations can be optimize to use the correct doctype handler before needing to sync parts of the document.


## Backwards Compatibility
Previously testnet versions of Ceramic has been using CIDs to represent DocIDs. This is a breaking change to only allow the DocID format specified above.


## Implementation
<!--The implementations must be completed before any CIP is given status "Final", but it need not be completed before the CIP is accepted.-->
No implementation yet.


## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).