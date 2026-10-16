# Firefox window partially hidden when using split screen on Ubuntu 26.04

## Problem

On Ubuntu 26.04 with GNOME and Wayland, Firefox could become partially positioned outside the visible screen area when using window tiling.

The issue appeared while using Ubuntu's bundled **Tiling Assistant** extension.

Instead of fitting correctly inside the selected region, part of the Firefox window could remain outside the visible desktop area.

## Environment

Tested with:

```text
Ubuntu 26.04.1 LTS
GNOME Shell 50.1
Wayland
Ubuntu Tiling Assistant 54
Firefox DEB from Mozilla
```

## Diagnosis

The desktop session and extension were checked with:

```bash
echo "$XDG_SESSION_TYPE"
lsb_release -ds
gnome-shell --version
gnome-extensions list | grep -i tiling
gnome-extensions info tiling-assistant@ubuntu.com
```

The environment showed:

```text
Wayland
GNOME Shell 50.1
Ubuntu Tiling Assistant 54
```

## Initial workaround

Disabling the Tiling Assistant immediately stopped the Firefox positioning problem:

```bash
gnome-extensions disable tiling-assistant@ubuntu.com
```

After disabling the extension, Firefox could again be tiled using GNOME's native window management without being partially hidden.

However, this was only a **workaround**, not a complete solution.

## Important side effect

Ubuntu's Tiling Assistant provides functionality beyond GNOME's basic two-column tiling.

When the extension was disabled, quarter-screen tiling was lost.

Instead of being able to arrange four windows in:

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

the desktop was limited mainly to left/right half-screen tiling.

Therefore, permanently disabling Tiling Assistant is not an ideal fix for users who rely on quarter-screen layouts.

## Re-enabling quarter tiling

The extension can be enabled again with:

```bash
gnome-extensions enable tiling-assistant@ubuntu.com
```

Its status can be checked with:

```bash
gnome-extensions info tiling-assistant@ubuntu.com
```

In the tested configuration, the quarter-screen shortcuts were still configured as:

```text
Super + Num7  -> top-left
Super + Num9  -> top-right
Super + Num1  -> bottom-left
Super + Num3  -> bottom-right
```

They can be inspected with:

```bash
gsettings get org.gnome.shell.extensions.tiling-assistant tile-topleft-quarter
gsettings get org.gnome.shell.extensions.tiling-assistant tile-topright-quarter
gsettings get org.gnome.shell.extensions.tiling-assistant tile-bottomleft-quarter
gsettings get org.gnome.shell.extensions.tiling-assistant tile-bottomright-quarter
```

## About GNOME `edge-tiling`

While Tiling Assistant is active, this command may report:

```bash
gsettings get org.gnome.mutter edge-tiling
```

```text
false
```

This does not necessarily mean tiling is disabled.

Tiling Assistant can override GNOME/Mutter's native tiling settings while the extension is active.

The original values can be inspected with:

```bash
gsettings get org.gnome.shell.extensions.tiling-assistant overridden-settings
```

For example:

```text
{'org.gnome.mutter.edge-tiling': <true>, ...}
```

This means the extension has stored the previous Mutter setting while providing its own tiling behaviour.

## Current conclusion

Disabling:

```text
tiling-assistant@ubuntu.com
```

is a useful diagnostic step and can work around Firefox positioning issues.

However, it should not be presented as a universal or permanent fix because it also removes functionality provided by the extension, including quarter-screen tiling.

A better troubleshooting approach is:

1. Confirm whether Tiling Assistant is involved.
2. Temporarily disable it and test Firefox.
3. If the problem disappears, the extension is part of the interaction.
4. Re-enable it if quarter-screen tiling is required.
5. Test the specific Firefox and tiling behaviour again after re-enabling.
6. Avoid treating extension removal as a complete fix unless losing its additional tiling features is acceptable.

## Commands used during troubleshooting

```bash
echo "$XDG_SESSION_TYPE"
lsb_release -ds
gnome-shell --version

gnome-extensions info tiling-assistant@ubuntu.com

gsettings get org.gnome.shell.extensions.tiling-assistant default-move-mode

gsettings get org.gnome.shell.extensions.tiling-assistant tile-topleft-quarter
gsettings get org.gnome.shell.extensions.tiling-assistant tile-topright-quarter
gsettings get org.gnome.shell.extensions.tiling-assistant tile-bottomleft-quarter
gsettings get org.gnome.shell.extensions.tiling-assistant tile-bottomright-quarter

gsettings get org.gnome.mutter edge-tiling

gsettings get org.gnome.shell.extensions.tiling-assistant overridden-settings
```

## Lesson learned

A workaround that removes the visible symptom can also remove useful desktop functionality.

When documenting troubleshooting, it is important to record both:

- what solved or isolated the original symptom;
- what functionality was lost as a consequence.

This case is therefore documented as an interaction between Firefox, GNOME/Wayland and Ubuntu Tiling Assistant rather than as a confirmed universal Firefox bug.
