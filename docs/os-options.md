# Going from stock Manjaro ARM to something more current

If you're on the PinePhone's stock Manjaro ARM image and it's gone
stale (which happens easily — this is a low-traffic image, updates can
sit unapplied for a long time before anyone notices), you've got two
real paths: **keep Manjaro but actually maintain it**, or **move to a
more actively-maintained image**. Both are legitimate. Here's how to
decide, current as of August 2026 — verify anything version-specific
against the projects' own sites before committing, this space moves
fast.

## Option 1: stay on Manjaro, but actually update it

If you like Arch-family package management and Plasma Mobile, staying
put is fine — the problem was never Manjaro itself, it was letting
updates lapse for two years. Run `sudo pacman -Syu` regularly (weekly
is reasonable for a phone you actually rely on), and see
[resolving-upgrade-conflicts.md](resolving-upgrade-conflicts.md) for
how to handle what comes up when you do.

## Option 2: move to a different image entirely

Three real options, each with real tradeoffs:

### postmarketOS
- Alpine-based (musl/BusyBox, not glibc) — a genuine departure from
  Ubuntu/Debian-family assumptions, some software needs adjustment.
- **Quarterly releases with kernel + critical CVE fixes** — meaningfully
  faster security patching than the alternatives below.
- No verified boot implementation (a gap if that matters to your threat
  model).
- Broad, currently-active hardware support across the whole Pine64
  lineup.

### Mobian
- Debian-based, currently tracking Debian Bookworm — if you know
  Ubuntu, this transfers directly (apt, .deb, systemd).
- **Tracks Debian Stable's ~2-year release cycle** — security patches
  can lag months to years. This is structurally the same category of
  problem that causes stock-image staleness in the first place, just
  with a different distro underneath. Worth being honest with yourself
  about whether you'll actually keep up with manual updates.
- PinePhone (non-Pro) is a **1st Tier** supported device on Mobian's
  device hierarchy — strong support, not an afterthought port.
- **The graphical/Calamares-style installer has been broken since
  January 2025** (Qt6 compatibility) — but this doesn't affect the
  actual install method for PinePhone, which uses a pre-built image
  flashed with `dd`/`bmaptool`, not that installer. See
  [migrating-to-mobian.md](migrating-to-mobian.md).

### Ubuntu Touch (UBports)
- Shipped **OTA 2.0 in July 2026**, rebased on Ubuntu 24.04 LTS —
  genuinely active, modern, currently-shipping development. Chromium
  134 in the browser, Widevine DRM, better legacy app support.
- **But**: the original PinePhone's port specifically is flagged as
  community-porter-dependent, "under heavy development... many features
  including basic functionality may not work." The OS's own health and
  this specific device's port quality are two different questions —
  don't conflate "the project is thriving" with "your hardware is
  well-supported by it."
- No standard UBports Installer support for PinePhone — manual
  install only (Tow-Boot prerequisite, then flash a `.img.xz`).
- Known limitations on this hardware: camera doesn't work, battery life
  well under the "24+ hours" target.

## Our take

If you're coming from Ubuntu and want the most direct transfer of
existing knowledge, **Mobian** is the practical choice — just go in
knowing about the Debian Stable patch-cadence tradeoff, and don't let
this image go stale either. If fast security patching matters more to
you than familiarity, **postmarketOS** is the stronger technical
answer despite the steeper learning curve. **Ubuntu Touch** is worth
watching — the OS itself is clearly healthy — but the PinePhone-specific
port quality is the real open question before committing your daily
driver to it.
