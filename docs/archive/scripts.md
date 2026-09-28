> **Archived 2026-09-28.** Local radio-toggle scripts from when this
> phone was more of a daily driver — not part of the current SSH/sensor
> project (that work talks to `mmcli`/`rfkill` directly). Kept in case
> day-to-day local use comes back into the picture. Still functionally
> fine on postmarketOS as far as we've verified.

# Radio toggle scripts

Two small scripts for power management: `modem-toggle` and
`wifi-toggle`. Both live in `scripts/` (next to this file, under
`docs/archive/`), both follow the same interface, both actually query
live state on every invocation rather than assuming — if you ask for
status, you get a fresh read every time, not a cached guess.

## Installing

```bash
mkdir -p ~/bin
cp scripts/modem-toggle scripts/wifi-toggle ~/bin/
chmod +x ~/bin/modem-toggle ~/bin/wifi-toggle
```

Add `~/bin` to your `PATH` if it isn't already (add to `~/.bashrc`):
```bash
export PATH="$HOME/bin:$PATH"
```
This only takes effect in new *interactive* shells — most `.bashrc`
files (Manjaro's default included) have a guard near the top
(`[[ $- != *i* ]] && return`) that skips the rest of the file for
non-interactive shells, e.g. one-shot `ssh host "command"` invocations.
That's normal, not a bug — your actual terminal sessions will see it
fine.

## Usage

Both scripts take the same four subcommands, default to `toggle` if
none given:

```bash
modem-toggle          # toggles, then shows resulting state
modem-toggle on
modem-toggle off
modem-toggle status    # just reports current state, changes nothing

wifi-toggle            # same interface
wifi-toggle on
wifi-toggle off
wifi-toggle status
```

`modem-toggle` uses ModemManager (`mmcli -m 0 --enable`/`--disable`).
`wifi-toggle` uses `rfkill block`/`unblock wifi`.

**On postmarketOS specifically:** there's a second layer involved now
that Manjaro didn't have — `eg25-manager`, a systemd service that
actually pulses the modem's power-on GPIO sequence (ModemManager only
talks to the modem once it's already up; it doesn't power it on).
`modem-toggle` still works as documented, but if the modem is ever
missing entirely (`mmcli -L` finds nothing, not just "disabled"), that's
`eg25-manager`'s problem, not this script's — check
`systemctl status eg25-manager` before assuming the toggle script is
broken. See
[../postmarketos-1.2b-fixes.md](../postmarketos-1.2b-fixes.md#problem-2-fixing-the-magnetometer-silently-kills-the-cellular-modem)
for a real case of this.

## ⚠️ If you're SSH'd in over WiFi, do not test `wifi-toggle off` that way

This is not a hypothetical warning — we did this and it cut the
connection immediately, no way to recover it remotely. If your SSH
session to the phone is itself going over its own WiFi radio, running
`wifi-toggle off` (or `toggle`, if WiFi is currently on) kills the very
link your session depends on mid-command. The command actually
succeeds on the phone's end — you just can't see the result or run
`wifi-toggle on` again to undo it, because you're no longer connected
to anything.

**If this happens to you:** the only way back in remotely is a
different network path — a wired/USB-ethernet connection if the device
supports one, or physical access to toggle WiFi back on locally. There
is no remote fix once the radio carrying your only connection goes
down.

**Safe ways to test `wifi-toggle off`:**
- From a local console session on the device itself (not over the WiFi
  you're about to disable).
- Over a wired connection, if available.
- Accept that if you test it over WiFi-SSH, you're committing to
  physical access (or another network path) to recover.

## Why `modem-toggle` uses enable/disable, not power-state directly

An earlier version tried `mmcli -m 0 --set-power-state-off/on`
directly. This fails with `Cannot set power state: modem either
enabled or initializing` whenever the modem is actively enabled —
ModemManager's state machine wants you to disable first. `--enable`/
`--disable` doesn't have this precondition and is the more reliable
toggle mechanism.
