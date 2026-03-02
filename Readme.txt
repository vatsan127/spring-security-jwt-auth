Spring Security JWT Authentication

What is JSON Web Token (JWT) ?
- It provides a secure way of transmitting information between parties as a JSON object.
- This information can be verified because its digitally signed using RSA (public / private key pair) etc.

Advantages:
============

Compact: Because of its size, it can be sent inside an HTTP header itself. And, due to its size transmission is fast.
Self Contained / Stateless: The payload contains all the required information about the user, thus it avoid querying the database.
Can be signed using a Symmetric (HMAC) or Asymmetric (RSA).
Build in expiry mechanism.
Custom claim (addition data) can be added in the JWT.


Where to use?
=============

- Used for AUTHENTICATING (confirming the user identity).
- Used for AUTHORIZATION (checks the user permission before providing the access to resources).
- Used for SSO (Single Sign On) i.e.., Authenticate once and access multiple applications.

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
        "sub": 9876543210,
        "exp": 1709341200,
        "email": test@gmail.com
    }

Signature
    HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret-key)


Header:
-------

- Header contains metadata information about the token.
- typ: Type of token, generally mentioned as "JWT"
- alg: Signing algorithm used like RSA or HMAC etc..,

Payload:
--------

- Contains claims (User information or any additional information is added here)

Claims:
    Registered Claims (Predefined names and meaning):
        - iss (Issuer): entity that issued the JWT
        - sub (Subject): identifies the user
        - aud (Audience): identifies the token for which token is intended
        - exp (expiration time): after this time, token is not valid
        - nbf (not before): Time before this, token should not be accepted
        - iat (Issued At): time at which token is issued.
        - jti (JWT ID): Unique JWT ID.
    Public claims: Its a custom claim which can be shared and understood by multiple parties.
    Private claims: Its a custom claim which are intended only for internal use only and not standardized, nor expected that other parties understand this



Signature:
----------
- Encode JWT Header and Payload sperately using Base64 encoding.
- Concatenate the Encoded Header and Payload strings using "."
- Use Symmetric or Asymmetric to create a digital signature.
- Encode the signature generated
- Concatenate with "."