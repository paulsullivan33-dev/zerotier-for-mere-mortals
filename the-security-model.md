# The Security Model

The outcome of this chapter: you can say
exactly who can see what in a ZeroTier
network — in particular, the answer to the
question everyone asks first: *can ZeroTier
(the company) read my traffic?* It cannot,
and this chapter explains the machinery
that makes that a property of the design
rather than a promise in a policy.

## Two planes, two kinds of trust

Keep the two halves of the system apart,
because they hold different secrets:

**The control plane** is membership: who
asked to join, who was authorised, which
addresses were assigned, what the rules
are. This lives with the **controller**.
The controller knows the *shape* of your
network — member addresses, names you gave
them, when they were last seen, their
apparent public addresses. If you use
Central, that metadata lives on ZeroTier's
service. The controller signs
configurations; it holds no keys to member
traffic, because it never needs them.

**The data plane** is the traffic itself,
and it is **end-to-end encrypted between
nodes**. Each node's identity is a key
pair (chapter 4); when two nodes talk,
they authenticate each other with those
keys and encrypt the conversation between
themselves. Roots and relays that forward
frames handle ciphertext — ZeroTier's NAT
article says it flatly: its servers relay
encrypted traffic and cannot read the
payload. A moon you run yourself is in the
same position: it can introduce your
machines and carry their ciphertext, and
that is all.

## What each actor can and cannot do

- **Another member** can reach your node's
  services on the networks you share —
  subject to the rules (chapter 10) —
  exactly as if plugged into your switch.
  This is the real exposure of the system,
  and it is governed by authorisation, not
  by the cryptography. The encryption keeps
  *outsiders* out; it does nothing about
  *insiders*, because insiders are the
  point.
- **A root or relay** sees encrypted frames
  and the metadata of who is rendezvousing
  with whom, and can drop or delay traffic
  but not read or forge it.
- **The controller** can admit or expel
  members and redefine the network — a
  serious power — but cannot decrypt member
  traffic or impersonate a member, because
  it does not hold any member's private key.
- **Whoever steals an `identity.secret`**
  *becomes that member*, completely; the
  defence is file hygiene (chapter 4) and,
  after the fact, deauthorising that member
  in the console (chapter 8) — the one
  remedy that works even while the thief
  holds the keys, because membership is
  granted by the controller, not by
  possession alone.

## Signatures everywhere

Notice how much of the system is *signed*
rather than merely sent: the `planet` file
naming the roots is signed (your daemon
trusts those four addresses because the
signature checks out, not because they
were compiled in as gospel); network
configurations are signed by the
controller; capabilities and tags are
signed credentials a member presents and a
peer verifies. A configuration pushed by
an impostor controller fails verification
and is ignored. This is also why the
install chapter bothered with a signature
chain for a humble .deb: the guide is
about a system whose entire design says
*verify, then trust* — it seemed right to
install it that way.

## The boundary of the model

Encryption ends at the member's interface.
What a member does with traffic after
decryption — the services it runs, the
passwords on them — is ordinary network
security, unchanged by ZeroTier. A badly
secured service on a ZeroTier network is
safe from the internet and exposed to the
network, and chapter 1's house-keys
warning is the practical summary: the
model guarantees that only *your invited
machines* can knock. Whether you should
have invited them, and what they can reach
once inside (chapter 10's answer), remains
— as it should — your decision.
