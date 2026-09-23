# Linux 03: Break a systemd Service

**Category:** Linux · **Date:** 2026-09-23 · **Env:** Ubuntu EC2, nginx

## What I broke
Typo'd the binary path directly in the vendor unit file, simulating a bad
manual edit or config-management mistake.

```bash
sudo sed -i 's|^ExecStart=.*|ExecStart=/usr/sbin/nginxx|' /lib/systemd/system/nginx.service
sudo systemctl daemon-reload
sudo systemctl restart nginx
```

## Symptom
Failure was loud and immediate — unlike the permissions lab, there was no
"looks healthy but isn't" gap:

```
Job for nginx.service failed because the control process exited with error code.
```

```
Process: 32285 ExecStart=/usr/sbin/nginxx (code=exited, status=203/EXEC)
(nginxx)[32285]: nginx.service: Unable to locate executable '/usr/sbin/nginxx': No such file...
```

`status=203/EXEC` specifically means the `exec()` syscall itself failed — the
kernel couldn't even launch the binary, as opposed to nginx starting and then
erroring internally. `errno 2` (`ENOENT`) confirmed it was a missing-file
problem, not a permissions or config-syntax problem.

## Root cause
A typo in the `ExecStart=` path inside the unit file. `nginx -t` (the
`ExecStartPre` syntax check) still passed, since it only validates nginx's own
config, not the systemd unit file pointing at the binary — so the pre-check
gave false confidence before the actual start step failed.

## Fix
```bash
sudo sed -i 's|^ExecStart=.*|ExecStart=/usr/sbin/nginx -g "daemon on; master_process on;"|' /lib/systemd/system/nginx.service
sudo systemctl daemon-reload
sudo systemctl restart nginx
```

**Prevention tool, tested directly:** `systemd-analyze verify nginx.service`
actually stats the `ExecStart=` binary and flags it if missing —
`Command /usr/sbin/nginxx is not executable: No such file or directory` —
catching this before any restart or downtime, not just a syntax linter.

## Lesson
Editing a vendor-shipped unit file directly (under `/lib/systemd/system/`) is
itself a mistake independent of the typo — package upgrades can silently
overwrite it later. The correct pattern is `systemctl edit <service>`, which
creates a drop-in override under `/etc/systemd/system/<service>.d/` instead.
Also: run `systemd-analyze verify <unit>` after any manual unit file edit,
before `daemon-reload` + `restart` — it catches exactly this class of error
with zero downtime.
