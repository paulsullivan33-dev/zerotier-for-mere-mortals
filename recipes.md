# Recipes

Short, complete jobs. Where a step depends
on the console or a live network, the
recipe says so; everything else here was
either run in the lab or is assembled from
commands the lab verified.

## Back up a node's identity

Your node's address lives in two small
files in the daemon's home. Back them up
somewhere safe — losing them means a new
address and re-authorisation everywhere.

```bash
sudo cp /var/lib/zerotier-one/identity.secret \
        /var/lib/zerotier-one/identity.public \
        /path/to/safe/backup/
```

Treat the backup of `identity.secret` like
a private SSH key: encrypted storage, never
a shared folder, never a chat window. For
a controller node, back up the whole home —
`controller.d/` and its database are the
network definitions themselves.

## Move a node to a new machine

Install ZeroTier on the new machine but do
not start it (or stop it if it started).
Copy the old machine's `identity.secret`
and `identity.public` into the new home,
fix ownership and permissions, then start:

```bash
sudo systemctl stop zerotier-one
sudo cp identity.secret identity.public /var/lib/zerotier-one/
sudo chmod 600 /var/lib/zerotier-one/identity.secret
sudo systemctl start zerotier-one
zerotier-cli info     # the old address, on the new hardware
```

Retire the old machine's copy — one
identity in two running daemons means two
machines claiming one address, and the
network will believe whichever it heard
from last. (This is also the theft scenario
from chapter 11, done by the owner: the
file really is the node.)

## Make your own vanity address (for fun)

```bash
zerotier-idtool generate cafe.secret cafe.public cafe
```

…will grind until it finds an identity
whose address starts with `cafe`. Four hex
digits took "about two hours on one core"
by the manual's arithmetic in 2016; modern
cores are faster, and five digits is sixteen
times the work regardless. Install the
result as a node's identity with the
previous recipe. Harmless, useless, and a
fair demo of why proof of work works.

## Pre-join a network before first boot

Because the daemon scans `networks.d/` at
startup (chapter 2), you can stage a join
in an image or a provisioning script:

```bash
sudo touch /var/lib/zerotier-one/networks.d/<network ID>.conf
sudo systemctl start zerotier-one
```

The node will request configuration on its
first breath; all that remains is the
owner's authorisation in the console.

## Let a regular user run the CLI

By the manual's own recipe: copy the API
token into the user's home, owned by them,
mode `0600`:

```bash
sudo cp /var/lib/zerotier-one/authtoken.secret ~/.zeroTierOneAuthToken
sudo chown $USER ~/.zeroTierOneAuthToken
chmod 0600 ~/.zeroTierOneAuthToken
```

`zerotier-cli` then works without sudo.
Grant this thoughtfully — the manual warns
that the token's holder can connect or
disconnect the machine from any network,
"a significant permission".

## A sixty-second health check

```bash
zerotier-cli info          # ONLINE? address as expected?
zerotier-cli listnetworks  # every network OK, with an address?
zerotier-cli peers         # DIRECT links to the peers you use?
```

Read it as a mechanic would: OFFLINE sends
you to chapter 15's UDP questions;
`ACCESS_DENIED` to the console; a peer
stuck on RELAY to chapter 6's firewall
paragraph; all three green means the
machinery is fine and your problem lives
in an application, not the network.

## Read a node's whole status as JSON

```bash
zerotier-cli -j status | python3 -m json.tool
```

The fields chapter 5 used — `online`,
`listeningOn`, `planetWorldId`,
`tcpFallbackActive` — plus everything the
terse `info` line hides. When a remote
helper asks "what does it say", this
output is the answer worth pasting.
