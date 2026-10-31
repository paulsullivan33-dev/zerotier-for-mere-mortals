# Troubleshooting

Each entry is a symptom, what it actually
means, and the check that separates the
causes. The lab met the first two in
person; the rest follow from the messages
and states the tools themselves produce.

## `info` says OFFLINE

The daemon cannot reach the roots — an
underlay problem, before any network
enters into it. Check, in order:

1. **Is it just young?** A node takes a few
   seconds after starting. Wait ten, ask
   again.
2. **Is UDP getting out?** The lab's
   defining failure: the kernel refused its
   UDP outright
   (`sendto failed: [Errno 1] Operation not permitted`),
   and no configuration could have fixed
   it. On a real machine the equivalent is
   a firewall or gateway blocking outbound
   UDP, or a network that only permits a
   web proxy. ZeroTier wants UDP 9993 and
   peers on assorted ports — "don't restrict
   outbound UDP" is the project's own
   network guidance.
3. **Did it fall back to TCP?**
   `zerotier-cli peers` prints
   `NOTE: Currently tunneling through a TCP relay. Ensure that UDP is not blocked.`
   when it believes it has, and
   `zerotier-cli -j status` shows
   `tcpFallbackActive`. TCP fallback is
   life support, not health — find the UDP
   blockage.
4. **Is the daemon's clock sane?** Signed
   configurations and identities are
   time-sensitive; a machine with a wildly
   wrong clock can fail checks that look
   like network failures.

## A network sits at ACCESS_DENIED

The healthy waiting room (chapter 8): the
controller knows the node and has not
authorised it. Confirm the address in the
console's members list matches
`zerotier-cli info` on the node — a
mismatch means you are looking at a
different node than you think (usually a
reinstall minted a fresh identity). Tick
Auth; the node flips to OK on its own. If
the network is *public* and you still see
ACCESS_DENIED, you are reading the status
of a different network ID than the one you
created — count the hex digits.

## listnetworks is empty after a join

Join records a request; it does not create
a row until a configuration exists to
describe (transcript t03 is the pure
case: `200 join OK`, a `.conf` file in
`networks.d/`, and silence). If the node
is ONLINE and stays silent for a minute,
the request is not reaching a controller —
the network ID is wrong (one wrong digit
names a controller that will never answer;
IDs are not checksummed for your comfort),
or the controller for that ID is a
self-hosted one that is down.

## Status OK, address present, cannot ping the other member

The network is fine; something narrower is
wrong. Check that the *other* member is
also OK and authorised (a deauthorised
peer drops out of the world quietly);
that no flow rule drops the traffic —
chapter 10's engine is stateless, so check
for a rule allowing the *reply* direction
too; that the target service listens on
the ZeroTier interface and not only on the
physical one; and that both nodes are not
fighting a managed-pool collision with
their local LANs (chapter 7's warning —
symptoms are "works from some places, not
others" and routes that point somewhere
embarrassing; `ip route get <managed IP>`
tells you which interface a packet would
really take).

## Everything is RELAY, nothing is DIRECT

Paths fall back when direct UDP between
the two nodes cannot get through. The
usual causes: one or both ends behind
**symmetric NAT** (the hostile kind,
common on mobile networks and some
business gateways); double NAT (router
behind router); or an outbound firewall
passing UDP to the roots but not to
arbitrary peer ports. UPnP or NAT-PMP on
the router, or real IPv6 at both ends,
fixes most of it; a **moon** placed on a
well-connected address (chapter 6) gives
the pair a nearer, better relay when it
cannot.

## PORT_CONFIGURATION_ERROR

The configuration arrived but the
interface could not be built. On Linux
this is usually the tunnel device: the
daemon needs root (or the equivalent
capability) and access to `/dev/net/tun`.
The lab could open its sandbox's tun
device, and it still would not have
helped — configuration never arrived to
configure it with. Order your suspicions:
ONLINE first, authorisation second,
device last.

## The CLI says "500" or cannot find the service

`zerotier-cli` talks to the daemon's API on
127.0.0.1:9993 and discovers it through
`zerotier-one.port` and `authtoken.secret`
in the daemon's home. Point it at the
right home with `-D<path>` if the daemon
runs somewhere non-standard (the lab did
this for every command), and confirm the
daemon is actually running — a stale home
directory reads exactly like a broken one,
minus the process.

## A node vanished after a reinstall

Reinstalling (or wiping the home) mints a
**new identity** unless the old
`identity.secret` was preserved — the node
in the console is the *old* address,
sitting authorised and silent forever.
Either restore the identity (chapter 12's
move recipe) or authorise the new address
and delete the ghost from the members
list, so the list stays an honest
inventory.
