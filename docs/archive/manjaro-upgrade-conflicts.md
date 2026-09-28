> **Archived 2026-09-28.** We're off Manjaro now — see
> [../setup-from-scratch.md](../setup-from-scratch.md) for the current
> OS (postmarketOS). Kept in case Manjaro ever comes back into the
> picture, or is useful to someone else running it on a PinePhone.

# Resolving pacman upgrade conflicts on Manjaro ARM

Field notes from actually living through a badly-stale `pacman -Syu` on a
PinePhone (roughly 2 years of accumulated drift) and coming out the
other side with a working system. This is what actually happened, not
generic advice.

## Before you start

- **Back up anything you can't lose.** A full-system upgrade after a
  long gap is not a routine operation — it can and will break things.
- Check your free disk space. A big upgrade downloads and unpacks a lot
  of packages at once.
- If you're on a device with a desktop session running (Plasma Mobile,
  Phosh, etc.), be prepared for the GUI to not survive the upgrade even
  if the upgrade itself "succeeds." More on this below.

## The upgrade itself

```bash
sudo pacman -Syu
```

That's the whole command. The work is in what happens *after* it
reports success.

## Package conflicts during the upgrade

Pacman will sometimes refuse to proceed because two packages conflict,
or because a package needs to be removed before another can be
installed. Read the actual error — it usually names the exact
conflicting packages. Common patterns:

- **A package was replaced by another with a different name**
  (e.g. an old Qt5-specific package superseded by a combined one).
  Fix: let pacman remove the old one when it asks, or
  `sudo pacman -Rdd <old-package>` first if it won't ask cleanly.
  `-Rdd` skips dependency checks — only use it when you're sure the
  removal is safe (i.e. pacman itself is telling you this package is
  obsolete/replaced, not that you're guessing).
- **Orphaned packages left over from a removed dependency chain.**
  Find them with `pacman -Qtdq` (lists packages nothing depends on
  anymore). Review the list — don't blindly remove everything, some
  orphans are things you actually installed on purpose and just don't
  have a reverse-dependency. Remove what's genuinely cruft with
  `sudo pacman -Rns $(pacman -Qtdq)`.

## `.pacnew` files — don't ignore these

When pacman updates a config file you've customized, it doesn't
overwrite your version — it drops the new default next to it as
`whatever.pacnew`, leaving your customized version in place untouched.
Find them all:

```bash
sudo find /etc -name '*.pacnew'
```

For each one, **diff before deciding anything**:

```bash
diff /etc/path/to/file /etc/path/to/file.pacnew
```

Three outcomes:
1. **No meaningful difference** (e.g. the "new" file just has your
   exact settings but uncommented differently) — safe to just delete
   the `.pacnew`.
2. **New file has genuinely new options you want** — merge by hand,
   don't just overwrite. Copy the new lines you want into your existing
   file, keep your customizations.
3. **New file's defaults would revert something you deliberately
   hardened** — this is the one to watch for. We hit this exact case
   with `sshd_config`: the `.pacnew` reverted `PermitRootLogin` and
   `PasswordAuthentication` to more permissive defaults. If your
   current file is already stricter than the new template and there's
   nothing else new in the diff, just delete the `.pacnew` — don't
   apply it.

**Never blindly run `pacman-mksafe` or similar auto-merge tools without
reading the diff first**, especially for anything under `/etc/ssh/`,
`/etc/sudoers.d/`, `/etc/pam.d/`, or network configs. A silently-applied
default can quietly undo security hardening you put there on purpose.

## If the GUI doesn't come back after rebooting

This happened to us. The upgrade itself reported success, but Plasma
Mobile / SDDM didn't. What actually happened, and how we fixed it:

**Root cause:** an orphaned package (in our case `qt5-es2-declarative`)
that had no `Replaces`/`Conflicts` relationship with the new Qt5
packages it should have been superseded by. It just sat there, mismatched
against the rest of a freshly-upgraded Qt5 stack, and SDDM crash-looped
on startup because of the version mismatch.

**How to diagnose:** check the actual failure, don't guess.
```bash
journalctl -u sddm -b   # boot-time SDDM logs
journalctl -u plasma-* -b
```
Look for version-mismatch errors, missing symbol errors, or crash
loops. If you see a specific package named repeatedly in crash
tracebacks, that's your suspect.

**How we actually fixed it:**
```bash
pacman -Qtdq                      # look for orphans first
sudo pacman -Rdd <suspect-package> # remove the specific broken orphan
sudo pacman -S <correct-package>   # install what should have replaced it
```

**A second, unrelated trap we hit on the same upgrade:** even after
fixing the actual package issue, the home screen showed no icons and
the desktop looked broken. This was **not a configuration problem** —
config was already correct. It was **stale live session state** left
over from upgrading packages while a desktop session was still running
underneath. Fix:
```bash
# find and kill the actual running session processes
pkill -9 kwin_wayland
pkill -9 plasmashell
# let the display manager (sddm, with autologin) restart cleanly
```
A plain reboot sometimes doesn't clear this if the same broken session
state gets restored — force-killing the specific processes and letting
the login manager start fresh is what actually worked.

## General principle

When something breaks after an upgrade, **verify empirically rather
than guess**. Read the actual log output for the actual failing
service. Check what package versions are actually installed
(`pacman -Q <package>`) rather than assuming the upgrade did what the
changelog says it should have. Byte-for-byte diffs beat assumptions,
every time.
