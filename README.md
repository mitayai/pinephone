# pinephone

Field notes and small tools for a **PinePhone Beta Edition (motherboard
rev 1.2b)**, written from actually living with one — not theory. This
repo is kept as a straight line from "blank/unknown phone" to our
current working state, not a running diary — see `docs/archive/` if you
want the history (a stale Manjaro install we moved off, the Mobian path
we considered but didn't take, the full OS comparison).

## Current state

Running **postmarketOS** (rebranded to **Nura** the same week — see
[docs/setup-from-scratch.md](docs/setup-from-scratch.md) for why the
rename doesn't change anything), Phosh UI, magnetometer and modem both
fixed for this hardware revision.

## Start here

1. **[docs/hardware.md](docs/hardware.md)** — identify your chassis,
   bootloader, and exact sensor chips first. This device's hardware
   revision (Beta Edition, 1.2b) has a specific sensor substitution that
   the rest of these docs assume you know about.
2. **[docs/setup-from-scratch.md](docs/setup-from-scratch.md)** — Tow-Boot
   (check for it, install it if it's not there), flashing postmarketOS,
   first boot, default login.
3. **[docs/postmarketos-1.2b-fixes.md](docs/postmarketos-1.2b-fixes.md)**
   — the stock image boots, but on a 1.2b unit the magnetometer is
   silently disabled and fixing that breaks the cellular modem in a
   non-obvious way. Both fixed here, in the order you'll actually hit
   them.

## Archive

**[docs/archive/](docs/archive/)** — superseded content, kept for
reference rather than deleted:
- `os-comparison-2026-08.md` — the full postmarketOS vs Mobian vs Ubuntu
  Touch vs staying-on-Manjaro comparison, and the dated decision log.
- `migrating-to-mobian.md` — the Mobian path, if that fits your situation
  better than ours did.
- `manjaro-upgrade-conflicts.md` — recovering a badly-stale Manjaro ARM
  install, from before we moved off it entirely.
- `scripts.md` / `scripts/` — `modem-toggle` and `wifi-toggle`, local
  radio-toggle tools from when this phone was more of a daily driver.
  Not part of the current SSH/sensor work, but still functional if
  day-to-day local use comes back into the picture.

None of these are maintained going forward; re-verify anything
version-specific before following them.

## Contributing

This started as one person's field notes, kept honest by only
documenting things actually verified against the hardware or the
projects' own current documentation — not assumptions. If you send a
fix, please keep that standard: cite where something came from, and
note the date you verified it, since PinePhone-adjacent projects move
fast and stale advice is worse than no advice.
