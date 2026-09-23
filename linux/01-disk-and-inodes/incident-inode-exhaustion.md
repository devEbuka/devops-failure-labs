# Linux 01b: Exhaust the Inodes

**Category:** Linux · **Date:** 2026-09-23 · **Env:** Ubuntu EC2, `/dev/root` 878,080 inodes

## What I broke
Created empty files in a loop until the filesystem's inode table filled up, with
plenty of raw disk space still free.

```bash
python3 -c "
import os
for i in range(800000):
    try:
        open(f'f{i}', 'w').close()
    except OSError as e:
        print(f'FAILED at {i}: {e}')
        break
"
```

## Symptom
```
FAILED at 774912: [Errno 28] No space left on device: 'f774912'
```

Same error code as a disk-full failure — but `df -h` and `df -i` told two
different stories at the exact same moment:

```
/dev/root   6.7G  2.7G  4.0G  41%   /     <- 4.0G free
/dev/root   878080 878080  0  100%        <- 0 inodes free
```

## Root cause
Inodes and disk space are independent limits. You can exhaust one while the
other is nearly empty. The kernel's error text (`errno 28`) doesn't distinguish
between them — only checking `df -i` separately reveals which one actually failed.

## Fix
```bash
rm -rf /root/inodetest   # delete the files, frees inodes immediately
df -i /                  # confirm inodes recovered
```

## Lesson
"No space left on device" is ambiguous. Always check `df -h` (bytes) and
`df -i` (inodes) separately before assuming which resource is exhausted —
especially on filesystems with lots of small files (mail queues, session
stores, log directories).
