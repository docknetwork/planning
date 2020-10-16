Authors: *Andrew Dirksen*

## Abstract

This RFC proposes an on-chain generic proof of existence module and accompanying tooling for posting anchors to the chain.

## Suggested Reading

None

## Background

Anchoring in Dock chain has been planned for some time now. Proof of existence is a well known application of blockchain.

## Motivation

Explicit support for PoE on the Dock chain.

## Previous Work

- [Chainpoint](https://chainpoint.org/) is a standard for anchors on bitcoin.
- [Blockcerts](https://www.blockcerts.org/) is a standard for on-chain attestations. A handy side effect of blockcerts' attestations is that they also serve as PoE anchors.

## Goals

- simple proof of existence using dockchain
- simple and pleasant UX for creating and verifying anchors
- SDK example for posting an anchor
- SDK example for checking an anchor

## Non-Goals

- proof of attestation
- SDK wrapper around the auto-generated rpc interface for the anchoring module. Developers can use that interface directly.

## Expected Outcomes

A new on-chain Anchor module exposing a single on-chain function. A command line tool for posting anchors.

## RFC

This RFC proposes a very simple on-chain module to allow posting of fixed size anchors.

This RFC proposes an easy to install and user friendly command line tool for posting anchors to dockchain.

## Deferred Decisions

Anchors are expected to be modeled as a mapping from fixed size byte array, `[u8; K]`, to block number. `[u8; K] => BlockNumber`. The on-chain module is agnositic to hash algorithm.

The command line tool for posting anchors is expected to use either sha256 or blake2s.

Consider whether 32 bytes is large enough to represent anchors for the life of the chain. Use larger or dynamically sized anchors if necessary.

## Other Considerations

[Linked data proofs](https://w3c-ccg.github.io/ld-proofs/) is work-in-progress standard for attaching proofs of attestation, authentication, and more, to linked data (usually jsonld). It is, however, a work in progress; it's apt to have bugs and to change in the future. In addition, implementing and using a new "existence" proofPurpose is expected to pose unwanted technical challenges. Check out the [older version of this RFC](https://github.com/docknetwork/planning/blob/f411e2a357fe03e292d6aea1efedd0348b902087/rfc/0007-anchoring.md#problems) for more info.

Blockcerts implements "MerkleProof2019" which is, from what I can tell, a proof of on-chain attestation. The [spec](https://w3c-dvcg.github.io/lds-merkle-proof-2019/) seems to have disappeared. It could be possible to use blockcerts 3.0 by standardizing a [blink](https://w3c-ccg.github.io/blockchain-links/) for dockchain, but:
- we would run into the same problems implementing linked data proofs and
- AFAIK blockcerts prove attestation, which is not our goal.

Use of the on-chain [Blob](https://github.com/docknetwork/dock-substrate/blob/master/runtime/src/blob.rs) storage was considered for storing anchors but the blob module doesn't store block numbers. Blob entries store extra information (more than just a hash) that is not needed for PoE. Blobs require a DID to be created in order to post. This extra step would be frictitious to users.

## Open Questions

Should accompanying tooling implement merkle batching? https://github.com/lovesh/merkle_trees https://github.com/docknetwork/mrkl

This RFC proposes a method for proving some data existed before a specific block. On PoW chains, it's possible to also prove a lower time bound on the creation of certain types of data by incorporating a block hash when constructing the data. This is out of scope, but maybe this type of proof can be implemented using Dockchain as well. Unsure if/how this method can be applied to generic payloads.

The chain's runtime not expected to ever read from anchor storage. Would it be better to not write anchors to chainstate, but rather look them up in transaction history? Future light client functionality may be hindered if anchors are not in chainstate.
