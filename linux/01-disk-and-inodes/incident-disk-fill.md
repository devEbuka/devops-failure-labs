# Linux 01a: Fill the Disk

**Category:** Linux · **Date:** 2026-09-23 · **Env:** Ubuntu EC2, `/dev/root` 6.7G

## What I broke
Filled the root filesystem step by step with `fallocate` until it hit 100% used,
then tried normal operations (`touch`, `apt update`, log writes, package installs)
against it.

```bash
fallocate -l 4G /root/diskfill/bigfile
fallocate -l 500M /root/diskfill/bigfile2
fallocate -l 320M /root/diskfill/bigfile3
apt install -y build-essential
```

## Symptom
At 92–96% used (hundreds of MB free), everything worked normally — no slowdown,
no warnings. The first real failure only hit at true near-zero free space, and a
large install (`build-essential`) failed **mid-unpack**, not at the start:

```
dpkg: error processing archive .../gcc-15-x86-64-linux-gnu...deb (--unpack):
 cannot copy extracted data for './usr/libexec/gcc/.../lto1' to '....dpkg-new':
 failed to write (No space left on device)
```

This left `apt` in a broken dependency state — even a tiny `apt install cowsay`
afterward failed with unrelated-looking dependency errors, not a disk error.

## Root cause
Disk-full failures aren't gradual — `df`'s `Use%` is a misleading proxy for how
much room is actually left. A multi-package install that runs out of space
mid-write doesn't just fail cleanly; it leaves dpkg's package database in a
partially-configured, broken state that outlives the original disk-full condition.

## Fix
```bash
rm /root/diskfill/bigfile        # free real space first
dpkg --configure -a              # surfaces exactly which packages are stuck
apt --fix-broken install -y      # re-unpacks the broken ones, reconfigures the rest
dpkg -l | grep <package>         # verify "ii" status, not just "no error"
```

## Lesson
A disk-full error during an install can corrupt the package manager's state, not
just fail the one command. Freeing space isn't enough on its own — you have to
actively repair with `dpkg --configure -a` + `apt --fix-broken install`.
