# Fluxbox patch for Qt/KDE file dialogs under X11

Date: 2026-07-14

## Repository state

This repository now contains a `debian/` directory that was added manually from the MX Linux source package metadata archive:

- https://mxrepo.com/mx/repo/pool/main/f/fluxbox/fluxbox_1.3.7+git20220731-0mx23+1.debian.tar.xz

That means this tree is no longer just a plain upstream/autotools source snapshot. It is now an upstream Fluxbox source tree plus Debian/MX packaging files imported from the MX Linux package source.

## Problem this patch tries to address

Under Fluxbox on X11, some Qt/KDE applications such as:

- Kate
- Kdenlive
- Ksnip
- Dolphin
- other Qt programs that open modal file dialogs

can appear to freeze when the user chooses `File -> Open...`.

In the observed failure mode, the application is not always truly hung. Instead:

- the modal dialog window is created,
- the app is blocked waiting inside the dialog event loop,
- but Fluxbox does not reliably bring that transient dialog to the front and focus it.

This makes the program look stuck even though the file dialog already exists.

## Root cause targeted here

The patch is aimed at an inconsistency in Fluxbox transient handling in `src/Window.cc`:

- during the initial creation path, transient dialogs already receive special treatment,
- but in `mapRequestEvent()` the focus path only considered `isFocusNew()`,
- and in `mapNotifyEvent()` Fluxbox only called `focus()` instead of explicitly raising the dialog first.

That asymmetry can leave a modal transient dialog mapped but not clearly visible or active.

## Change applied

File changed:

- `src/Window.cc`

### 1. Treat transient dialogs like focus candidates in `mapRequestEvent()`

Old code:

```cpp
if (isFocusNew()) {
```

New code:

```cpp
if (isFocusNew() || client->isTransient()) {
```

Effect:

- remapped transient dialogs now go through the same focus decision path,
- this makes the `MapRequest` path more consistent with the initial creation path already used for transients.

### 2. Raise the window before focusing it in `mapNotifyEvent()`

Old code:

```cpp
focus();
```

New code:

```cpp
raiseAndFocus();
```

Effect:

- if Fluxbox has already decided that the mapped window should receive focus,
- it is now also raised explicitly before focus is sent,
- which is important for modal file dialogs that may otherwise exist but remain visually buried.

## Exact diff

```diff
diff --git a/src/Window.cc b/src/Window.cc
index 342e4885..b956c2e8 100644
--- a/src/Window.cc
+++ b/src/Window.cc
@@ -2086,7 +2086,7 @@ void FluxboxWindow::mapRequestEvent(XMapRequestEvent &re) {
     setCurrentClient(*client, false); // focus handled on MapNotify
     deiconify();
 
-    if (isFocusNew()) {
+    if (isFocusNew() || client->isTransient()) {
         m_focused = false; // deiconify sets this
         Focus::Protection fp = m_focus_protection;
         m_focus_protection &= ~Focus::Deny; // goes by "Refuse"
@@ -2139,7 +2139,7 @@ void FluxboxWindow::mapNotifyEvent(XMapEvent &ne) {
     // we use m_focused as a signal that this should be focused when mapped
     if (m_focused) {
         m_focused = false;
-        focus();
+        raiseAndFocus();
     }
 
 }
```

## Expected result

This patch is intended to improve the behavior of transient modal dialogs, especially file-open dialogs from Qt/KDE applications under Fluxbox.

The expected improvement is:

- the dialog should be raised,
- the dialog should receive focus more reliably,
- the parent application should stop appearing frozen when the file dialog opens.

## Limits of this patch

This is a targeted workaround/fix attempt, not a full rewrite of Fluxbox focus handling.

It does not guarantee that every dialog bug is solved, because the final behavior can still depend on:

- application-side modality,
- X11 focus model,
- Fluxbox focus settings,
- session startup environment,
- and any external session scripts.

## Build dependencies

This tree is an upstream/autotools source tree.

Install the usual build requirements first on MX Linux / Debian:

```bash
sudo apt update
sudo apt install build-essential autoconf automake pkg-config \
    libx11-dev libxft-dev libxinerama-dev libxpm-dev libimlib2-dev \
    libfribidi-dev libfontconfig1-dev libxrandr-dev libxrender-dev
```

Depending on the exact branch and your system, you may already have some of them installed.

## Compile from this tree

From this directory:

```bash
cd /home/wachin/Dev/fluxbox-dev/fluxbox
./autogen.sh
make -j"$(nproc)"
```

If compilation succeeds, the patched binary should be in the build tree.

## Quick local install test

If you want to test quickly without building a Debian package first:

```bash
cd /home/wachin/Dev/fluxbox-dev/fluxbox
./configure --prefix=/usr/local
make -j"$(nproc)"
sudo make install
```

Important:

- this usually installs under `/usr/local`,
- it is useful for testing,
- but it is not as clean as installing a proper `.deb`.
- `make uninstall` only removes files from the same `prefix` used when `./configure` generated the current `Makefile`.

After installing, log out and start a Fluxbox session again before testing `Kate`, `Kdenlive`, `Ksnip`, or `Dolphin`.

To uninstall this local test build later, use the same configured tree:

```bash
cd /home/wachin/Dev/fluxbox-dev/fluxbox
sudo make uninstall
```

If `make uninstall` does not remove files from `/usr/local`, reconfigure the tree with the same prefix used for the local install and run uninstall again:

```bash
cd /home/wachin/Dev/fluxbox-dev/fluxbox
./configure --prefix=/usr/local
sudo make uninstall
```

For example, a tree configured with `--prefix=/usr` will try to uninstall from `/usr`, not from `/usr/local`.

## Power menu on MX Linux with SysVinit

On MX Linux, the Fluxbox session may be running on SysVinit instead of systemd. Check it with:

```bash
ps -p 1 -o comm=
```

If the output is `init`, Fluxbox menu entries that call `systemctl poweroff`, `systemctl reboot`, or `systemctl suspend` will not work because systemd is not PID 1.

Use `pkexec` with the classic power commands in `~/.fluxbox/menu`:

```fluxbox
[submenu] (Salir) </usr/share/icons/gnome/16x16/actions/stop.png>
   [exec] (Reiniciar) {pkexec /sbin/reboot} </usr/share/icons/gnome/16x16/actions/reload.png>
   [exec] (Apagar) {pkexec /sbin/poweroff} </usr/share/icons/gnome/16x16/actions/system-shutdown.png>
   [exec] (Suspender) {pkexec /usr/sbin/pm-suspend} </usr/share/icons/gnome/16x16/actions/player_pause.png>
   [exit] (Cerrar Sesión) </usr/share/icons/gnome/16x16/actions/system-log-out.png>
[end]
```

The graphical password prompt requires a Polkit authentication agent. A Fluxbox startup file can launch the GNOME Polkit agent like this:

```sh
/usr/lib/policykit-1-gnome/polkit-gnome-authentication-agent-1 &
```

After editing the menu, reload Fluxbox:

```bash
fluxbox-remote reconfig
```

## About creating a `.deb`

This repository **now does contain** a `debian/` directory.

That `debian/` directory was added from the MX Linux packaging archive at:

- https://mxrepo.com/mx/repo/pool/main/f/fluxbox/fluxbox_1.3.7+git20220731-0mx23+1.debian.tar.xz

So, unlike a plain upstream snapshot, this tree may now be usable for Debian package building, provided that:

- the imported `debian/` files match this exact source tree closely enough,
- the build dependencies are installed,
- and the package rules still work correctly on the current system.

## Recommended way to build a `.deb` from this repository

Suggested workflow:

1. Keep the patch in `src/Window.cc` as it is now, or move it later into `debian/patches/` if you want formal Debian patch management.
2. Run `./autogen.sh` if needed.
3. Build the package directly from this repository with `dpkg-buildpackage`.

Example command:

```bash
dpkg-buildpackage -b -us -uc
```

That should generate one or more `.deb` files in the parent directory if the packaging metadata is compatible with this tree.

If packaging fails, the next things to check are:

- whether the imported `debian/` directory expects a slightly different upstream tree,
- whether any MX/Debian patches need adjustment,
- and whether the current source version still matches the imported package metadata closely enough.

## Install the generated `.deb`

From the parent directory of the packaged source tree:

```bash
sudo dpkg -i ../fluxbox*.deb
sudo apt -f install
```

Use `apt -f install` only if `dpkg` reports missing dependencies.

## Testing checklist

After building and installing the patched Fluxbox:

1. Log out from the current session.
2. Log back into a Fluxbox session.
3. Start `kate`.
4. Open `File -> Open...`.
5. Repeat with `kdenlive`, `ksnip`, and `dolphin`.
6. Confirm whether the dialog is immediately visible and focused.

## If the problem persists

If the delay or fake-freeze still happens after this patch, the next diagnostic step should be to compare:

- patched Fluxbox behavior,
- the app's actual X11 transient windows,
- and focus/stacking events during `MapRequest` and `MapNotify`.

At that point the issue may involve more than one path in Fluxbox focus control.
