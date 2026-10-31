# Writing criteria for this guide

Every "for Mere Mortals" guide is written to the same nine-point
standard, so a reader who trusts one can trust the rest. Here it
is, with the verification notes for this particular guide below.

1. **Fix it first.** Reader outcomes lead, not action items: each
   chapter opens by naming the outcome it delivers, shows a check
   the reader can run to confirm they got it, and a reasonable
   reader finishes able to state what was done, what it means,
   and what to try next.
2. **Plain language.** Short sentences, everyday vocabulary,
   technical terms defined on first use. No slang, and no
   idioms that only make sense in one country — the reader may
   be translating as they go.
3. **No dead ends.** A guide never tells the reader to do
   something it cannot show them how to verify. Every claim a
   reader relies on — an install, a command, an output — is
   something this guide demonstrates or clearly labels
   documented-only below.
4. **Live-verified where it matters.** Procedures are run for
   real in the guide's lab wherever the build machine allows it,
   and the transcripts kept in the repository are the evidence.
   What could not be run is listed honestly below, never
   presented as if it had been.
5. **General audiences.** Nothing in this guide — examples,
   fixtures, names, addresses — is modelled on any particular
   reader or author's real life, network, or organisations.
   The lab's one invented network ID is a recognised joke
   string (`deadbeefdeadbeef`), and its identities were minted
   for the lab and destroyed with it.
6. **Standalone.** A reader needs this guide, its lab, and
   nothing else. Where another guide would genuinely help, it
   is pointed to once, by name, and the chapter still completes
   without it.
7. **Honest about limits.** Chapter 14 names what the tool
   cannot do: it cannot live without UDP, relayed traffic is
   slower traffic, the data plane is userspace, membership has
   a centre, the current licence is source-available and
   non-commercial rather than open, layer 2 has edges, and it
   is not an anonymity tool.
8. **The ethics rule is in the text, early.** ZeroTier creates
   real networks between real machines, so chapter 1 and the
   README draw the line before any procedure depends on it:
   only ever authorise devices you would hand a house key,
   because an authorised member is inside the perimeter by
   design. Chapter 11 states what the operators of the roots
   and the controller can and cannot see, so the reader's
   trust is placed knowingly.
9. **Builds clean.** The site builds with MkDocs in strict mode
   with no warnings, and a fresh reader following the chapters
   in order reaches the stated outcome of each.

## Verification notes for this guide

The build machine is a shared Ubuntu 24.04 sandbox whose kernel
**refuses outgoing UDP packets** (`sendto` fails with
`Operation not permitted`, proven in transcript t04). That one
fact decides this guide's shape: the daemon, its identity
system, its local API and its failure states were all run for
real, while everything past a successful connection to the
roots is documented-only and labelled as such, in the chapters
and here.

ZeroTier was obtained as the official Debian package
`zerotier-one_1.16.2_amd64.deb` (3,514,154 bytes) from the
`noble` pool at download.zerotier.com. Its genuineness was
checked link by link before anything ran: the repo `Release`
file's signature verified **Good** against ZeroTier's release
signing key (fingerprint `74A5 E9C4 58E1 A431 F1DA 57A7 1657
1988 23E5 2A61`) extracted from the official install script at
install.zerotier.com; the signed Release vouches for the
`Packages.gz` hash (matched, `14bb0edd…`); the index vouches
for the .deb's SHA-256 (matched, `fe78e8d3…`). The package was
**extracted with `dpkg-deb -x` into a private tree rather than
installed system-wide**, a sandbox accommodation disclosed
here and in chapter 3; the binary ran unmodified from that
tree. The daemon ran only with its home inside the lab
directory, and was stopped and confirmed gone when the lab
ended (transcript t07). All transcripts live in
`zerotier-lab/transcripts/`; the source list for documented
material is `zerotier-lab/sources.txt`.

Run for real, with transcripts as evidence:

- The download and the full signature/hash chain (t00).
- Package extraction, file inventory, dependency list, and
  `zerotier-one -v` → 1.16.2 (t01).
- **A lab accident, kept in the record**: invoking
  `zerotier-one` with no arguments starts the service with the
  default home instead of printing help. The stray daemon was
  stopped, its home at `/var/lib/zerotier-one` deleted, and
  the episode is written into chapters 2 and 3 as a warning
  (t01).
- Daemon start in a lab home: identity generated (address
  `2076ee07cf`), home-directory anatomy listed file by file,
  API port 9993 read from `zerotier-one.port` (t01, t02, t04).
- `zerotier-cli info` → `200 info 2076ee07cf 1.16.2 OFFLINE`,
  reproducibly, for the daemon's whole life (t02).
- `listpeers` in the empty state: the four PLANET roots
  (`62f865ae71`, `778cde7190`, `cafe04eba9`, `cafe9efeb9`)
  with path `-` and latency `-1`; `peers` adding the tool's
  own line: "Currently tunneling through a TCP relay. Ensure
  that UDP is not blocked."; `listmoons` → `[]` (t02, t06).
- The local API by hand: HTTP 401 without the token; with
  `X-ZT1-Auth`, `/status` as JSON (`"online": false`,
  planetWorldId 149604618, listening ports 9993 plus two
  derived high ports), `/peer` (planets with `"paths": []`),
  `/network` → `[]` (t02, t03).
- A join of the invented network `deadbeefdeadbeef`:
  `200 join OK`, the `.conf` file created in `networks.d/`,
  listnetworks remaining header-only because no configuration
  could ever arrive; then `200 leave OK` (t03).
- `zerotier-idtool generate` and `validate` on a scratch
  identity (address `bc8249ff1a`, deleted with the lab), and
  both tools' full help surfaces, including idtool's moon
  commands and the 1.16.2 licence notice printed by the
  binary itself (t04, t05).
- The sandbox's walls, measured: UDP `sendto` refused with
  `Errno 1`; `/dev/net/tun` present and openable here — noted
  as moot, since no configuration could arrive to use it (t04).
- The shipped systemd unit and the three manual pages read in
  full as primary sources (t06; zerotier-lab/sources.txt).
- Clean shutdown: process gone, port 9993 refusing (t07).

Documented only — described from the sources in
`zerotier-lab/sources.txt`, not run, and the text does not
claim otherwise:

- Everything from a successful root connection onward: ONLINE
  state, rendezvous and direct paths in action, relay and TCP
  fallback carrying real traffic, and actual throughput
  (figures quoted are ZeroTier's own published numbers).
- ZeroTier Central end to end: account creation, creating a
  network, the members list, authorising a member, pinning a
  managed IP, deauthorising. **No account was created for this
  guide** and no real network was joined; the console workflow
  rests on the shipped manual pages, ZeroTier's knowledge
  base, and three router vendors' documentation of Central,
  which agree with each other.
- Controller settings: IPv4 auto-assign pools, managed routes
  (including the exit-node `0.0.0.0/0` pattern and the member's
  `allowDefault` opt-in), IPv6 modes (RFC 4193, 6PLANE), and
  pushed DNS — from the controller's configuration model as
  documented by the open-source consoles that implement it.
- The flow rules engine, capabilities and tags — from
  ZeroTier's knowledge-base article on its rules engine
  (statelessness, enforcement at both ends, the tag example).
- Moons in operation (the `initmoon`/`genmoon` commands are
  real and quoted from the binary; no moon was built, orbited
  or relied on), and self-hosted controllers in operation.
- The Windows, macOS, iOS and Android apps, and router/NAS
  builds beyond noting that they exist.
- docs.zerotier.com itself: the site refused the lab's page
  reader (HTTP 403), so no claim in this guide rests on a page
  the lab could not open. It is named as further reading, not
  cited as evidence.

Fixtures are this guide's own: one invented network ID
(`deadbeefdeadbeef`, a joke string that names no real network
and could not be reached from the lab in any case) and two
identities minted in the lab and destroyed with it. The public
address of the lab node (`2076ee07cf`) is quoted because an
address is a public identifier and the node no longer exists;
no identity.secret content, and no API token, appears anywhere
in this repository.
