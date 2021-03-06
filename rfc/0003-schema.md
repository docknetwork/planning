Authors: Lovesh Harchandani

## Abstract
The schema in a credential describes the structure of the credential, meaning the fields present, their data types and the constraints on the field values. The format of the schema is JSON and follows the JSON schema standard. The schema is stored on the chain in a blob storage and is referenced inside the credential. The schema applies to the claims (properties of credential subject) in the credential.

## Suggested Reading
- [VCDM spec section about Data Schemas](https://www.w3.org/TR/vc-data-model/#data-schemas)
- [Schema](https://json-schema.org/understanding-json-schema/about.html)
- [Basics of JSON Schema](https://json-schema.org/understanding-json-schema/basics.html)
- [Step-By-Step guide to JSON Schema](https://json-schema.org/learn/getting-started-step-by-step.html)
- [Verifiable Credentials JSON Schema Specification](https://w3c-ccg.github.io/vc-json-schemas/)
- [Github repo for CCG's JSON Schema Spec](https://github.com/w3c-ccg/vc-json-schemas), contains [sample code](https://github.com/w3c-ccg/vc-json-schemas/blob/master/hypothetical-use-cases/cmtr/cmtr-v0.1.spec.js) showing schema validation
- Optionally, read the [JSON schema core spec](http://json-schema.org/draft/2019-09/json-schema-core.html) and [validation spec](http://json-schema.org/draft/2019-09/json-schema-validation.html)

## Background
None

## Motivation
In pursuit of [extensibility](https://w3c.github.io/vc-data-model/#extensibility), VCDM makes an Open World Assumption; a credential can state anything. Schemas allow issuers to "opt-out" of some of the freedom VCDM allows. Issuers can concretely limit what a specific credential will say. In a closed world, a verifier can rely on the structure of a credential to enable new types of credential processing e.g. generating a complete and human-friendly graphical representation of a credential.

## Previous Work
This RFC is suggesting the use of JSON schema as defined by the CCG [here](https://w3c-ccg.github.io/vc-json-schemas).

## Goals

- Support an immutable single-owner blob storage on chain.
- Support storing schemas in the blob storage
- Support querying schemas (using blob id) from chain
- Support adding a schema to a credential using SDK.
- Support validating a credential (and presentation) with the specified schema using SDK.

## Non-Goals
- On-chain validation of JSON schema.

## Expected Outcomes
- A new substrate module for blob storage and its integration in SDK.
- SDK supports schema in the credentials and validates the credential's structure according to the schema.
- New SDK tutorials for schema. The testnet is updated with schema support.
- Showcase standard schemas like [PR card](https://github.com/w3c-ccg/vc-examples/blob/master/plugfest-2020/vendors/sicpa/credentials/PermanentResidentCard.json), [BoL](https://github.com/w3c-ccg/vc-examples/blob/master/plugfest-2020/vendors/mavennet/credentials/BillOfLading.json) , [QP Inbond](https://github.com/w3c-ccg/vc-examples/blob/master/plugfest-2020/vendors/mavennet/credentials/QPInBond.json) and some Covid-19 creds like [Healthcare Worker Passport](https://docs.google.com/document/d/1F5TLvAqCxj1kaPuPe6JhdECixwpbhKpEAb8eeQuDGT4/edit#heading=h.kdkhzpmqto5s), [Proof of Health Status](https://docs.google.com/document/d/1F5TLvAqCxj1kaPuPe6JhdECixwpbhKpEAb8eeQuDGT4/edit#heading=h.4371z63wgb1t)

## RFC

- A blob storage module is added to the chain where each blob has a unique id. Writing a blob requires a DID.
- The blob storage module stores a map of id to the blob and the blob author. The id is 32 bytes and the blob is treated as 
arbitrary bytes so it can be JSON, XML, CSV, etc.
- Blobs once written cannot be deleted.
- The VCDM spec does not mandate how a schema should be represented but gives the freedom of choosing JSON schema, JSON-LD schema, or some other data verification schema.
- The RFC is choosing JSON schema over JSON-LD schema for now as it has implementation in JS and and is documented. Attempt to use JSON-LD schema failed (more detail in **Other Considerations** section). 
- The schema of a credential is determined by the issuer but not necessarily created by the issuer.
- Each schema will be stored on chain as a blob with a unique id of 32 bytes and the value for the id will be a JSON object and the author DID. 
- The schema thus has an associated DID as the author's identity implicitly attaches weight to the schema. 
- Schemas can only be created but cannot be deleted as deleting a schema can cause the dependent credentials to be unparsable. 
- Schema can only be retrieved with their id and thus schema can be stored in map in chain-storage with a fixed size id and (max-size) JSON string as value. 
- Note that Substrate runtime will not parse JSON during extrinsic and will treat it as a bytearray with maximum size of 1024 bytes. 
- Schema validation is not done by the chain as it is expensive and the cost to validate cannot be merely predicted by the size of the schema and thus the current weight system in Substrate is ineffective. As a result, it is the client's (SDK) responsibility to validate the schema. 
- The reason for supporting only JSON and not a hashlink for schema extrinsic is that hashlink schemas can anyway be resolved in the SDK, no need to support them on chain.
- The SDK supports creating, verifying, writing and reading schema from the chain. 
- A schema is created by directly passing the complete JSON to the SDK. The SDK should also support creating credentials with schema (in the builder) in the `credentialSchema` key and validating credentials with the schema and complain when credential (presentation's credentials as well) fails to satisfy the schema. 
- The SDK should continue to support credentials without a schema (`credentialSchema`).  

To write a schema on chain, the entire JSON document should be sent with a unique schema id and the DID's signature. The JSON document is serialized using JCS serialization.
For example the following JSON document could be sent:
```json
{
  "name": "AlumniCredSchema",
  "version": "1.0.0",
  "$schema": "http://json-schema.org/draft-07/schema#",
  "description": "Alumni",
  "type": "object",
  "properties": {
    "emailAddress": {
      "type": "string",
      "format": "email"
    },
    "alumniOf": {
      "type": "string"
    }
  },
  "required": ["emailAddress", "alumniOf"],
  "additionalProperties": false
}
```

The schema is queried from the chain using its id resulting in the above JSON document and the author DID. The SDK 
will format the received information as follows:
```json
{
   // Note that the value of id key is same as the value of `id` in the `credentialSchema` field of the credential below.
   "id": "blob:dock:5C78GCA......",
   "name": "AlumniCredSchema",
   "version": "1.0.0",
   "author": "did:dock:5CEdyZkZnALDdCAp7crTRiaCq6KViprTM6kHUQCD8X6VqGPW",
   "schema": {
      "$schema": "http://json-schema.org/draft-07/schema#",
      "description": "Alumni",
      "type": "object",
      "properties": {
        "emailAddress": {
          "type": "string",
          "format": "email"
        },
        "alumniOf": {
          "type": "string"
        }
      },
      "required": ["emailAddress", "alumniOf"],
      "additionalProperties": false
    }
}
```

The json representation of the `credentialSubject` within a credential specifying the above as schema is expected to satisfy the schema.
The above schema states that there will be 2 properties called `emailAddress` which is a string and has the format of an email and `alumniOf` which is a string. Both properties are required to be present in the `credentialSubject` due to their mention in `required`. Also, `additionalProperties` is set to false indicating that `credentialSubject` must not contain any properties apart from the properties listed in `properties` (`id` is popped out before validation).  

The schema in a credential will be specified as
```json
"credentialSchema": {
  "id": "blob:dock:5C78GCA......",
  "type": "JsonSchemaValidator2018"
}
```

The resulting credential will look like this
```json
{
   "@context": [
      "https://www.w3.org/2018/credentials/v1",
      "https://www.w3.org/2018/credentials/examples/v1"
   ],
   "id": "uuid:0x9b561796d3450eb2673fed26dd9c07192390177ad93e0835bc7a5fbb705d52bc",
   "type": [
      "VerifiableCredential",
      "AlumniCredential"
   ],
   "issuanceDate": "2020-03-18T19:23:24Z",
   "credentialSchema": {    // this is the schema
      "id": "blob:dock:5C78GCA......",
      "type": "JsonSchemaValidator2018"
   },
   "credentialSubject": {
      "id": "did:dock:5GL3xbkr3vfs4qJ94YUHwpVVsPSSAyvJcafHz1wNb5zrSPGi",
      "emailAddress": "john.smith@example.com",
      "alumniOf": "Example University"
   },
   "credentialStatus": {
      "id": "rev-reg:dock:0x0194...",
      "type": "CredentialStatusList2017"
   },
   "issuer": "did:dock:5GUBvwnV6UyRWZ7wjsBptSquiSHGr9dXAy8dZYUR9WdjmLUr",
   "proof": {
      "type": "Ed25519Signature2018",
      "created": "2020-04-22T07:50:13Z",
      "jws": "eyJhbGciOiJFZERTQSIsImI2NCI6ZmFsc2UsImNyaXQiOlsiYjY0Il19..GBqyaiTMhVt4R5P2bMGcLNJPWEUq7WmGHG7Wc6mKBo9k3vSo7v7sRKwqS8-m0og_ANKcb5m-_YdXC2KMnZwLBg",
      "proofPurpose": "assertionMethod",
      "verificationMethod": "did:dock:5GUBvwnV6UyRWZ7wjsBptSquiSHGr9dXAy8dZYUR9WdjmLUr#keys-1"
   }
}
```

Notice how the `credentialSubject` section has `emailAddress` and `alumniOf` as its schema in `credentialSchema` requires those 2 fields.  
If a credential has multiple subjects, each subject should conform to the schema (after popping out the `id`).

### Example: Specifying the type of fields not defined in the schema

The schema can allow for fields not defined in the schema but with certain restrictions using `additionalProperties`
```json
{
   "type":"object",
   "properties":{
      "emailAddress":{
         "type":"string",
         "format":"email"
      },
      "alumniOf":{
         "type":"string"
      }
   },
   "required":["emailAddress", "alumniOf"],
   "additionalProperties":{
      "type": "bool" 
    }
}
``` 
In the above schema, the JSON object can have additional properties but they have to be of bool type
```json
// This is valid as no additional property
{
   "alumniOf":"Example University",
   "emailAddress":"john@example.org"
}

// This is valid as additional properties are of string type
{
   "alumniOf":"Example University",
   "emailAddress":"john@example.org",
   "name":"John Smith",
   "location":"earth"
}

// This is invalid as additional property is not of string type
{
   "alumniOf":"Example University",
   "emailAddress":"john@example.org",
   "wasHonored":false,
}
```

### Example: Nested fields

The schema can have nested fields

```json
{
  "type": "object",
  "properties": {
    "emailAddress": {
      "type": "string",
      "format": "email"
    },
    "alumniOf": {
      "type": "string"
    },
    "degree": {   // notice the nesting here
      "type": "object",
      "properties": {
        "type": {
          "type": "string"
        },
        "name": {
          "type": "string"
        },
      },
      "required": ["name", "type"],
      "additionalProperties": true,
    }
  },
  "required": ["emailAddress", "alumniOf"],
  "additionalProperties": false,
}
```
In the above example, the schema defines a nested structure where the `degree` field has nested properties `name` and `type` and allows additional properties as well. The following are valid examples of JSON compliant with the above schema.
```json
{
   "alumniOf":"Example University",
   "degree":{
      "type":"BachelorDegree",
      "name":"Bachelor of Science and Arts",
   },
   "emailAddress":"john@example.org"
}

{
   "alumniOf":"Example University",
   "degree":{
      "type":"BachelorDegree",
      "name":"Bachelor of Science and Arts",
      "location":"earth"    // additional property, wasn't specified in the schema.
   },
   "emailAddress":"john@example.org"
}
```


## Deferred Decisions
The schema extrinsic allows a large blob of data to be sent to the node. Currently this is limited by a small cap of 1KB but in future this might need to expanded. Moreover, a client writing schema of 100 bytes should pay less for state storage compared to someone writing schema of 900 bytes. For this dynamic weights for extrinsics should be implemented.  
JSON-schema supports schema references with `$ref` attribute but the W3C spec [Verifiable Credentials JSON Schema Specification](https://w3c-ccg.github.io/vc-json-schemas/) does not specify them yet. We can however support them in the future where the `$ref` key's value will be the Dock schema id (prepended with `/` to indicate root) and the SDK will have to recursively scan the credential schema to find all the dependency schemas and then scan each of those schemas for more dependencies. Once all schemas have been discovered, add them to the schema validator (using `addSchema` if using `jsonschema`). Eg, consider the following case where the address schema is referenced in the person schema which is referenced in the `manSchema` schema
```js
const addressSchema = {
  "id": "/blob:dock:address", // the schema id
  "type": "object",
  "properties": {
    "lines": {
      "type": "array",
      "items": {"type": "string"}
    },
    "zip": {"type": "string"},
    "city": {"type": "string"},
    "country": {"type": "string"}
  },
  "required": ["country"]
}

const personSchema = {
  "id": "/blob:dock:person",
  "type": "object",
  "properties": {
    "name": {"type": "string"},
    "address": {"$ref": "/blob:dock:address"}, // notice the address schema id being referenced
    "votes": {"type": "integer", "minimum": 1}
  }

const manSchema = {
    "id": "/blob:dock:man",
    "type": "object",
    "properties": {
      "type": {"$ref": "/blob:dock:person"},
      "gender": {"type": "string"}
      },
      "votes": {"type": "integer", "minimum": 1}
    }
};
```
When the scanner is run over `personSchema`, the referenced schema `blob:dock:address` is returned.
```js
import {scan} from 'jsonschema/lib/scan';
const s1 = scan('/', personSchema);
console.log(s1.ref);
> ref: { '/blob:dock:address': 0 }

const s2 = scan('/', manSchema);
console.log(s2.ref);
> ref: { '/blob:dock:person': 0 }
```
`s1.ref` is a map of referenced schemas with the key contains the schema id (with leading `/`).  
When the scanner is run over `manSchema`, the referenced schema `blob:dock:person` is returned but the schema `addressSchema` referenced by `personSchema` is not returned. Hence the recursive application of `scan` to the references.  
An alternate is using `unresolvedRefs` as shown [here](https://github.com/tdegrunt/jsonschema#dereferencing-schemas).  

## Other Considerations
- JSON-LD schemas were the first choice for schemas but no open-source library exists to support validation according to schema. Using JSON-LD framing, the structure of the data can be matched but not the data-type, eg. number should be greater than 0, etc.
- Briefly looked at [Shex](https://shex.io/) and [RDF schema](https://www.w3.org/TR/rdf-schema/) but this will need more research time.

## Open Questions
- Should JSON-LD research be pursued more?
