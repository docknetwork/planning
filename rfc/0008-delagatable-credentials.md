
Authors: Andrew Dirksen

## Abstract

This RFC proposes defining an extendable ontology for expressing credential delegations. This RFC also proposes writing a set of logical rules for reasoning over said ontology.

## Suggested Reading

- [Claim Deduction](https://docknetwork.github.io/sdk/tutorials/concepts_claim_deduction.html)
- [Linked Biometrics Demo](https://github.com/docknetwork/linked-biometrics-demo)

## Background

A working PoC for offline credential delegation exists (shown in the linked biometrics demo).

## Motivation

A PoC is insufficient to make users comfortable using credential delegation. Credential Delegation vocabulary should be published as an official standard.

## Previous Work

See Suggested Reading.

## Goals

- Features should be extendable. The implementation should not prevent future innovations by Dock or by other parties. e.g. schema-based delegations.
- Interoperability
  - Interoperability between parties using the new ontology. Issuers and verifiers should be able to agree on semantics.
  - Use well known RDF ontologies like foaf where possible.
- Ontology semantics should be clear, succinct, non-ambiguous, and logically valid with respect to the RDF data model. 
- Should not be VCDM specific. Other methods of attestation should be possible [e.g.](./0014-public-attestation.md) .

## Non-Goals

- On-chain delegations, see [Public Delegation](./0013-public-delegation.md).
- Schema-based delegations. Logic-based and Schema-based delegation are not mutually exclusive, but Logic-based delegation is what this RFC proposes.
- Offline credential delegation required holders store and present credential delegation chains with their credentials. This requirement is expected to damage UX for holders but is out of scope for this RFC. The issue is addressed by [Public Delegation](./0013-public-delegation.md).

## Expected Outcomes

- A rigorously thought-out Credential Delegation ontology, published as an "alpha" version.
- Executable definitions for the ontology written as deductive rules.
- Concept doc and Tutorial.

## RFC

Proposed actions:

- Specify and publish the alpha RDF ontology.
- Implement and publish the related logical rules, most likely in the Dock javascript SDK repo.
- Concept doc.
- Tutorial doc.

It's possible there already exists a ontology with support for delegation. The implementer should attempt to find such an ontology. If it suits this RFC's use-case it should be used instead of a custom ontology.
<!-- https://www.google.com/search?q=rdf+delegation+chain+ontology -->

## Deferred Decisions

Unspecified

## Other Considerations

Unspecified

## Open Questions

Unspecified

### Data Restrictions

How can logic-based delegations specify generic predicates on data like "the object ?a is an integer literal" or "?a == not(?b)"

Perhaps [Generic Predicates](https://github.com/docknetwork/rify/issues/7) would be required.

Schema formats often allow expressing restrictions on values, like "?a is less than 4", but to my knowledge, these restrictions are not extensible. There isn't a way to introduce new predicates like "?a is even". Unsure whether multi-element predicates like "?a == not(?b)" are supported in any existing graph schema language. https://www.w3.org/TR/shacl/
