# About

{% hint style="warning" %}
Currently, Earendil is in its earliest stages of development.&#x20;

The following is an _aspirational_ README describing the goals of the project. Very few features are done at the moment.
{% endhint %}

[**Earendil**](https://github.com/nullchinchilla/earendil) is a decentralized, censorship-resistant packet-routing overlay network designed for performance and censorship resistance. It allows any two nodes connected to Earendil to communicate freely, even against powerful state-level attackers.

At first sight, Earendil seems similar to existing peer-to-peer onion routing networks like I2P or mixnets like Nym. But it has several distinguishing features:

## Robust censorship resistance

Earendil resists both [type-I](https://nullchinchilla.me/2023/05/two-kinds-of-censorship-resistance/) censorship (removing content or users from the network) and [type-II](https://nullchinchilla.me/2023/05/two-kinds-of-censorship-resistance/) censorship (blocking access to Earendil entirely). Strong type-II censorship resistance is rare in other projects. Even when present, it's generally limited to special-case defenses (e.g. Tor obfuscated bridges) against nation-state firewalls like the Great Firewall of China.

On the other hand, Earendil is designed to work _even_ _if the GFW were deployed worldwide_. It makes no assumptions as to most of the network existing in the "free world". This is possible because:

* Earendil traffic is, by default, difficult to distinguish from "normal" network traffic. Furthermore, the protocol used for any particular node-to-node link can be switched out, using a "pluggable transport" architecture similar to those used for Tor bridges, for particular severe network environments (e.g. networks that only allow plaintext HTTP and man-in-the-middle all HTTPS traffic)
* Earendil routes traffic using a unique "friend-to-friend" routing system that does not reveal information about the entire network to every node, making it difficult for even powerful attackers to compile a list of Earendil nodes useful for surveillance or censorship.

## Confederal, non-egalitarian topology

Earendil embraces a [confederal](https://nullchinchilla.me/2023/03/confederal/) rather than classical peer-to-peer architecture. This means that we use a server-client (or "supernode-node" if you prefer) distinction for its scalability and usability benefits: users who do not choose so do not have to contribute infrastructure to the network.&#x20;

But unlike federated protocols like Matrix, we retain strong decentralization through user sovereignty and choice:&#x20;

* Clients can switch servers seamlessly, and anyone can run a server. There's a free and competitive marketplace for servers.&#x20;
* Clients do not have to share servers in common to communicate. Servers generate no network effects that lock in their clients.
* Clients end-to-end encrypt all messages among themselves.

## Decentralized, sybil-resistant incentives

Earendil **optionally** allows every node to set a price that their peers must pay to consume its resources, through cryptocurrency micropayments. This is only possible due to an integration with Astramel, a light-client-centric, privacy-protecting payment channel network built on the Mel blockchain.

Micropayments elegantly solve sybil-resistance (preventing bad nodes from flooding the network), incentives for honest nodes, and DoS resistance. Nodes behaving badly will not be paid by their peers, while honest nodes are incentivized to compete in a free market to provide the best service to their clients. Bad actors attempting to spam Earendil must pay the network accordingly.

This has important advantages over other incentive/sybil-resistance mechanisms:

* _Requiring nodes to consume some fixed resources to join_ (proof of work, proof of stake, proof of donation, etc) involves difficult central planning and economic inefficiency. \[...]
* _Centralized whitelisting_ \[...]
* _"Tokenomic" incentives_ \[...]

## User-tunable anonymity/performance tradeoff

