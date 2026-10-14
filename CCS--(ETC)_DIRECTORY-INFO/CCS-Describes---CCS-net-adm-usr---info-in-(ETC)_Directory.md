each one with three angles: **what it does**, **what you (admin/user) can do with it**, and **what an attacker could do with it** — since you work in cyber security, knowing the attack surface of each file is half the value.

## Core System Files

| File | What it does | User/Admin power | Attacker power |
|------|--------------|------------------|-----------------|
| `/etc/passwd` | Maps usernames → UID, home dir, default shell | Enumerate all accounts, find UID 0 dupes, spot system vs. human users | **Reconnaissance goldmine** — reveals valid usernames for brute-force/spray attacks. Also, if world-writable (misconfiguration), an attacker can add a user |
| `/etc/shadow` | Stores salted+hashed passwords (root-readable only) | Audit hash algorithms (`$6$` SHA-512, `$y$` yescrypt) — legacy `$1$` MD5 hashes should be re-set | If readable due to bad perms (should be `640`/`000`), hashes can be cracked offline |
| `/etc/group` | Defines groups and membership | Grant sudo/wheel/docker group membership; audit for unexpected group adds | Adding a user to `sudo` or `docker` group here = instant root escalation |
| `/etc/fstab` | Auto-mounts filesystems at boot | Add encrypted partitions, tmpfs, bind mounts, `noexec`/`nosuid` flags for hardening | Craft a mount that shadows `/etc` or drops `nosuid` — historic LPE vector |
| `/etc/hosts` | Static DNS overrides — checked *before* resolvers | Block domains (point them at `127.0.0.1` or `0.0.0.0`) — this is the heart of your telemetry blocking | **Local DNS poisoning** — redirect banking sites, mirror packages, silently hijack traffic |
| `/etc/hosts.deny` | TCP-wrappers deny list | Blacklist IPs/subnets per-service (`ALL: 192.168.1.`) | Historically bypassed (services not compiled with libwrap), but can also be flipped to lock you out of your own host |
| `/etc/issue` | Pre-login banner on TTY | Legal warning text; customization | Information disclosure — reveals distro/hostname if not sanitized |
| `/etc/profile` + `bash.bashrc` | System-wide shell init scripts | Push env vars, aliases, hardening (umask, HISTCONTROL) for all users | **Persistence foothold** — injected commands execute as every user logs in, including root |
| `/etc/environment` | pam_env-injected variables system-wide | Proxy settings, PATH additions | Modifying `PATH` here can hijack binaries for every session |
| `/etc/ld.so.conf` | Where the dynamic linker looks for libraries | Point to custom/optimized libs (e.g., hand-tuned builds) | **Library hijacking** — prepend a malicious lib path, intercept any dynamically-linked binary |
| `/etc/sudoers` | Who can run what as whom | Granular command whitelisting — e.g., allow only `systemctl restart nginx` without full root | Any misgrant (`NOPASSWD: ALL`, wildcards like `vim` or `tar` via sudo) = trivial LPE |
| `/etc/security/limits.conf` | Resource caps (ulimits) per user/group | Prevent fork bombs, cap file handles on servers | Lockout/DoS if malformed — or runaway processes if unset |

## Network Stack

| Item | What it does | User/Admin power | Attacker power |
|------|--------------|------------------|-----------------|
| `/etc/netplan/` | Declarative YAML network config rendered to systemd-networkd/NetworkManager | Static IPs, VLAN tagging, bonding, routes | Swap DNS/gateway → MITM the whole machine silently |
| `/etc/NetworkManager/` | Connection profiles (Wi-Fi PSKs stored here!) | Manage per-connection privacy settings, MAC randomization | Stored Wi-Fi credentials in plaintext-ish files; altering a profile redirects traffic |
| `/etc/resolv.conf` | Which DNS servers to query | Point to DoT-capable resolvers, local caches, blocklisting middlemen (like Portmaster) | **DNS redirection** — classic exfil/MITM pivot; you've caught foreign resolvers sneaking in before, this is where they'd land |
| `/etc/nsswitch.conf` | Order of lookups: files vs DNS vs LDAP | Tune lookup order for speed or override behavior | Pointing passwd → LDAP/compat historically enabled weird resolution exploits |
| `/etc/hosts.allow/.deny` pair | Together form TCP-wrapper ACLs | Rapid IP bans per daemon | Superseded largely by nftables; gives false sense of coverage |
| `/etc/protocols`, `/etc/services` | Read-only reference tables (protocol numbers, port→service names) | Informative — used by netstat/ss to pretty-print | Low risk, but identifies what a box expects to run |

## Boot, Kernel & Hardware

| Item | What it does | User/Admin power | Attacker power |
|------|--------------|------------------|-----------------|
| `/etc/default/grub` | GRUB kernel boot parameters | Lock bootloader with password, set `module.sig_enforce=1`, disable USB at boot | Passing `init=/bin/sh` at GRUB prompt = instant root (unless password-protected) |
| `/etc/modprobe.d/` | Blacklist/force-load kernel modules | Blacklist vulnerable or unnecessary drivers (`usb-storage`, `dccp`) — key hardening | Loading a malicious module = kernel-level persistence/rootkit |
| `/etc/udev/rules.d/` | Device plug-in behavior (permissions, symlinks, run-on-attach) | Auto-mount rules, device tagging — you've used this for renaming NICs | Rule with `RUN+=` on device attach executes as root — a nasty persistence trick |
| `/etc/systemd/` | Unit definitions, drop-ins, timers | Your hardening playground: `ProtectSystem=strict`, `PrivateTmp=yes`, `CapabilityBoundingSet=` | Malicious unit/timer placed here = root persistence surviving reboot detection; also `XDG_RUNTIME_DIR` abuse on multi-user boxes |
| `/etc/sysctl.conf` (+ `sysctl.d/`) | Live kernel tunables | Disable IPv4 forwarding, enable rp_filter, `kernel.kptr_restrict=2`, SYN cookie toggles | Relaxing these reverses your hardening invisibly |

## Scheduling, Logs & PAM

| Item | What it does | User/Admin power | Attacker power |
|------|--------------|------------------|-----------------|
| `/etc/cron.d/`, `cron.daily/` etc. | Timed jobs | Automated maintenance — and where your backup automation decisions live | **Classic persistence** — every pentest checks cron for writable dirs + root-run scripts |
| `/etc/pam.d/` | Auth pipeline per service | Enforce 2FA modules, password complexity (`pwquality`), faillock, session limits | One malicious PAM module logs *every password* on the box — this is how "pam_trojan" kits work |
| `/etc/logrotate.d/` | Rotation/compression policies | Retention tuning — keep enough for forensics, small enough for disk health | Trimming retention destroys your forensic trail post-compromise |

## Package Management (APT)

| Item | What it does | User/Admin power | Attacker power |
|------|--------------|------------------|-----------------|
| `/etc/apt/sources.list` + `sources.list.d/` | Repository definitions | Pin priorities, signed-by routing, distro upgrades | Injecting an evil repo supplies backdoored packages — mitigated only by signing |
| `/etc/apt/trusted.gpg.d/` + `keyrings/` | Repo signing keys | Verify publisher identity; prune stale keys like your Shiftkey cleanup | Adding an attacker key legitimizes their repo — apt treats it as trusted going forward |
| **GPG chain** overall | APT validates Release files against these keys | This is your provenance gate — exactly why you audit repos/keys | Key confusion or expired keys force fallback behavior that can be exploited |

## Desktop, Apps & Permissions

| Item | What it does | User/Admin power | Attacker power |
|------|--------------|------------------|-----------------|
| `/etc/X11/` | Display server config | Multi-monitor setups, keyboard layouts, DPI | Historical X server vulns; MITM local screensharing/keystroke capture on multi-user |
| `/etc/fonts/` | Font rendering/substitution | Antialiasing prefs, font-family fallbacks | Mostly benign — bad hinting config; remote-font exploits are rare |
| `/etc/apparmor/` | Mandatory Access Control profiles | Constrain daemons (like Firefox, Cups) to narrow syscall/filesystem scope | Disabling profiles widens blast radius post-exploit |
| `/etc/kde/` | System-wide Plasma settings | Theme/lock-down defaults for kiosk-style setups | Low risk |
| `/etc/ssh/` | SSH config (server & client) | Your toolkit — non-standard port, `PermitRootLogin no`, `AllowUsers`, key-only auth | If weakened (`PasswordAuthentication yes`, weak ciphers), that's the front door into the box |
| `/etc/tailscale/` | VPN mesh config/state (mostly under `/var/lib/tailscale` in practice) | Coordination with your headscale/Tailscale mesh | If modified or `--accept-routes` flipped on, traffic egress shifts unexpectedly |
| `/etc/portmaster/` | Portmaster local firewall/DNS filter rules | Your DNS-blocklist companion | Corrupting blocklist state can silently unblock telemetry |

## The Big-Picture Pattern

Notice the recurring themes — because this is where hardening effort pays off the most:

1. **Persistence vectors**: `profile`, `bash.bashrc`, `modprobe.d/`, `udev rules.d/`, cron, systemd units, PAM — all run as root or on every login. If any are mispermissioned, you've handed out root persistence. Worth auditing with `ls -la` to confirm nothing is world-writable.

2. **Traffic hijacking**: `hosts`, `resolv.conf`, `netplan`, NetworkManager profiles — controlling where DNS and traffic point means controlling what the box believes. Your blocklist work targets exactly this layer.

3. **Trust chain**: APT sources + GPG keys + `ld.so.conf` — the "what software can we trust" question, which is basically the philosophical core of how you evaluate every package.

4. **Escalation**: `passwd`, `group`, `sudoers`, PAM — who gets what privilege, checked at every privileged boundary.
