# Glossary

**6PLANE** — An IPv6 addressing mode in
which a node's address is computed
deterministically from the network ID and
its node address, needing no assignment.

**ACCESS_DENIED** — The listnetworks status
of a node that has asked to join a private
network and has not yet been authorised.

**Authorisation** — The controller-side act
of admitting a node to a network (the Auth
checkbox in Central). The node's half is
joining; this is the owner's half.

**Capability** — A controller-signed bundle
of extra flow rules issued to selected
members, granting permissions beyond the
network's base rules.

**Central (ZeroTier Central)** — ZeroTier
Inc.'s hosted network controller and its
web console, at my.zerotier.com.

**Controller** — The authority holding a
network's definition — members, addresses,
routes, rules — and signing configurations
for its members. Hosted (Central) or
self-hosted in the daemon itself.

**Flow rules** — The per-network rule
program, enforced by every member's daemon
on sending and receiving, that decides
which frames the virtual switch forwards.

**Identity** — A node's key pair and the
address derived from it, stored in
`identity.public` and `identity.secret`.

**Managed IP** — The per-network private
address a controller assigns an authorised
member from the network's pool.

**Managed route** — A route (destination
CIDR, optional via) the controller hands
to all members, used to reach LANs behind
a member or to build an exit node.

**Moon** — A self-hosted root server:
introduces peers and can relay for nodes
that have explicitly joined its world.

**Network ID** — A network's 16-hex-character
name; its first 10 characters are the
address of the network's controller.

**Node** — One machine (or app) running
ZeroTier; one identity, potentially many
networks.

**OFFLINE / ONLINE** — The daemon's report
of whether it can reach the roots at all —
underlay health, before any network's
status is considered.

**Planet** — The official root servers
collectively, and the signed file in the
daemon's home that lists them; the default
"world", named Earth.

**Proof of work** — The deliberate
computation baked into identity generation
that makes addresses cheap to mint singly
and expensive to mint in floods.

**Relay** — Forwarding of encrypted traffic
through a root, moon or the TCP fallback
service when two nodes cannot (yet) talk
directly.

**RENDEZVOUS** — The message by which a
root introduces two nodes, carrying the
endpoint hints both use to attempt a direct
path.

**Root** — A fixed, well-known node that
introduces peers to each other and keeps
track of where nodes can be reached.

**Tag** — A controller-signed label on a
member (a key/value pair) that flow rules
can match on, enabling micro-segmentation.

**VL1 / VL2** — ZeroTier's names for its
peer-to-peer underlay (VL1) and the virtual
Ethernet it presents to the operating
system (VL2).

**ZeroTier address** — A node's public
identifier: 10 hex characters derived from
its public key. Quoted like a MAC address;
used by owners to authorise the node.
