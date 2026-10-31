# Installing ZeroTier

The outcome of this chapter: ZeroTier
installed from the official source, its
genuineness checked rather than assumed,
and one command that proves the binary
runs. The lab did this on Ubuntu 24.04;
the shape is the same on any Debian-family
system, and the last section covers the
other platforms in brief.

## The official routes

ZeroTier offers three doors into the same
room on Linux:

1. **The install script** at
   `install.zerotier.com` — one command,
   and it sets up door number two for you:
   `curl -s https://install.zerotier.com | sudo bash`
2. **The ZeroTier apt repository** at
   `download.zerotier.com` — your package
   manager installs and updates it like any
   other package.
3. **The .deb file directly** from the
   repository's pool, for one machine by
   hand.

The script is the documented front door,
and it is honest about what it does — it is
a readable shell script, and the lab read
it. Buried in it is something worth seeing:
the complete text of ZeroTier's repository
signing key, which the script writes to
`/tmp/zt-gpg-key` and installs into apt's
trusted keys. That embedded key becomes
important below.

## What the lab actually did

The lab's sandbox could not take over the
system package manager, so it used door
three, with the verification done by hand —
the same verification apt would have done
silently. First, the package itself:
`zerotier-one_1.16.2_amd64.deb` (3,514,154
bytes) from the `noble` pool of
`download.zerotier.com`. Then the chain of
vouches, each link checked
(`zerotier-lab/transcripts/t00-download-and-verify.txt`
has the whole session):

- The repository's `Release` file carries
  a detached signature, `Release.gpg`.
  Checked against the signing key extracted
  from the official install script,
  fingerprint
  `74A5 E9C4 58E1 A431 F1DA 57A7 1657 1988 23E5 2A61`,
  GnuPG's verdict was **Good signature**
  from "ZeroTier, Inc. (ZeroTier Support
  and Release Signing Key)".
- The signed `Release` file lists the hash
  of the package index (`Packages.gz`) —
  the downloaded index matched:
  `14bb0edd…`.
- The index lists the SHA-256 of the .deb —
  the downloaded package matched:
  `fe78e8d3b03fe77d1607bffd67dcd6723e2bacc4ac063afb5e311400fda12ea6`.

So the bytes the lab ran are the bytes
ZeroTier signed, with the signature checked
against a key obtained from a second,
independent official artefact (the script)
rather than from the same directory as the
package. On your own machine, `apt` after
door one or two performs the same chain
without showing it to you — which is fine,
but now you know what it is doing.

The package was then **extracted, not
installed** (`dpkg-deb -x` into a private
tree — a lab accommodation, disclosed in
`criteria.md`; on a real machine you would
simply install it). Its contents are
reassuringly small:

```text
/usr/sbin/zerotier-one        the binary (plus the cli/idtool links)
/usr/lib/systemd/system/zerotier-one.service
/etc/init.d/zerotier-one      for non-systemd systems
/var/lib/zerotier-one/        the default home, holding one SELinux file
man pages for all three tools
```

Its declared dependencies are just
`adduser`, `libstdc++6` and `openssl` — the
binary is nearly self-contained.

## The proof it runs

ZeroTier's version check is also its
licence notice, and both are worth seeing
once:

```bash
zerotier-one -v
# 1.16.2

zerotier-cli help
# ZeroTier One version 1.16.2 build 0 (platform 1 arch 2)
# Licensed under a Source-Available License for Non-Commercial
# Use (nonfree/LICENSE.md). Use of this build for Commercial Use
# requires a paid subscription plan or a commercial license
# agreement with ZeroTier, Inc.
```

On an installed system the service starts
itself at boot. Started by hand with a
chosen home — the lab's method, and the one
to imitate when experimenting — it looks
like this:

```bash
sudo zerotier-one -d /var/lib/zerotier-one   # -d: run as a daemon
zerotier-cli info
```

That second command is chapter 5's whole
subject. One caution from chapter 2 bears
repeating here: `zerotier-one` with no
arguments **starts the service** with the
default home; it is not a help command.
`-h` and `-v` are the safe probes.

## The other platforms, briefly

Documented-only in this guide: Windows and
macOS have first-class installers from
zerotier.com with a graphical app for
joining networks and reading status (the
same daemon underneath — the Windows
service, the macOS system extension — and
the same `zerotier-cli` in a bundled
console). iOS and Android have apps in
their stores, joining networks by ID the
same way. Routers and NAS boxes are a
long tail: MikroTik ships ZeroTier inside
RouterOS, and several NAS vendors offer
packages. Every one of these is the same
node with the same identity model — which
is why the next chapter, about that
identity, is platform-free.
