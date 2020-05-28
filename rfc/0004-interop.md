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
(stands for DID-aware communication) is any communication that relies on DIDs as identifiers. DIDCOMM is a standard way to exchange DID-aware encrypted messages, regardless of transport.

## Motivation

Dock is providing tools for a standards-based ecosystem, proving interoperability is required to show standards compliance. It also helps showing potential users that chosing Dock is a more future-proof and enabling option than using vendors that use proprietary solutions.

## Previous Work

*What previous efforts, if any, have been made to solve these problems?*
*Any other related efforts?*

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

## Open Questions
- Are we planning to submit 
- Should the resulting Web apps (Issuer/Wallet/Verifier) complement/replace the previously-planned demo app? (Which at the time of writing this doc only has support for verification of credentials, but was planned to support issuing too)
- Do we _need_ too implement all three (Issuer/Wallet/Verifier) in order


---

*Final Checklist*

- *Would your RFC benefit from some last-minute visuals?*
