Authors: Fausto Woelflin

## Abstract
Proving interoperability with other players in the space is key to survive and thrive in a standards-based ecosystem like the Verifiable Credentials one. This RFC explores the interop initiatives CHAPI and DIDCOMM, and the steps that would allow Dock to prove compliance to them.

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
###CHAPI
A non-opinionated pipe to enable a communication channel between two web applications running on the same user agent (without the data leaving the local machine to do so).

###DIDCOMM
_(DID communication)_ is any communication that relies on DIDs as identifiers. DIDCOMM is a standard way to exchange DID-aware encrypted messages, regardless of transport.

## Motivation

Dock is providing tools for a standards-based ecosystem, showing interoperability is required to empirically prove standards compliance. It also helps showing potential users that choosing Dock is a more future-proof and enabling option than going with vendors implementing proprietary solutions.

## Previous Work

_Please refer to the [suggested reading section](#suggested-reading)._

## Goals
1. Show some degree of interoperability with other players in the space.
1. Open source example applications to support these claims.

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

The main goal is to show some degree of interoperability with other players in the space. To that end, 

### Unified Interoperability Test Plan
The following is a summary of the Unified Interoperability Test Plan (section 2 in [this document](https://drive.google.com/file/d/1XvwGzYYy7ZrdElmz4_zj3qVBXRH8DydO/view)).
#### Wallets
1. [x] Support for the Verifiable Credentials data model.
1. [ ] Support create and read operations for at least two different DID methods
1. [ ] Support CHAPI or DIDComm for DIDAuth with an Issuer front-end website.
1. [ ] Support CHAPI or DIDComm for obtaining and storage of Verifiable Credentials.
1. [ ] Support CHAPI or DIDComm for presentation of Verifiable Credentials.
1. [x] Support the Ed25519 Cryptographic Suite.
1. [ ] Support the interaction with multiple Issuer demo sites and multiple Verifier demo sites through CHAPI and/or DIDComm.

#### Issuers
1. [x] Support for the Verifiable Credentials data model.
1. [ ] Support the Permanent Resident Card or Raw Materials Credential (data model).
1. [ ] Issue a Verifiable Credential to a subject, supporting at least two different DID methods for the issuer DID.
1. [x] Support the Ed25519 Cryptographic Suite
1. [ ] Support the Verifiable Credential Issuer HTTP API

#### Verifiers
1. [x] Support for the Verifiable Credentials data model.
1. [x] Verify a Verifiable Credential, supporting at least two different DID methods for the subject DID.
1. [ ] Verify a Verifiable Presentation, supporting at least two different DID methods for the
subject DID.
1. [x] Support the Ed25519 Cryptographic Suite.
1. [ ] Support the Permanent Resident Card or Raw Materials Credential (data model).
1. [ ] Support the Verifiable Credential Verifier HTTP API

Goals marked with a checkmark [x] above are believed to be done with from previous work, or that they will not require additional work from our side to pass them. Readers are encouraged to find discrepancies.

The subjects of CHAPI and DIDCOMM appear above as the two possible options to pass some of the goals. Research was done to understand them and how they could help us reach that goal. This new understanding allows for a wildly simplistic summary of the concepts, hoping to quickly bring readers up to speed for discussion:

- **CHAPI**: a way to allow two web application instances running in the same agent (browser) to communicate with each other (explicitely needing some user interaction), where user data never leaves the local device. 
- **DIDCOMM**: a way to wrap messages between two DID-aware entities in order to ensure TODO: READ MORE TO SUMMARIZE THIS

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
