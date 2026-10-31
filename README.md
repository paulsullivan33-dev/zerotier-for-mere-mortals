# ZeroTier for Mere Mortals

You have a computer at home, a laptop in a
café, and a small server in a rack somewhere,
and you would like them to behave as if they
were plugged into the same switch in the same
room — same private addresses, reachable from
each other, invisible to everyone else.
**ZeroTier** does exactly that. It is a
program that builds a **virtual Ethernet
network** over the ordinary internet: every
machine that joins gets a virtual network
port, a place on a private network that
exists only as software, and encrypted
paths to the other members. No port
forwarding, no public addresses to hand
out, no router configuration at either end.

This guide explains how ZeroTier works, how
to install it, and how to read what it tells
you. Then it does something most networking
guides skip: it shows you a node that
**cannot connect at all**, and reads the
failure honestly. The lab that wrote this
guide ran the real service — **ZeroTier One
1.16.2**, the genuine signed package, on a
small Linux machine — inside a sandbox whose
outgoing **UDP** traffic is refused by the
kernel. ZeroTier finds its way around the
internet with UDP, so the lab's node sat at
`OFFLINE` for its whole life. That turns out
to be a gift: everything that does *not*
need the network — the identity a node is
born with, the control API on its own
machine, the shape of a join request, the
exact wording of the failure — could be
examined slowly and quoted verbatim.

What the lab could not do, it did not fake.
It never joined a real network, so creating
a network in ZeroTier's web console,
authorising a member, and watching traffic
flow are taught from ZeroTier's own manual
pages, its knowledge base, and the
documentation of the controller itself —
and labelled as documented-only wherever
they appear. No account was created for
this guide. `criteria.md` keeps the full
ledger of what was run and what was only
read.

One warning belongs up front, because it
shapes how you should use any tool like
this: a ZeroTier network is a real network.
A machine you authorise can reach the other
members as surely as if it were plugged
into your switch at home — that is the
point of the tool, and also the risk.
Chapter 1 states the ethics of it plainly,
and chapter 11 explains what ZeroTier's
servers can and cannot see of your traffic.

## What this guide covers

- [**What ZeroTier is**](what-zerotier-is.md) —
  the mental model: a virtual Ethernet
  switch stretched over the internet, and
  what "layer 2" buys you that a tunnel
  doesn't
- [**The moving parts**](the-moving-parts.md) —
  the daemon, the command-line tool, the
  identity tool, the local API, and the
  files in the daemon's home — and which
  part holds what
- [**Installing ZeroTier**](installing-zerotier.md) —
  the official package, its signature
  checked link by link, and the version
  command that proves it runs
- [**Identity — your address and your keys**](identity-and-addresses.md) —
  the 10-character address every node is
  born with, the proof of work inside it,
  and the one file you must never lose or
  leak
- [**Reading your node**](reading-your-node.md) —
  `info`, `listpeers` and the local API:
  what a healthy node reports, and what
  this lab's OFFLINE node reported instead
- [**Roots, moons, relays and direct paths**](roots-moons-relays-and-direct-paths.md) —
  how two machines behind home routers find
  each other, why most traffic goes direct,
  and what relays and moons are for
- [**Networks and ZeroTier Central**](networks-and-central.md) —
  where networks are defined: the
  controller, the web console, and the
  16-character network ID (documented)
- [**Joining and being authorised**](joining-and-authorising.md) —
  the join itself, the wait for
  authorisation, and the managed address
  that arrives when it comes
- [**Addresses, routes and DNS**](addresses-routes-and-dns.md) —
  address pools, the addresses a node can
  compute for itself, managed routes, exit
  nodes, and pushing DNS settings
- [**Rules, capabilities and tags**](rules-capabilities-and-tags.md) —
  the firewall that lives inside the
  network: flow rules, and the signed
  labels that keep them manageable
- [**The security model**](the-security-model.md) —
  what is encrypted, who holds which keys,
  and what the controller and the roots
  can see (membership, not traffic)
- [**Recipes**](recipes.md) — back up a
  node's identity, move a node to a new
  machine, pre-join a network at first
  boot, and let a non-root user drive the
  CLI
- [**The neighbours**](the-neighbours.md) —
  Tailscale, hand-rolled WireGuard, Nebula
  and the ghost of Hamachi: what each does
  that ZeroTier doesn't, and vice versa
- [**Limits**](limits.md) — the UDP it
  cannot live without, userspace
  performance, the controller in the
  middle of membership, and a licence that
  is not the one you may remember
- [**Troubleshooting**](troubleshooting.md) —
  OFFLINE and its causes, ACCESS_DENIED,
  authorised but unreachable, and the
  relay note the CLI prints when it is
  worried about your firewall
- [**Glossary**](glossary.md) — the terms,
  collected in one place

## What it doesn't cover

Running a production network for a company
— the ideas are the same, but the paid
plans, the audit requirements and the
support contracts are their own subject.
ZeroTier's embedded and router builds are
pointed at, not taught. And this guide
never creates a ZeroTier account: the
console chapters describe the documented
workflow so you can follow it with your
own account open beside you.

## How to use this guide

Read chapter 1 for the model and chapter 2
before installing anything — knowing which
part holds what turns later confusion into
a named problem. Chapters 3 to 5 get you a
node and teach you to read it; chapter 5's
checks are the ones you will reuse every
time something seems wrong. Chapters 7 to
10 describe the other half of the system —
the controller and its console — which the
lab documents rather than runs; keep your
own console open alongside them. The lab's
transcripts live in `zerotier-lab/transcripts/`
and its source list in `zerotier-lab/sources.txt`.

---

## Part of the series

This guide is one of the [For Mere Mortals](https://paulsullivan33-dev.github.io/mere-mortals/) guides: plain-language guides to the tools programmers and makers use.

Related guides:

- [Tailscale for Mere Mortals](https://paulsullivan33-dev.github.io/tailscale-for-mere-mortals/)
- [WireGuard for Mere Mortals](https://paulsullivan33-dev.github.io/wireguard-for-mere-mortals/)
- [Networking for Mere Mortals](https://paulsullivan33-dev.github.io/networking-for-mere-mortals/)

## License

The writing in this guide is licensed under [CC BY 4.0](https://github.com/paulsullivan33-dev/zerotier-for-mere-mortals/blob/main/LICENSE). Code examples and lab scripts are licensed under the [MIT License](https://github.com/paulsullivan33-dev/zerotier-for-mere-mortals/blob/main/LICENSE-CODE).
