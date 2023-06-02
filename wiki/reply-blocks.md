# Reply blocks

A particular challenge in packet-based onion routing is when Alice sends a message to Bob, how can Bob reply.

The trivial solution is to attach a header to the message with a "source address". This is okay in the normal case, where anonymity against the counterparty is not desired, but not okay if Alice wants to hide her identity from Bob.

In that case, we use **reply blocks**. Let's say $$A$$ sends a message to $$B$$ over intermediary relays $$R_1,\dots,R_n$$. To help $$B$$ respond, $$A$$ attaches a reply block containing:

* An arbitrary, fixed **source ID**
* The first hop on the return-path, $$R_n$$. WLOG we assume there is only one path between the parties, and the reply packet follows the same path but in the other direction.
* An onion-encrypted header encoding the path $$R_n,\dots,R_1$$, as well as ephemeral pubkeys for each layer
* A randomly generated key $$k_r$$

$$B$$ then AEAD-encrypts the response with $$k_r$$ with a random, attached nonce, attaches the header verbatim, then sends a packet off to $$R_n$$.

The message will eventually pass back to $$A$$, garbled by the intermediary nodes' attempts to "decrypt" the message with layers of stream-ciphers. But $$A$$ can reconstruct the keystream that was XOR'ed against the AEAD-encrypted payload, and thus reconstruct the ciphertext that $$B$$ sent.

$$A$$ is then able to decrypt the response from $$B$$.

## Reusing reply blocks?

Can reply blocks be reused? The main danger seems to be that when a reply block is reused, the nodes along the way will know that it's the same reply block being used (since the header will look the same).

But given different nonces for the AEAD encryption, this seems fine?

Reply-block reuse has the advantage of saving bandwidth, but how much bandwidth can that save? Is it a lot?

We can periodically send packets full of nothing but reply blocks, so that lack of reply blocks is never an issue, and per-packet overhead is minimized.

## How this is exposed in the API

The `recv_from` function will return a packet and _source_. This source is either

* a _direct_ address: a fingerprint of a relay or client
* a _reply_ address: something encoding the source ID

When `send_to` processes a reply address, it looks up an (unused?) reply block corresponding to the source ID and routes the message through it.

## DoS resistance

Only the last N reply blocks received from a source, within the last T seconds are kept around. Memory usage can be tightly bound with something like a weighted `moka` cache.

Source IDs that are too inactive become dead.
