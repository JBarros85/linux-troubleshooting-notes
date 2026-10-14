# Firefox window partially hidden when using split screen on Ubuntu 26.04

## Problem

When Firefox was snapped to half of the screen, part of the browser window remained outside the visible desktop area.

Repeatedly resizing, maximizing or snapping the window again did not solve the problem.

## Environment

The issue was observed with:

```text
Ubuntu 26.04.1 LTS
GNOME Shell 50.1
Wayland
tiling-assistant@ubuntu.com
```

## Symptoms

When using split-screen mode:

- Firefox did not fit correctly inside half of the display
- Part of the window remained hidden
- Manual resizing did not permanently correct the position
- Re-snapping the window produced the same behaviour

Initially, the problem appeared to be related to Firefox itself.

## Diagnosis

The desktop environment and active tiling extensions were checked with:

```bash
echo "$XDG_SESSION_TYPE"
lsb_release -ds
gnome-shell --version
gnome-extensions list | grep -i tiling
```

The environment reported the Ubuntu Tiling Assistant extension:

```text
tiling-assistant@ubuntu.com
```

To isolate the issue, the extension was disabled without changing any Firefox configuration.

## Solution

Disable the Ubuntu Tiling Assistant extension:

```bash
gnome-extensions disable tiling-assistant@ubuntu.com
```

After disabling the extension, Firefox immediately started snapping correctly using GNOME's native window tiling.

No Firefox configuration changes were required.

## Re-enable Tiling Assistant

If needed, the extension can be enabled again with:

```bash
gnome-extensions enable tiling-assistant@ubuntu.com
```

## Result

In this specific environment, disabling only:

```text
tiling-assistant@ubuntu.com
```

resolved the split-screen positioning problem.

The issue therefore appeared to be related to the interaction between the Tiling Assistant extension and window placement rather than Firefox itself.

## Troubleshooting Principle

When an application window behaves incorrectly under GNOME, it can be useful to isolate desktop extensions before modifying the application.

A simple diagnostic approach is:

```text
Application problem suspected
        |
        v
Check desktop session
        |
        v
Check GNOME extensions
        |
        v
Disable suspected extension
        |
        v
Retest application
        |
        v
Identify affected layer
```

This avoids unnecessary application changes when the actual problem is in the desktop environment.

## Notes

This solution was confirmed on:

- Ubuntu 26.04.1 LTS
- GNOME Shell 50.1
- Wayland
- Firefox
- Ubuntu Tiling Assistant

Behaviour may differ on other GNOME, Ubuntu or Firefox versions.
