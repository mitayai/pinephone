# pinephone

Field notes and small tools for the original PinePhone, written from
actually living with one — not theory. Everything here reflects real
problems hit and how they actually got resolved, not generic advice
copied from elsewhere.

## What's here

- **[docs/resolving-upgrade-conflicts.md](docs/resolving-upgrade-conflicts.md)**
  — how to handle `pacman -Syu` conflicts, `.pacnew` files, and a broken
  GUI after upgrading a badly-stale Manjaro ARM install. Written after
  recovering from ~2 years of accumulated update drift.
- **[docs/os-options.md](docs/os-options.md)** — postmarketOS vs Mobian
  vs Ubuntu Touch vs staying on Manjaro, as of August 2026. Real
  tradeoffs, not marketing.
- **[docs/migrating-to-mobian.md](docs/migrating-to-mobian.md)** —
  concrete steps for moving from stock Manjaro to Mobian, including the
  Tow-Boot prerequisite.
- **[docs/scripts.md](docs/scripts.md)** — usage and a real warning
  about `wifi-toggle` (see below).
- **[scripts/](scripts/)** — `modem-toggle` and `wifi-toggle`, small
  power-management scripts for the cellular modem and WiFi radio.

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
