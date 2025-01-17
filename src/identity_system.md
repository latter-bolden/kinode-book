# Identity System

One of the most important features of a peer-to-peer network is the ability to maintain a unique and persistent identity.
This identity must be self-sovereign, unforgeable, and easy to discover by peers.
NectarOS uses a domain system similar to [ENS](https://ens.domains/) to achieve this.
It should be noted that, in our system, the concepts of `domain`, `identity`, and `username` are identical and interchangeable.

Like ENS, Nectar domains (managed by our NDNS) are registered by a wallet and owned in the form of an NFT.
However, unlike ENS, Nectar domains never expire. Additionally, they contain metadata necessary to both:
- demonstrate the provenance of a given identity.
- route messages to the identity on the Nectar network.

NDNS provides both sensible defaults and flexibility.
The cheapest option is also the default: minting a new NFT, a `.nec` TLD.
However, unlike ENS, NDNS is not restricted to a single set of NFTs.
Instead, it is designed to easily extend and wrap existing NFTs, enabling users to use identities they are already attached to as their Nectar identity.

What does this look like in practice?

It's easy enough to check for provenance of a given identity.
If you have an Nectar domain, you can prove ownership by signing a message with the wallet that owns the domain.
However, to essentially use your Nectar identity as a domain name for your personal server, NDNS domains have routing information, similar to a DNS record, that points to an IP address.

A NDNS domain can either be `direct` or `indirect`.
When users first boot a node, they may decide between these two domain types as they create their initial identity.
Direct nodes share their literal IP address and port in their metadata, allowing other nodes to message them directly.
Again, this is similar to registering a WWW domain name and pointing it at your web server.
However, running a direct node is both technically demanding (you must maintain the ability of your machine to be accessed remotely) and a security risk (you must open ports on the server to the public internet).
Therefore, indirect nodes are the best choice for the majority of users that choose to run their own node (TODO: what about those who have someone else run it? A hosting service?).

Instead of sharing their IP and port, indirect nodes simply post a list of *routers* onchain.
These routers are other *direct* nodes that have agreed to forward messages to indirect nodes.
When a node wants to send a message to an indirect node, it first finds the node onchain, and then sends the message to one of the routers listed in the node's metadata.
The router is responsible for forwarding the message to the indirect node and similarly forwarding messages from that node back to the network at large.

For more information about the architectural specifics of the networking protocol, see [Networking Protocol](./networking_protocol.md).
The main takeaway for the identity system is that *domain provenance* and *domain resolution* are unified by NDNS.

Like .eth for ENS, the NDNS domain space is fixed inside the `.nec` top-level domain.
However, we reserve the ability to expand domain availability in the future, and governance of the Nectar protocol will include the ability to manage domain names.
Eventually, we hope to link various TLDs to existing NFT communities and other identity systems.
