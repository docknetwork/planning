Authors: Andrew Dirksen

## Abstract

This RFC describes a system for deducing new "composite" claims from sets verifiable credentials. Specifically, this RFC describes a system for deducing composite claims from RDF claim graphs. This document also proposes use of a deductive reasoning system to automatically generate machine verifiable formal proofs of such claims.

## Suggested Reading

- https://www.w3.org/TR/rdf11-primer
- https://en.wikipedia.org/wiki/Deductive_reasoning

## Background

The terms "derivation", "proof" and "argument" refer to [formal proofs](https://en.wikipedia.org/wiki/Formal_proof). "Derivation" will be used when possible.

A composite claim, thoerem, or conclusion is the logical result of a derivation. "Theorem" will be used when possible.

Each valid derivation proves a logical if-then (implication) relationships between a set of premises and a thoerem. The derivations proposed in this RFC split premises into two types: claims (represented as RDF tuples) and rules. Rules will likely be represented as Ontologies.

A derivation is "valid" if it is impossible for its premises to be true while its thoerem is false. An derivation is "sound" if it is valid and the premises are true. https://en.wikipedia.org/wiki/Deductive_reasoning#Validity_and_soundness

"Deductive reasoning" or "theorem proving" is the process of finding a derivation that proves a given theorem.

*Write an introduction to the topic if helpful.*

## Motivation

*What technical problems does this RFC aim to solve?*

## Previous Work

*What previous efforts, if any, have been made to solve these problems?*
*Any other related efforts?*

## Goals

*Bulleted list suggested.*

## Non-Goals

- logical induction, logical abduction

*Here is a chance to limit feature creep and reduce confusion.*

## Expected Outcomes

*Mention the outcomes you expect, intentional and unintentional, beneficial and harmful.*

## RFC

*Main body. Make the plan descriptive enough to encourage productive suggestions.*

> *"I didn’t have time to write a short letter, so I wrote a long one instead."*
> *--Mark Twain*

## Deferred Decisions

*List the decisions that this RFC explicitly does not make. Optionally enumerate potential approaches to the listed decisions.*

## Other Considerations

*Discuss approaches you considered (but ultimately decided against). This serves as a form of documentation and can also preempt suggestions from reviewers to investigate approaches you’ve already discarded.*

## Open Questions

How to represent logical rules. SPARQL? OWL?

What are some engaging example use-cases?

*Use this section to invite specific feedback from reviewers.*

---

*Final Checklist*

- *Would your RFC benefit from some last-minute visuals?*
