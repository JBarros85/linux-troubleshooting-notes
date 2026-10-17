# Firefox window partially hidden when using split screen on Ubuntu 26.04

## Problem

On Ubuntu 26.04 with GNOME and Wayland, Firefox could become partially positioned outside the visible screen area when using window tiling.

The issue was initially observed while Ubuntu's bundled Tiling Assistant extension was active.

Instead of fitting correctly inside the selected region, part of the Firefox window could remain outside the visible desktop area.

## Environment

The original issue was observed with:

```text
Ubuntu 26.04.1 LTS
GNOME Shell 50.1
Wayland
Firefox DEB from Mozilla
Ubuntu Tiling Assistant 54
```

The system extension was:

```text
tiling-assistant@ubuntu.com
```

## Initial diagnosis

The desktop environment and tiling extension were inspected with:

```bash
echo "$XDG_SESSION_TYPE"
lsb_release -ds
gnome-shell --version
gnome-extensions info tiling-assistant@ubuntu.com
```

The Ubuntu-provided extension reported:

```text
Version: 54
```

As a diagnostic test, it was disabled:

```bash
gnome-extensions disable tiling-assistant@ubuntu.com
```

After disabling it, the Firefox positioning problem stopped occurring.

This showed that the tiling extension was involved in the behaviour observed in this environment.

## Side effect of disabling the extension

Disabling Ubuntu Tiling Assistant also removed its quarter-screen tiling functionality.

GNOME's remaining tiling behaviour was mainly limited to left/right half-screen layouts.

This meant that permanently disabling the extension was not an ideal solution because four-window layouts were still required.

Example quarter-screen layout:

```text
+-----------+-----------+
|           |           |
|     1     |     2     |
|           |           |
+-----------+-----------+
|           |           |
|     3     |     4     |
|           |           |
+-----------+-----------+
```

## Follow-up investigation

A later audit showed that two different Tiling Assistant installations existed on the system.

### Ubuntu-packaged extension

```text
UUID: tiling-assistant@ubuntu.com
Path: /usr/share/gnome-shell/extensions/tiling-assistant@ubuntu.com
Version: 54
Enabled: No
State: INITIALIZED
```

### Active user extension

```text
UUID: tiling-assistant@leleat-on-github
Path: ~/.local/share/gnome-shell/extensions/tiling-assistant@leleat-on-github
Version: 56
Enabled: Yes
State: ACTIVE
```

The metadata of the active extension identifies the project as:

```text
https://github.com/ubuntu/Tiling-Assistant
```

and declares support for GNOME Shell:

```text
48
49
50
51
```

## Current working configuration

The current tested configuration is therefore:

```text
Ubuntu 26.04.1 LTS
GNOME Shell 50.1
Wayland
Firefox DEB from Mozilla

tiling-assistant@ubuntu.com
Version: 54
Status: disabled

tiling-assistant@leleat-on-github
Version: 56
Status: active
```

With Tiling Assistant 56 active:

- Half-screen tiling works
- Quarter-screen tiling works
- Four-window layouts work again
- Firefox can be tiled normally
- The previous window-positioning behaviour has not reappeared during subsequent normal use
- No further desktop freezes related to this workflow have been observed during subsequent use

These observations describe this specific system and should not be interpreted as proof that version 56 universally fixes every Firefox or GNOME tiling issue.

## Quarter-screen shortcuts

The configured quarter-screen shortcuts can be checked with:

```bash
gsettings get org.gnome.shell.extensions.tiling-assistant tile-topleft-quarter
gsettings get org.gnome.shell.extensions.tiling-assistant tile-topright-quarter
gsettings get org.gnome.shell.extensions.tiling-assistant tile-bottomleft-quarter
gsettings get org.gnome.shell.extensions.tiling-assistant tile-bottomright-quarter
```

In this environment they were:

```text
Super + Num7  -> top-left
Super + Num9  -> top-right
Super + Num1  -> bottom-left
Super + Num3  -> bottom-right
```

## How to identify which extension is actually active

When troubleshooting multiple installations, checking only one known UUID can produce the wrong conclusion.

List active extensions related to tiling:

```bash
gnome-extensions list --enabled | grep -Ei 'tiling|tile'
```

Inspect the Ubuntu version:

```bash
gnome-extensions info tiling-assistant@ubuntu.com
```

Inspect the user-installed version:

```bash
gnome-extensions info tiling-assistant@leleat-on-github
```

Find their installation locations:

```bash
find ~/.local/share/gnome-shell/extensions \
     /usr/share/gnome-shell/extensions \
     -maxdepth 1 \
     -type d \
     -iname '*tiling*' \
     -print 2>/dev/null
```

## About GNOME edge tiling

With Tiling Assistant controlling window placement, this setting may report:

```bash
gsettings get org.gnome.mutter edge-tiling
```

```text
false
```

This alone does not mean that window tiling is unavailable.

Tiling Assistant manages its own tiling behaviour and can override Mutter settings while active.

Stored overridden values can be inspected with:

```bash
gsettings get org.gnome.shell.extensions.tiling-assistant overridden-settings
```

## Final conclusion

The original advice to permanently disable:

```text
tiling-assistant@ubuntu.com
```

was incomplete.

Disabling version 54 was useful as a **diagnostic workaround** because it isolated the original Firefox positioning problem, but it also removed quarter-screen tiling.

The current working configuration uses:

```text
tiling-assistant@leleat-on-github
Version 56
```

while Ubuntu's bundled:

```text
tiling-assistant@ubuntu.com
Version 54
```

remains disabled.

This restores four-quadrant tiling while, in subsequent testing on this machine, the original Firefox positioning issue and desktop freezes have not been observed again.

## Lesson learned

When multiple versions of the same GNOME extension can exist simultaneously, troubleshooting should verify:

- UUID
- version
- installation path
- enabled state
- active state

The extension name alone is not enough to determine which implementation is actually controlling the desktop.

A workaround should also be documented together with its side effects rather than presented as a permanent fix before the final system state has been verified.
