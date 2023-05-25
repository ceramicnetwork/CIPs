---
cip: 69
title: Batched Anchor Data Structure
author: Joel Thorstensson (@oed)
discussions-to: https://github.com/ceramicnetwork/CIP/issues/69
status: Final
category: Core
created: 2020-11-02
edited: 2020-11-02
---

## Simple Summary
This CIP describes the data structure of a blockchain anchor that batches multiple document updates together. 


## Abstract
An anchor service accepts document update anchor request from clients and batches them together in a single merkle tree, which is then anchored into a specific blockchain. The anchor service also sorts all of the requests and creates a bloom filter which describes which documents where updated in this batch, which is also included in the blockchain transaction. This allows external observers to more effectively determine if a document which they are interested in was updated.


## Motivation
There are two main motivations for this CIP. Firstly having an agreed upon spec for the structure of the batched anchor merkle tree makes it easier for all consumers of the ceramic protocol to know what to expect when parsing an *anchor record*. Secondly the sorting of updates and the inclusion of a bloom filter allows indexing services to effectively determine if an update was made which is relevant, and find this update in the tree effectively.


## Specification

First lets define the basic IPLD data structures used.
```ipldsch
type MerkleNode [Link]

type BloomMetadata struct {
  type String
  data {String:any}
}

type TreeMetadata struct {
  numEntries Int
  bloomFilter BloomMetadata
}
```
Here the *MerkleNode* is simply an array of CIDs, which is used as the main building block for the merkle tree. The *TreeMetadata* contains some metadata about the tree, namely the number of entries and a bloom filter. The `bloomFilter.type` is used to describe which specific type of bloom filter is being used and is there for upgradeability purposes.

### Tree structure
The structure of the merkle tree is very straight forward. Starting from the bottom each individual update is paired with another update and a *MerkleNode* is created which has the leftmost update in position `0`, and the other update at position `1`. *MerkleNodes* are then created on each level until we reach the root. The root *MerkleNode* has an additional entry at position `2` which simply points to a *TreeMetadata* object. In the figure below we see a simple tree with four leafs.

![merkletree(1)](https://user-images.githubusercontent.com/3909429/97851426-a83bf900-1cf5-11eb-9d9f-e7b568138850.png)

In this example an *IPLD Path* can be used to reach any of the leafs. For example, to reach *Update 3* we would use the path `<root-cid>/1/0`. Similarly we can always get the *TreeMetadata* using the path `<root-cid>/2`.

The merkle tree should be constructed as a balanced tree. See the [implementation](https://github.com/ceramicnetwork/ceramic-anchor-service/blob/ced4ea9f8c70aa09e6c87e3c1b2de5bbdf505157/src/merkle/merkle-tree.ts#L26-L61) for reference.


### Leaf sorting
Before creating the tree the leafs (updates) should be sorted. The sorting should be based on a few different properties of the Ceramic document that is being updated:

1. `family` - sort by the family in the document metadata
2. `schema` - if multiple documents have the same topic, sort by the *schema*
3. `controllers` - if multiple documents have the same topic and schema, sort by the first controller, then subsequent ones
4. `StreamID` - finally sort by the StreamID


### Bloom filter
The bloom filter should include the following data for each document that is being updated:

* The `family`, prepend each string with `family-`

* The first 5 `tags`, prepend each tag string with `tag-`
* The `schema`, prepend the schema StreamID string with `schema-`
* All DID strings in the `controllers` array, prepend each DID with `controller-`
* The StreamID string of the document, prepend with `streamid-`

The bloom filter is created using the javascript [bloom-filters](https://github.com/Callidon/bloom-filters) library. Specifically using the *Classic Bloom Filter*. An example for how to create the filter can be observed below.

```js
const { BloomFilter } = require('bloom-filters')
const filterEntries = [...] // An array of strings
const errorRate = 0.0001 // .01 % error rate
const filter = BloomFilter.from(items, errorRate)
const exported = filter.saveAsJSON()
```

Now we can create the *TreeMetadata* object, which can be added into ipfs and the CID of which is added to position `2` in the root *MerkleNode*.

```js
{
  numEntries: <number-of-leafs>,
  bloomFilter: { type: 'jsnpm_bloom-filters', data: exported }
}
```


## Rationale
This CIP provides the most efficient merkle tree implementation possible in IPLD DAG-CBOR. The more interesting aspects of this CIP are however the sorting algorithm and the bloom filter implementation.

The sorting allows for more efficient lookup in the merkle tree, primarily based on the family of the updated Ceramic document. Further sorting is also provided by the additional properties. If an implementer knows the family of a document they can do a binary search though the tree to find the correct update.

The bloom filter allows an observer to efficiently determine if any of the given properties are most likely in the merkle tree. Since the filter itself is stored in a separate IPLD object the filter doesn't add any overhead when verifying a merkle witness for a document update. The bloomfilter implementation that was chosen is currently only implemented in javascript, we failed to find an implementation that is compatible with multiple languages. However, since we introduce a `type` property we can easily determine which implementation was used. This also allows us to change the implementation used in the future.


## Backwards Compatibility
These changes should be fully backwards compatible with the current implementation of Ceramic.


## Implementation
* [Ceramic Anchor Service](https://github.com/ceramicnetwork/ceramic-anchor-service) - Not started yet.


## Security Considerations
The main consideration to take into account when implementing the *Batched Anchor Data Structure* is to have a limit on the number of leafs, i.e. tree depth. If the tree is too large it will become more expensive to verify the merkle witness for any individual update. 
In the bloom filter we limit the `tags` inclusion to the first 5 tags. The reason for this is simply to limit the size of the generated bloom filter. 


## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
