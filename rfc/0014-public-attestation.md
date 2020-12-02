*This document is a written proposal of some future effort.*

*The italicized items should be deleted or commented out before submission.*

*If a section has no content, simply write "None", "Unspecified" or "NA".*

Authors: Andrew Dirksen

## Abstract

This RFC proposes a way for DIDs to publically attests to arbirarily large RDF claimgraphs.
These attestations are not stored on-chain; rather, the attester chooses a storage method.

## Suggested Reading

*Bulleted list suggested.*

*If a team member would require additional technical information before reading the rest of this document, link to that information from this section. Each item listed here adds to the total reading time for this document.*

## Background

I've played with the idea for a public attestation registry for years now. The team discussed attaching attestation semantics to dereferencable anchors (Danks).

This RFC proposes public attestations, but keeps dereferencable anchors out of scope.

## Motivation

Public attestations are a generalization of public delegation attestations.

## Previous Work

*What previous efforts, if any, have been made to solve these problems?*
*Any other related efforts?*

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

*Here is a chance to limit feature creep and reduce confusion.*

## Expected Outcomes

*Mention the outcomes you expect, intentional and unintentional, beneficial and harmful.*

## RFC

Attestations may be mutable without making changes on-chain. For example, when an attestation is an http\[s\] url the referred document can be changed. When dereferencing an attestation, the dereferencer may choose to reject such urls.

For some URL schemes, http[s] for example, dereferencing of attestations may be tracked by the owner of the target webserver. The choice of storage location belongs to the attester, but a dereferencer wishing to avoid tracking may choose to reject such urls.

## Deferred Decisions

*List the decisions that this RFC explicitly does not make. Optionally enumerate potential approaches to the listed decisions.*

## Other Considerations

*Discuss approaches you considered (but ultimately decided against). This serves as a form of documentation and can also preempt suggestions from reviewers to investigate approaches you’ve already discarded.*

## Open Questions

*Use this section to invite specific feedback from reviewers.*

---

*Final Checklist*

- *Would your RFC benefit from some last-minute visuals?*
