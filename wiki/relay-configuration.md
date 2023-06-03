# Relay configuration

Every connection between relays is bidirectional and symmetric once established.

But how these connections are configured is not symmetric. There is a distinction between accepting a connection from a peer passively, and contacting a peer actively.

This is reflected in the configuration file, where the `in_routes` mapping specifies _ways to accept_ incoming connections, while the `out_routes` mapping specifies _specific outgoing connections_.

{% code title="/etc/earendil/config.yaml" %}
```yaml
identity: /etc/earendil/identity.asc
state_cache: /etc/earendil/state_cache.db

# list of all listeners for incoming connections
in_routes:
    # arbitrary names, used for diagnositics and logging
    main_udp:
        protocol: obfsudp
        listen: 0.0.0.0:19999
        secret: correct horse battery staple
    main_http:
        protocol: http-longpoll
        listen: 0.0.0.0:19998
        path: /correct/horse/battery/staple
        tls:
            domain: laboo.example.com
            certificate: autoconf
            
# list of all outgoing connections
out_routes:
    # arbitrary nicknames too
    alice:
        fingerprint: KCKUhWZfluAzMzwiw721CNrvyhc
        protocol: obfsudp
        connect: 100.1.2.3:18232
        cookie: d9aeca8eb2517c18ecf6f24769161be7049187a38c7c8a3391896d502b9bc462
    bob:
        fingerprint: eveIb0XRU8gULsiYxPBa1aUqjy0
        protocol: http-longpoll
        connect: https://nala-goosha.example.com/correct/horse/battery/staple
        tls-fingerprint-seed: helloworld
```
{% endcode %}
