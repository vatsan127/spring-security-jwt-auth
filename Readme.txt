Spring Security JWT Authentication

What is JSON Web Token (JWT)?
- It provides a secure way of transmitting information between parties as a JSON object.
- This information can be verified because it is digitally signed using a Symmetric (HMAC) or Asymmetric (RSA) algorithm.

Advantages:
============

- Compact: Because of its small size, it can be sent inside an HTTP header, making transmission fast.
- Self-Contained / Stateless: The payload contains all the required information about the user, thus avoiding querying the database.
- Built-in expiry mechanism.
- Custom claims (additional data) can be added in the JWT.


Where to use?
=============

- Used for AUTHENTICATION (confirming the user's identity).
- Used for AUTHORIZATION (checking the user's permissions before providing access to resources).
- Used for SSO (Single Sign-On) i.e., authenticate once and access multiple applications.

JWT Structure:
==============

Structure: <header>.<payload>.<signature>

Header (Base64URL encoded)
    {
        "alg": "HS256",
        "typ": "JWT"
    }

Payload (Base64URL encoded)
    {
        "iss": "http://example.com/auth",
        "sub": "9876543210",
        "exp": 1709341200,
        "email": "test@gmail.com"
    }

Signature
    HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret-key)


Header:
-------

- Contains metadata about the token.
- typ: Type of token, generally mentioned as "JWT".
- alg: Signing algorithm used, such as RSA or HMAC.

Payload:
--------

- Contains claims (user information or any additional data).

    Registered Claims (Predefined names and meanings):
        - iss (Issuer): Entity that issued the JWT.
        - sub (Subject): Identifies the user.
        - aud (Audience): Identifies the recipient for which the token is intended.
        - exp (Expiration Time): After this time, the token is not valid.
        - nbf (Not Before): Token should not be accepted before this time.
        - iat (Issued At): Time at which the token was issued.
        - jti (JWT ID): Unique identifier for the JWT.

    Public Claims:
        - Custom claims registered in the IANA JSON Web Token Claims registry
          or defined with collision-resistant names.
        - These are shared and understood by multiple parties.
        - Example:
            {
                "email": "user@example.com",
                "name": "John Doe",
                "picture": "https://example.com/photo.jpg"
            }

    Private Claims:
        - Custom claims intended only for internal use between specific parties.
        - They are not standardized or registered, and other parties are not expected
          to understand them.
        - Example:
            {
                "department": "engineering",
                "employee_id": "EMP-12345",
                "access_level": 3
            }


Signature:
----------

- Encode the JWT Header and Payload separately using Base64URL encoding.
- Concatenate the encoded Header and Payload strings using ".".
- Apply a Symmetric (HMAC) or Asymmetric (RSA) algorithm to create a digital signature.
- Base64URL encode the generated signature.
- Final token: base64Url(header).base64Url(payload).base64Url(signature)


Sequence Diagram:
=================

User                 Auth Server              Resource Server
  |                       |                          |
  |  POST /login          |                          |
  |  (username, password) |                          |
  |---------------------->|                          |
  |                       |                          |
  |                       | Validate credentials     |
  |                       | Generate JWT             |
  |                       |                          |
  |    200 OK + JWT token |                          |
  |<----------------------|                          |
  |                       |                          |
  |                       |                          |
  |  GET /api/resource    |                          |
  |  Authorization: Bearer <JWT>                     |
  |------------------------------------------------->|
  |                       |                          |
  |                       |          Verify signature|
  |                       |          Check expiration|
  |                       |          Extract claims  |
  |                       |                          |
  |                  200 OK + resource data          |
  |<-------------------------------------------------|
  |                       |                          |
  |                       |                          |
  |  GET /api/resource (expired JWT)                 |
  |------------------------------------------------->|
  |                       |                          |
  |                       |          Verify -> FAILED |
  |                       |                          |
  |                  401 Unauthorized                 |
  |<-------------------------------------------------|
  |                       |                          |

Key point: Auth Server and Resource Server never talk to each other.
The Resource Server validates the JWT locally using the shared secret or public key.


Symmetric vs Asymmetric Cryptography:
======================================

Symmetric (e.g., HMAC-SHA256):
------------------------------

- Uses a single shared secret key for both signing and verification.
- Both the Auth Server and Resource Server must know the same secret.
- Faster than asymmetric algorithms.
- Suitable when the signer and verifier are the same service or fully trust each other.
- JWT algorithm examples: HS256, HS384, HS512

    Sign:   HMAC(header.payload, shared-secret) -> signature
    Verify: HMAC(header.payload, shared-secret) -> compare with received signature

Asymmetric (e.g., RSA, ECDSA):
-------------------------------

- Uses a key pair: private key (kept secret) and public key (shared openly).
- Auth Server signs the JWT with the private key.
- Resource Server verifies the JWT using the public key.
- Slower than symmetric algorithms, but the verifier never needs to know the private key.
- Suitable for distributed systems where multiple services need to verify tokens independently.
- JWT algorithm examples: RS256, RS384, RS512, ES256, ES384, ES512

    Sign:   RSA(header.payload, private-key) -> signature
    Verify: RSA(header.payload, public-key, signature) -> valid or invalid


Encoding vs Encryption:
========================

Encoding:
---------

- Transforms data into a different format for safe transport or storage.
- NOT a security mechanism -- anyone can decode it without a key.
- Reversible by design.
- Example: Base64URL encoding used in JWT header and payload.

    Original:  {"alg":"HS256","typ":"JWT"}
    Encoded:   eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
    Decoded:   {"alg":"HS256","typ":"JWT"}   (anyone can do this)

- This is why JWT payload should never contain sensitive data like passwords.
  The payload is only encoded, not encrypted -- it can be read by anyone.

Encryption:
-----------

- Transforms data so that only authorized parties with a key can read it.
- A security mechanism -- cannot be reversed without the correct key.
- Example: AES-256, RSA encryption.

    Original:   "Hello"
    Encrypted:  "3dF8k9..." (unreadable without the key)
    Decrypted:  "Hello"     (only with the correct key)

- JWT uses JWE (JSON Web Encryption) if encrypted payloads are needed,
  but standard JWT (JWS) only signs -- it does not encrypt.

Key Difference:
    Encoding   -> format conversion, readable by anyone (Base64URL in JWT)
    Encryption -> data protection, readable only with a key (AES, RSA)
    Signing    -> integrity verification, proves data was not tampered with (HMAC, RSA)
