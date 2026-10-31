# The Neighbours

ZeroTier is one answer to "my machines
should form one network". This chapter
characterises the other answers honestly —
what each does that ZeroTier doesn't, and
vice versa. The neighbours were not run in
this guide's lab; the two that have guides
of their own in this series are the deeper
references.

## Tailscale — the closest cousin

[Tailscale](https://paulsullivan33-dev.github.io/tailscale-for-mere-mortals/)
solves the same problem with a different
cut. Its data plane is WireGuard — the
lean, kernel-resident protocol — and each
node gets an IP address on a layer-3
network: no virtual Ethernet, no broadcast,
no pretending to be a switch. Coordination
is a hosted service much like Central;
relays (DERPs) cover the cases where direct
paths fail, just as ZeroTier's roots and
TCP relays do. The honest trade: Tailscale
is simpler to hold in your head and its
WireGuard core is famously fast and
audited; ZeroTier's layer-2 emulation
carries things Tailscale won't (LAN
discovery protocols, non-IP traffic,
bridging a whole physical segment in),
offers rules, capabilities and tags inside
the network, and lets you run your own
roots *and* controller with the tools in
the box. People's choices usually turn on
whether they need Ethernet behaviour —
games and gadgets say yes, fleets of
servers usually say no.

## WireGuard by hand — the engine, exposed

[WireGuard](https://paulsullivan33-dev.github.io/wireguard-for-mere-mortals/)
is the protocol Tailscale automates.
Run it yourself and you hold every piece:
your keys, your config files, your choice
of hub. What you give up is everything
this guide spent chapters on — there is
no rendezvous service (one end needs a
reachable address), no membership approval
flow (a peer is a stanza you type), no
address management, no rules beyond what
your firewall already does. For two or
three machines with one public endpoint,
that leanness is a virtue. For a scattered
fleet behind other people's NATs, you
will rebuild ZeroTier badly, slowly,
yourself. The WireGuard guide in this
series builds its lab by hand precisely so
you can feel the difference.

## Nebula — the self-hosted purist

Nebula (from Slack, open source) is layer-3
like Tailscale, but *everything* is yours:
you run a certificate authority, issue each
node a certificate, and run your own
**lighthouse** nodes — its rendezvous
points — and there is no hosted control
plane to depend on at all. Firewall rules
travel inside the certificates'
configuration. It asks more of you up
front (certificates have lifetimes; files
must be edited) and gives back complete
independence. If chapter 14's controller
dependency is the thing that worries you
most, Nebula is the comparison to study.

## Hamachi — the ancestor

LogMeIn Hamachi proved, in the 2000s, that
gamers and small offices wanted exactly
this: a virtual LAN over the internet,
join by name, play. It is a hosted,
proprietary service, much diminished since
its heyday, and ZeroTier is in most
respects its modern, more capable heir —
the same intuition, with better
cryptography, real rules, and self-hosting
doors Hamachi never opened. It is named
here because half the internet's old forum
advice about virtual LANs says "Hamachi"
the way it says "Hoover": the category
had a first famous name.

## The shape of the choice

Hosted membership with layer-2 powers and
self-hosting escape hatches: **ZeroTier**.
Hosted membership, layer-3 simplicity,
WireGuard speed: **Tailscale**. No hosted
anything, certificates and lighthouses of
your own: **Nebula**. No machinery at all,
a handful of machines, keys in files you
wrote: **WireGuard by hand**. None of these
is the wrong answer; they disagree about
where the complexity should live, and this
guide's tool puts it in a controller and a
rule engine so that joining can be one
command. Chapter 14 now turns the same
honesty on ZeroTier itself.
