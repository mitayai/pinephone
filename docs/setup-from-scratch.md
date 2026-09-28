# Setting up a PinePhone Beta Edition from scratch: Tow-Boot + postmarketOS

The full path from "whatever's currently on this phone" (or nothing at
all) to our current working state. First, confirm you actually have the
same hardware — see [hardware.md](hardware.md) before following this,
since the fixes doc that comes after this one is revision-specific.

**Note on the name:** postmarketOS rebranded to **Nura** on 2026-09-27.
Genuine rebrand (announced at a community conference, covered by
multiple independent outlets, their own blog post exists), not a domain
hijack, even though `postmarketos.org` now redirects to `nura.eco`
(`images.postmarketos.org` still works too, redirecting to
`images.nura.eco`). We use "postmarketOS" throughout since that's still
the more recognizable name as of writing.

**Why postmarketOS**, briefly: broadest, most actively-maintained
hardware support across the Pine64 lineup (concretely true for us — our
exact 1.2b revision has a known, already-tracked, already-fixable
magnetometer gap, see the fixes doc below), and faster security patching
than Mobian's Debian-Stable cadence, which mattered more than usual since
the plan for this phone includes SSH access and remote sensor polling —
a bigger attack surface than "phone in your pocket." Ubuntu Touch was
ruled out — the non-Pro PinePhone port is still "under heavy development"
as of when we checked. Full comparison and reasoning, including the
Mobian path if you'd rather take that one:
[archive/os-comparison-2026-08.md](archive/os-comparison-2026-08.md).

## Step 1: check for Tow-Boot, install it if it's not there

Tow-Boot (a U-Boot-based bootloader) has to be on the eMMC before any of
the rest of this works.

**Check:** power on and watch the LED. Solid **blue** during a
Volume-Up hold (see step 2 below) means it's already there and you can
skip to Step 2. A **red-then-yellow** sequence with its own boot screen
during a *normal* power-on means the installer itself is present but
Tow-Boot isn't installed to eMMC yet — follow "Install" below. If the
phone just boots straight into whatever OS is on it with no LED color
change or boot menu at all, Tow-Boot isn't there either — same steps.

**Install:**

1. Download the eMMC Boot installer image (`mmcboot.installer.img`)
   from the [official Tow-Boot releases page](https://github.com/Tow-Boot/Tow-Boot/releases).
2. Write it to a microSD card:
   ```bash
   dd if=mmcboot.installer.img of=/dev/XXX bs=1M oflag=direct,sync status=progress
   ```
   Replace `/dev/XXX` with the SD card's actual device — double-check
   this, `dd` won't ask twice.
3. Power the phone off completely, remove the battery, insert the SD
   card, reinsert the battery, power on.
4. Watch for the LED: red, then yellow. Screen shows a blue screen or
   the installer GUI.
5. In the GUI, select **"Install Tow-Boot to eMMC Boot"**. No storage
   erasure needed first — this only replaces the boot stage, whatever OS
   is already on the eMMC (if any) is untouched. Remove the SD card once
   done.

## Step 2: get eMMC access (Tow-Boot's USB Mass Storage mode)

1. Power the phone off completely (long-press power ~5-6s; if it
   restarts instead of powering off, that's a known quirk on some
   Tow-Boot builds — just catch the reboot and continue to step 2 on it).
2. Press and hold **Volume Up**, then press **Power** while still
   holding Volume Up.
3. Keep holding Volume Up through the phone's *second* vibration pulse.
4. Watch for the LED to go **solid blue** — that's USB Mass Storage
   mode, distinct from the installer's red-then-yellow sequence in Step 1.
5. The eMMC now shows up as a USB block device on whatever machine it's
   plugged into (`lsblk` on Linux). Confirm it's really the phone before
   touching it (size, any partition labels you recognize from a previous
   OS). Unmount any auto-mounted partitions before writing to it.

## Step 3: get the postmarketOS image

1. Grab the current stable image for `pine64-pinephone` (not
   `pine64-pinephonepro` — easy to grab the wrong one) from
   `images.postmarketos.org/bpo/<version>/pine64-pinephone/<ui>/<date>/`.
   We used Phosh; `gnome-mobile`, `plasma-mobile`, and `sxmo-de-sway` are
   also built.
2. The page lists sha256 and sha512 for the `.img.xz` directly — verify
   both before writing anything to the eMMC.

## Step 4: flash it

```bash
unxz -k the-image.img.xz
sudo dd bs=64k if=the-image.img of=/dev/sdX status=progress conv=fsync
sync
```

Took about 6.5 minutes for a 650MB compressed / 3GB raw image over USB at
~8.2MB/s. Safely disconnect afterward (`udisksctl power-off -b /dev/sdX`
on Linux) before unplugging.

## Step 5: first boot

Power on normally (no button-holding needed once the eMMC has the new
image). **First boot is slow** — filesystem resize happens automatically,
let it sit even if the screen looks stuck for a few minutes.

**Default login:** username `user`, password `147147`. **Change it
immediately** — it's a known default, not something to leave in place on
a device you carry.

## Step 6: the revision-specific fixes

The stock image boots fine, but on a 1.2b unit the magnetometer is
silently disabled by default, and fixing that has a second, non-obvious
consequence for the cellular modem. Go straight to
[postmarketos-1.2b-fixes.md](postmarketos-1.2b-fixes.md) before you hit
either problem blind.
