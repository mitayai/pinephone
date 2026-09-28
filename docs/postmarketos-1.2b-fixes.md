# Getting a PinePhone Beta Edition (1.2b) fully working on postmarketOS

Two real problems hit after flashing stock postmarketOS to a 1.2b unit
(see [hardware.md](hardware.md) for what that means), both solved, in the
order we actually hit them — the second one only exists *because* of the
fix for the first, so do read this in order if you're following along.

Verified 2026-09-28 against postmarketOS v26.06 (Phosh), kernel
`6.18.3_git20260105-r3`. Both fixes are config/data changes, not kernel
patches — nothing here needs recompiling anything.

## Problem 1: the magnetometer doesn't show up at all

No `af8133j` (or `lis3mdl`) device under `/sys/bus/iio/devices/`, no
mention of either in `dmesg`, even after `sudo modprobe af8133j` (which
loads fine — the driver's been in mainline since kernel 6.9 — but never
binds to anything).

**This is a known, tracked upstream gap, not something wrong with your
setup:** postmarketOS's own wiki confirms it for exactly this revision —
"the magnetometer cannot be used on the 1.2b revision PinePhone... the
driver is present in the kernel but the device tree does not define it
[as enabled]." Tracked at `pmaports#1945`.

**Root cause, precisely:** the `af8133j` node already exists, fully
wired (address `0x1c` on `i2c1`, `reset-gpios` on PB1, both supplies on
`reg_dldo1`), in mainline's shared `sun50i-a64-pinephone.dtsi` — but with
`status = "disabled"` in the generic `sun50i-a64-pinephone-1.2.dts`. A
**separate, already-correct** `sun50i-a64-pinephone-1.2b.dts`/`.dtb`
exists upstream, ships as part of the kernel package, and is sitting
right there on the phone's `/boot/dtbs/allwinner/` — with `af8133j`
flipped to `okay` and the (unpopulated, on this unit) `lis3mdl` flipped
to `disabled`. Confirmed by decompiling both with `dtc` and diffing them:
the *only* differences are those two status lines plus the model/
compatible strings.

**The actual gap:** postmarketOS's own `deviceinfo_dtb` (in
`/usr/share/deviceinfo/deviceinfo` for the `pine64-pinephone` device
package) only lists `allwinner/sun50i-a64-pinephone-1.1` and `-1.2` — it
doesn't know `1.2b` exists as a deployable option, even though the file
is built and present.

### The fix

```bash
# 1. Point postmarketOS's own override mechanism at the right tree
echo 'deviceinfo_dtb=allwinner/sun50i-a64-pinephone-1.2b' | sudo tee -a /etc/deviceinfo

# 2. Redeploy
sudo mkinitfs
```

**This alone is not enough.** `mkinitfs`/`boot-deploy` installs
`/boot/sun50i-a64-pinephone-1.2b.dtb` correctly, but the compiled U-Boot
script (`/boot/boot.scr`) loads the dtb via a `${fdtfile}` **U-Boot
environment variable** — set by Tow-Boot's own board-revision detection,
which can't tell 1.2 from 1.2b apart, not by anything `deviceinfo_dtb`
touches. `fw_printenv`/`fw_setenv` exist but need a `/etc/fw_env.config`
pointing at the right eMMC offset, which isn't set up — we deliberately
didn't guess at that offset; a wrong write there risks the boot
environment itself.

Instead, the safe fix: keep the filename `fdtfile` already resolves to,
just replace its *content*.

```bash
# Back up the original first — this is the recovery path if anything's wrong
sudo cp /boot/sun50i-a64-pinephone-1.2.dtb /boot/sun50i-a64-pinephone-1.2.dtb.orig-backup

# Overwrite it with the correct, already-built 1.2b tree
sudo cp /boot/dtbs/allwinner/sun50i-a64-pinephone-1.2b.dtb /boot/sun50i-a64-pinephone-1.2.dtb

sudo reboot
```

Recovery if boot ever misbehaves: this is *not* a bricking risk. Tow-Boot
itself is completely untouched by this. Boot back into Tow-Boot's USB
mass-storage mode (see [setup-from-scratch.md](setup-from-scratch.md)) from another
machine and restore the `.orig-backup` file over the live one.

**Known cosmetic wart:** the file is now named `sun50i-a64-pinephone-1.2.dtb`
on disk despite containing 1.2b content, because `fdtfile`'s resolution
couldn't be safely redirected. A correctly-named
`sun50i-a64-pinephone-1.2b.dtb` also exists from `boot-deploy` but isn't
the one actually loaded. Harmless, but confusing if you go looking later
— and **check this again after any future OS/kernel update**, since a
fresh `boot-deploy` run could regenerate the wrongly-named file from the
(still-wrong) `deviceinfo_dtb`/`fdtfile` resolution and silently revert
this fix.

### Verifying it worked

```bash
cat /proc/device-tree/model                    # should say "(1.2b)" now
cat /sys/bus/iio/devices/iio:device*/name       # af8133j should be listed
cat /sys/bus/iio/devices/iio:deviceN/in_magn_x_raw   # should return real, changing numbers
```

## Problem 2: fixing the magnetometer silently kills the cellular modem

After the reboot above, the modem stops enumerating on USB entirely —
`lsusb` shows nothing, `mmcli -L` says "No modems were found." **This
survives a full power cycle** (not just a reboot), which rules out a
PMIC warm-reset quirk — it's a real regression from the fix above, not a
timing issue.

**Root cause:** postmarketOS runs a dedicated daemon,
**`eg25-manager`**, that actually pulses the modem's PWRKEY/GPIO
power-on sequence (ModemManager just talks to the modem once it's up —
it doesn't power it on). `eg25-manager` picks its config file by an
*exact* match on the board's device-tree `compatible` string:
`/usr/share/eg25-manager/pine64,pinephone-<rev>.toml`. Configs exist for
`1.0`, `1.1`, `1.2`, and `pinephone-pro` — but not `1.2b`. The fix above
changes the compatible string from `pine64,pinephone-1.2` to `-1.2b` (as
part of correctly flipping to the 1.2b tree), and `eg25-manager` has no
fallback for an unmatched revision — it just crashes
(`journalctl -u eg25-manager` shows "unable to find a suitable config
file!" then a core-dump, repeatedly, until `start-limit-hit`).

The modem's actual wiring is identical between 1.2 and 1.2b (confirmed —
the dtb diff in Problem 1 showed zero modem/GPIO differences, only the
two sensor nodes), so reusing the 1.2 config is correct, not a
workaround.

### The fix

```bash
sudo cp "/usr/share/eg25-manager/pine64,pinephone-1.2.toml" \
        "/usr/share/eg25-manager/pine64,pinephone-1.2b.toml"
sudo systemctl reset-failed eg25-manager.service
sudo systemctl restart eg25-manager.service
```

Verify: `journalctl -u eg25-manager -n 10` should show "Executed
power-on sequence"; `lsusb` should show `2c7c:0125 Quectel EG25-G`
within a few seconds; `mmcli -L` should list the modem.

### The general lesson

**Changing this board's `compatible` string has a wider blast radius
than the device tree itself.** Anything doing exact-string config
lookups keyed on `pine64,pinephone-<rev>` needs its own `-1.2b` file —
`eg25-manager` is the one we found; there may be others we haven't hit
yet. **After any future revision/compatible-string change on this
device, check `systemctl --failed` before assuming everything's fine.**

## SSH access notes (systemd, not OpenRC)

This image runs **systemd**, not the OpenRC you might expect from older
postmarketOS docs (it's mid-migration away from OpenRC — `rc-update`,
`ss`, `lsb_release` are all absent; the shell is `/bin/ash`). Relevant if
you're setting up remote access:

- SSH isn't installed by default: `sudo apk add openssh` (still real
  Alpine `apk` under the hood). This creates a plain `sshd.service` — no
  `ssh.socket` socket-activation on this image, unlike some other
  systemd-ssh setups. Enable with `sudo systemctl enable --now sshd.service`.
- Lock to key-only auth once you've got a key in
  (`ssh-copy-id user@<phone-ip>`) via a drop-in, not editing the main
  config: `/etc/ssh/sshd_config.d/99-key-only.conf` with
  `PasswordAuthentication no`, `KbdInteractiveAuthentication no`,
  `PermitRootLogin no`. Test with `sudo sshd -t` before restarting.
- If you're using the archived `modem-toggle` script
  ([archive/scripts.md](archive/scripts.md)), be aware `eg25-manager` is
  now a *third* layer involved in modem state beyond ModemManager and
  the toggle script, on this OS specifically — Manjaro didn't have it.
