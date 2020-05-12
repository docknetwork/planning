[RFC](../rfc/0003-schema.md)

## Design Summary
Schemas are stored on chain and identified and retrieved by their unique id. A credential specifies the unique id of the schema that it adheres to.
Schemas are stored as blob in the blob storage module of the chain. The chain is agnostic to the contents of the blob and thus to schemas.

## Post-RFC Decisions
None

## Spec
Before continuing with the spec, read the RFC on schemas.  

### Node

#### API

Constants
```rust
/// Size of the blob id in bytes
pub const ID_BYTE_SIZE: usize = 32;
/// Maximum size of the blob in bytes
 // implementer may choose to implement this as a dynamic config option settable with the `parameter_type!` macro
pub const BLOB_MAX_BYTE_SIZE: usize = 1024;

/// The type of the blob id
pub type Id = [u8; ID_BYTE_SIZE];
```

Storage
```rust
/// For each blob id, its author's DID and the blob is stored in the map
decl_storage! {
    trait Store for Module<T: Trait> as BlobModule {
        Blobs get(id): map dock::blob::Id => Option<(dock::did::Did, Vec<u8>)>;
    }
}
```

Types
```rust
/// When a new blob is being registered, the following object is sent
/// When a blob is queried, the following object is returned.
#[derive(Encode, Decode, Clone, PartialEq, Debug)]
pub struct Blob {
    id: dock::blob::Id,
    blob: Vec<u8>,
    author: dock::did::Did,
}
```

New blob
```rust
/// Register a new blob after ensuring blob with the same id is not registered and then 
/// verifying `did`'s signature on the blob
/// `schema_detail` contains the id, author DID and the blob. The size of blob should be at 
/// most `BLOB_MAX_BYTE_SIZE` bytes. The `blob` is wrapped in a [StateChange][statechange] before 
/// serializing for signature verification.
/// `signature` is the signature of the blob author on the `blob`
pub fn new(
            origin,
            blob: dock::blob::Blob,
            signature: dock::did::DidSignature,
        ) -> DispatchResult
```

Errors
```rust
decl_error! {
    /// Error for the token module.
    pub enum Error for Module<T: Trait> {
        /// The blob is greater than `BLOB_MAX_BYTE_SIZE`
        BlobTooBig,
        /// There is already a blob with same id
        BlobAlreadyExists,
        /// There is no such DID registered
        DidDoesNotExist,
        /// Signature verification failed while adding blob
        InvalidSig
    }
}
```

Events:  
No events for now.


Add `Blob` to `StateChange`.
```rust
#[derive(Encode, Decode)]
pub enum StateChange {
    KeyUpdate(did::KeyUpdate),
    ....,
    Blob(blob::Blob)
}
```

Add `BlobModule` in `construct_runtime` in `lib.rs`
```rust
pub enum Runtime where
		....
	{
    System: system::{Module, Call, Storage, Config, Event},
    .....,
    DIDModule: did::{Module, Call, Storage, Event},
    ....,
    BlobModule: blob::{Module, Call, Storage}
	}
```

#### Tests
1. Can add a blob with unique id, blob data of < `BLOB_MAX_BYTE_SIZE` bytes and a valid signature.
1. Can add a blob with unique id, blob data of `BLOB_MAX_BYTE_SIZE` bytes and a valid signature.
1. Adding a blob with size > `BLOB_MAX_BYTE_SIZE` fails with error `BlobTooBig`.
1. Can retrieve a valid blob and the blob contents and author match the given ones.
1. Adding a blob with already used id fails with error `BlobAlreadyExists`.
1. Adding a blob with an unregistered DID fails with error `DidDoesNotExist`.
1. An invalid signature while adding a blob should fail with error `InvalidSig`.

### SDK

For validating with JSON-schema, the package [jsonschema](https://www.npmjs.com/package/jsonschema) is used.

#### API

Constants
```js
const BLOB_MAX_BYTE_SIZE = // Retrieve them from chain constants;
const BLOB_ID_MAX_BYTE_SIZE = // Retrieve them from chain constants;
```

A new module for blob in `src/modules`
```js
class BlobModule {
  constructor(api) {
    this.api = api;
    this.module = api.tx.blobModule;
  }
  
  /**
    Register a new blob on the chain
    `blob` matches the `Blob` object on chain.
  */
  new(blob, signature) {
  }

  /**
    Get blob with given id from the chain. Accepts a full blob id like blob:dock:0x... or just the 
    hex identifier. Returns error if cannot find blob.
  */
  async getBlob(id) {
  }

  // Serializes the `Blob` for signing before sending to the node
  getSerializedBlob(blob) {
  }
}

class Schema {
  // Create a new `Schema` object
  // id is optional, if not given, generate a random id
  constructor(id) {

  }

  // Add the JSON schema to this object after checking that `jsonSchema` is a valid JSON schema.
  // Check if JSON is valid.
  setJSONSchema(jsonSchema) {
  }

  // Update the object with `author` key. Repeatedly calling it will keep resetting the author
  // did can be a DID hex identifier or full DID
  setAuthor(did) {

  }

  // Update the object with `signature` key. This method is used when 
  // signing key/capability is not present and the signature is received from outside.
  // Repeatedly calling it will keep resetting the `signature` key.
  // The signature must be one of the supported objects
  setSignature(signature) {

  }

  // Serializes the object using `getSerializedBlob` and then signs it using the given 
  // polkadot-js pair. The object will be updated with key `signature`. Repeatedly calling it will 
  // keep resetting the `signature` key
  sign(pair) {

  }

  // Serializes to JSON for sending to the node, as `Blob`. A full DID is converted to the 
  // hex identifier and signature object is converted to the enum
  toJSON() {
  }

  // Check that the given JSON schema is compliant with JSON schema spec mentioned in RFC
  static function validateSchema(json) {
  }

  /**
    Get schema from from the chain using the given id, by querying the blob storage. 
    Accepts a full blob id like blob:dock:0x... or just the hex identifier and the `DockAPI` object.
    The returned schema would be formatted as specified in the RFC (including author DID, schema id) or an error is 
    returned if schema is not found on the chain or in JSON format.
  */
  async static function getSchema(id, dockApi) {
  }
}
```

The `VerifiableCredential` class in `src/verifiable-credential.js` is updated with new methods and some old methods are updated
```js
class VerifiableCredential {
  // Sets the `credentialSchema` field of the credential with the given id and type as specified in the RFC. 
  // `id` must be a URI.
  setSchema(id, typ) {
  }

  // Check that the credential is compliant with given JSON schema, meaning `credentialSubject` has the 
  // structure specified by the given JSON schema. Use `validateCredentialSchema` but exclude subject's id.
  // Allows issuer to validate schema before adding it.
  validateSchema(schema) {
  }

  // The method will check that the `credentialSubject` is consistent with `credentialSchema` 
  // if `credentialSchema` if `credentialSchema` is present. Uses `validateSchema`.
  async verify(resolver, compactProof = true, forceRevocationCheck = true, revocationAPI) {
  }
}
```
Similarly, for `VerifiablePresentation` in `src/verifiable-presentation.js`, the `verify` method must be updated to verify the credential schema
```js
class VerifiablePresentation {
  // The method will check for schema validation using the same logic used by verify credential
  async verify(challenge, domain, resolver, compactProof = true, forceRevocationCheck = true, revocationAPI) {
  }
}
```

For credential and presentation verification to validate according to the schema, update functions `verifyCredential` 
and `verifyPresentation` in `src/utils/vc.js`. Declare a function called `validateCredentialSchema` in `src/utils/vc.js`.
```js
import Validator from 'jsonschema/lib/validator';
const validator = new Validator();

// The function uses `jsonschema` package to verify that the `credential`'s subject `credentialSubject` has the JSON schema `schema`
function validateCredentialSchema(credential, schema) {
  // validator.validate(credential, schema) should `errors` as an empty array.
}

```

#### Tests

1. `BlobModule` can write new blobs on chain and read them.
1. `BlobModule` fails to write blob with size greater then allowed.
1. `BlobModule` fails to write blob with id already used.
1. `BlobModule` can should throw error when cannot read blob with given id from chain.
1. `Schema` accepts the id optionally and generates id of correct size when id is not given.
1. `Schema`'s `setAuthor` will set the author and accepts a DID identifier or full DID. 
1. `Schema`'s `setJSONSchema` will only accept valid JSON schema and set the schema key of the object. 
1. `Schema`'s `setSignature` will only accept signature of the supported types and set the signature key of the object. 
1. `Schema`'s `sign` will generate a signature on the schema detail, this signature is verifiable.
1. `Schema`'s `validateSchema` will check that the given schema is a valid JSON-schema.
1. `Schema`'s `toJSON` will generate a JSON that can be sent to chain.
1. `Schema`'s `getSchema` will return schema in correct format.
1. `Schema`'s `getSchema` throws error when no blob exists at the given id.
1. `Schema`'s `getSchema` throws error when schema not in correct format.
1. `validateCredentialSchema` should validate given credential's schema with the given JSON-schema. Test the following cases
    1. `credentialSubject` has same fields and fields have same types as JSON-schema
    1. `credentialSubject` has same fields but fields have different type than JSON-schema
    1. `credentialSubject` is missing required fields from the JSON-schema and it should fail to validate.
    1. The schema's `properties` is missing the `required` key and `credentialSubject` can omit some of the `properties`.
    1. `credentialSubject` has extra fields than given schema specifies and `additionalProperties` is false.
    1. `credentialSubject` has extra fields than given schema specifies and `additionalProperties` is true.
    1. `credentialSubject` has extra fields than given schema specifies and `additionalProperties` has certain type.
    1. `credentialSubject` has nested fields and given schema specifies the nested structure.
1. `VerifiableCredential`'s `setSchema` should appropriately set `credentialSchema`.
1. `VerifiableCredential`'s `validateSchema` should validate the `credentialSubject` with given JSON schema.
1. Utility methods `verifyCredential` and `verifyPresentation` should check if schema is incompatible with the `credentialSubject`.
1. The `verify` and `verifyPresentation` should detect a subject with incompatible schema in `credentialSchema.`

### Dev tasks

The task breakdown as per above spec for both the node and SDK is as follows:

- Mock API for getting and putting blob in Substrate
- Node API for write and read blob, includes tests mentioned in the spec
- Add class `BlobModule` in SDK. Support reading and writing blobs on chain through SDK. Write tests mentoned in impl spec
- Write and read schemas from chain using the `Schema` class. Used `BlobModule` internally. Covers validating JSON shcema, serializing, signing, sending and querying the chain. Write corresponding tests mentoned in impl spec.
- Integration with credentials: 
  - `validateCredentialSchema` implementation and tests 15.1-15.4
  - `validateCredentialSchema` tests 15.5-15.7
  - `validateCredentialSchema` tests 15.8
- Add `setSchema` in VerifiableCredential and test from impl spec
- Add `validateSchema` in VerifiableCredential and test from impl spec
- Update `verifyCredential` to check for valid schema and test from impl spec
- Update `VerifiableCredential`'s verify method
- Update `verifyPresentation` to check for valid schema and test from impl spec
- Update `VerifiablePresentation`'s verify method and test from impl spec 
- Add example script for blob module showing put and get
- Add example script for schema module's builder pattern and then reading and writing the schema on chain
- Add example script for showing schema validation.
- Add example script for showing schema addition to credentials and their verification.
- Add schemas specified in the RFC (Product, Bill of landing, covid-19 schemas) as JSON schemas in json files. Create a separate directory called schemas. 
- Add example script to show integration of above schemas
- Add concept doc describing schema, some content can be borrowed from RFC
- Add tutorial for schema writing, reading, and integration in credentials.
- Deployment of node. This must be done once we know the SDK works with node


## Measuring Impact
Dock chain will have a the schemas specified in the RFC written on chain and the SDK would be able to issue credentials referring those. The written schemas should be present in the SDK as JSON so that they can be imported by the calling library.

## Teaching

The SDK will have example scripts for

- writing and reading a blob in chain storage.
- writing and reading schema on chain.
- showing how to use schemas in credentials.

A tutorial (concepts and impl tutorial) would be added for teaching schema.

## Security, Privacy, Risks
None

## Milestones
- Blob storage support in node.
- Blob storage support in node.
- Schema creation and query support in SDK.
- Schema integration with current credentials and presentations.

## Open Questions
None
