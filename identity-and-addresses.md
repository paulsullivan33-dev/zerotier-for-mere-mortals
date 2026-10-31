# Identity — Your Address and Your Keys

The outcome of this chapter: you can read a
ZeroTier identity, say what each part is
for, and handle the one dangerous file
correctly. Everything here was run live —
identities are minted locally, need no
network, and cost nothing but a moment of
computation.

## Born with an address

The first time `zerotier-one` starts in a
fresh home, before it speaks to anyone, it
**generates an identity**: an ECC key pair
and, derived from the public key, the
node's **ZeroTier address** — 10 hex
characters, like a MAC address for the
whole ZeroTier world. The lab's node was
born at 00:36 UTC with this one:

```text
2076ee07cf:0:5c5991f7845c7d77c8dc77b965d…
```

That is the beginning of the node's
`identity.public` file (141 bytes): the
address, a version field, and the public
key, colon-separated. The address is a
public identifier — you quote it the way
you would quote a MAC address, and chapter
8 has you hand it to a network's owner for
authorisation. It is not a secret.

The full identity, including the private
key, is `identity.secret` (270 bytes, mode
`0600`). The manual page is blunt about
this file, and the guide will not soften
it: the file *is* the node. Copy it to
another machine and that machine **is**
this node, on every network, able to
decrypt its traffic. Lose it, and the
address is gone forever — a new start
mints a new address, and every network's
owner must authorise you again. Steal it,
and the thief is you, as far as the
cryptography can tell. Back it up like a
key (chapter 12), guard it like a key, and
never paste it anywhere — including,
especially, into a transcript. This guide's
transcripts quote the public file only,
truncated at that.

## The proof of work inside

Addresses are not just hashes of keys.
ZeroTier deliberately makes identity
generation cost computation — a **proof of
work** baked into the address — so that
minting a million identities to flood a
network is expensive while minting one is
a moment's work (the manual warns it can
take ten seconds or more on a slow system,
once). `zerotier-idtool` exposes the
machinery:

```bash
zerotier-idtool generate scratch.secret scratch.public
# scratch.secret written
# scratch.public written

cut -d: -f1 scratch.public
# bc8249ff1a        (a second identity, made in about a second)

zerotier-idtool validate scratch.secret
# scratch.secret is a valid identity
```

(The lab's scratch identity was deleted
with the lab and never joined anything —
it is safe to print its address because it
no longer exists anywhere.) You can even
ask for an address beginning with chosen
digits — a *vanity* prefix. The manual
prices that honestly: a 4-digit prefix
takes about two hours on one core of a
desktop processor. The proof of work is
the reason; each lucky guess must be paid
for in full.

## Addresses you will meet

Three related numbers run through the rest
of the guide, so fix them now:

- A **node address** is 10 hex characters
  (`2076ee07cf`). It names a machine.
- A **network ID** is 16 hex characters
  (chapter 7). It names a network — and,
  neatly, its first 10 characters are the
  address of the network's **controller**:
  `8056c2e21c000001`, the Earth network from
  chapter 1, is network number `000001` on
  the controller whose address is
  `8056c2e21c`. Every network ID tells you
  who runs it, if you know how to read it.
- A **managed IP** is the ordinary private
  address (like `10.147.20.5`) the
  controller assigns a member *inside* one
  network (chapter 9). It is per-network,
  not an identity: the same node has a
  different managed IP on each network it
  joins, and none at all until authorised.

One node, one identity, many networks —
that is the whole model. Next, chapter 5
starts the daemon and learns to read what
a node says about itself, which in the
lab's case was a short, honest confession.
