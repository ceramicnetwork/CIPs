---
cip: 59
title: StreamID encoding
author: Joel Thorstensson (@oed)
discussions-to: https://github.com/ceramicnetwork/CIP/issues/59
status: Final
category: Core
created: 2020-08-12
edited: 2020-09-24
---

## Simple Summary

Specification for how to encode a Stream Id (StreamID)


## Abstract
A StreamID is composed of a streamid-multicodec, a stream type varint, and a CID.


## Motivation
A specific encoding for StreamIDs allows us to distinguish them from CIDs as well as provide more information about the given stream. CommitIDs are a subset of a StreamID which refers to a specific commit in a stream.


## Specification
#### StreamID

StreamIDs are defined as:

```html
<streamid> ::= <multibase-prefix><multicodec-streamid><stream-type><genesis-cid-bytes>

# e.g. using CIDv1
<streamid> ::= <multibase-prefix><multicodec-streamid><stream-type><multicodec-cidv1><multicodec-content-type><multihash-content-address>
```

Where

- `<multibase-prefix>` is a [multibase](https://github.com/multiformats/multibase) code (1 or 2 bytes), to ease encoding StreamIDs into various bases. **NOTE:** *Binary* (not text-based) protocols and formats may omit the multibase prefix when the encoding is unambiguous.
- `<multicodec-streamid>` is a [multicodec](https://github.com/multiformats/multicodec) used to indicate that it's a StreamID.
- `<stream-type>` is a [varint](https://github.com/multiformats/unsigned-varint) representing the stream type of the stream.
- `<genesis-cid-bytes>` is the bytes from the [CID](https://github.com/multiformats/cid) of the *genesis record*,  stripped of the multibase prefix.

#### CommitID

The CommitID adds some additional information at the end. If it represents the genesis commit the zero byte is added (`0x00`) otherwise the CID that represents the commit is added.

CommitIDs are defined as:

```html
<streamid> ::= <multibase-prefix><multicodec-streamid><stream-type><genesis-cid-bytes><commit-reference>

# e.g. using CIDv1 and representing the genesis commit
<streamid> ::= <multibase-prefix><multicodec-streamid><stream-type><multicodec-cidv1><multicodec-content-type><multihash-content-address><0x00>

# e.g. using CIDv1 and representing the an arbitrary commit in the log
<streamid> ::= <multibase-prefix><multicodec-streamid><stream-type><multicodec-cidv1><multicodec-content-type-1><multihash-content-address-1><multicodec-cidv1><multicodec-content-type-2><multihash-content-address-2>

```

Where

- `<commit-reference>` is either the zero byte (`0x00`) or a [CID](https://github.com/multiformats/cid).

### StreamID multicodec
The multicodec for StreamID is `0xce`

### Recommendations 
For compatibility with browser urls it's recommended to encode the StreamID using [`base36`](https://github.com/multiformats/multibase).

### Registered values
Using the table linked below stream types can be be publicly registered by submitting a new CIP that adds it to the table.

* [StreamType table](../assets/streamtypes-table.csv)

## Rationale
A Ceramic stream can be identified using the CID of the *genesis commit* of the event log, as well as the stream type of the stream. Using the StreamID a Ceramic node can query the network for the latest *tip* of the stream. 

By having the stream type in the StreamID, Ceramic implementations can be optimize to use the correct stream type handler before needing to sync parts of the event log.


## Backwards Compatibility
Previously testnet versions of Ceramic has been using CIDs to represent StreamIDs. This is a breaking change to only allow the StreamID format specified above.


## Implementation
* [js-ceramic](https://github.com/ceramicnetwork/js-ceramic)


## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
