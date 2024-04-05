# JSON Web Token

## Fundamentals
A JWT (pronounced "jot") is a string that contains three sections that are url-safe base64 encoded, each separated by a dot (.).

The base64 encoded values are JSON objects, so, something like this:

```jsonc
// Header
{
  "alg": "HS256",
  "typ": "JWT"
}

// Payload
{
  "sub": "1234567890", // Subject
  "name": "John Doe",
  "iat": 1516239022, // Issued at time
  "iss": "urn:myapp.com", // Issuer
  "exp": 1697795266, // Expires at
  "aud": "urn:myapp:api1", // Audience
  "nbf": 1697795266 // Not before
}

// Signature
// "BIDfNrs_26wEO8f4aE6rnLB2qb4kKGtpSHjMEF-BIr0"
```

Becomes this:

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyLCJpc3MiOiJ1cm46bXlhcHAuY29tIiwiYXVkIjoidXJuOm15YXBwOmFwaTEiLCJleHAiOjE2OTc3OTUyNjZ9.BIDfNrs_26wEO8f4aE6rnLB2qb4kKGtpSHjMEF-BIr0
```

### The header
This section provides some basic information about how the JWT is signed.  

Treat the values in the header as hints, and not instructions.

### The payload
The payload contain claims. These are often used for authorization purposes, where you'll inspect the token's claims to see if the caller is authorized to perform an action.

The claims serves at least two purposes: token validation and authorization.  
Standard claims (also known as Registered Claims) to check validity includes:
- `sub`: The subject. Identifies the entity that the token represents. That being a user or system. Typically the unique identifier for a subject within the issuer system. Ideally, this is a unique, immutable value. Never something like an e-mail, phone number, and the like.
- `iss`: Always validate the issuer is what you'd expect. Never trust tokens from an unknown party.
- `aud`: Always validate that the audience is the intended value. This is the unique identifier of the token recipient. This can also be an array of values, so that the token can be used for multiple audiences.
- `exp`: Always validate that the token hasn't expired. The issuer purposefully added the expiration to make sure that the token wouldn't be used beyond this point.
- `nbf`: This may not always be there, but, if it is, then the time must be in the past at the time of validation. This may be useful if you e.g. sell access to a resource that is due to release in the future.

When validating the `exp`, expires at, it's a good practice to use a clock skew, allowing a few extra minutes (not hours), in case the systems are not 100% aligned in terms of time. 

If you define your own claims, then these are non-Registered claims, also referred to as "Private Claims". Alwyas check the registered claims list to see if there's one that fits your purpose, before making up your own. 

- (List of registered claims)[https://www.iana.org/assignments/jwt/jwt.xhtml#claims]

### The signature
The Header and Payload is signed and a signature digest is generated, that is attached at the end of the JWT.  
You can use different means to sign a JWT described in "Signing algorithms".

A general rundown of how this happens in practice is the following: you base64Url encode the JSON header and payload separately. Then, take the two values and join them with a dot. (base64UrlEncodedHeader.base64UrlEncodedPayload). This single value is used to generate a digital signature, using a signing algorithm. Say HMAC-SHA256 when using a private key:
`HMACSHA256(base64UrlEncodedHeader + "." + base64UrlEncodedPayload, your-256-bit-secret)`

The digital signature is base64Url encoded and attached to the end, so you end up with:  
`base64UrlEncodedHeader.base64UrlEncodedPayload.base64UrlEncodedSignature`

### Keys and Signing algorithms

#### JSON Web Algorithms (JWA)
Algorithms are generally broken into the signature algorithm family and the hashing algorithm.  
E.g. RS256 means the RSA (RSASSA-PKCS1-v1_5) algorithm family using a SHA 256 hashing algorithm.

Note: With asymmetric keys, use RSAASSA-PSS (PS256) and not the PKCS1-v1.5 (RS256) for digital signatures.

Another popular algorithm is ECDSA. It's harder to break than RSA, it can use shorter keys, and generates shorter signatures. A short key of 256 bits provides same strength as 3072-bit RSA key.

Note. Use a deterministic variance of ECDSA if you decide to use this algorithm.  
ECDSA is hard to implement, so there are cases where the signature validation has been bypassed.

Consider using EdDSA instead. The EdDSA offers:
- Fast signature creation and validation
- Short signatures
- Modern cryptography doesn't have same vulnerabilities as older algorithms.

EdDSA is not offered by all JWT libraries. E.g. Microsofts identity library doesn't have EdDSA yet (as of .NET8).

HMACs are an alterantive to digital signatures. HMACs offfer:
- Integrity
- Authenticity
- Superior speed

It doesn't offere non-repudiation because the private key must be shared with everyone that need to verify the signature.

Use HS256 (HMAC SHA256) to sign JWTs if there's only a single JWT issuer and validator.


#### Asymmetric key
Public-Private Key cryptography ensures that only the party with the private key can generate new JWTs, while everyone with the public key can verify the digital signature. This approach provides:
- Authentication
- Integrity
- Non-reppudiation

When using asymmetric keypair for signing, you may also put the public key's ID in the header, so others know which public key they can use to verify the JWT signature.

```jsonc
{ // header
    // other fields
    "kId": "some_key"
}
```

##### X.509
It may be a good idea to provide an X.509 public certificate that others can verify the JWT with. When doing this, you freely distribute the X.509 certificate, and communicate this in the header part:

```jsonc
{ // header
    // other fields
    "x5t#S256": "thumbprint" // x.509 SHA-256 certificate thumbprint
}
```
In a JWT header, the x5t#S256 field represents the X.509 certificate SHA-256 thumbprint.
The x5t#S256 is the base64url-encoded SHA-256 hash (also known as thumbprint or digest) of the DER (a specific binary format) encoding of the X.509 certificate.  

By comparing the x5t#S256 field to the SHA-256 hash of your available certificates, you can determine which certificate was used.

#### Symmetric key
With only a private key you can issue and validate tokens that you've created yourself. Others won't be able to verify the signature, since you'll need the private key for that.

Symmetric key encryption with HMAC-SHA256 is fine if you control the resource and identity server, and just want to issue keys that are used within your system.

#### JSON Web Key (JWK)

## JSON Web Encryption (JWE)
Creating digital signatures is slow. Especially when signing large amounts of data, such as a JWT.

Instead, JWE uses a combination of asymmetric encryption and the faster symmetric encryption.

https://app.pluralsight.com/course-player?clipId=11d367d5-cee4-466b-b666-bfb50129b332


## Resources
- https://www.scottbrady91.com/c-sharp/eddsa-for-jwt-signing-in-dotnet-core
- https://github.com/scottbrady91/IdentityModel