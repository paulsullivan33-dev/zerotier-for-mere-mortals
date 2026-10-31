# What ZeroTier Is

You have machines in more than one place —
a desktop at home, a laptop that travels, a
little server in a cupboard or a data
centre — and the ordinary internet keeps
them apart. Each sits behind a router that
gives it a private address and hides it
from the world. Reaching the home machine
from the café normally means punching a
hole in that router: a forwarded port, a
public address, something for the whole
internet to knock on.

**ZeroTier removes the wall instead of
punching it.** It is a program — one
service, `zerotier-one`, running on each
machine — that creates a **virtual network**
and gives every member a virtual network
port on it. The members find each other
across the internet, build encrypted paths
between themselves, and from then on behave
exactly as if they were plugged into the
same Ethernet switch. The operating system
sees an ordinary network interface with an
ordinary private address. Programs cannot
tell the difference, because at the level
they work at, there isn't one.

That last point is the design decision
everything else follows from. ZeroTier
emulates **Ethernet** — the "layer 2" of
networking, where machines address each
other by hardware-style addresses and
broadcasts reach the whole segment. Most
VPN-style tools work one layer up, at the
IP layer ("layer 3"), carrying addressed
packets between known endpoints. A virtual
Ethernet carries anything Ethernet carries:
IPv4, IPv6, broadcasts, the chatty
discovery protocols gadgets use to find
each other. A game that looks for a LAN
server, a printer that announces itself —
on a ZeroTier network they simply work,
because as far as they are concerned they
*are* on a LAN. ZeroTier calls this virtual
layer VL2, riding on a peer-to-peer
underlay it calls VL1.

## A first taste

On a machine where ZeroTier is installed
and a network already exists (chapter 7
covers where networks come from), joining
is one line, and the manual page shipped
with the program uses ZeroTier's own big
public test network, **Earth**, as the
example:

```bash
sudo zerotier-cli join 8056c2e21c000001
sudo zerotier-cli listnetworks
```

Wait a moment, and the second command shows
a network interface and an assigned address.
From then on the machine is *on* that
network — pingable, SSH-able, file-shareable
by every other authorised member, from
wherever they are. Leaving is the same
command with `leave` in place of `join`.

## What it is not

ZeroTier is not an anonymiser: it connects
*your* machines to each other, it does not
hide you from the internet, and it is not
trying to. It is not a remote-desktop or
file-sharing program: it builds the network
those programs then run over (the SSH guide
in this series is the natural companion —
SSH over ZeroTier is the combination most
people actually live in). It is not a
replacement for a firewall inside the
network — though it grows one of its own,
in chapter 10. And it is not magic about
membership: knowing a network's ID lets a
machine *ask* to join, and on a private
network — the default, and the setting you
should keep — the network's owner must
approve the request before a single packet
flows. Chapter 8 lives in that approval.

## Who it is for

The person with a home server they want to
reach from anywhere without exposing it.
The small team whose machines are scattered
across houses and offices and should still
form one workgroup. The maker whose
Raspberry Pi sits behind a router they do
not control — a landlord's, a mobile
network's — and still needs to be reachable.
The common thread is a handful of machines,
all *yours*, that should form one private
network despite the internet's best efforts
to keep them strangers.

## The honest catch, up front

Two catches, and this guide will not bury
either. First: a ZeroTier network is a real
network, with a real network's dangers. Any
machine you authorise can knock on every
service the other members are running — the
laptop you approve today is inside the
perimeter tomorrow, malware and all.
Authorise devices the way you would hand
out house keys, because cryptographically
that is close to what you are doing.

Second: ZeroTier is not fully
self-contained. Your traffic is encrypted
end to end and usually travels directly
between your machines — but *membership*
is managed by a **controller**, normally
ZeroTier's own hosted service, ZeroTier
Central. The controller decides who is in;
it cannot read what members say (chapter 11
draws that line precisely). If the hosted
controller disappeared tomorrow your
existing networks would keep passing
traffic for a while, but no new member
could be approved. Chapters 6 and 7 cover
the escape hatches — moons and self-hosted
controllers — and chapter 14 prices the
dependency honestly. And one more, new
since older guides were written: the
current 1.16 builds are **not** open source
in the old sense. The binary prints it in
its own help text — a source-available
licence for non-commercial use, with
commercial use requiring a paid plan.
Chapter 14 comes back to that too.
