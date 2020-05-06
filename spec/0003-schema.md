[RFC](../rfc/0003-schema.md)

## Design Summary
Schemas are stored on chain and identified and retrieved by their unique id. A credential specifies the unique id of the schema that it adheres to.

## Post-RFC Decisions
None

## Spec
Before continuing with the spec, read the RFC on schemas.  

### Node

#### API

Constants
```rust
/// Size of the schema id in bytes
pub const ID_BYTE_SIZE: usize = 32;
/// Maximum size of the schema object in bytes
pub const SCHEMA_MAX_BYTE_SIZE: usize = 1024;

/// The type of the schema id
pub type Id = [u8; ID_BYTE_SIZE];
```

Storage
```rust
/// For each schema id, its author's DID and the schema object is stored in the map
decl_storage! {
    trait Store for Module<T: Trait> as SchemaModule {
        Schemas get(id): map dock::schema::Id => Option<(dock::did::Did, Vec<u8>)>;
    }
}
```

Types
```rust
/// When a new schema is being registered, the following object is sent
/// When a schema is queried, the following object is returned.
#[derive(Encode, Decode, Clone, PartialEq, Debug)]
pub struct SchemaDetail {
    id: dock::schema::Id,
    schema: Vec<u8>,
    author: dock::did::Did,
}
```

New schema
```rust
/// Register a new schema after ensuring schema with the same id is not registered and then 
/// verifying `did`'s signature on the schema
/// `schema_detail` contains the id, author DID and bytes of the schema. The size of schema should be at 
/// most `SCHEMA_MAX_BYTE_SIZE` bytes. The `schema_detail` wrapped in a [StateChange][statechange] before 
/// serializing for signature verification.
/// `signature` is the signature of the schema author on the `schema_detail`
pub fn new(
            origin,
            schema_detail: dock::schema::SchemaDetail,
            signature: dock::did::DidSignature,
        ) -> DispatchResult
```

Errors
```rust
decl_error! {
    /// Error for the token module.
    pub enum Error for Module<T: Trait> {
        /// The schema is greater than `SCHEMA_MAX_BYTE_SIZE`
        SchemaTooBig,
        /// There is already a schema with same id
        SchemaAlreadyExists,
        /// There is no such DID registered
        DidDoesNotExist,
        /// Signature verification failed while adding schema
        InvalidSig
    }
}
```

Events:  
No events for now.


Add `SchemaDetail` to `StateChange`.
```rust
#[derive(Encode, Decode)]
pub enum StateChange {
    KeyUpdate(did::KeyUpdate),
    ....,
    SchemaDetail(schema::SchemaDetail)
}
```

Add `SchemaModule` in `construct_runtime` in `lib.rs`
```rust
pub enum Runtime where
		....
	{
    System: system::{Module, Call, Storage, Config, Event},
    .....,
    DIDModule: did::{Module, Call, Storage, Event},
    ....,
    SchemaModule: schema::{Module, Call, Storage}
	}
```

#### Tests
1. Can add a schema with unique id, schema data of < `SCHEMA_MAX_BYTE_SIZE` bytes and a valid signature.
1. Can add a schema with unique id, schema data of `SCHEMA_MAX_BYTE_SIZE` bytes and a valid signature.
1. Can retrieve a valid schema and the schema details and author match the given ones.
1. Adding a schema with size > `SCHEMA_MAX_BYTE_SIZE` fails with error `SchemaTooBig`.
1. Adding a schema with already used id fails with error `SchemaAlreadyExists`.
1. Adding a schema with an unregistered DID fails with error `DidDoesNotExist`.
1. An invalid signature while adding a schema should fail with error `InvalidSig`.

### SDK

For validating with JSON-schema, the package [jsonschema](https://www.npmjs.com/package/jsonschema) is used.

#### API
TODO: More details should be added for the parameters of functions in the actual code in conformance with JSDoc.  
TODO: Add constants for schema id size and max schema size from Node section. 

A new module for schema in `src/modules`
```js
class SchemaModule {
  constructor(api) {
    this.api = api;
    this.module = api.tx.schemaModule;
  }
  
  /**
    Register a new schema on the chain
    `schemaDetail` matches the `SchemaDetail` object on chain.
  */
  new(schemaDetail, signature) {
  }

  /**
    Get schema with given id from the chain. Accepts a full schema id like schema:dock:0x... or just the 
    hex identifier.
    The returned schema would be formatted as specified in the RFC (including author DID, schema id) or an error is 
    returned if schema is not in JSON format.
  */
  async getSchema(id) {
  }

  // Serializes the schema detail for signing 
  // before sending to the node
  getSerializedSchemaDetail(schemaDetail) {
  }
}

class SchemaDetail {
  // Create a new `SchemaDetail` object
  // id is optional, if not given, generate a random id
  constructor(id) {

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

  // Serializes the object using `getSerializedSchemaDetail` and then signs it using the given 
  // polkadot-js pair. The object will be updated with key `signature`. Repeatedly calling it will 
  // keep resetting the `signature` key
  sign(pair) {

  }

  // Serializes to JSON for sending to the node. A full DID is converted to the 
  // hex identifier and signature object is converted to the enum
  toJSON() {
  }

  // Check that the given JSON schema is compliant with JSON schema spec mentioned in RFC
  static function validateSchema(json) {
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

1. `SchemaDetail` accepts the id optionally and generates id of correct size when id is not given.
1. `SchemaDetail`'s `setAuthor` will set the author and accepts a DID identifier or full DID. 
1. `SchemaDetail`'s `setSignature` will only accept signature of the supported types and set the signature key of the object. 
1. `SchemaDetail`'s `sign` will generate a signature on the schema detail, this signature is verifiable.
1. `SchemaDetail`'s `validateSchema` will check that the given schema is a valid JSON-schema.
1. `SchemaDetail`'s `toJSON` will generate a JSON that can be sent to chain.
1. `validateCredentialSchema` should validate given credential's schema with the given JSON-schema. Test the following cases
    - `credentialSubject` has same fields and fields have same types as JSON-schema
    - `credentialSubject` has same fields but fields have different type than JSON-schema
    - `credentialSubject` is missing required fields from the JSON-schema and it should fail to validate.
    - The schema's `properties` is missing the `required` key and `credentialSubject` can omit some of the `properties`.
    - `credentialSubject` has extra fields than given schema specifies and `additionalProperties` is false.
    - `credentialSubject` has extra fields than given schema specifies and `additionalProperties` is true.
    - `credentialSubject` has extra fields than given schema specifies and `additionalProperties` has certain type.
    - `credentialSubject` has nested fields and given schema specifies the nested structure.
1. `VerifiableCredential`'s `setSchema` should appropriately set `credentialSchema`.
1. `VerifiableCredential`'s `validateSchema` should validate the `credentialSubject` with given JSON schema.
1. Utility methods `verifyCredential` and `verifyPresentation` should check if schema is incompatible with the `credentialSubject`.
1. The `verify` and `verifyPresentation` should detect a subject with incompatible schema in `credentialSchema.`


## Measuring Impact
Dock chain will have a the schemas specified in the RFC written on chain and the SDK would be able to issue credentials referring those. The written schemas should be present in the SDK as JSON so that they can be imported by the calling library.

## Teaching
The SDK will have example scripts for writing and reading schema on chain. There would be example scripts showing how to use schemas in credentials. A tutorial (concepts and impl tutorial) would be added for teaching schema.

## Security, Privacy, Risks
None

## Milestones
- Schema support in node
- Schema creation and query support in SDK
- Schema integration with current credentials and presentations.

## Open Questions
None
