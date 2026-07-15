# ROADMAP

Date: 2026-07-14

This file summarizes what has already been done in this repository and what remains pending so work can continue directly from here.

## Current Status

[x] Reviewed the full upstream Fluxbox source tree.
[x] Identified a possible root cause for the `File -> Open...` dialog issue in Qt/KDE applications under Fluxbox on X11.
[x] Reviewed `transient` and `modal` window handling in `src/Window.cc`, `src/FocusControl.cc`, and `src/WinClient.cc`.
[x] Confirmed that the observed issue matches a modal dialog that is created, but is not always raised or focused correctly.
[x] Applied a minimal patch in `src/Window.cc`.
[x] Created `README.md` in this repository documenting the patch and how to test it.
[x] Added a `debian/` directory taken from the MX Linux package to support later packaging work.
[x] Confirmed that `autogen.sh` works in this tree and generates `configure` correctly.
[x] Verified with `./configure` that the tree generates `Makefile` and has an `uninstall` target.
[x] Built the patched project with `make -j4`.
[x] Tested the patched binary in a real Fluxbox session.
[x] Built a `.deb` package from this repository with the added `debian/` directory.

## Applied Patch

Modified file:

- `src/Window.cc`

Changes made:

[x] In `mapRequestEvent()`, also treat `transient` windows as candidates for the same focus path used for new windows.
[x] In `mapNotifyEvent()`, use `raiseAndFocus()` instead of `focus()` when Fluxbox has already decided that the window should receive focus.

Patch goal:

- improve behavior for modal/transient dialogs,
- especially in `Kate`, `Kdenlive`, `Ksnip`, `Dolphin`, and similar Qt/KDE applications,
- preventing the dialog from existing but remaining hidden, covered, or apparently unfocused.

## Recommended Immediate Work

[x] Run `./configure` in this repository.
[x] Confirm that `Makefile` and `install`/`uninstall` rules are generated.
[x] Run `make -j"$(nproc)"`.
[x] Build the `.deb` package with `dpkg-buildpackage -b -us -uc`.
[x] Install the generated `.deb` and test it in a real Fluxbox session.

## Pending Functional Tests

[x] Log into Fluxbox with the patched binary.
[ ] Open `kate` and test `File -> Open...`.
[ ] Repeat with `kdenlive`.
[ ] Repeat with `ksnip`.
[ ] Repeat with `dolphin`.
[ ] Confirm whether the dialog appears immediately, visible and focused.
[ ] Check whether any apparent freeze lasting more than several seconds still occurs.
[ ] Compare behavior with and without `[transient]` rules in `~/.fluxbox/apps`.

## Debian / MX Packaging

[x] The repository already has a `debian/` directory.
[x] That `debian/` directory was added manually from the MX Linux packaging archive:
[x] https://mxrepo.com/mx/repo/pool/main/f/fluxbox/fluxbox_1.3.7+git20220731-0mx23+1.debian.tar.xz
[x] Check that the added `debian/` directory is basically coherent with this exact source tree:
[x] `dpkg-buildpackage` reaches `dpkg-source --before-build` and correctly applies `debian/patches/series`.
[x] The tree returns to a clean state with `dpkg-source --after-build .`.
[ ] Review `debian/patches/series` to decide whether the new patch should be added there as a formal Debian patch.
[ ] Decide whether to keep the change directly in `src/Window.cc` or move it to `debian/patches/`.
[x] Run `dpkg-buildpackage -b -us -uc` with all build dependencies installed.
[x] Resolve missing build dependencies detected on this machine.
[x] Verify with `dpkg-checkbuilddeps` that no build dependencies are missing.
[x] Generate `../fluxbox_1.3.7+git20220731-0mx23+1_amd64.deb`.
[x] Generate `../fluxbox-dbgsym_1.3.7+git20220731-0mx23+1_amd64.deb`.
[x] Install the generated `.deb` and test it in a real session.

## Verification Result On This Machine

[x] `./configure` completed successfully.
[x] The `uninstall` target exists (`make -n uninstall`).
[x] `make -j4` completed successfully and generated the `./fluxbox` binary.
[x] `dpkg-buildpackage -b -us -uc` starts correctly with the imported `debian/` directory.
[x] `dpkg-buildpackage -b -us -uc` completed successfully and generated the binary package.
[x] The generated `.deb` package installed correctly and the `Fluxbox` session appeared in the login manager.
[x] Successfully entered a real Fluxbox session from the MX Linux 23 login manager.

Notes observed during `./configure`:

- The tree initially managed to build without `imlib2` or `xpm` because `configure` did not find those optional libraries.
- For the Debian/MX package, `debian/control` requires the complete build dependency set.
- On this machine they are now installed and verified; `dpkg-checkbuilddeps` reports no missing dependencies.

Build-dependency install command used/recommended according to `debian/control`:

```bash
sudo apt install autoconf automake autotools-dev bzip2 debhelper \
    dh-autoreconf libfribidi-dev libgtk2.0-dev libimlib2-dev libtool \
    libx11-dev libxext-dev libxft-dev libxinerama-dev libxpm-dev \
    libxrandr-dev libxt-dev
```

## Future Technical Improvements For Fluxbox In 2026

These are not done yet. They are work tracks for a broader review of the project.

[ ] Review modern compatibility with Qt5/Qt6 and current KDE applications.
[ ] Review focus, raising, and stacking of transient/modal windows in paths beyond `Window.cc`.
[ ] Add automated tests specific to `WM_TRANSIENT_FOR`, modals, and dialogs.
[ ] Review behavior with portals (`xdg-desktop-portal`) and modern dialogs.
[ ] Review integration with `systemd --user`, `dbus-update-activation-environment`, and current X11 sessions.
[ ] Review environment variables that can affect Qt/KDE inside minimal sessions.
[ ] Audit old `autoconf`/`automake` dependencies and macros that already produce warnings.
[ ] Review `AC_LANG_CPLUSPLUS`, `AC_HEADER_STDC`, `AC_CHECK_LIBM`, `AC_TYPE_SIGNAL`, `AC_CONFIG_HEADER`, and other obsolete macros.
[ ] Evaluate build system modernization.
[ ] Review HiDPI support, themes, and modern icons.
[ ] Review behavior with multiple monitors and current XRandR setups.
[ ] Review interaction with docks, systray, notifications, and recent applications.
[ ] Review historical issues in `ChangeLog` related to transients to detect unresolved patterns.
[ ] Check whether similar bugs have been reported recently downstream in Debian, antiX, or MX.

## Recommended Priority For Modernizing Fluxbox

Fluxbox is an old project, so it is not advisable to try to modernize everything at once. The safest path is to work in layers, starting with the areas that have the most impact on modern applications.

### 1. Build System

First reasonable area to modernize:

- `configure.ac`
- `Makefile.am`
- macros in `m4/`
- `autoconf`/`automake` warnings

Suggested tasks:

[ ] Replace obsolete Autoconf/Automake macros.
[ ] Review warnings emitted by `autogen.sh` and `configure`.
[ ] Update obsolete uses such as `AC_LANG_CPLUSPLUS`, `AC_HEADER_STDC`, `AC_TYPE_SIGNAL`, `AC_CONFIG_HEADER`, and similar macros.
[ ] Preserve compatibility with Debian/MX packaging while modernizing the build.

This layer should not change Fluxbox behavior; it is meant to make the project more maintainable.

### 2. Focus, Transient Windows, And Modern Compatibility

Most important area for the dialog issue:

- `src/Window.cc`
- `src/WinClient.cc`
- `src/FocusControl.cc`
- `src/Ewmh.cc`

Suggested tasks:

[ ] Review `WM_TRANSIENT_FOR` handling.
[ ] Review modal and transient windows from Qt5/Qt6/KDE.
[ ] Review when Fluxbox should raise a window before focusing it.
[ ] Review paths related to `MapRequest`, `MapNotify`, and focus requests from clients.
[ ] Improve compatibility with EWMH/ICCCM hints used by current applications.

This is the priority area if the `File -> Open...` problem persists.

### 3. Session And Current Desktop Integration

Relevant files:

- `util/startfluxbox.in`
- `data/fluxbox.desktop.in`
- `debian/fluxbox.desktop`
- login/session startup scripts

Suggested tasks:

[ ] Review DBus integration in minimal sessions.
[ ] Review integration with `systemd --user` or `elogind`, depending on the init system in use.
[ ] Review use of `dbus-update-activation-environment`.
[ ] Review useful environment variables for GTK/Qt/KDE: `DISPLAY`, `XAUTHORITY`, `DBUS_SESSION_BUS_ADDRESS`, `XDG_CURRENT_DESKTOP`, `XDG_SESSION_DESKTOP`.
[ ] Review whether the login manager `.desktop` file should distinguish this patched build or keep the name `Fluxbox`.

This layer helps modern applications work better inside a minimal Fluxbox session.

### 4. Debian/MX Packaging

Relevant files:

- `debian/control`
- `debian/rules`
- `debian/patches/`
- `debian/fluxbox.desktop`

Suggested tasks:

[ ] Decide whether the `src/Window.cc` change should be moved to `debian/patches/`.
[ ] Clean up and keep build dependencies up to date.
[ ] Verify that artifacts generated by `dpkg-buildpackage` are not versioned.
[ ] Clearly document how to build, install, uninstall, and revert the package.

This layer is important because the repository now includes Debian/MX packaging.

### 5. Gradual C++ Code Modernization

This part must be done more carefully than the previous ones.

Possible tasks:

[ ] Replace manual pointers with `std::unique_ptr` only where ownership is clear.
[ ] Use `nullptr` where safe.
[ ] Add `override` in derived classes.
[ ] Reduce dangerous casts.
[ ] Enable compiler warnings progressively.
[ ] Avoid large refactors without functional tests.

It is not advisable to start with a general C++ modernization. It is better to prioritize focus, EWMH, session, and build system work first, because those are the areas with real impact on modern applications.

## Useful Notes For Resuming Later

[x] The original upstream tree did not include `debian/`; that directory was added later.
[x] `README.md` already documents the applied patch and the technical logic behind the change.
[x] The original user issue reproduces as a fake freeze: the application waits for the modal dialog, but the window may not be visible or focused.
[ ] Confirm with real `File -> Open...` tests whether this patch is enough or whether more focus paths are involved.

## Suggested Next Step When Opening Codex Directly Here

[ ] Open Codex in `/home/wachin/Dev/fluxbox-dev/fluxbox`.
[x] Run `./configure`.
[x] Build.
[x] Test entry into a real Fluxbox session with the generated package.
[x] Install build dependencies and package into a `.deb`.
[ ] Test `File -> Open...` in Qt/KDE applications; if it does not work, continue tracing `MapRequest`, `MapNotify`, `focusRequestFromClient()`, and `FocusControl`.

## Next Safe Step

Next safe step:

  1. Test `Kate`, `Kdenlive`, `Ksnip`, and `Dolphin` with `File -> Open...` inside the Fluxbox session installed from the `.deb`.
  2. Confirm whether dialogs appear visible and focused immediately.
  3. If reverting is needed, reinstall the Fluxbox package from the MX repositories or rebuild/reinstall from this tree.
