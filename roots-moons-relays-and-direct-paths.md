# Roots, Moons, Relays and Direct Paths

The outcome of this chapter: you can
explain how two machines behind two
different home routers ever manage to talk
to each other — the hardest problem in this
whole subject — and what each supporting
actor (root, relay, moon) actually does.
The lab's node never completed any of this;
its failure is the chapter's running
example, and the mechanics are ZeroTier's
own account of its design.

## The problem: everyone is hiding

A home router hands its machines private
addresses and **NAT** — network address
translation — stands between them and the
internet: outgoing packets are rewritten to
look like they came from the router, and
unsolicited incoming packets have nowhere
to go and are dropped. Two such machines
cannot simply dial each other; neither has
a reachable address. Every tool in this
guide's neighbourhood exists to solve some
form of this problem.

## Roots: the fixed points

ZeroTier's answer starts with **root
servers** — a small set of always-on,
globally distributed machines whose
addresses are published in the signed
`planet` file your daemon carries (chapter
2). The lab decoded its four:

```text
62f865ae71   778cde7190   cafe04eba9   cafe9efeb9
```

A node's first act is to say hello to the
roots over **UDP** (port 9993 to start,
plus the two derived ports you saw in its
status). The roots keep track of where
every node can currently be reached — its
*paths* — the way a phone book keeps
numbers. That is their main job: not
carrying your life, just knowing where
everybody is standing.

## Rendezvous: introductions, then direct

When node A wants to talk to node B, ZeroTier
does not make them shout into the void. From
ZeroTier's own engineering write-up of its
NAT traversal: A sends its first packet for
B *via a root*. The root forwards it — and
also sends both parties a **RENDEZVOUS**
message carrying hints about each other's
apparent addresses. Both sides then try
those addresses with test packets. Home
NATs usually cooperate just enough: the
outgoing test packet opens a pinhole in
the sender's router, and the other side's
packet slips through it. ("Usually" is
measured: ZeroTier's figures say roughly
**92–96% of connections end up direct**,
and only about 4–8% of traffic needs
relaying for good.)

Two properties of this design deserve
emphasis. The introduction traffic and the
backup relay are the *same* path: packets
flow via the root from the first moment,
so a conversation works instantly, and
keeps working if a direct path later
opens, or never does — ZeroTier calls this
*lazy* NAT traversal. And when traffic does
ride a root or relay, it rides as
ciphertext: the root forwards encrypted
frames it cannot read (chapter 11).

If UDP is blocked on the road — a hostile
firewall, or a sandbox like the lab's —
the fallback is a **TCP relay**: the node
wraps its traffic in an ordinary TCP
connection to a relay service. It is
slower and it is a last resort; the status
JSON carries a `tcpFallbackActive` flag
for it, and the `peers` command prints its
pointed note ("Ensure that UDP is not
blocked") when it thinks you have fallen
back. The lab's node could not even begin
this dance: with UDP refused at the first
send, there was no hello, no rendezvous,
no relay negotiation — just `OFFLINE`.
Ports, if you are configuring a firewall
for a real node: allow outbound UDP
generally (peers listen on assorted ports),
expect the node to use 9993 plus two
derived high ports, and enable UPnP or
NAT-PMP on the router if you can — they
let the node ask the router for a proper
mapping and skip the cleverness entirely.

## Moons: roots of your own

The official roots are ZeroTier's. If you
want the introducer layer under your own
control — a fleet on a ship, an office
that must keep working if ZeroTier's
infrastructure is unreachable, a network in
a place with poor routes to the official
roots — you run your own root, called a
**moon**, on a machine with a stable public
address. The moon tooling ships in
`zerotier-idtool`, and the lab captured its
command list from the real binary:

```text
initmoon <identity.public of first seed>
genmoon <moon json>
```

The workflow, documented: `initmoon` drafts
a moon definition from a node's identity;
you edit the draft to list the moon's
stable public endpoints; `genmoon` signs
it into a `.moon` file; members join that
moon-world by placing the file in their
daemon's `moons.d` directory or running
`zerotier-cli orbit <moon id> <seed>` — and
from then on the moon introduces and relays
alongside (or instead of) the planets.
Honesty about scope: a moon is a *root*,
not a controller. It introduces peers and
can relay; it does not define networks or
authorise members. And a moon only helps
people who have explicitly joined it —
your machines will not find it by
themselves.

## The self-hosted controller, in one paragraph

The other supporting role, the controller,
can also be yours: the daemon contains one
(the `controller.d/` in every fresh home),
and networks defined there have IDs built
from *your* node's address. Self-hosting
the controller means the membership of
your networks answers to a machine you
own; it also means that machine must be
reachable and maintained, and its
`identity.secret` becomes the keys to every
kingdom (chapter 4's warning, squared).
ZeroTier Central exists because most people
would rather not. Chapter 7 uses Central;
chapter 14 counts what that choice costs.
