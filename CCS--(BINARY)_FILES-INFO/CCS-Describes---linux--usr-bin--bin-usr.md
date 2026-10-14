| Directory | Purpose | Users |
|---|---|---|
| `/bin` | Essential user commands needed even in single-user/no-network boot | All users |
| `/sbin` | System administration binaries (often need root) | Root/admins |
| `/usr/bin` | The bulk of user-facing programs | All users |
| `/usr/sbin` | Non-essential admin tools | Root/admins |
| `/usr/local/bin` | Locally compiled/installed software — untouched by the package manager | Site admin |
| `/opt/<vendor>/bin` | Self-contained third-party packages (Chrome, some commercial apps) | Varies |
| `/snap/bin` | Snap package shims | All users |

**What lives where, concretely:**

- **`/bin`** (traditional): `bash`, `sh`, `cp`, `mv`, `ls`, `cat`, `grep`, `mount`, `umount`, `ps`, `ping`, `su`, `dd`, `tar`
- **`/sbin`**: `fdisk`, `fsck`, `ip`, `iptables`, `reboot`, `shutdown`, `ifconfig` (legacy), `ip6tables`, `mkfs.*`, `swapon`
- **`/usr/bin`**: the big bucket — `apt`, `dpkg`, `python3`, `gcc`, `git`, `curl`, `ssh`, `dig`, `nslookup`, `nano`, `vim`, plus hundreds of desktop/tool packages
- **`/usr/sbin`**: `useradd`, `visudo`, `ntpd`, `syslogd`, `update-grub`, `tcpdump`
- **`/usr/local/bin`**: your compiles, manual installs — the one place the package manager won't overwrite
- **`/etc/alternatives`** + symlinks: Debian's mechanism for choosing between same-named binaries (`editor`, `java`, `awk`)

**The critical modern detail — the `/usr` merge:**

Since systemd-era distros unified the tree, **`/bin` is now just a symlink to `/usr/bin`** on all your machines. Check it yourself:

```bash
ls -ld /bin /sbin /usr/bin
```

On KDE Neon, Zorin 18, Kubuntu, and Ubuntu Server you'll see `/bin -> usr/bin` and `/sbin -> usr/sbin`. Same content, one directory — this was done so `/usr` can be mounted read-only or shared across machines, and so packaging stops being split-brained. Arch pioneered it, Debian/Ubuntu followed around Debian 12 / Ubuntu 23.04+ era (your kernels 6.8 and 6.17 systems all postdate it). Notably, Fedora/openSUSE did the same — virtually every mainstream distro has now.

One more distinction worth having in your mental model, security-wise:

- **Binaries owned by the package manager** (`dpkg -S /usr/bin/dig` verifies which package installed a file, and `debsums -c` can detect tampering with packaged files)
- **Binaries NOT owned by anyone** — anything in `/usr/local/bin`, `/opt`, or dropped directly into `/usr/bin` stands out in audits: `dpkg -S <path>` returning "no path found" is a red-flag triage step when hunting for implant persistence.

Given your security work, that last point pairs nicely with `debsums`, `find /usr/bin -newer /etc/os-release -type f` (recently modified binaries), and checking for binaries in odd places — common persistence locations include `/tmp`, `/dev/shm`, and hidden dirs, since legitimate executables never belong there.

Want me to walk through a provenance audit script — something that sweeps your PATH-listed directories and flags anything unowned by dpkg, unusual timestamps, or unset hardening flags?

## Essential commands (`/bin` → `/usr/bin`)

| Binary | What it does |
|---|---|
| `bash` / `sh` | Your shells — `sh` is dash on Ubuntu, deliberately minimal and fast for scripts |
| `cp`, `mv`, `rm`, `ls`, `ln`, `mkdir` | Coreutils file ops — copying, moving, deleting, listing, linking |
| `cat`, `head`, `tail`, `wc` | Reading/streaming files; `tail -f` for live log watching |
| `grep` | Text/pattern searching — your constant companion for log parsing |
| `sed`, `awk` | Stream editors — find-and-replace and column-wise processing |
| `find`, `xargs` | File searching and piping results to commands |
| `tar`, `gzip`, `xz`, `zstd` | Archiving and compression — `.pkg.tar.zst` uses zstd |
| `dd` | Raw byte-level copying — disk imaging, writing ISOs to USB |
| `ps`, `top` | Process snapshots and live monitor |
| `kill` | Send signals to processes (TERM, KILL, HUP) |
| `chmod`, `chown`, `umask` | Permission and ownership control — root of the security model |
| `mount`, `umount` | Attach/detach filesystems — USB mounts, LVM, bind mounts |
| `su`, `sudo` | Privilege elevation; `sudo` is logged and per-command configurable |
| `df`, `du`, `free`, `sync` | Disk/space/memory stats; flush buffers to disk before pulling a drive |
| `hostname`, `uname`, `dmesg` | System identity and kernel ring buffer (hardware/driver messages — you've lived here during GPU debugging) |
| `sleep`, `date`, `env`, `echo` | Scripting plumbing |
| `ping` | ICMP reachability/l latency checks |

## Admin tools (`/sbin` → `/usr/sbin`)

| Binary | What it does |
|---|---|
| `fdisk`, `parted`, `blkid` | Partition tables and block device identification |
| `mkfs.ext4`, `mkswap`, `fsck` | Create filesystems, swap, and repair them |
| `swapon`/`swapoff` | Manage swap devices |
| `ip` / legacy `ifconfig` | Interface/IP/route config — `ip a`, `ip r`, `ip -br addr` |
| `iptables`, `nft` | Kernel firewall rule management (nftables replaced iptables as backend) |
| `shutdown`, `reboot`, `halt`, `poweroff` | Power lifecycle |
| `useradd`, `usermod`, `userdel`, `groupadd` | Account management |
| `visudo` | Safely edit sudoers — syntax-checks before saving, prevents lockout |
| `rsyslogd`, `systemd-*` | Logging daemon and systemd helpers (note: most systemd binaries live in `/usr/lib/systemd` now) |
| `update-grub` | Debian/Ubuntu wrapper regenerating GRUB config |
| `tcpdump` | Packet capture from CLI |
| `lvm` suite | Logical volume management — the tools behind your NVMe LVM2 setup |
| `dhclient` | Legacy DHCP lease acquisition (Netplan/systemd-networkd handles this on modern installs) |

## The workhorse tier (`/usr/bin`)

Grouped by your likely use:

**Networking & DNS** — `ss` (socket stats, replaced netstat), `dig` + `nslookup` + `host` (DNS queries — your domain-monitoring staple), `traceroute`, `curl` + `wget` (transfers), `ssh`/`scp`/`sftp` (remote access and encrypted file movement)

**System introspection** — `strace` (trace syscalls), `lsof` (open files/sockets by process), `lspci`/`lsusb` (hardware enumeration — your `/dev` driver checks), `nproc`, `uptime`, `id`, `journalctl` (systemd journal reader — your primary log interface now)

**Package management** — `apt`, `apt-get`, `dpkg` and friends: `apt-cache policy <pkg>` shows repo/version candidates, `dpkg -S` maps file→package, `dpkg -L` the reverse

**Build/dev** — `gcc`, `make`, `python3`, `perl`, `git`, `ld`, `ar`, `nm`, `objdump` (the binutils family we discussed for ELF inspection), `pkg-config`

**Editors/terminals** — `nano`, `vim.tiny` (full vim is separate on modern Ubuntu), `less`, `pager` wrappers

**Userspace daemons & agents** — `curl`-adjacent things like `gnome-keyring-daemon` bits and desktop helpers live alongside; roughly half of `/usr/bin` is populated by desktop-environment apps

## `/usr/local/bin` — yours alone

Nothing "lives" here by default on a fresh install; it's conventionally empty. On your machines it holds whatever you've compiled or dropped in: your Proton Drive backup scripts, hand-built tools, symlinked AppImages. Package-managed `PATH` hygiene means this directory shadows system binaries in lookup order — which is exactly why it's worth auditing periodically, since malware dropped there executes in preference to genuine tools if names collide.

Worth knowing the full precedence chain on your systems, since it affects script behavior:

```bash
echo $PATH
```

Typically: `/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin` — local wins, then administrative, then system. That ordering is why `~/.local/bin` (where user-installed pip/pipx tools go) and `~/bin` sometimes shadow system commands too.

Want me to map any of these onto a live exercise — e.g., running `ls /usr/bin | wc -l` to see your actual population count, then auditing which ones aren't dpkg-owned? That's a good baseline security snapshot per machine.
