# Reading Your Node

The outcome of this chapter: given any
ZeroTier node, you can ask it who it is,
whether it is connected, and whom it has
been talking to — and interpret the answers,
including the answer this lab got, which
was "nobody, and here's why".

## info: the one-line health check

```bash
zerotier-cli info
```

On a healthy node on an ordinary
connection, this prints your address, the
version, and the word **ONLINE** within
seconds of starting, and stays that way.
The lab's node printed this, every time,
for its entire life:

```text
200 info 2076ee07cf 1.16.2 OFFLINE
```

(The leading `200` is the API's status
code, echoed by the CLI — every reply
begins with one.) Read the line left to
right: request OK; the address from
chapter 4; version 1.16.2; and **OFFLINE** —
the daemon is running, holds an identity,
and cannot reach the rest of ZeroTier at
all. OFFLINE is a statement about the
*underlay* — the road to the roots — not
about any particular network. A node that
is OFFLINE cannot join, fetch
configurations, or pass traffic; a node
that is ONLINE may still be unauthorised
on your network, which is a different
problem with a different message (chapter 8).

Why was the lab OFFLINE? The lab is a
sandbox whose kernel refuses outgoing UDP
packets outright. ZeroTier's first move on
any network is to send UDP to the root
servers. The refusal is not subtle — the
lab tested it directly with a one-packet
probe:

```text
sendto failed: [Errno 1] Operation not permitted
```

Chapter 6 explains why UDP, of all things,
is the load-bearing protocol. For now, file
the lesson: **OFFLINE means the daemon
cannot speak UDP to the internet**, and the
usual real-world causes are a firewall
blocking outbound UDP, a network that only
allows web traffic through a proxy, or —
on a fresh install — simply that you read
the answer in the first two seconds.

## listpeers: the phone book and the logbook

```bash
zerotier-cli listpeers
```

On a working node this lists the roots
plus every peer the node has spoken with
recently — other members of your networks,
with the path each conversation takes and
its latency. The lab's version is the
empty-state, worth reading once in your
life:

```text
200 listpeers <ztaddr> <path> <latency> <version> <role>
200 listpeers 62f865ae71 - -1 - PLANET
200 listpeers 778cde7190 - -1 - PLANET
200 listpeers cafe04eba9 - -1 - PLANET
200 listpeers cafe9efeb9 - -1 - PLANET
```

Four entries, all **PLANET** — the official
root servers, decoded from the signed
`planet` file in the daemon's home. The
path column is `-` and the latency `-1`:
known, but never reached. The manual makes
a point worth remembering on a *working*
node too: listpeers shows the underlay —
roots and controllers appear here alongside
your fellow members, and a machine can be
a peer without sharing any network with
you. It answers "whom can this node talk
*to*", not "who is on my network".

The prettier sibling command, `zerotier-cli
peers`, adds the summary the lab cherished
most:

```text
62f865ae71 -      PLANET    -1 RELAY
...
NOTE: Currently tunneling through a TCP relay. Ensure that UDP is not blocked.
```

The tool itself, unprompted, diagnosing the
lab's exact condition. Keep that sentence;
chapter 15 builds on it.

## The API underneath

Everything the CLI prints comes from the
local API (chapter 2), and asking it
directly is the same skill with the gloves
off:

```bash
curl -s http://127.0.0.1:9993/status
# {}                         with HTTP 401 — no token, no answer

TOKEN=$(cat /var/lib/zerotier-one/authtoken.secret)
curl -s -H "X-ZT1-Auth: $TOKEN" http://127.0.0.1:9993/status
```

The lab's reply, trimmed to the lines the
guide uses:

```json
{
  "address": "2076ee07cf",
  "online": false,
  "version": "1.16.2",
  "planetWorldId": 149604618,
  "tcpFallbackActive": false
}
```

Two details reward attention. The full
status (the CLI's `-j` flag shows it too)
lists `"listeningOn"` addresses — the node
listening on port **9993** plus two
additional high ports, matching the
documented pattern of three UDP ports:
9993 itself, one derived from the node's
address, and one for NAT port-mapping
protocols. And `planetWorldId: 149604618`
is the ID of the "world" — the signed root
list — this node belongs to; the default
world is named **Earth**. `/peer` returns
the listpeers data as JSON (the planets
again, `"paths": []` — no routes to
anywhere), and `/network` returns `[]`:
this node belongs to nothing.

## listnetworks, and the shape of a join

```bash
zerotier-cli listnetworks
# 200 listnetworks <nwid> <name> <mac> <status> <type> <dev> <ZT assigned ips>
```

Just the header: no memberships. Chapter 8
performs a join; the preview belongs here
because it is so instructive. The lab asked
to join an invented network,
`deadbeefdeadbeef`:

```text
$ zerotier-cli join deadbeefdeadbeef
200 join OK
$ ls networks.d/
deadbeefdeadbeef.conf
$ zerotier-cli listnetworks
200 listnetworks <nwid> <name> <mac> <status> <type> <dev> <ZT assigned ips>
```

`join OK` means *the request is recorded* —
a `.conf` file exists in `networks.d/` —
nothing more. No configuration ever
arrived (it could not: the controller for
that ID lives behind roots this node cannot
reach), so the network never even earned a
line in listnetworks. On a real connection
the same command fills in within seconds:
a status of `OK` once authorised, or
`ACCESS_DENIED` while you wait for the
owner (chapter 8). The lab then left the
invented network (`200 leave OK`) and shut
its daemon down — no process remained, and
port 9993 stopped answering. Every part of
that little ceremony, minus the network
itself, was real.
