# kde-wallpaper-setter

A lightweight Python script for setting per-monitor wallpapers on KDE Plasma 6 (Wayland) using the native DBus interface. Built for multi-monitor setups where existing GUI tools fall short.

---

## The Problem

If you run KDE Plasma on Wayland with multiple monitors — especially mixed orientations and resolutions — you've probably already discovered that the built-in wallpaper picker doesn't let you browse a folder, and that most third-party wallpaper managers don't actually work on KDE Wayland.

**HydraPaper** has a clean UI and looks like exactly the right tool. It isn't, on KDE Wayland. The Apply button does nothing. The reason is that HydraPaper is a GTK/GNOME application that communicates with the desktop through `xdg-desktop-portal-gtk`. On KDE Wayland, that service can't acquire a display connection because the session environment variables (`WAYLAND_DISPLAY`, `DISPLAY`) aren't passed to systemd user services by default. The result is a silent failure — no error, no feedback, no wallpaper change.

**Superpaper** would be the next candidate. It's not in the Fedora/Nobara repos. Flatpak doesn't have it either, at least not under the ID that documentation suggests.

**Variety** is in the repos but doesn't support per-monitor configuration.

KDE's own right-click desktop menu lets you set wallpapers per monitor, but only by picking individual files — no folder browsing, no workflow for people with large image libraries.

So you end up here.

---

## The Solution

KDE Plasma exposes its wallpaper configuration through a native DBus interface (`org.kde.plasmashell`) that works correctly on Wayland without any portal dependencies. This script talks directly to that interface.

Two things are required that aren't obvious from the KDE documentation:

1. Image paths must be prefixed with `file://` — passing a bare filesystem path is accepted silently but does nothing
2. `reloadConfig()` must be called after `writeConfig()` — without it, the config write succeeds and the wallpaper doesn't change

Both of these were found through testing, not documentation.

The script pairs the DBus calls with `kdialog` (ships with KDE Plasma) to provide a native file picker, and creates a `.desktop` entry so it lives in your app launcher like any other application. No terminal required for day-to-day use.

---

## Environment

Developed and tested on:

- **OS:** Nobara Linux (Fedora 43 base)
- **Desktop:** KDE Plasma 6, Wayland session
- **Monitors:** 3 displays, mixed orientations (landscape / portrait / landscape) and resolutions
- **Python:** 3.x
- **Dependencies:** `python3-dbus` (likely already installed), `kdialog` (ships with KDE Plasma)

Should work on any KDE Plasma 6 / Wayland setup. KDE Plasma 5 is untested.

---

## Installation

### 1. Verify dependencies

```bash
which kdialog
python3 -c "import dbus; print('ok')"
```

If `python3-dbus` is missing:

```bash
# Fedora / Nobara
sudo dnf install python3-dbus

# Debian / Ubuntu
sudo apt install python3-dbus
```

### 2. Check your monitor layout

Monitors are indexed left-to-right by screen X position. Run this to see your layout before editing the script:

```bash
qdbus org.kde.plasmashell /PlasmaShell org.kde.PlasmaShell.evaluateScript "
var d = desktops()
    .filter(d => d.screen != -1)
    .sort((a, b) => screenGeometry(a.screen).left - screenGeometry(b.screen).left);
d.forEach((desk, i) => print(i + ': ' + screenGeometry(desk.screen).width + 'x' + screenGeometry(desk.screen).height));
"
```

Output example:
```
0: 1920x1080
1: 1080x1920
2: 2560x1440
```

### 3. Edit the script

Open `wallpaper.py` and update two things:

**Monitor labels** — update the `MONITORS` dict to match your layout:
```python
MONITORS = {
    0: "Left   (1920x1080)",
    1: "Middle (1080x1920, portrait)",
    2: "Right  (2560x1440)",
}
```

**Default file picker path** — update `pick_file()` to open in your wallpaper directory:
```python
["kdialog", "--getopenfilename", "/your/wallpaper/folder/", ...]
```

### 4. Install

```bash
sudo cp wallpaper.py /usr/local/bin/wallpaper
sudo chmod +x /usr/local/bin/wallpaper
```

### 5. Create the desktop launcher

```bash
cat << 'EOF' > ~/.local/share/applications/wallpaper.desktop
[Desktop Entry]
Name=Wallpaper Changer
Comment=Set per-monitor wallpapers
Exec=wallpaper
Icon=preferences-desktop-wallpaper
Terminal=false
Type=Application
Categories=Utility;
EOF

update-desktop-database ~/.local/share/applications/
```

Search for **Wallpaper Changer** in your app launcher. Pin it to your taskbar if you change wallpapers frequently.

---

## Usage

### Interactive (recommended)

```bash
wallpaper
```

A KDE dialog will ask which monitor, then open a file picker pointed at your wallpaper folder.

### Command line

```bash
# Single monitor (0 = leftmost, incrementing left to right)
wallpaper one 0 /path/to/image.jpg
wallpaper one 1 /path/to/image.jpg
wallpaper one 2 /path/to/image.jpg

# All monitors at once
wallpaper all /path/left.jpg /path/middle.jpg /path/right.jpg
```

---

## Important

**Do not prefix paths with `file://`** when calling the script from the command line. The script adds this prefix internally before passing to DBus. Providing it twice causes a silent failure.

**Monitor indexes can shift** if you change your display layout in KDE Display Settings or physically rearrange monitors. Re-run the detection query from Step 2 to verify order after any layout change.

**External drives must be mounted.** If your wallpapers live on a secondary drive and it isn't mounted, KDE will silently revert to a default. The script won't error — the path just won't resolve.

---

## How This Was Built

This script came out of a troubleshooting session, not a planned project. The goal was simple: change wallpapers across three monitors without doing it through three separate right-click menus.

The session ran through HydraPaper (broken on KDE Wayland), Superpaper (not in repos), Variety (no per-monitor support), manual portal debugging (D-Bus session corruption, `xdg-desktop-portal-gtk` crash loops), and eventually landed on the KDE DBus interface as the correct native solution.

This was built collaboratively with **Claude (Anthropic)** — specifically Claude Sonnet, via claude.ai. The architecture decisions, the debugging direction, and the final calls were mine. Claude handled implementation, helped reason through the portal failure chain, and wrote the code once we understood what the actual solution was. The `file://` prefix requirement and the `reloadConfig()` necessity were both found through that back-and-forth — neither is documented clearly in KDE's own resources.

That collaboration model is documented in my [homelab README](https://github.com/jpranter/homelab) for anyone curious about what AI-assisted infrastructure work actually looks like in practice.

---

## License

MIT. Take it, adapt it, improve it.

If you find that this works (or doesn't) on a KDE Plasma 5 setup, or on a non-Fedora distribution, opening an issue with your findings would be useful to others.
