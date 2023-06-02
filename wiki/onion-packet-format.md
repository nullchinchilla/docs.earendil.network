# Onion packet format

The onion-packet has two parts:

* The **header** which includes authenticated-encrypted routing messages
* The **body** which is encrypted with layers of ChaCha20, with intentionally no authentication.

## Box encryption

**Box encryption** (named after the similar construction from NaCl) is a generic way of encrypting a message, with integrity protection, so that only the owner of a particular X25519 secret key can read it.

The format is:

* 32 bytes: a X25519 ephemeral public key from the sender
* `n` bytes: encrypted message, with ChaCha20Poly1305 (RFC 8439), encrypted with the key derived from blake3-hashing the X25519 shared secret computed from the sender's ephemeral key and the X25519 key of the recipient, and an all-zero nonce.

## Header format

The header for a message whose first hop is "Rob" will have:

* Box-encrypted to Rob's X25519 public key:
  * 20 bytes: fingerprint for next hop
* ChaCha20-encrypted (no authentication, no nonce) with `KDF(s, "header")` where `s` is the shared-secret between the sender and our X25519 keys.
  * The rest of the headers
  * As much randomness to pad the entire header to 520 bytes

## Body format

The body is ChaCha20-encrypted with `KDF(s, "body")`
