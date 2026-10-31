# Addresses, Routes and DNS

*Documented-only, like chapter 7: these are
controller settings, and the lab ran no
controller. The settings described are the
controller's own configuration model — the
same fields whether the face on them is
Central or a self-hosted console.*

The outcome: on a network you administer,
you can decide where members' addresses
come from, what extra roads the network
hands them, and what names mean — and you
know the one option that must be granted
twice before it works.

## Managed addresses

When a member is authorised, the
controller assigns it a **managed IP** from
the network's **IPv4 auto-assign pool** —
the range chosen at creation (chapter 7),
one of the ordinary private ranges
(`10.x`, `172.x`, `192.168.x` presets, or
your own). The address appears in
`listnetworks` on the node, on the
interface the operating system created,
and in the console's members list, where
an administrator can override it with a
fixed one. Pin addresses for anything
other machines will look for by number —
servers, printers, the box running DNS —
and let laptops float.

Two collision warnings do real work. The
pool must not overlap the physical LANs
your members join from (chapter 7's
warning bears repeating: a virtual
`192.168.1.0/24` on a home network that is
also `192.168.1.0/24` makes routing
ambiguous in ways that present as hauntings).
And "managed" means *the controller's
word*: a member can also be given addresses
by hand on its interface, but the network
will not vouch for them, and the rules
engine (chapter 10) can be told to drop
frames claiming unauthenticated addresses.

## Addresses a node computes for itself

IPv6 on ZeroTier has two extra modes in
which no pool is needed, because the
address is *derived* rather than assigned:

- **RFC 4193** — each node gets a unique
  address in a private IPv6 range derived
  from the network ID and its own node
  address.
- **6PLANE** — a deterministic address that
  embeds both IDs in a fixed pattern; the
  CLI exposes it directly:
  `zerotier-cli get <network ID> ip6plane`
  (that setting is named in the 1.16.2
  help text the lab captured). Two nodes
  can compute each other's 6PLANE address
  from public information, which makes it
  handy for scripts and strangely durable
  documentation examples.

Both can run alongside pool assignment;
they are conveniences layered on the same
membership, not separate networks.

## Managed routes: roads beyond the segment

A member can be told about networks
*beyond* the virtual one. Central's
**Managed Routes** list holds entries of
the form *destination → via*: a CIDR range
and, optionally, the managed IP of a
member that acts as the gateway to it.
The classic uses:

- **Reach a whole LAN through one member.**
  A node sitting on the office LAN joins
  the network; add a route for the LAN's
  range via that node's managed IP, and
  every member can address machines that
  have never heard of ZeroTier. (The
  gateway node itself must be configured
  to forward — an operating-system setting
  on that machine, not a ZeroTier one.)
- **An exit node.** Add `0.0.0.0/0` via a
  member, and that member becomes a
  candidate default route — the whole
  internet, tunnelled through your server.

The exit-node pattern contains the
chapter's must-grant-twice option: a
managed default route is *offered* to
members, but a member only **accepts** it
if it has been set to allow overriding its
default route (`zerotier-cli set <network ID>
allowDefault true` on the node; the
setting exists in the node JSON the CLI
reads). A network cannot silently seize a
member's internet connection — the member
opts in, explicitly. That asymmetry is a
security feature wearing a settings page.

## DNS: names for the network

The controller can carry a **DNS
configuration** for the network: a search
domain (say `example.internal`) and a list
of DNS server addresses. Members whose
platform supports pushed DNS — the desktop
and mobile apps handle it; behaviour on
Linux varies with the system's resolver
setup — then resolve names inside that
domain through those servers, typically
ones *on* the virtual network: the DNS
guide and Pi-hole guide in this series
both describe servers that would happily
take the job. Treat pushed DNS as a
convenience to verify on each platform you
use, not a contract: check with
`nslookup` (or your resolver's own tools)
that queries for the network's domain
actually land where you intended,
especially before you lean anything
important on the names.

Addresses tell members where things are.
Chapter 10 is about the complementary
power: deciding what they are allowed to
say to each other once they get there.
