# pinephone

Field notes and small tools for a **PinePhone Beta Edition (motherboard
rev 1.2b)**, written from actually living with one — not theory.
Everything here reflects real problems hit and how they actually got
resolved, not generic advice copied from elsewhere. See
[docs/hardware.md](docs/hardware.md) for exactly what that hardware
identity means and why it matters (short version: it has a different
magnetometer chip than most PinePhone docs assume, and that has
consequences).

Originally ran a badly-stale Manjaro ARM (~2 years of accumulated update
drift). As of 2026-09-27/28, running **postmarketOS** (rebranded to
**Nura** the same day — see [docs/os-options.md](docs/os-options.md#decision-2026-09-27)
for why we chose it and why the rename doesn't change anything).

## What's here

- **[docs/hardware.md](docs/hardware.md)** — what this device actually
  is: chassis, bootloader, the sensor chip substitution that trips up
  generic PinePhone advice, camera limitations.
- **[docs/os-options.md](docs/os-options.md)** — postmarketOS vs Mobian
  vs Ubuntu Touch vs staying on Manjaro, plus the actual decision we made
  and why.
- **[docs/migrating-to-postmarketos.md](docs/migrating-to-postmarketos.md)**
  — the real steps we followed to move from stock Manjaro to postmarketOS.
- **[docs/postmarketos-1.2b-fixes.md](docs/postmarketos-1.2b-fixes.md)**
  — getting the magnetometer and (as a direct consequence of that fix)
  the cellular modem actually working on a 1.2b unit. Read this before
  you hit either problem blind.
- **[docs/migrating-to-mobian.md](docs/migrating-to-mobian.md)** —
  concrete steps for moving from stock Manjaro to Mobian instead,
  including the Tow-Boot prerequisite. Kept accurate as an alternative
  path, even though we went with postmarketOS.
- **[docs/resolving-upgrade-conflicts.md](docs/resolving-upgrade-conflicts.md)**
  — how to handle `pacman -Syu` conflicts, `.pacnew` files, and a broken
  GUI after upgrading a badly-stale Manjaro ARM install. Historical now
  that we've moved off Manjaro, kept for anyone still on it.
- **[docs/scripts.md](docs/scripts.md)** — usage and a real warning
  about `wifi-toggle` (see below).
- **[scripts/](scripts/)** — `modem-toggle` and `wifi-toggle`, small
  power-management scripts for the cellular modem and WiFi radio. Still
  work fine on postmarketOS.

## Quick start

```bash
mkdir -p ~/bin
cp scripts/*-toggle ~/bin/
chmod +x ~/bin/*-toggle
# add ~/bin to PATH if not already there -- see docs/scripts.md
modem-toggle status
wifi-toggle status
```

**Before you touch `wifi-toggle`, read the warning in
[docs/scripts.md](docs/scripts.md).** If you're SSH'd into the phone
over its own WiFi, disabling WiFi cuts the connection you're using to
run the command — no remote recovery possible, only a different
network path or physical access gets you back in. This happened to us;
it's not a hypothetical.

## Contributing

This started as one person's field notes, kept honest by only
documenting things actually verified against the hardware or the
projects' own current documentation — not assumptions. If you send a
fix, please keep that standard: cite where something came from, and
note the date you verified it, since PinePhone-adjacent projects move
fast and stale advice is worse than no advice.
