# Onion packet format

The onion-packet has two, fixed-size parts:

- The **header** which includes authenticated-encrypted routing messages
- The **body** which is encrypted with layers of ChaCha20, with intentionally no authentication.

```
Total: 8,872 bytes
  ├─ Header: 680 bytes
  │    ├─ Box-encrypted routing info for first hop: 68 bytes
  │    └─ Onion-encrypted routing info for next hops: 612 bytes
  └─ Onion-encrypted body: 8,192 bytes
```

One important thing to note is that the encoding within the fixed-size body is not specified here, but it _must_ have some sort of integrity protection.

## Box encryption

**Box encryption** (named after the similar construction from NaCl) is a generic way of encrypting a message, with integrity protection, so that only the owner of a particular X25519 secret key can read it.

The format is:

- 32 bytes: a X25519 ephemeral public key from the sender
- ?? bytes: encrypted message, with ChaCha20Poly1305 (RFC 8439), encrypted with the key derived from blake3-hashing the X25519 shared secret computed from the sender's ephemeral key and the X25519 key of the recipient, and an all-zero nonce.

## Header format

The header for a message whose first hop is "Rob" will have:

- 68 bytes: Box-encrypted to Rob's X25519 public key:
  - 20 bytes: fingerprint for next hop
- 612 bytes: ChaCha20-encrypted (no authentication, no nonce) with `KDF(s, "header")` where `s` is the shared-secret between the sender and our X25519 keys.
  - The rest of the headers
  - As much randomness to pad the entire header to 680 bytes

The padding of the header to 680 bytes is done to keep the header size fixed, which helps with anonymity and prevents traffic analysis. The maximum route length is 10 hops, so subsequent peeled header is exactly 68 bytes smaller than the previous one, but with 68 bytes more random noise added to the back. This means the intermediary nodes have no idea where they are in the route.

## Body format

The body is ChaCha20-encrypted at every layer of the onion with `KDF(s, "body")`, and is always exactly 8192 bytes in length.

## Key-derivation function

The function `KDF` is implemented by using `blake3`'s keyed-hash mode, with the "key" being padded to 32 bytes with `_`. For instance, `KDF(s, "header")` means blake3-hashing `s` with the key `"header__________________________"`.
