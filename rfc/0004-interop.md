Authors: Fausto Woelflin

## Abstract
Proving interoperability with other players in the space is key to survive and thrive in a standards-based ecosystem like the Verifiable Credentials one. This RFC explores the [DHS S&T Silicon Valley Innovation Program (SVIP) proposal on interoperability](https://drive.google.com/file/d/1XvwGzYYy7ZrdElmz4_zj3qVBXRH8DydO/view), and the steps that would allow Dock to reach the goals suggested in it.

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
- [DID Auth whitepaper](https://github.com/WebOfTrustInfo/rwot6-santabarbara/blob/master/final-documents/did-auth.pdf)
  - [Video](https://www.youtube.com/watch?v=Yq8yFYdCnxU)



## Background

To the best of our knowledge, our tools should be 100% compatible with the ones from other vendors. This RFC explores ways of empirically proving third parties that this is the case.

## Motivation

Dock is providing tools for a standards-based ecosystem, showing interoperability is required to empirically prove standards compliance. It also helps showing potential users that choosing Dock is a more future-proof and enabling option than going with vendors implementing proprietary solutions.

## Previous Work

_Please refer to the [suggested reading section](#suggested-reading)._

## Goals
1. Show some degree of interoperability with other players in the space.
1. Open source example applications to support these interop claims.
1. Appear listed as one of the interop-enabled options, [here](https://w3c-ccg.github.io/vc-examples/plugfest-2020.html) or wherever it makes the most sense for us to be discovered.

## Non-Goals
To the best of our knowledge, at the time of writing of this document, deliverables will not include changes to our SDK code, nor to our Node code.

## Expected Outcomes
1. A live hosted Issuer web application (with a link to its open source code on Github)
1. A live hosted Wallet web application (with a link to its open source code on Github)
1. A live hosted Verifier web application (with a link to its open source code on Github). This may or may not be an extension of our current verifier.


## RFC
The main goal of this effort is to show some degree of interoperability with other players in the space. To that end, the Unified Interoperability Test Plan (section 2 in the [DHS doc](https://drive.google.com/file/d/1XvwGzYYy7ZrdElmz4_zj3qVBXRH8DydO/view)) suggest a series of goals to meet, and a series of tests for Issuer-, Wallet- and Verifier applications to pass. The following is a summary of those:
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
1. [ ] [Verifiable Credential HTTP API Test Plan](https://github.com/w3c-ccg/vc-examples/tree/feat/test-suite/test-suite): automated suite designed to test end-to-end conformance to the Verifiable Credential data model standard and the HTTP API pre-standards between Issuer and Verifier products. 
1. [ ] [Credential Handler API (CHAPI) Test Plan](https://github.com/w3c-ccg/vc-examples/tree/master/plugfest-2020): manual test suite designed to test end-to-end conformance to the Credential Handler API pre-standard between the Issuer, Wallet, and Verifier products.

___

Goals/tests marked with a checkmark [x] above are believed to be done with from our earlier work, or that they will not require additional work from our side to pass them. Readers are encouraged to find discrepancies.

The subjects of CHAPI and DIDCOMM appear above as the two possible options to pass some of the goals. Research was done to understand them and how they could help us reach that goal. This new understanding allows for a wildly simplistic summary of the concepts, hoping to quickly bring readers up to speed for discussion:

- **CHAPI**: A non-opinionated pipe to enable a communication channel between two web applications running on the same user agent (without the data leaving the local machine to do so), explicitely needing some user interaction for security reasons.
- **DIDCOMM**: comes from _DID communication_, is any communication that relies on DIDs as identifiers. DIDCOMM can be seen as a standard way to exchange DID-aware encrypted messages, regardless of transport, between agents and agent-like things. 

Remember, CHAPI and DIDCOMM are suggested as _"means to achieve **some** of the interop goals for the different applications"_, not as goals by themselves. Implementing both may help in some way but in no place of the [DHS doc](https://drive.google.com/file/d/1XvwGzYYy7ZrdElmz4_zj3qVBXRH8DydO/view) is it suggested to do so, probably because it'd be an overkill. This detail suggests the need to somehow answer the following question: *which of the two should we choose?*

As per [CHAPI's specs](https://w3c-ccg.github.io/credential-handler-api/#model) Credential handlers are implemented as Service Workers. The notion of service workers is more applicable to a full web application than to an SDK (which is what we currently have). DIDComm on the other hand, seems to be a specific way of wrapping messages, which can definitely be implemented as SDK functionality (maybe as a wrapper over [existing toos](https://github.com/decentralized-identity/DIDComm-js)). This distinction may seem to favor DIDComm over CHAPI, although the question remains whether we *want* to add that functionality to our SDK given that the goal of this effort isn't to "extend the SDK" but to "prove interoperability between our tools and those from other vendors".

Another factor to take into account is that the three demo applications (Issuer/Wallet/Verifier) need to be built in any case, and a probably good option is to use the existing ones from Transmute or Digital Bazaar as example. All of them seem to be very similar, which suggests that they may just be forks of each other and share most of the code except for the addition of a specific vendor. If after deeper study this remains being the case, it may be wise to follow the trend and use what these example repos are using ([CHAPI in this case](https://github.com/digitalbazaar/chapi-demo-issuer)) which would a) ensure we'll end up proving interop with them, and b) probably save a good amount of effort/time/money by reusing a lot of their code.

Careful analysis of these repos should be done before writing the Spec, the final choice may just depend on that. At the end of the day neither option seem to add significant value to Dock's set of tools beside proving interoperability, so we should probably go with the cheaper alternative.

### The currently-preferred option: CHAPI
All current research seems to point to the fact that implementing CHAPI examples are the cheaper alternative, so the following sections will show some examples of how this implementation can look like

#### Wallet
##### Service worker
Here's an example service worker script:
```javascript
 async function activateWalletEventHandler() {
    try {
      await credentialHandlerPolyfill.loadOnce(MEDIATOR);
    } catch(e) {
      console.error('Error in loadOnce:', e);
    }

    console.log('Worker Polyfill loaded, mediator:', MEDIATOR);

    return WebCredentialHandler.activateHandler({
      mediatorOrigin: MEDIATOR,
      async get(event) {
        console.log('WCH: Received get() event:', event);
        return {type: 'redirect', url: WALLET_LOCATION + 'wallet-ui-receive.html'};
      },
      async store(event) {
        console.log('WCH: Received store() event:', event);
        return {type: 'redirect', url: WALLET_LOCATION + 'wallet-ui-store.html'};
      }
    })
  }

  console.log('worker.html: Activating handler, WALLET_LOCATION:', WALLET_LOCATION);
  activateWalletEventHandler();
```

##### Handling a credential receive event
Here's an example of how a wallet UI can handle a credential receive event:
```javascript
async function handleWalletReceiveEvent() {
    const event = await WebCredentialHandler.receiveCredentialEvent();
    console.log('Wallet processing store() event:', event);
    document.getElementById('respondBtn').addEventListener('click', () => {
      const data = JSON.parse(document.getElementById('responseText').value);
      event.respondWith(Promise.resolve({dataType: 'VerifiableCredential', data}));
    });
    document.getElementById('cancelBtn').addEventListener('click', () => {
      event.respondWith(Promise.resolve({dataType: 'Response', data: 'error'}))
    });
  }

  credentialHandlerPolyfill
    .loadOnce(MEDIATOR)
    .then(handleWalletReceiveEvent);
```

##### Handling a credential store event
Here's an example of how a wallet UI can handle a credential store event:
```javascript
  async function handleStoreEvent() {
    const event = await WebCredentialHandler.receiveCredentialEvent();

    console.log('Store Credential Event:', event);

    const credential = event.credential || {};

    document.getElementById('username').innerHTML = Cookies.get('username') || '';

    document.getElementById('hintKey').innerHTML = credential.hintKey || '';
    document.getElementById('credentialContents').innerHTML = JSON.stringify(credential.data, null, 2);

    document.getElementById('successBtn').addEventListener('click', () => {
      event.respondWith(new Promise(resolve => {
        return resolve({dataType: 'Response', data: 'success'});
      }))
    });

    document.getElementById('errorBtn').addEventListener('click', () => {
      event.respondWith(new Promise(resolve => {
        return resolve({dataType: 'Response', data: 'error'});
      }))
    });
  }

  onDocumentReady(() => {
    document.getElementById('loginButton').addEventListener('click', login);
    refreshUsername();
  })

  credentialHandlerPolyfill
    .loadOnce(MEDIATOR)
    .then(handleStoreEvent);
```

#### Issuer & Verifier examples
These felt like an overkill but can be provided if the readers find them useful.


### Sample stories
The involved work may include, but not be limited to, the following tasks:
- Develop a sample Issuer API (according to the [VC Issuer HTTP API specs](https://github.com/w3c-ccg/vc-issuer-http-api))
- Develop a sample Verifier API (according to the [VC Verifier HTTP API specs](https://github.com/w3c-ccg/vc-verifier-http-api))
- Develop a sample, CHAPI-compliant, Issuer App
- Develop a sample, CHAPI-compliant, Wallet App
- Develop a sample, CHAPI-compliant, Verifier App
- Make sure our sample APIs pass all the [plugfest2020 tests](https://github.com/w3c-ccg/vc-examples/blob/master/plugfest-2020/plugfest-2020.spec.js). This should probably also cover [the other ones mentioned by DHS](https://github.com/w3c-ccg/vc-examples/tree/feat/test-suite/test-suite) as well.
- Create a [PR back to plugfest-2020](https://github.com/w3c-ccg/vc-examples/pulls) so they include us as an option and we show up in the [latest report](https://w3c-ccg.github.io/vc-examples/plugfest-2020.html).

 
## Deferred Decisions
N/A

## Other Considerations

N/A

## Open Questions
- Should the resulting Web apps (Issuer/Wallet/Verifier) complement/replace the previously-planned demo app? (Which at the time of writing this doc only has support for verification of credentials, but was planned to support issuing too)
- Do we *_need_* to implement all three (Issuer/Wallet/Verifier) in order to prove interoperability?
- Do we want to take the demos one step further and show a bit more of our solutions? For example instead of hardcoding a demo credential to be issued every time, we could allow the user to manufacture one with our SDK. We could also take the chance to show other features like DID creation or Schemas.
- How do our schema-enabled VCs work in third party verifiers?
- How will we implement DIDauth ourselves?
- Are there other (better) ways out there to reach the same interop goal?

