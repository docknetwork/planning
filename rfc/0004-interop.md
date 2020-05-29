Authors: Fausto Woelflin

## Abstract
Proving interoperability with other players in the space is key to survive and thrive in a standards-based ecosystem like the Verifiable Credentials one. This RFC explores the [DHS S&T Silicon Valley Innovation Program (SVIP) proposal on interoperability](https://drive.google.com/file/d/1XvwGzYYy7ZrdElmz4_zj3qVBXRH8DydO/view), and the steps that would allow Dock to reach the suggested goals.

## Suggested Reading
- [Refined Roadmap Doc](https://www.notion.so/dockteam/Refined-Roadmap-2020-04-92673bd7155447b2a4ab109b813c079a)
  - [Interop Proposal](https://www.notion.so/dockteam/Interop-ec6566afb9b440778779ae5fabac2e41)
- [DHS S&T Silicon Valley Innovation Program (SVIP) proposal on interoperability](https://drive.google.com/file/d/1XvwGzYYy7ZrdElmz4_zj3qVBXRH8DydO/view)
- [Plugfest 2020 - Test suite](https://github.com/w3c-ccg/vc-examples/tree/master/plugfest-2020)
  - [Demo issuer](https://github.com/digitalbazaar/chapi-demo-issuer)
  - [Demo wallet](https://github.com/digitalbazaar/chapi-demo-wallet)
  - [Demo verifier](https://github.com/digitalbazaar/chapi-demo-verifier)
- [IIW Session Notes](https://iiw.idcommons.net/IIW_30_Session_Notes)
- [CHAPI: Credential Handler API](https://w3c-ccg.github.io/credential-handler-api/)
- [DIDCOMM RFC](https://github.com/hyperledger/aries-rfcs/blob/master/concepts/0005-didcomm/README.md)
- []()
- []()

## Background

TODO: We're pretty sure we're compliant, but showing it in a standarised way will be worth a lot

## Motivation

Dock is providing tools for a standards-based ecosystem, showing interoperability is required to empirically prove standards compliance. It also helps showing potential users that choosing Dock is a more future-proof and enabling option than going with vendors implementing proprietary solutions.

## Previous Work

_Please refer to the [suggested reading section](#suggested-reading)._

## Goals
1. Show some degree of interoperability with other players in the space.
1. Open source example applications to support these interop claims.
1. Appear listed as one of the interop-enabled options, [here](https://w3c-ccg.github.io/vc-examples/plugfest-2020.html) or wherever it makes the most sense for us to be discovered.

## Non-Goals
To the best of our knowledge, at the time of writing of this document:
- Deliverables will not include changes to our SDK code.
- Deliverables will not include changes to our Node code.

## Expected Outcomes

*Mention the outcomes you expect, intentional and unintentional, beneficial and harmful.*

1. Prove interoperability by making sure we pass each step in the [Unified Interoperability Test Plan](https://drive.google.com/file/d/1XvwGzYYy7ZrdElmz4_zj3qVBXRH8DydO/view). For the most part, 
passing all tests in the [Credential Handler API (CHAPI) Test Plan](https://github.com/w3c-ccg/vc-examples/tree/master/plugfest-2020) should be enough.
  1. This probably requires developing, hosting and open sourcing three demo web applications showing the Issuer, Wallet and Verifier use cases in action.

## RFC
*Main body. Make the plan descriptive enough to encourage productive suggestions.*

The main goal of this effort is to show some degree of interoperability with other players in the space. To that end, the Unified Interoperability Test Plan (section 2 in [this document](https://drive.google.com/file/d/1XvwGzYYy7ZrdElmz4_zj3qVBXRH8DydO/view)) suggest a series of goals to meet, and a series of tests for Issuer-, Wallet- and Verifier applications to pass. The following is a summary of those:
### Goals for Wallet Applications
1. [x] Support for the Verifiable Credentials data model.
1. [ ] Support create and read operations for at least two different DID methods
1. [ ] Support CHAPI or DIDComm for DIDAuth with an Issuer front-end website.
1. [ ] Support CHAPI or DIDComm for obtaining and storage of Verifiable Credentials.
1. [ ] Support CHAPI or DIDComm for presentation of Verifiable Credentials.
1. [x] Support the Ed25519 Cryptographic Suite.
1. [ ] Support the interaction with multiple Issuer demo sites and multiple Verifier demo sites through CHAPI and/or DIDComm.

### Goals for Issuer Applications
1. [x] Support for the Verifiable Credentials data model.
1. [ ] Support the Permanent Resident Card or Raw Materials Credential (data model).
1. [ ] Issue a Verifiable Credential to a subject, supporting at least two different DID methods for the issuer DID.
1. [x] Support the Ed25519 Cryptographic Suite
1. [ ] Support the Verifiable Credential Issuer HTTP API

### Goals for Verifier Applications
1. [x] Support for the Verifiable Credentials data model.
1. [x] Verify a Verifiable Credential, supporting at least two different DID methods for the subject DID.
1. [ ] Verify a Verifiable Presentation, supporting at least two different DID methods for the
subject DID.
1. [x] Support the Ed25519 Cryptographic Suite.
1. [ ] Support the Permanent Resident Card or Raw Materials Credential (data model).
1. [ ] Support the Verifiable Credential Verifier HTTP API

### Tests to pass
1. [ ] [Verifiable Credential HTTP API Test Plan](https://github.com/w3c-ccg/vc-examples/tree/feat/test-suite/test-suite): automated suite designed to test end-to-end conformance to the Verifiable Credential data model standard and the HTTP API pre-standards between Issuer and Verifier products. TODO: mention that we have most of these, only miss issuer/verifier APIs
1. [ ] [Credential Handler API (CHAPI) Test Plan](https://github.com/w3c-ccg/vc-examples/tree/master/plugfest-2020): manual test suite designed to test end-to-end conformance to the Credential Handler API pre-standard between the Issuer, Wallet, and Verifier products.

These tests seem to test similar things with regards to interoperability: comp
___

Goals/tests marked with a checkmark [x] above are believed to be done with from previous work, or that they will not require additional work from our side to pass them. Readers are encouraged to find discrepancies.

The subjects of CHAPI and DIDCOMM appear above as the two possible options to pass some of the goals. Research was done to understand them and how they could help us reach that goal. This new understanding allows for a wildly simplistic summary of the concepts, hoping to quickly bring readers up to speed for discussion:

- **CHAPI**: A non-opinionated pipe to enable a communication channel between two web applications running on the same user agent (without the data leaving the local machine to do so), explicitely needing some user interaction for security reasons.
- **DIDCOMM**: comes from _DID communication_, is any communication that relies on DIDs as identifiers. DIDCOMM can be seen as a standard way to exchange DID-aware encrypted messages, regardless of transport, between agents and agent-like things. 


TODO ADD DISCUSSION CHAPI VS DIDCOMM AND FINAL DECISION


 
## Deferred Decisions

*List the decisions that this RFC explicitly does not make. Optionally enumerate potential approaches to the listed decisions.*

## Other Considerations

*Discuss approaches you considered (but ultimately decided against). This serves as a form of documentation and can also preempt suggestions from reviewers to investigate approaches you’ve already discarded.*

## Open Questions
- Should the resulting Web apps (Issuer/Wallet/Verifier) complement/replace the previously-planned demo app? (Which at the time of writing this doc only has support for verification of credentials, but was planned to support issuing too)
- Do we _need_ to implement all three (Issuer/Wallet/Verifier) in order to prove interoperability?
- Do we want to take the demos one step further and show a bit more of our solutions? For example instead of hardcoding a demo credential to be issued every time, we could allow the user to manufacture one with our SDK. We could also take the chance to show other features like DID creation or Schemas.
- How do our schema-enabled VCs work in third party verifiers?

---

*Final Checklist*

- *Would your RFC benefit from some last-minute visuals?*
