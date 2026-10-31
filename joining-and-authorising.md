# Joining and Being Authorised

The outcome of this chapter: you can join a
node to a network, recognise each state the
join passes through, and complete the other
half — authorisation — in the console.
The lab performed the node's half of this
(the request, the local state, the leave)
against an invented network; the arrival
of a configuration is documented, from the
manual page's own description of the
ceremony.

## The request

```bash
sudo zerotier-cli join <network ID>
# 200 join OK
```

The manual page describes what you have
just set in motion: the node asks the
network's controller for a configuration.
"You'll either get a reply from the network
controller with a certificate and other info
such as IP assignments, or you'll get
'access denied.' In this case you'll need
the administrator of this network to
authorize your device by its 10-digit device
ID."

`listnetworks` reports where things stand:

```text
200 listnetworks <nwid> <name> <mac> <status> <type> <dev> <ZT assigned ips>
```

The statuses you will meet, in the order
you will meet them:

- **REQUESTING_CONFIGURATION** — the node
  has asked and is waiting. Seconds on a
  healthy connection; forever, in the lab.
- **ACCESS_DENIED** — the controller knows
  you, and you are not authorised. This is
  not an error; on a private network it is
  the expected waiting room.
- **OK** — authorised, configured, and the
  row now shows the interface name and your
  managed address. The network is live.
- **PORT_CONFIGURATION_ERROR** — the
  configuration arrived but the node could
  not build the interface (usually the
  tunnel device or permissions) — chapter
  15's territory.

Notice what the lab proved about the
*degenerate* case (transcript t03): with
no path to the controller at all, the join
still "succeeds" locally — `200 join OK`,
a `deadbeefdeadbeef.conf` file in
`networks.d/` — and listnetworks stays
silent, because there is no configuration
to describe. Join is a *request recorded*,
never a promise kept. The `.conf` file is
also why a joined network survives
restarts, and why `leave` matters:

```bash
sudo zerotier-cli leave <network ID>
# 200 leave OK
```

Leaving deletes the configuration and the
interface. (Peers may linger in
`listpeers` for up to half an hour — they
are underlay acquaintances now, unable to
send you anything that matters, as the
manual puts it.)

## The approval (documented)

On the console side, the documented
workflow is:

1. Open the network in Central and scroll
   to **Members**. Your node appears there
   once it has asked — by its 10-character
   address. This is why chapter 4 had you
   learn the address first: it is the name
   the owner knows you by, and the thing to
   send the owner, out of band, when you
   ask to be let in.
2. Tick the **Auth** box for that member.
   Until this moment the node sits at
   ACCESS_DENIED; the moment it is ticked,
   the controller signs its configuration
   and the node flips to OK by itself —
   nothing to restart.
3. Name the member and write a description.
   Six months from now, the members list is
   your inventory of everything with a key
   to this network; "laptop" is not an
   inventory.
4. Optionally, pin its **managed IP** —
   click the assigned address and set the
   one you want it to keep (chapter 9).

Deauthorising is the same checkbox in
reverse, and it is the correct response to
a lost device: the member's certificate is
no longer honoured, and its access ends
without touching the device itself.
Deleting a member removes the record
entirely; a rejoined node starts again
from the waiting room.

## The two halves, honestly

It is worth standing back: joining a
ZeroTier network is a *relationship*, not
a setting. The node's half is one command;
the owner's half is a judgement, made by a
human looking at an address a human sent
them. Everything else in this guide —
addresses, routes, rules — hangs off that
one act of trust, which is why the ethics
note in chapter 1 said to hand out
authorisations like house keys. Chapter 9
now spends the trust: what the controller
gives members once they are in.
