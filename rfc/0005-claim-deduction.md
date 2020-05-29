Authors: Andrew Dirksen

## Abstract

This RFC describes a system for deducing new "composite" claims from sets of verifiable credentials. Specifically, this RFC describes a system for deducing composite claims from RDF claim graphs. This document also proposes use of a deductive reasoning system to automatically generate machine verifiable formal proofs of such claims.

## Suggested Reading

- https://www.w3.org/TR/rdf11-primer
- https://en.wikipedia.org/wiki/Deductive_reasoning

## Background

The terms "derivation", "proof" and "argument" refer to [formal proofs](https://en.wikipedia.org/wiki/Formal_proof). "Derivation" will be used when possible.

A composite claim, thoerem, or conclusion is the logical result of a derivation. "Theorem" will be used when possible.

Each valid derivation proves a logical if-then (implication) relationship between a set of premises and a thoerem. The derivations proposed in this RFC split premises into two types: claims (represented as RDF tuples) and "rules". The data model for representing "rules" is undecided.

A derivation is "valid" if it is impossible for its premises to be true while its thoerem is false. An derivation is "sound" if it is valid and the premises are true. https://en.wikipedia.org/wiki/Deductive_reasoning#Validity_and_soundness

"Deductive reasoning" or "theorem proving" is the process of finding a derivation that proves a given theorem.

A "derivation validator" is a program which checks whether a derivation is "valid", but does not check whether a derivation is "sound".

## Motivation

Each VDCM credential attests some claim graph, but the exact claims expressed in the credential are not always what the verifier needs. In most cases, the verifier needs to peform additional processing of verified credentials before acting on them. The current state-of-the-art is to present verified credentials in some human readable form. It's the human's responsibility to reason about the credentials and possibly perform some action based on the conclusion. Removing the human from the process is assumed to be beneficial.

## Previous Work

Stacked Credentials are often [proposed](https://ccrc.tc.columbia.edu/media/k2/attachments/stackable-credentials-awards-for-future.pdf) and [frequently](https://cte.ed.gov/initiatives/community-college-stackable-credentials) [discussed](https://www.credentialingexcellence.org/blog/implementing-a-stackable-strategy-a-discussion-with-nfwa-and-atd) but rarely in the context of the VCDM. Credential Deduction is a generalisation of credential stacking.

## Goals

- Reduce necessary human interaction when acting on credentials.
- Stacked Credentials
- Developer outreach through open source implementation and engaging demos

## Non-Goals

- Logical induction, logical abduction
- On-chain components

## Expected Outcomes

- Derivation format, potentially usable as a vcdm:proof or dock:logic (custom property) in verifiable presentations
- Groundwork for new w3c vcdm:VerifiablePresentaion specialization type e.g. dock:DeductivePresentation2020
  - Note: Within this project "Derivation" already has a concrete meaning. Calling a dock:DeductivePresentaion2020 a derivation would be accurate, but confusing. "Deductive Presentation" should be used instead.
- Better understanding of the problem space
- Street cred

## RFC

This RFC proposes the development of, or integration with, two programs (functions).

1. A derivation validator
2. A theorem prover which finds derivations for use as input to the derivation validator

This RFC also proposes use of the derivation validator in combination with a VCDM Verifier to verify (check the soundness of) composite claims. The VCDM Verifier would verify VCDM credential and the derivation validator would validate derivations based on the premises extracted from those credentials.

### Example function signatures (psuedocode)

This section is non-normative. It's provided as a concrete example of what the api for this feature *may* look like.

Derivation validator:

```rust
fn validate(premises: RdfStore, derivation: &Proof) -> Result<Vec<ConcreteClaim>, InvalidProof>;
```

Theorem prover:

```rust
// naive thoerem prover
fn prove(premises: RdfStore, rules: &LogicalRules, composite_claims: &[Claim], limits: &Limits) -> Result<Proof, CantProve>;

// or

// theorem prover with rules embedded within the claim graph
fn prove(premises: RdfStore, composite_claims: &[Claim], limits: &Limits) -> Result<Proof, CantProve>;

// `Limits` are a way to bound memory consumption and cpu time when proving a thoerem. If the limits are exceeded an error is returned.
```

Use with VCDM presentations:

```js
// potential way to combine proof validation with credential verification
function verify_deductive_presentation(vcdm_presentation) {
    let verified_claims = ...; // verify and get the union of all credentials within the presentation
    let issuers = ...; // get every verified issuer in presentaion's credetials
    let composite_claims = [];
    for (proof of vcdm_presentation.get_prop("https://dock.io/rdf/logic")) {
        // the js wrapper around validate would throw and error on InvalidProof
        let new_claims = validate(verified_claims, proof);
        composite_claims.append(new_claims);
    }
    return {
        if_you_trust_all: issuers,
        then: composite_claims, // could also be `then: verified_claims + composite_claims`
	};
}
```

### Logical Rule Example

A logical conjuctive rules may be expressed like so:

```rust
struct Rule {
    if_all: Vec<Triple<usize>>,
    instantiation: BTreeMap<usize, Iri>,
    then: Vec<Triple<usize>>,
}

/// an RDF triple
struct Triple<T> {
    subject: T,
    predicate: T,
    object: T,
}
```

It's not decided exactly how the rule descriptions will be serialized, but they are expressable in this datalog-ish syntax:

```datalog
// For conciseness I'll use <#something> as shorthand for <http://example.com/something>.

// Forall a, b: if b is a parent of a then b is an ancestor of a
(?a <#parent> ?b) -> (?a <#ancestor> ?b) 

// forall a, b, c: if (a ancestor b) and (b ancestor c) then (a ancestor c)
(?a <#ancestor> ?b) ∧ (?b <#ancestor> ?c) -> (?a <#ancestor> ?c)
```

More rule examples:

```datalog
// forall a, b: if (a parent b) then (b child a)
(?b <#child> ?a) -> (?a <#parent> ?b)

// forall a, b: if (a child b) then (b parent a)
(?b <#parent> ?a) -> (?a <#child> ?b)

// Multiple statements can be implied in the same rule. It's left as an exercise to the reader to interpret this statement.
(?a <#mother> ?m) ∧ (?a <#father> ?f) -> (?m <#parentWith> ?f) ∧ (?f <#parentWith> ?m)

// Predicates can also be free values
(?a ?p ?b) ∧ (?p <#sameMeaning> ?q) -> (?a ?q ?b)
```

For reasoner performance. There will likely be a restriction that every implied claim must contain no unbound items that are not on the other side of the rule.

```
// This is not allowed
(<#alice> <#bob> <#charlie>) -> (<#alice> <#charlie> ?a)

// But this is allowed
(<#alice> <#bob> <#charlie>) -> (<#alice> <#charlie> <#bob>)
```

## Deferred Decisions

- Choice of derivation format
  - Useful to be human readable
- Choice of rule expression DSL
  - Use of claim graph representaion for rules
	- Allows both types of "premise" to be represented in the same claim graph.
	- Allows Issuers to include "rules" within the credential.
	- Downsides?
- How can derivations refer to [blank nodes](https://en.wikipedia.org/wiki/Blank_node)?
  - Without a way to uniquely specify blank nodes, a verifier would sometimes need to determine a shuffling of blank nodes that allows the derivation to be valid. That "find a shuffling" problem is computationaly difficult (https://en.wikipedia.org/wiki/Subgraph_isomorphism_problem).
- For storage and computaional efficiency, multiple theorems may be proved by a single "batch" derivation.
- Language
  - The derivation validator and theorem prover should be usable from other languages if possible. Rust is a likely candidate.

## Other Considerations

Some alternative aproaches were considered:

### Express rules in a turing complete language

As an alternative approach each composite claim could be described and shared as a program of some turing complete language. The program would analyze a "premises" claim graph to determine whether the a composite claim is true. Claim deduction would be performed by running the stored progam. This would make soundness checking computationally hard. It's not known how theorem proving could fit into this model.

### Express rules as [ShEx](https://shex.io/shex-primer/index.html)

It's expected that rules described as ShEx would not be flexible enough to allow theorem provers to find "creative" derivations. More research is required.

## Open Questions

### How to represent logical rules

SPARQL? OWL? Regular expressions applied to the claim graph?

### What are some engaging example use-cases?

### Expicit ethos

Should ethical (ethos) arguments be expressed as implication relationships like "issuer-A is trustworthy -> claim1 ∧ claim2 ∧ ..."? An expanded form may look something like this:

```
// The following would always be passed as an axiom:
∀ X, Y: (X 'claims Y) ∧ (X 'trustworthy 'true) -> Y

// then verified credentials would look like this
(issuer-A 'claims claim1)
(issuer-A 'claims claim2)

// and this assumption would be explicitly stated for each issuer
(issuer-A 'trustworthy true)

// claims would be "imported" like this
∀ X, Y: (X        'claims Y     ) ∧ (X        'trustworthy 'true) -> Y
        (issuer-A 'claims claim1) ∧ (issuer-A 'trustworthy 'true) -> claim1

// either the theorem prover needs to be general enough to extract claims as shown above, or all
// derivations need to be prefixed with claim "imports"
```

This may be a question best posed during implementation, when limitaions are better understood.
