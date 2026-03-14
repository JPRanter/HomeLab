# The Sysadmin Stick
### A persistent live Linux environment on a Ventoy USB toolkit — or: how hard could it possibly be?
 
---
 
I already have a 2TB external NVMe running Ventoy with every tool I've ever needed.
ISOs for days. WinPE environments. Rescue disks. The whole arsenal.
 
The problem is it's the size of a deck of cards and needs its own USB port.
What I wanted was something that fits on a lanyard. One stick. Plug it in anywhere,
boot into a familiar environment, open a browser, open a terminal, and get to work.
The tools live on the stick. The stick runs the tools.
 
How hard could it possibly be.
 
---
 
## The First Idea (It Didn't Work)
 
The obvious approach: Ventoy for ISO booting, MX Linux for the persistent live OS,
a `.dat` file for persistence. Ventoy supports dat file persistence. MX Linux is
Debian-based, XFCE by default, exactly what I wanted. Done. Ship it.
 
Except MX Linux uses antiX's live boot system, which has its own persistence
mechanism called `MX-Persist`. When Ventoy presents its `vtoy_persistent` device
mapper volume to the live system, antiX sees it and mounts it read-only. Not
because anything is broken. Not because the configuration is wrong. Just because
that's what antiX does with volumes it doesn't recognize as its own.
 
There's no fix for this. It's an architectural mismatch. MX and Ventoy dat file
persistence are simply not compatible. This is not documented clearly anywhere,
which is why I'm writing it down now.
 
Dead end number one.
 
---
 
## The Second Idea (Also Didn't Work, But For A Different Reason)
 
Fine. If persistence through a dat file won't work with MX, install MX directly
to a dedicated partition on the drive. Carve out 20GB, install to it, GRUB to
that partition only. Ventoy handles ISO booting, MX handles the persistent OS,
two separate partitions doing two separate jobs. Clean separation. Elegant, even.
 
The repartitioning went fine. sdb1 stayed as Ventoy's data partition, a new
sdb3 got carved out at 20GB ext4, and sdb2 — Ventoy's EFI partition — stayed
physically at the end of the drive where it belongs. Partition table looked like:
sdb1, sdb3, sdb2. Physically correct. Numerically out of order.
 
Ventoy has opinions about this.
 
```
!!!!!!!!!!!! WARNING !!!!!!!!!!!!
This is NOT a standard Ventoy device and is NOT supported (7).
Error message: <Disk partition layout check failed.>
```
 
Ventoy runs an integrity check on boot. It expects its own two partitions and
nothing else. A third partition — any partition, any number, any position —
fails the check. We tried renumbering sdb3 to sdb4 using `sfdisk` to make the
numbering sequential. Ventoy still rejected it. The check isn't about numbers.
It's about the presence of anything that isn't Ventoy.
 
The dedicated partition approach is a dead end. Ventoy will not coexist with a
third partition on the same drive. Full stop.
 
Dead end number two.
 
---
 
## Reconsidering The Problem
 
At this point I'd burned through MX Linux, a dat file persistence attempt,
a full repartition, a partition renumbering exercise involving `sfdisk` and
a backup partition table dump, and about three reformats of the drive.
 
The dat file approach was the right architecture. The problem was the distro.
 
MX uses antiX live boot — incompatible with Ventoy persistence.
The solution was a distro whose live boot system actually works with Ventoy's
dat file persistence mechanism. That means Ubuntu-family (casper-rw label,
automatic) or Debian Live (persistence label + a one-line config file).
 
I had an Xubuntu image handy. Ubuntu-based. Casper persistence. Should just work.
 
It did work. Persistence functioned correctly. Files survived reboots.
 
The boot time was six minutes. Shutdown hung and required a hard power-off.
On the second boot it took ten minutes to reach a usable desktop.
 
The culprit was obvious in retrospect: a full desktop environment running from
a dat file loop-mounted on a filesystem on a USB drive in a USB 2.0 port.
Every read, every write, every service starting at boot, every file manager
thumbnail — all of it fighting for the same 480Mbps theoretical ceiling that
USB 2.0 provides. In a production environment you take whatever port is
available. USB 2.0 is a real-world constraint, not an edge case.
 
Xubuntu works. It just doesn't work fast enough to be useful.
 
Dead end number three.
 
---
 
## The Architecture That Actually Works
 
The problem isn't the dat file. The problem is what gets written to it and when.
 
A Linux live system using overlayfs only writes the *diff* between the base ISO
and your changes to the persistence file. The ISO itself is read-only. If you
point high-churn directories — browser cache, systemd journal, apt cache,
thumbnail cache — at tmpfs instead of the dat file, the persistence layer
barely gets touched during a normal session. Writes go to RAM. Only meaningful
changes hit the dat file.
 
And then there's `toram`.
 
`toram` is a boot parameter that tells the live system to load the entire ISO
into RAM at boot. After that, the USB is almost completely idle. The system runs
entirely from memory. The USB 2.0 bottleneck gets compressed into a single hit
at startup — load the ISO into RAM, done — instead of being a constant drag on
every operation throughout the session.
 
A server you're trying to fix probably has RAM to spare. That's the resource
you want to use. The USB is just the delivery mechanism.
 
The final architecture:
 
- **Ventoy** — ISO bootloader, nothing else
- **Debian Trixie Live XFCE** — minimal base, overlayfs persistence, kernel 6.12
- **4GB dat file** — labeled `persistence`, contains a one-line `persistence.conf`
- **`guesttoram` in ventoy.json** — loads ISO into RAM at boot
- **Firefox ESR with disk cache disabled** — cache goes to RAM, dies on reboot;
  bookmarks, passwords, and session data persist to the dat file
- **Standard sysadmin toolkit** — installed to persistence, survives reboots
 
Boot time on USB-C: under two minutes.
Boot time on USB 2.0: acceptable. Not fast. Acceptable.
Shutdown: clean.
Persistence: confirmed across multiple reboots.
 
---
 
## Why Debian and Not Something Else
 
The distro selection went through several rounds.
 
Ubuntu-family was the first candidate because casper persistence just works with
Ventoy — no extra configuration, automatic detection. But Ubuntu carries overhead
that doesn't belong on an emergency toolkit, and the ecosystem is increasingly
opinionated in ways that get in the way. Xubuntu proved the mechanism. It didn't
prove the distro.
 
Kali was briefly considered because the use case — sysadmin emergency toolkit —
overlaps with what Kali ships with. The counterargument: Kali's toolset is deep
and interwoven, removing tools is painful on a persistence-constrained environment,
and most of the offensive security toolset isn't relevant here. Kali is the right
tool for a different job. When that job comes up, it gets its own stick.
 
Fedora and RHEL are off the table entirely. No live persistence mechanism exists
in the Red Hat family. Full stop.
 
Debian Live was home turf. Every container in the homelab cluster runs Debian.
The conventions are familiar. The package manager is familiar. When something
breaks at 2am in a server room, familiar is worth a lot. Trixie ships kernel 6.12,
which covers essentially all hardware you're likely to encounter. The persistence
setup requires one extra step compared to Ubuntu — a `persistence.conf` file
inside the dat — and that step takes about thirty seconds.
 
---
 
## What's On The Stick
 
```
/
├── DebianTrixie.iso          — persistent live OS (boots to RAM)
├── DebianTrixie.iso.dat      — 4GB overlayfs persistence
├── ventoy/
│   └── ventoy.json           — persistence + guesttoram config
├── Rescue/
│   ├── systemrescue-12.03-amd64.iso
│   ├── clonezilla-live-3.3.1-35-amd64.iso
│   ├── shredos-2025.11_28_x86-64_v0.40_20260204.iso
│   └── mt86plus_8.00_i586.iso
├── WinPE/
│   ├── HBCD_PE_x64.iso
│   └── Lazesoft_UE.iso
└── Tools/
    ├── Sysinternals/
    ├── BleachBit/
    ├── AngryIP/
    ├── HWInfo/
    ├── HDTUNE/
    ├── Revo/
    ├── Patch My PC/
    ├── Virus Removal/
    ├── WizTree64.exe
    ├── MBAM.exe
    ├── BatteryInfoView.exe
    └── rufus-4.6p.exe
```
 
The persistent Debian environment has Firefox ESR (disk cache disabled, RAM only),
KeePassXC, and a standard network/filesystem toolkit: nmap, wireshark, tcpdump,
iperf3, netcat, testdisk, smartmontools, tmux, vim, git, remmina, ansible.
 
---
 
## The Configuration Files
 
**ventoy/ventoy.json**
```json
{
    "persistence": [
        {
            "image": "/DebianTrixie.iso",
            "backend": "/DebianTrixie.iso.dat",
            "backend_type": "ext4"
        }
    ],
    "boot_menu": [
        {
            "image": "/DebianTrixie.iso",
            "guesttoram": true
        }
    ]
}
```
 
**persistence.conf** (inside the dat file)
```
/ union
```
 
That's it. One line. Everything else flows from the overlayfs union mount.
 
**Firefox ESR user.js** (~/.mozilla/firefox/[profile]/user.js)
```javascript
user_pref("browser.cache.disk.enable", false);
user_pref("browser.cache.memory.enable", true);
user_pref("browser.cache.memory.capacity", 524288);
user_pref("privacy.clearOnShutdown.cache", true);
user_pref("privacy.clearOnShutdown.cookies", false);
user_pref("privacy.clearOnShutdown.history", false);
user_pref("privacy.clearOnShutdown.sessions", false);
```
 
Cache is RAM-only. Every session is disposable. Passwords and bookmarks survive.
 
---
 
## Building It
 
**Prerequisites**
- Ventoy installed on the drive (standard install, no modifications)
- Debian Trixie Live XFCE ISO on the Ventoy data partition
 
**Create and format the persistence dat file**
```bash
# Create 4GB dat file
sudo dd if=/dev/zero of=/path/to/Ventoy/DebianTrixie.iso.dat \
    count=4096 bs=1M status=progress
 
# Format with persistence label
sudo mkfs.ext4 -L persistence /path/to/Ventoy/DebianTrixie.iso.dat
```
 
**Write persistence.conf into the dat file**
```bash
sudo mkdir -p /mnt/persistence_dat
sudo mount /path/to/Ventoy/DebianTrixie.iso.dat /mnt/persistence_dat
echo "/ union" | sudo tee /mnt/persistence_dat/persistence.conf
sudo umount /mnt/persistence_dat
```
 
**Create ventoy.json**
```bash
mkdir -p /path/to/Ventoy/ventoy
cat > /path/to/Ventoy/ventoy/ventoy.json << 'EOF'
{
    "persistence": [
        {
            "image": "/DebianTrixie.iso",
            "backend": "/DebianTrixie.iso.dat",
            "backend_type": "ext4"
        }
    ],
    "boot_menu": [
        {
            "image": "/DebianTrixie.iso",
            "guesttoram": true
        }
    ]
}
EOF
```
 
**Boot and verify**
 
Boot Ventoy, select DebianTrixie.iso, choose the persistence boot option.
Create a test file. Reboot. Confirm it survived.
 
Then install your toolkit inside the persistent environment while connected
to a fast USB port. Write everything to persistence once, on good hardware.
Don't do it over USB 2.0.
 
---
 
## What I Learned
 
**MX Linux and Ventoy dat persistence are not compatible.** antiX's live boot
system treats the Ventoy persistence volume as read-only. This is not fixable
through configuration. Don't try.
 
**Ventoy will not coexist with a third partition.** Its integrity check rejects
any drive layout that isn't exactly its own two partitions. Partition numbers,
physical position, filesystem type — none of it matters. A third partition fails
the check regardless.
 
**`toram` changes the equation on USB 2.0.** Without it, a full DE on a USB 2.0
dat file is too slow to be useful. With it, the USB bottleneck is front-loaded
at boot and the rest of the session runs from RAM.
 
**Do your setup work on a fast port.** apt installs, browser configuration,
KeePass setup — all of that writes to the persistence dat. Do it on USB-C or
USB 3.x. The result persists to whatever port you boot from in the field.
 
**The dat file only needs to hold your changes.** 4GB is enough. You don't need
20GB of persistence for a toolkit that's mostly running from RAM anyway.
 
---
 
## A Note On Process
 
This was built collaboratively with Claude (Anthropic) as a technical partner
across a long working session. The direction, the decisions, and the dead ends
are mine. Claude helped reason through failure modes, suggested the `toram`
architecture, and kept track of what we'd already tried so we didn't
re-diagnose settled problems.
 
That collaboration model is documented in the main infrastructure README
for anyone curious about what AI-assisted infrastructure work actually
looks like in practice. The short version: it looks like this.
 
---
 
*Part of the [HomeLab](https://github.com/JPRanter/HomeLab) infrastructure project.*
