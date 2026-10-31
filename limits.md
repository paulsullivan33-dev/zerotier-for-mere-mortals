# Limits

What ZeroTier cannot do, will not do, or
does only at a price — stated the way the
rest of the guide would want them stated
about any other tool.

**It cannot live without UDP.** ZeroTier's
native tongue is UDP, and networks that
strangle it — hotel and corporate networks
that pass only web traffic, proxies, the
lab's own sandbox — push it onto the TCP
fallback relay or off the air entirely.
The lab's whole existence in this guide
was one long demonstration: `sendto`
refused, `OFFLINE` forever. If your world
is hostile networks, test from one before
committing.

**Relayed traffic is slower traffic.** A
direct path is your packets on the
shortest route the internet offers; a
relayed path detours through someone
else's machine, and a TCP-relayed path
(the fallback of the fallback) adds the
head-of-line queueing of a stream protocol
carrying datagrams. ZeroTier's own numbers
put the relayed share at roughly 4–8% of
traffic — a minority, but if *your* pair
of machines is in it (two symmetric NATs
are the usual cause), it is your whole
experience.

**The data plane is userspace.** The
daemon encrypts and forwards in a user
program, not in the kernel. That buys
portability (the same node runs on nearly
anything) and safety from kernel bugs, and
it costs throughput next to kernel
WireGuard on multi-gigabit paths. For
remote access, file sync and admin traffic
you will never notice; for saturating a
fast link, measure before you migrate.

**Membership has a centre.** Networks are
defined and members authorised by a
controller. On Central — the default —
that centre is ZeroTier's service: your
existing networks keep running if it is
unreachable (nodes hold their signed
configurations and keep talking), but
nobody new can be authorised and nothing
can be redefined until it returns, and
the metadata of membership lives there
(chapter 11). Moons and a self-hosted
controller remove the dependency at the
price of becoming the operator yourself.
This is a genuine architectural limit, not
a bug, and it is the one to weigh longest.

**The licence changed.** Builds up to the
1.14 line were open source in the familiar
sense (the project's earlier licence had a
commercial carve-out). The 1.16.2 binary
the lab ran states its terms in its own
help text: a *source-available licence for
non-commercial use*, with commercial use
requiring a paid subscription or a
commercial licence. Home users are
unaffected in practice; anyone deploying
at work should read the current terms
before, not after. Older how-tos that call
ZeroTier simply "open source" are out of
date.

**Layer 2 is a power with edges.** A
virtual Ethernet carries broadcasts and
multicast to every member — wonderful for
discovery protocols, and a real cost in
chatter on large networks, and a real risk
if a member is compromised: it is *on
your LAN*, with a LAN's lateral reach,
mitigated by rules (chapter 10) rather
than by the tunnel. Networks also have
practical membership ceilings on hosted
plans, and a node's interface count is
one more thing your operating system must
digest.

**It is not a VPN for privacy.** ZeroTier
connects machines you own to each other.
It does not hide your browsing from your
ISP (unless you deliberately build the
exit-node pattern of chapter 9, accepting
a member's word for your whole internet
connection), does not make you anonymous,
and does not pretend to. Tools that
promise those things are a different
category with different trade-offs.

**And the small print of this guide's
lab**: the console chapters are
documented, not run; performance claims
are the project's own numbers, quoted as
such; and nothing here was measured
against a live peer. The limits of the
*guide* are listed in `criteria.md`,
alongside the lab's evidence for
everything else.
