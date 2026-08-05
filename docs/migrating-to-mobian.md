# Migrating a PinePhone from stock Manjaro ARM to Mobian

Verified against the official Debian wiki install page as of August
2026 — re-check before following this if it's been a while, install
processes for niche hardware do shift.

## Prerequisite: Tow-Boot

Mobian's install method needs Tow-Boot (a U-Boot-based bootloader)
installed on the PinePhone's eMMC first. If you're not sure whether
it's already there, the simplest check is physical: power on and watch
for Tow-Boot's distinctive LED sequence (red, then yellow) and its own
boot screen — if the phone just boots straight into your existing OS
with no LED color change or boot menu, it's not installed.

**To install it:**

1. Download the eMMC Boot installer image (`mmcboot.installer.img`)
   from the [official Tow-Boot releases page](https://github.com/Tow-Boot/Tow-Boot/releases).
2. Write it to an SD card:
   ```bash
   dd if=mmcboot.installer.img of=/dev/XXX bs=1M oflag=direct,sync status=progress
   ```
   Replace `/dev/XXX` with your SD card's actual device — double-check
   this, `dd` won't ask twice.
3. Power the phone off completely, remove the battery, insert the SD
   card, reinsert the battery, power on.
4. Watch for the LED: red, then yellow. Screen shows a blue screen or
   the installer GUI.
5. In the GUI, select **"Install Tow-Boot to eMMC Boot"**. No storage
   erasure needed first. Remove the SD card once done, confirm the
   phone still boots your existing OS normally afterward (Tow-Boot
   just replaces the boot stage, it doesn't touch your OS).

## Installing Mobian

Mobian doesn't use a graphical installer for PinePhone (the
Calamares-style installer has been broken since January 2025 due to
Qt6 issues — irrelevant here, since this method doesn't use it).
Instead, you flash a pre-built image directly.

1. **Download the image** from
   [images.mobian.org/sunxi/](https://images.mobian.org/sunxi/) — grab
   the `.img.xz`, plus the matching `.sha256sum` and `.sha256sum.sig`.

2. **Verify it** — worth doing, you're about to overwrite your eMMC
   with this:
   ```bash
   curl -O https://repo.mobian.org/mobian.gpg
   gpg --import mobian.gpg
   gpg --verify *.sha256sums.sig
   shasum -c *.sha256sums
   ```

3. **Get eMMC access.** Either boot Tow-Boot's USB storage mode (hold
   volume-up during power-on) so the phone appears as a block device on
   another machine, or boot a JumpDrive image from SD card first.

4. **Flash it:**
   ```bash
   unxz mobian-sunxi-phosh-YYYYMMDD.img.xz
   sudo dd bs=64k if=mobian-sunxi-phosh-YYYYMMDD.img of=/dev/mmcblkX status=progress
   ```
   Confirm `/dev/mmcblkX` is really the PinePhone's eMMC before running
   this, not something on your own machine.

5. **First boot** takes noticeably longer than normal — automatic
   filesystem resize happens on first startup, this is expected, let it
   run.

## Default login

- Username: `mobian`
- Password: `1234` (also the PIN)
- Root is locked by default

**Change that password immediately** — it's a known default, not
something to leave in place even briefly on a device you carry around.

## After install

Same principle as the [upgrade conflict notes](resolving-upgrade-conflicts.md)
apply going forward, adjusted for apt instead of pacman — don't let
this image go stale either. Given Debian Stable's slower patch cadence
is the real tradeoff of choosing Mobian in the first place (see
[os-options.md](os-options.md)), staying current here matters more than
it would on a faster-moving distro.
