# Launch a Firefox website as a separate GNOME application on Ubuntu 26.04

## Problem

On Ubuntu with GNOME and Wayland, I wanted to open a self-hosted web application as if it were a native desktop application.

The goal was to have:

- Its own icon
- A separate window
- A separate entry in the GNOME dock
- No grouping with normal Firefox windows
- The ability to pin it to the dock like a regular application

Example application URL:

```text
http://192.168.10.20:8888
```

The application itself continued to run inside Firefox.

## Environment

Tested with:

```text
Ubuntu 26.04
GNOME Shell 50
Wayland
Firefox DEB from Mozilla
```

The Firefox installation was the Mozilla DEB package rather than the Ubuntu Snap version.

Example binary:

```text
/usr/lib/firefox/firefox-bin
```

## Attempt 1 — Basic `.desktop` launcher

The first attempt used a standard desktop launcher:

```bash
firefox --new-window http://192.168.10.20:8888
```

The site opened correctly, but GNOME still identified the window as Firefox.

When pinned to the dock, it remained associated with the Firefox application.

## Attempt 2 — `--name` and `StartupWMClass`

The next attempt used:

```bash
firefox --name nomad
```

together with:

```ini
StartupWMClass=nomad
```

This still did not produce consistent application separation under GNOME/Wayland.

## Attempt 3 — Firefox experimental Taskbar Tabs / Web Apps

Firefox experimental taskbar functionality was also tested.

In:

```text
about:config
```

the following preference was enabled:

```text
browser.taskbarTabs.enabled = true
```

The related Firefox interface became available, but in this Ubuntu/GNOME/Wayland environment it did not create a usable Web App or `.desktop` launcher.

No corresponding application entry was created under:

```text
~/.local/share/applications/
```

## Solution

The working solution was to launch Firefox as a separate instance using all three options:

```text
--no-remote
--name nomad
--class nomad
```

The identifiers must match the desktop launcher configuration.

Example launcher:

```ini
[Desktop Entry]
Version=1.0
Name=NOMAD
Comment=NOMAD Command Center
Exec=/usr/bin/firefox --no-remote --name nomad --class nomad --profile /home/<user>/.mozilla/firefox/<profile> --new-window http://192.168.10.20:8888
Icon=/home/<user>/.local/share/icons/nomad.svg
Terminal=false
Type=Application
StartupNotify=true
StartupWMClass=nomad
Categories=Utility;
```

Save the file as:

```text
~/.local/share/applications/nomad.desktop
```

Then make it executable and update the desktop application database:

```bash
chmod +x ~/.local/share/applications/nomad.desktop
update-desktop-database ~/.local/share/applications
```

## Application Icon

In this case, the application's own SVG icon was copied from its Docker container:

```bash
docker cp <container-name>:/app/public/nomad-primary.svg \
  ~/.local/share/icons/nomad.svg
```

The desktop launcher then referenced:

```text
/home/<user>/.local/share/icons/nomad.svg
```

## Result

After using:

```text
--no-remote
--name nomad
--class nomad
```

together with a matching:

```text
StartupWMClass=nomad
```

the application:

- Opened in its own Firefox window
- Used its own application icon
- Appeared separately from Firefox in the GNOME dock
- Could be pinned like a normal desktop application
- Continued using Firefox underneath
- Retained Firefox features such as offline translation

## Why it worked

The key was giving the Firefox instance a distinct application identity that GNOME could associate with the `.desktop` launcher.

Conceptually:

```text
nomad.desktop
      |
      +---- StartupWMClass=nomad
      |
      v
Firefox
      |
      +---- --name nomad
      +---- --class nomad
      +---- --no-remote
      |
      v
Separate GNOME application identity
```

Using only `--new-window` was not enough because GNOME continued associating the window with the normal Firefox application.

## Useful diagnostic commands

Check the current display session:

```bash
echo "$XDG_SESSION_TYPE"
```

Check Ubuntu version:

```bash
lsb_release -ds
```

Check GNOME version:

```bash
gnome-shell --version
```

Check Firefox binary:

```bash
which firefox
readlink -f "$(which firefox)"
```

Check installed Firefox-related desktop launchers:

```bash
find ~/.local/share/applications -maxdepth 1 \
  -iname '*firefox*.desktop' -o \
  -iname '*nomad*.desktop'
```

## Notes

This solution was confirmed in a specific Ubuntu 26.04 / GNOME / Wayland environment.

Behaviour may differ depending on:

- Firefox version
- Firefox packaging format
- GNOME version
- X11 versus Wayland
- Desktop extensions
- Firefox profile configuration

The example IP address, user name, profile path and container name in this document are intentionally sanitized.
