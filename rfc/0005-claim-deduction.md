TODO: read https://www.w3.org/TR/2014/REC-rdf11-mt-20140225

Authors: Andrew Dirksen

## Abstract

This RFC describes a system for deducing new "composite" claims from sets verifiable credentials. Specifically, this RFC describes a system for deducing composite claims from RDF claim graphs. This document also proposes use of a deductive reasoning system to automatically generate machine verifiable formal proofs of such claims.

## Suggested Reading

- https://www.w3.org/TR/rdf11-primer
- https://en.wikipedia.org/wiki/Deductive_reasoning

## Background

The terms "derivation", "proof" and "argument" refer to [formal proofs](https://en.wikipedia.org/wiki/Formal_proof). "Derivation" will be used when possible.

A composite claim, thoerem, or conclusion is the logical result of a derivation. "Theorem" will be used when possible.

Each valid derivation proves a logical if-then (implication) relationship between a set of premises and a thoerem. The derivations proposed in this RFC split premises into two types: claims (represented as RDF tuples) and "rules". The data model for representing "rules" is undecided.

A derivation is "valid" if it is impossible for its premises to be true while its thoerem is false. An derivation is "sound" if it is valid and the premises are true. https://en.wikipedia.org/wiki/Deductive_reasoning#Validity_and_soundness

"Deductive reasoning" or "theorem proving" is the process of finding a derivation that proves a given theorem.

This RFC proposes a "Derivation Validation Program" which checks whether a derivation is "valid", but does not check whether a derivation is "sound".

## Motivation

Each VDCM credential attests some claim graph, but the exact claims expressed in the credential are not always what the verifier needs. In most cases, the verifier needs to peform additional processing of verified credentials before acting on them. The current state-of-the-art is to present verified credentials in some human readable form. It's the human's responsibility to reason about the credentials and possibly perform some action based on the conclusion. Removing the human from the process is assumed to be beneficial.

## Previous Work

Stacked Credentials often [proposed](https://ccrc.tc.columbia.edu/media/k2/attachments/stackable-credentials-awards-for-future.pdf) and [frequently](https://cte.ed.gov/initiatives/community-college-stackable-credentials) [discussed](https://www.credentialingexcellence.org/blog/implementing-a-stackable-strategy-a-discussion-with-nfwa-and-atd) but rarely in the context of the VCDM. Credential Deduction is a generalisation of credential stacking.

## Goals

- Reduce necesary human interaction when acting on credentials
- Stacked Credentials
- Developer outreach through open source implementation and engaging demos

## Non-Goals

- logical induction, logical abduction
- on-chain components

## Expected Outcomes

- Derivation format, potentially usable as a "proof" in verifiable credentials.
- Better understanting of the problem space.

## RFC



## Deferred Decisions

- Choice of derivation format
- Choice of rule expression DSL
- Language
- Use of claim graph representaion for rules

## Other Considerations

Some alternative aproaches were considered:

Each composite claim is describe as in some turing complete language. The program analyses the claim graph representing the "premises" to determine whether the a composite claim is true. TODO: consider this approach.

*Discuss approaches you considered (but ultimately decided against). This serves as a form of documentation and can also preempt suggestions from reviewers to investigate approaches you’ve already discarded.*

## Open Questions

How to represent logical rules. SPARQL? OWL? Regular expressions applied to the claim graph?

What are some engaging example use-cases?

---

*Final Checklist*

- *Would your RFC benefit from some last-minute visuals?*
