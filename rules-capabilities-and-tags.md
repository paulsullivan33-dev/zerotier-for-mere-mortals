# Rules, Capabilities and Tags

*Documented-only, from ZeroTier's own
knowledge-base article on its rules engine
(listed in `zerotier-lab/sources.txt`) and
the manual pages. The lab administered no
network; it can, however, vouch for the
shape of the system from the artefacts it
did run — the certificates and credentials
below are what a controller signs, and the
daemon enforces them.*

The outcome: you know that a ZeroTier
network can have a firewall inside itself,
how that firewall thinks, and why it
scales better than a pile of per-machine
rules.

## A switch with opinions

Plain Ethernet trusts the segment: any
port can send anything to any port. A
ZeroTier network starts more cautiously.
The controller issues each member a signed
configuration that includes **flow rules** —
a small program in a rule language, run
against every frame. Rules match on the
things a switch can see: Ethernet type,
source and destination (ZeroTier address,
MAC, IP), IP protocol, ports, and more;
actions include `accept`, `drop`, and
`tee`/`watch` (copy a frame to a watching
member — the network can have a designated
witness). Rules are evaluated in order,
and what no rule accepts is dropped.

Two properties, from ZeroTier's own
knowledge base, define the engine's
character:

- **It is stateless.** There is no
  connection tracking, no "replies to
  established sessions are fine" shortcut.
  If you want A to be able to *answer* B,
  a rule must allow the answering traffic
  too. ZeroTier calls this a deliberate
  trade-off, and its knowledge base
  publishes patterns for the common shapes.
- **It is enforced at both ends.** The
  sender's node applies the rules to
  outgoing frames and the receiver's node
  applies them again to incoming ones. To
  smuggle a forbidden packet, an attacker
  would have to compromise *both* nodes'
  daemons — a member running a hacked
  client still cannot talk to an honest
  one outside the rules.

A typical starting posture (the knowledge
base's own example shape): drop anything
that is not IPv4, ARP or IPv6, and drop
frames whose IP source is not the address
the controller authenticated; then accept
the services you mean to offer — say, TCP
to ports 22, 80 and 443 — and let the
default drop handle the rest.

## Capabilities: permission slips

Writing every exception into the base
rules would make them enormous. A
**capability** is a separate, numbered,
controller-signed bundle of extra rules,
issued to *some* members: "whoever holds
capability 100 may reach the web servers
on 80/443." When a member presents a
capability, its rules are consulted after
the base set. Think of the base rules as
the building's standing orders and a
capability as a signed pass: presentable,
revocable by the issuer, and worthless to
anyone it wasn't issued to, because the
signature won't verify for them.

## Tags: labels the rules can read

**Tags** are controller-signed labels —
key/value pairs stamped onto members: a
`department` tag reading `engineering` or
`sales`, a `clearance` tag with a number.
Rules can match on them (`teq`, "tag
equals", and friends). The knowledge base's
showcase example is file sharing allowed
only between members whose `department`
tags agree — a rule that stays one line
long no matter how many thousands of
machines join, where per-machine rules
would multiply into millions. Tags and
capabilities together are how the system
does **micro-segmentation** — many small
rooms inside one network — without a
rules file the size of a novel.

## The honest limits of the engine

Statelessness cuts both ways: no stateful
inspection, no notion of "this flow was
legitimate five minutes ago," and ICMP's
error messages — which refer to other
packets — need explicit thought. Rules
also cannot see inside encrypted payloads
(nothing can; that is chapter 11's point):
they filter the envelope, not the letter.
And the rules editor in Central is a text
field with a compiler: it will tell you
about syntax errors, but a rule set that
compiles can still lock every member out
of the network — change rules the way you
would change firewall rules on a machine
you are SSH'd into, which is to say,
carefully, and with the console session
kept open until a second node confirms the
new rules still let it speak.
