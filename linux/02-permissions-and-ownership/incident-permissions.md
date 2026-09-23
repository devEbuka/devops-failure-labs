# Linux 02: Permission and Ownership Breakage

**Category:** Linux · **Date:** 2026-09-23 · **Env:** Ubuntu EC2, nginx

## What I broke
Locked down the web root directory so nginx's worker couldn't traverse it, then
separately made the config file unreadable by anyone.

```bash
chmod 700 /var/www/html
chmod 000 /etc/nginx/nginx.conf
systemctl restart nginx
```

## Symptom
Directory lockdown: nginx stayed `active (running)`, but every request returned
**403 Forbidden**, both from `curl` and a real browser. The service looked
healthy to a naive uptime check while every visitor got an error.

```
"/var/www/html/index.html" is forbidden (13: Permission denied)
```

Config lockdown: **nginx restarted successfully anyway.** `chmod 000` on
`nginx.conf` had no effect at all, because the process runs as root, and root
bypasses standard Unix permission checks. Confirmed by testing as a non-root
user instead:

```bash
sudo -u www-data cat /etc/nginx/nginx.conf
# cat: /etc/nginx/nginx.conf: Permission denied
```

## Root cause
Directory-level permissions block access even when the file itself is
world-readable — Linux checks execute (`x`) permission on every directory in
the path before it ever looks at the file. Separately, root is exempt from
standard file permission checks entirely, so `chmod 000` only protects a file
from non-root processes — a fact a root-owned nginx master process never
exposes in a naive test.

## Fix
```bash
chmod 755 /var/www/html
chmod 644 /etc/nginx/nginx.conf
systemctl restart nginx
```

## Lesson
"Service is active" and "service works for users" are different claims — a 403
looks identical to a healthy process in `systemctl status`. And any permission
test run as root will silently pass even when the permissions are wrong,
because root ignores them; test as the actual service user (`sudo -u www-data`)
to get a result that means anything.
