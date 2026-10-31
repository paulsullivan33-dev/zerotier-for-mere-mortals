# The Moving Parts

ZeroTier arrives as one package containing
one binary that plays three roles, plus a
folder of small files that *are* the node's
life. This chapter names them all, because
every later chapter — and every error
message — refers to one of them.

## The daemon: zerotier-one

The service itself. It runs in the
background (your init system starts it; the
package ships a systemd unit whose whole
job is `ExecStart=/usr/sbin/zerotier-one`
with `Restart=always`), and it does
everything that matters:

- holds the node's **identity** — its
  address and keys (chapter 4);
- talks to the **roots** and to other
  nodes, finding and keeping paths
  (chapter 6);
- asks **controllers** for network
  configurations and applies them;
- creates the **virtual network interfaces**
  (one per joined network) that the
  operating system sees, named from the
  network's ID;
- encrypts and decrypts every frame that
  crosses those interfaces.

A bare `zerotier-one` with no arguments
*starts the service*, using the default home
directory — it does not print help. The lab
learned this the direct way: an accidental
bare invocation quietly created a home at
`/var/lib/zerotier-one` and began running
before anyone noticed. The transcript of
that small accident is kept in
`zerotier-lab/transcripts/t01-extract-and-first-run.txt`,
because on your machine the same surprise
would look like a mystery process.

## The two other personalities

`zerotier-cli` and `zerotier-idtool` are
symbolic links to the same binary; it
behaves according to the name it was called
by (the switches `-q` and `-i` select the
same personalities by hand).

**zerotier-cli** is how you talk to the
running daemon: `info`, `listpeers`,
`listnetworks`, `join`, `leave`, and
`get`/`set` for per-network settings. It
does no network work itself — it is a
remote control.

**zerotier-idtool** works with identities
as files: `generate` a new one, `validate`
it, extract its public half with
`getpublic`, `sign` and `verify` files with
its key, and build **moons** with `initmoon`
and `genmoon` (chapter 6). Chapter 4 uses it
live.

## The local API

Under the CLI sits the real interface: a
JSON-over-HTTP API served by the daemon on
**127.0.0.1, port 9993** — the tool's own
designed port, the same one its peer
traffic starts from. Every request must
carry a token in the `X-ZT1-Auth` header;
the token lives in a file in the daemon's
home (below). Chapter 5 calls this API
directly with curl: without the token it
answers `401` and an empty `{}`; with it,
it hands over the node's whole status as
JSON. Anything that can speak HTTP can
drive ZeroTier — that is how the desktop
apps' interfaces and third-party control
panels are built.

## The home directory

The daemon keeps everything it is in one
directory — `/var/lib/zerotier-one` on
Linux by default, or any path you hand it
at launch. The lab ran its daemon with a
home inside the lab folder, which is how
its anatomy could be listed safely. A fresh
home, seconds after first start:

```text
authtoken.secret    the API token (mode 0600)
controller.d/       this node as a controller (empty here)
identity.public     address + public key (141 bytes)
identity.secret     the full identity, private key included (0600)
metrics.prom        Prometheus metrics output
metricstoken.secret token for the metrics endpoint
networks.d/         one <network ID>.conf per joined network
planet              the signed list of the root servers (570 bytes)
zerotier-one.pid    the running daemon's process id
zerotier-one.port   the API port it chose: 9993
```

Three of these deserve a sentence each now.
**identity.secret** is the node's soul:
whoever holds it *is* this node, on every
network it belongs to — chapter 4 is mostly
about respecting that file. **planet** is a
small signed document, shipped with the
program and copied into the home, naming
the official root servers — the node's
phone book for the whole system, verifiable
because it is signed. **networks.d/** is
the node's memory of its memberships: touch
an empty file named `<network ID>.conf`
there and the daemon will try to join that
network at startup, a trick chapter 12 uses.

## The parts that are not on your machine

Two roles live elsewhere. **Roots** (in the
vocabulary of the `planet` file, *planets*;
self-hosted ones are *moons*) are the
introducers and fallback relays of chapter
6 — fixed points a node can always find
first. **Controllers** hold the definition
of each network — its members, addresses,
routes and rules — and sign configurations
for the members. Most people use ZeroTier's
hosted controller (Central, chapter 7);
the daemon itself contains a controller
too — that empty `controller.d/` is its
desk — which is the seed of the
self-hosting story in chapters 6 and 14.

With the cast named, the rest of the guide
is putting each part to work: install it
(chapter 3), meet the identity (chapter 4),
and read a living node (chapter 5).
