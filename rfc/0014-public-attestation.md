
Authors: Andrew Dirksen

## Abstract

This RFC proposes a way for DIDs to publically attests to arbirary (and arbitrarily large) RDF claimgraphs.
These attestations are not stored on-chain; rather, the attester chooses a storage method by specifying a Uri.

## Suggested Reading

- [RDF](https://en.wikipedia.org/wiki/Resource_Description_Framework)
- [RDF Containers](https://www.w3.org/TR/rdf-schema/#ch_containervocab)

## Background

We discussed attaching attestation semantics to dereferencable anchors but have not persued the feature. This RFC proposes Public Attestations, but keeps dereferencable anchors out of scope.

A Url can be said to dreference to an RDF claimgraph if and only if the document to which it points has a mime-type that is an RDF serialization format and the document body is a valid instance of that serialization format.

### Terms

**The Supergraph**:

All public machine accessible RDF data, [reified](https://www.w3.org/wiki/RdfReification) such that each statement is known to be true.

**Curious Agent**:

A Supergraph crawler directed by a programmable sense of curiosity.

## Motivation

Public Attestations are a generalization of Public Delegations. They will enable the proposed [Public Delegation](./0013-public-delegation.md) feature.

## Previous Work

Unspecified

## Goals

- Keep the feature flexible. Empower users to invent and implement new use-cases.
- Keep on-chain logic general and reusable.
- Minimize on-chain resource use
  - Minimize chain-state utilization.
  - Minimize on-chain logic.
- Public data should be discoverable.
- Interoperability.
  - Allow attestations from other DID methods.
  - Expect claimgraphs from public attestation to be merged with those from credentials.
  - Represent data as RDF for portability.
  - Use well known RDF ontologies like foaf where possible.
  - Data should be cross platform.
- Prevent data silos and gatekeepers.
  - Access to, and verification of, public attestations should be difficult to restrict.
- Information should be verifiable [via ethos].
- User privacy
  - Users should control their own data.
  - Users should have the power to decide which data they publish.

## Non-Goals

- Hashlinks/Dereferencable Anchors
- Vendor lock-in (to Dock)

## Expected Outcomes

The proposed feature is highly generic and extensible. It will enable third parties to invent new use-cases.

## RFC

### On-Chain Component

The runtime module is as simple as possible. The design aims to keep as much data off-chain as possible.

```
// Storage
map claims: Did => Url,

// Extrinsics
fn set_claim(origin: AccountId, claimer: Did, claim: Option<Url>, sig: DidSignature) -> _;
```

*Note: Replay protection needs to be implemented but is omitted from this example for simplicity.*

### Semantics

In RDF terms, the following implication relationship will be considerd an axiom:

If an entry `did => url` exists in the "claims" map at the current block, then `[did dock:attestsDocumentContents url]`.

### Curious Agent

Use cases like [Public Credential Delegation](./0013-public-delegation.md) will require a method for automatic discovery of data. This RFC proposes a Supergraph crawler with programmable "Curiosity". Curiosity will be represented as logical rules (as in Claim Deduction) or as SPARQL queries. (The difference between the two is that logical rules are applied recursively while SPARQL queries are not.)

Following is an example of the logic-based Curiosity Suite that might be used for discovering delegations. It would be appended to the ruleset introduced by [RFC 0008](./0008-delegatable-credentials.md) before being applied by the Curious Agent.

```
if [?a dock:mayClaim ?b] then [?a rdf:type curiosity:Interesting]
if [?a dock:mayDelegate ?b] then [?a rdf:type curiosity:Interesting]
if [?a rdf:type curiosity:Interesting]
  and [?a dock:attestsDocumentContent ?b]
  then [?b rdf:type curiosity:Interesting]
```

On infering a tuple of form `[?a rdf:type curiosity:Interesting]`, the Curious Agent will initialize an asyncronous lookup of the resource in question. While the lookup is in progress reasoning can continue. When the lookup is complete, a semantically true RDF representaion of the resource will be inserted into the agent's knowlege graph. For example, when the resource is a DID document, the new knowlege would look like this:

```turtle
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix dock: <http://dock.io/rdf/> .

<did:example:a> dock:dereferencesTo [
  rdfs:member [
    rdf:subject <did:example:a> ;
    rdf:predicate dock:attestsDocumentContent
  ] ;
  # ... rest of the did doc
] .
```

The crawler will be written in Rust.

### Addition to the Dock DID Resolver

Public attestations will be displayed in the DID document of the attester. This enables compatability accross DID methods. DIDs from other methods, non-dock DIDs, may also include public attestations.

```
{
  "@context": "https://www.w3.org/ns/did/v1",
  "id": "did:dock:<Did>",
  "verificationMethod": [{
    "id": "did:dock:<Did>#key-1",
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
  - All data, posts, favorites, and [foaf](http://www.foaf-project.org/) are contained in attested documents or sub-documents. Media is optionally pinned on ipfs.
- Attested builds
  - A trusted party compiles a program and attests that `[src.tar.gz <example:compilesTo> ipfs://adfa/bin.tar.gz]` a user can download the trusted build rather than build from source themself.
- Self Publishing Music
  - An independent musical artist self-publishes a free album. They attest to the album metadata using an open a specification like [music ontology](http://musicontology.com/specification). The actual audio data for tracks may be pinned in ipfs. The Curious Agent can locate the music of known authors and pass it to the users streaming program.
- Automatic Retrieval of Media
  - A curator scans a movie database and publically attests to the resultant data. Another party attests to the links between movies and urls where they can be streamed legally. A user indicates they want to e.g. watch all the movies featuring [Michael Cera](https://www.wikidata.org/wiki/Q309555). The Curious Agent seeks out the movies in question and the users browser automatically begins streaming.
- Public attestations to linked biometrics.
  - If a DID is authorized to open an electronically secured door, that DID may publically attest to the biometric data of the person controlling it. The door can use that attestation in place of a private attestation (a credential signed by the DID). This allows authorized party to gain secure access to the door without carrying a keycard. The authorized party controls the attested biometrics data and can change/revoke it at will.

I think it's worth dedicating some time to implement a demo for Public Attestations.

## Deferred Decisions

- The choice of representation for Curiosity Suites is left to the implementer. It is assumed Curiosity Suites will be represented as either [rify](https://github.com/docknetwork/rify) rules or as SPARQL, but other options exist; Prolog may even be a viable option.

## Other Considerations

Unspecified

## Open Questions

We need a cool nickname for the feature.

How will Public Attestation be demoed?
