# Migrating a PinePhone Beta Edition from stock Manjaro to postmarketOS

What we actually did, 2026-09-27/28, on a **PinePhone Beta Edition (rev
1.2b)** — see [hardware.md](hardware.md) if you're not sure that's what
you have too. Assumes Tow-Boot is already on the eMMC (it was, for us);
if you're not sure, see the LED-check in hardware.md, and the
Tow-Boot-installer steps in [migrating-to-mobian.md](migrating-to-mobian.md#prerequisite-tow-boot)
if you need to install it first (that part isn't Mobian-specific).

**Note on the name:** postmarketOS rebranded to **Nura** on 2026-09-27 —
literally the same day we did this migration. Genuine rebrand (announced
at a community conference, covered by multiple independent outlets, their
own blog post exists), not a domain hijack, even though
`postmarketos.org` now redirects to `nura.eco`. `images.postmarketos.org`
still works too (redirects to `images.nura.eco`). We use "postmarketOS"
throughout since that's still the more recognizable name as of writing;
expect docs/URLs to fully migrate to Nura branding over time.

## Why postmarketOS over Mobian or Ubuntu Touch

See [os-options.md](os-options.md) for the full comparison; the decision
that mattered for us specifically:

- **Broadest, most actively-maintained hardware support** across the
  Pine64 lineup, and — this turned out to matter concretely — an already
  *known, already fixable* gap for our exact 1.2b revision's magnetometer
  (see [postmarketos-1.2b-fixes.md](postmarketos-1.2b-fixes.md)), tracked
  upstream at `pmaports#1945`. A less-maintained project wouldn't have
  had that tracked, let alone half-fixed already.
- **Quarterly security patching** beats Mobian's Debian-Stable cadence
  (patches can lag months to years there) — this mattered more than
  usual for us because the plan was to expose SSH on this phone and poll
  its sensors remotely, which is a meaningfully bigger attack surface
  than "phone in your pocket."
- **Ubuntu Touch ruled out** for this exact device — re-checked live on
  UBports' own device page during this migration, the *original* (non-Pro)
  PinePhone port is still "under heavy development... many features
  including basic functionality may not work." Don't confuse this with
  the PinePhone Pro port, which is in much better shape — they're
  different codebases/ports.

If you just want the most direct transfer of Ubuntu/Debian knowledge and
don't care about the above, Mobian is still a completely legitimate
choice — see the existing [migrating-to-mobian.md](migrating-to-mobian.md).

## Getting eMMC access

1. Power the phone off completely (long-press power ~5-6s; if it restarts
   instead of powering off, that's a known quirk on some Tow-Boot builds —
   just catch the reboot and continue to step 2 on it).
2. Press and hold **Volume Up**, then press **Power** while still holding
   Volume Up.
3. Keep holding Volume Up through the phone's *second* vibration pulse.
4. Watch for the LED to go **solid blue** — that's Tow-Boot's USB Mass
   Storage mode. (This is different from the installer's red-then-yellow
   sequence in migrating-to-mobian.md — that one's only for installing
   Tow-Boot itself, not for using it once it's there.)
5. The eMMC now shows up as a USB block device on whatever machine it's
   plugged into (`lsblk` on Linux). Confirm it's really the phone before
   touching it — ours showed up as `/dev/sda`, 28.9G, with a partition
   labeled `BOOT_MNJRO` from the old Manjaro install. Unmount any
   auto-mounted partitions before writing.

## Getting the image

1. Grab the current stable image for `pine64-pinephone` (not
   `pine64-pinephonepro` — easy to grab the wrong one) from
   `images.postmarketos.org/bpo/<version>/pine64-pinephone/<ui>/<date>/`.
   We used Phosh; `gnome-mobile`, `plasma-mobile`, and `sxmo-de-sway` are
   also built.
2. The page lists sha256 and sha512 for the `.img.xz` directly — verify
   both before writing anything to the eMMC.

## Flashing

```bash
unxz -k the-image.img.xz
sudo dd bs=64k if=the-image.img of=/dev/sdX status=progress conv=fsync
sync
```

Took about 6.5 minutes for a 650MB compressed / 3GB raw image over USB at
~8.2MB/s. Safely disconnect afterward (`udisksctl power-off -b /dev/sdX`
on Linux) before unplugging.

## First boot

Power on normally (no button-holding needed once the eMMC has the new
image). **First boot is slow** — filesystem resize happens automatically,
let it sit even if the screen looks stuck for a few minutes.

## Default login

- Username: `user`
- Password: `147147`
- **Change it immediately** — same principle as Mobian's default, it's a
  known password, not something to leave in place on a device you carry.

## Before you do anything else

Read [postmarketos-1.2b-fixes.md](postmarketos-1.2b-fixes.md) if you're
on a 1.2b unit — the stock image boots fine, but the magnetometer is
silently disabled by default, and fixing that has a second, non-obvious
consequence for the cellular modem that you'll want to know about before
you hit it blind.
