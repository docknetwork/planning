*This document is a written proposal of some future effort.*

*The italicized items should be deleted or commented out before submission.*

*If a section has no content, simply write "None", "Unspecified" or "NA".*

Authors: *Andrew Dirksen*

## Abstract

This RFC proposes use of [Public Attestation](./0014-public-attestation.md) to give issuers the option to publicly attest delegations rather than requiring delegation chains be stored and presented by a holder.

This RFC depends on both [Public Attestation](./0014-public-attestation.md) and [Delegatable Credentials](./0008-delegatable-credentials.md).

## Suggested Reading

- [Public Attestation](./0014-public-attestation.md)
- [Delegatable Credentials](./0008-delegatable-credentials.md)
- Curiosity Suite: A specification for what new information a Crawler should investigate, specified as rify rules.
- Curious DID RDF Supergraph Crawler: A web-crawler as proposed in the [Public Attestation RFC](./0014-public-attestation.md). (Actual name is TBD)

## Background

### Terms

- Delegator: An entity who grants authority to make certain attestations on their behalf.
- Delegatee: An entity who is granted authority to make certain attestations on behalf of a delegator.

## Motivation

To use [Delegatable Credentials](./0008-delegatable-credentials.md), Holders are required to carry and present not only the credential in question, but also additional supporting credentials which comprise a delegation chain. Public Delegation will allow Verifiers to automatically locate and make use of delegation attestations so the Holder need not present a delegation chain along with thier credential.

## Previous Work

[Delegatable Credentials](./0008-delegatable-credentials.md)

## Goals

- Minimize chain-state utilization.
- Keep delegation flexible. Issuers should be allowed extreme flexiblity in delegation.
- Keep on-chain logic general and reusable. New applications of dockchain should be inventable by anyone, without runtime upgrades.
- Minimize on-chain logic.
- Make delegations discoverable.
- Simplify UX for credential holders.
- Interoperability. (Delegation chains may span multiple DID methods.)

## Non-Goals

- Forced public delegation: A delegator will not have the power to force downstream delegations to be public. Delegations can be a mix of public and private attestations. (The Verifier could potentially enforce such a rule but that is out of scope.)
- Restrict the number of credentials issued by a delegator.

## Expected Outcomes

This feature will drive implementation of [Public Attestation](./0014-public-attestation.md).

## RFC

[Delegatable Credentials](./0008-delegatable-credentials.md) will result in and ontology and associated rules. This RFC will be implemented by plugging those rules into the Crawler. The verifier will have the option to run the crawler to locate delegations before verifying credentials.

A Curiosity Suite will be written to direct the Crawler.

## Deferred Decisions

Schema-based delegation vs. pure claim deduction. This decision belongs in [Delegatable Credentials](./0008-delegatable-credentials.md).

## Other Considerations

### Fully on-chain delegation

A fully on-chain delegation graph was considered:

```
map delegations: Did => BTreeMap<Policy, Vec<Did>>
...
#[bitmap]
enum Permissions {
	MayDelegate,
	MayClaim,
}
struct Policy {
	allowed: Vec<BlobName>,
	perms: Permissions,
}
```

Cons:

- chain-state utilization scales with number of delegations, and also with complexity of delegation logic
- Hard coding delegation rules on-chain precludes other types of delegation logic implemented by other parties in the future e.g. `if [?did type Knight] then [did:example:self claims [?did mayClaim _:0]]`
- not interoperable? could other did methods express this same data somehow?
- The on-chain logic is complex

### capabilityDelegation

The `object` of [https://www.w3.org/TR/did-core/#capability-delegation](capabilityDelegation) from the DID working draft (08 November 2020) is a verification method, not an arbitrary entity.

## Open Questions

*Use this section to invite specific feedback from reviewers.*

---

*Final Checklist*

- *Would your RFC benefit from some last-minute visuals?*
