
Authors: Andrew Dirksen

## Abstract

This RFC proposes a way for DIDs to publically attests to arbirary (and arbitrarily large) RDF claimgraphs.
These attestations are not stored on-chain; rather, the attester chooses a storage method by specifying a Uri.

## Suggested Reading

- [RDF](https://en.wikipedia.org/wiki/Resource_Description_Framework)
- [RDF Containers](https://www.w3.org/TR/rdf-schema/#ch_containervocab)

## Background

We discussed attaching attestation semantics to dereferencable anchors but have not persued the feature. This RFC proposes Public Attestations, but keeps dereferencable anchors out of scope.

## Motivation

Public attestations are a generalization of public delegations. They will enable the proposed [Public Delegation](./0013-public-delegation.md) feature.

## Previous Work

Unspecified

## Goals

- Keep the feature flexible. Users should be allowed extreme flexiblity in how they employ the feature.
- Keep on-chain logic general and reusable. New applications of dockchain should be inventable by anyone, without runtime upgrades.
- on-chain resource use
  - Minimize chain-state utilization.
  - Minimize on-chain logic.
- Public data should be discoverable.
- Interoperability.
  - Allow attestations from other DID methods.
  - Expect claimgraphs from public attestation to be merged with those from credentials.
  - Represent data as RDF to keep data portable.
  - Use well known RDF ontologies like foaf where possible.
- Prevent data silos and gatekeepers.
  - Access to, and verification of, public attestations should be difficult to restrict.
- Information should be verifiable [via ethos].
- Data should be cross platform.
- User privacy
- Users should control their own data.
  - Users should have the power to decide which data they publish.

## Non-Goals

- Hashlinks/Dereferencable Anchors

## Expected Outcomes

The proposed feature is highly generic and extensible. It will enable third parties to invent new use-cases.

## RFC

### On-Chain Component

The runtime module is as simple as possible. The design aims to store as much data off-chain as possible.

```
// Storage
map claims: Did => Url,

// Extrinsics
fn set_claim(origin: AccountId, claimer: Did, claim: Option<Url>, sig: DidSignature) -> _;
```

*Note: Replay protection needs to be implemented but is omitted from this example for simplicity.*

### Semantics

In RDF terms, the following implication relationship is axiomized for dock DIDs.

If an entry `did => url` exists in the "claims" map at the current block `[did dock:attestsDocumentContents url]`.

A Url can be said to dreference to an RDF claimgraph if and only if the document to which it points has a mime-type that is an RDF serialization format and the document body is a valid instance of that serialization format.

### Curious Agent

Use cases like [Public Credential Delegation](./0013-public-delegation.md) will require a method for automatic discovery of data. This RFC proposes a supergraph crawler with programmable "Curiosity". Curiosity will be represented as logical rules (as in claim deduction) or as SPARQL queries. The difference between the two is that logical are applied recursively while SPARQL queries are not.

Following is an example of the logic-based Curiosity Suite that might be used for discovering delegations. It would be appended to the ruleset introduced by [RFC 0008](./0008-delagatable-credentials.md) before being applied by the Curious Agent.

```
if [?a dock:mayDelegate ?b] then [?a rdf:type dock:Delegate]
if [?a dock:mayClaim ?b] then [?a rdf:type dock:Delegate]
if [?a rdf:type dock:Delegate] then [?a rdf:type curiosity:Interesting]
if [?a rdf:type dock:Delegate]
  and [?a dock:attestsDocumentContent ?b]
  then [?b rdf:type curiosity:Interesting]
```

On infering a tuple of form `[?a rdf:type curiosity:Interesting]`, the Curious Agent will initialize an asyncronous lookup of the resource in question. While the lookup is in progress reasoning can continue. When the lookup is complete, a semantically true RDF representaion of the resource will be inserted into the agent's knowlege graph. For example, when the resource is a DID document, the claims added to would look like this:

```turtle
did:example:a dock:dereferencesTo [
	rdfs:member [
		rdf:subject did:example:a ;
		rdf:predicate dock:attestsDocumentContent
	] ;
	... # rest of the did doc
] .
```

The crawler will be written in Rust.

### Addition to the Dock DID Resolver

Public attestations will be displayed in the DID document of the attester. This enables compatability accross DID methods. Other DID methods may include the same predicate.

```
{
  "@context": "https://www.w3.org/ns/did/v1",
  "id": "<Did>",
  "verificationMethod": [{
    "id": "<Did>#key-1",
    "type": "Ed25519VerificationKey2018",
    "controller": "did:example:123456789abcdefghi",
    "publicKeyBase58": "H3C2AVvLMv6gmMNam3uVAjZpfkcJCwDwnZn6z3wXmqPV"
  }],
  "authentication": [ "#key-1" ],
  "https://dock.io/rdf/attestsDocumentContent": "<Url>"
}
```

### Demo

Lots of tempting use cases for this one that could serve as a demo.

- Decentralized social network
  - All data, posts, favorites, [foaf](http://www.foaf-project.org/) statements is contained in attested documents, or sub-documents. Media can be pinned in ipfs.
- Attested builds
  - A trusted party compiles a program and attests that `[src.tar.gz <compilesTo> ipfs://adfa/bin.tar.gz]` a user can download the trusted build rather than build the source themself.
- Self Publishing Music
  - An independent musical artist self-publishes a free album. They attest to the album metadata using an open specifications like [music ontology](http://musicontology.com/specification). The actual audio data for tracks is pinned on ipfs. The Curious Agent can locate the music of known authors and pass it to the users streaming program.
- Automatic Retrieval of Media
  - A curator scans a movie database and attests to the resultant data. Another party attests to the links between movies and urls where they can be streamed legally. A user indicates they want to watch all the movies featuring [Michael Cera](https://www.wikidata.org/wiki/Q309555). The Curious Agent seeks out the movies in question and the users browser automatically begins streaming.
- Public attestations to linked biometrics.
  - If a DID is authorized to open a door, that DID may attest to the biometric data of the person controllong it. The door can use that attestation in place of a credential signed by the DID. This allows authorized party to gain secure access to the door without a keycard. The authorized party controls the attested biometrics data and can change/revoke it at will.

I think it's worth dedicating some time to implement a demo for Public Attestations.

## Deferred Decisions

- The choice of representation for Curiosity Suites is left to the implementer. It is assumed Curiosity Suites will be represented as either [rify](https://github.com/docknetwork/rify) rules or as SPARQL, but other options exist; Prolog may even be a viable option.

## Other Considerations

Unspecified

## Open Questions

We need a cool nickname for the feature.
