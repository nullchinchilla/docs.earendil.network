# Architecture

## Nodes

There are two kinds of nodes in the network:

* **Relays** form the backbone of the Earendil network, and relay messages between their **neighbors**: nodes that are directly connected to this relay.
* **Clients** do not relay any traffic, and they access the network with the help of relays. None of their neighbors can be other clients.

Earendil operates on an _invite-only_ model: to bootstrap into the network, every node (relay and client) must _manually_ configure at least one node as a neighbor. This means obtaining their **route secret**: a secret document that describes how to reach that node.

The secrecy of route secrets is critical to the type-II censorship resistance of the network (though not its confidentiality or its type-I censorship resistance), so nodes in censored regions would generally be very tight-lipped about these secrets except to trusted friends.&#x20;

Nodes in the "free world" may publicly list their route secrets (e.g. on some website) to help other fellow "free world" netizens bootstrap, but it must be noted that these netizens' ISPs will easily tell that they are using Earendil as a result.

## Routing

### Public relay graph

Up to now, this sounds like a typical friend-to-friend or "darknet", similar to a proposed model for the Freenet project, where every node in a peer-to-peer network only knows its neighbors and is forbidden access to information about further nodes.

However, Earendil departs from this model in that all nodes do know the **relay graph** of the network: what relays there are (identified by public key fingerprint) and what relays are neighbors to which relays, even though the complete "contact information" contained in the route secrets is kept secret.

This is done by continually gossipping through the network **adjacency descriptors**: documents signed by both neighbors of a connection that contain their unique fingerprints, but no other route information.

### Packet-based onion routing

With the entire relay topology, an arbitrary _client_ can send a message to an arbitrary _relay_ with **onion routing**. The basic procedure is analogous to other onion routing systems:

* Client finds a path through the relay graph, starting at itself and ending at the relay. This can be done through a shortest-path algorithm or any other heuristic, depending on the client's desired anonymity/performance tradeoff.
* Client produces a nested-encrypted [onion packet](onion-packet-format.md). Given intermediate relays $$R_1,\dots,R_n$$, the packet is roughly the fingerprint of $$R_n$$ attached to the message encrypted to the pubkey of $$R_n$$, then encrypted to the pubkey of $$R_{n-1}$$ with the fingerprint of $$R_{n-1}$$attached, etc.
* Client sends the onion packet to $$R_1$$. Every relay "peels off" one layer of of encryption, revealing the next hop, to whom they send the remaining layers.
* Eventually, $$R_n$$ gets the fully "peeled off" message: a message encrypted to its pubkey, which it then decrypts and reads.

This has the nice property that no relay, including the destination, knows the entire path. Given paths picked from a high-entropy enough probability distribution, this provides strong, Tor/I2P-like anonymity to the user (modulo timing and other side-channels)

### Relay->client packets

The above method only works for unidirectional communication from a client to a relay. This is not enough for a useful network.

Fortunately, as long as the client contacts the relay first, relays can send messages back to clients while preserving the client's anonymity. This uses [reply blocks](reply-blocks.md).

### Client->client packets

Client-to-client communication uses **rendezvous routing**. In short, if Alice and Bob want to communicate, this procedure is followed (WLOG we assume that Alice is sending a packet to Bob rather than the other way around):

* (This step only needs to be done infrequently, as a "setup") Bob picks a relay, Rob, in the network to act as his **rendezvous relay**.
* Bob arranges with Rob so that when Rob receives messages with a particular unique tag, they are forwarded to Bob, using the usual relay->client reply blocks.
* Bob publishes a signed binding into a network-wide **rendezvous DHT**, binding his fingerprint to a **rendezvous descriptor**, containing Rob's fingerprint and the Bob-specific unique tag.
* When Alice needs to send a packet to Bob:
  * She looks up Bob's fingerprint in the rendezvous DHT, getting Rob's fingerprint
  * She sends usual client->relay packets to Rob tagged with Bob's unique tag
  * Rob forwards the content to Bob using the relay->client system to Bob

Bob's choice of his "Rob" depends on his anonymity needs. If he just wants to optimize for performance, simply picking his fastest neighbor works. But if he wants to optimize for anonymity, he should pick a relay from the whole network uniformly, without taking into account his position in the network.
