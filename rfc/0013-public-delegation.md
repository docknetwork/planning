
Authors: Andrew Dirksen

## Abstract

This RFC proposes use of [Public Attestation](./0014-public-attestation.md) to give issuers the option to publish delegations globally rather than requiring delegation chains be stored and presented by a holder.

The proposed feature depends on both [Public Attestation](./0014-public-attestation.md) and [Delegatable Credentials](./0008-delegatable-credentials.md).

## Suggested Reading

- [Public Attestation](./0014-public-attestation.md)
- [Delegatable Credentials](./0008-delegatable-credentials.md)
- [Curious DID RDF Supergraph Crawler Agent](./0014-public-attestation.md#curious-agent) (Actual name TBD)

## Background

### Terms

- Delegator: An entity who grants authority to make certain attestations on their behalf.
- Delegatee: An entity who is granted authority to make certain attestations on behalf of a delegator.
- Curiosity Suite: An executable specification for what new information a Curious Crawler should investigate, likely expressed as logical rules.

## Motivation

To use [Delegatable Credentials](./0008-delegatable-credentials.md), Holders are required to carry and present not only the credential in question, but also additional supporting credentials which comprise a delegation chain. Public Delegation will allow Verifiers to automatically locate and make use of delegation attestations so the Holder need not present a delegation chain along with their credential.

## Previous Work

[Delegatable Credentials](./0008-delegatable-credentials.md)

## Goals

- Minimize chain-state utilization.
- Keep delegation flexible. Issuers should be allowed extreme flexibility in delegation.
- Keep on-chain logic general and reusable. New applications of dockchain should be inventable by anyone, without runtime upgrades.
- Minimize on-chain logic.
- Make delegations discoverable.
- Simplify UX for credential holders.
- Interoperability. (Delegation chains may span multiple DID methods.)

## Non-Goals

- Forced public delegation
  - A delegator will not have the power to force downstream delegations to be public. The Verifier could potentially enforce such a rule but that is out of scope.
  - Delegations can be a mix of public and private attestations. 
- Restrict the number of credentials issued by a delegator. This would likely rely on some sort of forced public delegation, as well as forced public issuance.
- Deductive Proofs. It is possible to offload to a Holder the task of locating delegations. The Holder would provide a proof of delegation to the Verifier. Verifying these proofs would involve dereferencing the URLs specified by the creator of the proof. Support for deductive proofs with IO capability is out of scope for this RFC. [related](https://github.com/docknetwork/rify/issues/7#issuecomment-736840701).

## Expected Outcomes

This feature will drive implementation of [Public Attestation](./0014-public-attestation.md).

## RFC

[RFC 0008 - Delegatable Credentials](./0008-delegatable-credentials.md) proposes composition of an ontology and associated rules. This RFC will be implemented by plugging those rules into the Curious Agent proposed by  [RFC 0014 - Public Attestations](./0014-public-attestation.md). The verifier will have the option to run the crawler to locate delegations before verifying credentials.

A Curiosity Suite will be written to direct the Curious Agent in finding public delegations.

## Deferred Decisions

Schema-based delegation vs. pure claim deduction. This decision belongs in [Delegatable Credentials](./0008-delegatable-credentials.md).

## Other Considerations

### Fully on-chain delegation

A fully on-chain delegation graph was considered:

```rust
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
- This approach keeps more logic on-chain.

### capabilityDelegation

The `object` of [https://www.w3.org/TR/did-core/#capability-delegation](capabilityDelegation) from the DID working draft (08 November 2020) is a verification method, not an arbitrary entity.

## Open Questions

To use the Curious Agent from javascript an interface must be defined. What will that interface look like? Some possibilities:
- Stuff the agent in a [docker] container and let it perform exploration. Expose a query interface.
- Run the Curious Agent in a wasm VM. Let the VM call back to js when it needs a Url dereferenced.
