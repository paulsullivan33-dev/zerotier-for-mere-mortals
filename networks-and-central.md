# Networks and ZeroTier Central

*This chapter is documented-only. The lab
created no account and no network; what
follows is ZeroTier's documented workflow,
assembled from the manual pages shipped in
the package, ZeroTier's knowledge base,
and the documentation of vendors who build
on the same controller. `zerotier-lab/sources.txt`
lists them. Keep your own console open
beside this chapter and expect small
differences — ZeroTier has been rolling
out a redesigned console ("New Central")
alongside the long-standing one, and the
layout differs while the concepts do not.*

The outcome: you know where a network
*lives* before you touch one, and you can
create one with the settings that matter
set deliberately instead of by default.

## The controller is the network's home

Every ZeroTier network has exactly one
**controller**: the authority that holds
the network's definition — its name, its
member list, its address pools, its routes,
its rules — and issues signed
configurations to members. When a node
joins, it asks the controller "what is
this network, and am I in?" and believes
only answers signed by that controller.

A **network ID** names both the network
and its controller at once: 16 hex
characters, of which the first 10 are the
controller's own node address and the last
6 are a counter (chapter 4). `8056c2e21c000001`
is network 1 on controller `8056c2e21c` —
ZeroTier's own, running the public Earth
network. A network you create in Central
will carry the address of one of
ZeroTier's controller nodes, which is why
your IDs look random.

## Central: the hosted controller with a face

**ZeroTier Central** (my.zerotier.com) is
ZeroTier Inc.'s hosted controller service
with a web console. Creating an account is
free and there is a free tier aimed at home
users; paid tiers exist for larger fleets
and — recall chapter 3's licence notice —
commercial use of the current builds
requires one. The console is where most
networks live, and its workflow is the one
the rest of this guide assumes.

Creating a network, in the documented
shape:

1. Sign in at my.zerotier.com and choose
   **Create A Network**. You are handed a
   fresh network ID immediately — write it
   down; it is the network's name everywhere
   that matters.
2. Give the network a human name and
   description. Hex IDs are for machines;
   the next person to administer this
   network (possibly you, in two years)
   needs words.
3. Choose **Access Control**. The default
   is **Private**: no node becomes a member
   until you authorise it (chapter 8).
   **Public** means any node that knows the
   ID is a member. Public networks have
   exactly one good use — deliberately
   public party lines like Earth — and a
   long tail of bad ones. Private is the
   default because it should be.
4. Set the **IPv4 auto-assign** pool — the
   range managed addresses are drawn from
   (chapter 9). The presets are ordinary
   private ranges; pick one that does not
   collide with the LANs your members sit
   on, because a member whose home router
   also uses `192.168.1.x` will bind itself
   in knots if the virtual network claims
   the same range. This is the single most
   consequential choice on the page.

Everything else on the network's page —
members, managed routes, DNS, flow rules —
gets its own chapter: 8, 9 and 10
respectively.

## The same controller, other doors

Central is a face on an API: the
controller exposes the network definition
as a JSON object (members, `ipAssignmentPools`,
`routes`, `dns`, `rules`, and friends), and
anything that can authenticate to it can
administer networks — ZeroTier's own
console, third-party panels, or your own
scripts. And the controller itself does
not have to be ZeroTier's: the daemon you
installed contains one (the empty
`controller.d/` in its home), and
open-source consoles exist for it. Running
your own controller moves chapter 14's big
dependency onto your own desk — the
network's fate then rests on a machine you
must keep reachable, patched and backed
up, whose `identity.secret` is now the
master key to every network it runs. Most
readers should start on Central and treat
self-hosting as a graduation, not a first
step.

With a network in existence, at last there
is something to join. Chapter 8 crosses
the room: the node's request, your
approval, and the address that arrives
when both halves meet.
