Authors: *Andrew Dirksen*

## Abstract

This RFC proposes an on-chain generic proof of existence module. This RFC also proposes additions to the Dock sdk to support [linked data proofs](https://w3c-ccg.github.io/ld-proofs/) of existence over credentials and possibly over presentations and possibly over generic jsonld.

## Suggested Reading

- https://w3c-ccg.github.io/ld-proofs/

## Background

*Write an introduction to the topic if helpful.*

## Motivation

We want to be able to prove that a credential was issued before a specific time.

## Previous Work

- [Chainpoint](https://chainpoint.org/) is a standard for anchors on bitcoin.
- [Blockcerts](https://www.blockcerts.org/) is a standard for on-chain attestations. A handy side effect of blockcerts' attestations is that they also serve as PoE anchors.

## Goals

*Bulleted list suggested.*

## Non-Goals

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

## Problems

### Use Cases Are Hard To Predict

What will this be used for?

### Composition of linked data proofs

The linked data proofs working group draft allows for multiple proofs to be composed and expressed as a [proof chain](https://w3c-ccg.github.io/ld-proofs/#proof-chains). The nature of a proof chains dictates that they need to be *ordered* lists. The current working group draft suggests an encoding for these proof chains.

```json
{
  "@context": [
    "https://www.w3.org/2018/credentials/v1",
    "https://www.w3.org/2018/credentials/examples/v1"
  ],
  "title": "Hello World!",
  "proofChain": [{
    "type": "Ed25519Signature2018",
    "proofPurpose": "assertionMethod",
    "created": "2019-08-23T20:21:34Z",
    "verificationMethod": "did:example:123456#key1",
    "domain": "example.org",
    "jws": "eyJ0eXAiOiJK...gFWFOEjXk"
  },
  {
    "type": "RsaSignature2018",
    "proofPurpose": "assertionMethod",
    "created": "2017-09-23T20:21:34Z",
    "verificationMethod": "https://example.com/i/pat/keys/5",
    "domain": "example.org",
    "jws": "eyJ0eXAiOiJK...gFWFOEjXk"
  }]
}
```

The above document is jsonld. In jsonld "lists", as used above, don't express order so this representaion may not be sound. If proof-chains are off the table, then [proof-sets](https://w3c-ccg.github.io/ld-proofs/#proof-sets) are an alternative. The limitation of proof-sets is that elements of a proof-set don't refer to previous elements. Consider the following:

```json5
{
  "@context": [
    "https://www.w3.org/2018/credentials/v1",
    "https://www.w3.org/2018/credentials/examples/v1"
  ],
  "title": "Hello World!",
  "proof": [{
    "type": "Ed25519Signature2018",
    "proofPurpose": "assertionMethod",
	...
  },
  {
    "type": "MerkelProof2018",
    "proofPurpose": "proofOfExistence",
	...
  }]
}
```

In that example, the PoE proves that the credential was constructed before a certain block. It never proves an upper bound on when the credential was issued which makes it pretty useless.

### vc-js support for proofs of existence

[vc-js](https://github.com/digitalbazaar/vc-js), has no idea how to handle proof of existence (citation needed), and doesn't provide an interface we can use to plug into our poe cabability.

### Special Care must be paid to the "issuer" field

vc-js verifies the "issuer" field of each credential against the "proof" field. Users may rely on this behaviour. Explicit Ethos for Claim Deduction [relies](https://github.com/docknetwork/sdk/blob/f346eb7cdd596c56aa685f3dcc8cb467751c7adb/src/utils/cd.js#L76) on this behaviour. Proofs of existence don't relate solely to credentials and the "issuer" field is not verifiable via a PoE.

## Open Questions

This RFC proposes a method for proving some data existed before a specific block. On PoW chains, it's possible to also prove a lower bound on credential issuance by including a block hash in the credential before issuing. This is out of scope, but maybe this type of proof can be implemented using Dockchain as well.

*Use this section to invite specific feedback from reviewers.*

---

*Final Checklist*

- *Would your RFC benefit from some last-minute visuals?*
