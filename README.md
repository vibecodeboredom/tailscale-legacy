# Tailscale Legacy

An unofficial Tailscale client for older Android — **Android 5.1+ (API 22)**.

This is a fork of the open-source Tailscale Android client
([tailscale/tailscale-android](https://github.com/tailscale/tailscale-android),
BSD licensed). It is **not affiliated with, endorsed by, or supported by
Tailscale Inc.** If you have a device on Android 8.0 or newer, use the
official Tailscale app instead — this fork exists only for hardware the
official app no longer supports.

## Why

Current Tailscale for Android requires Android 8.0+ (API 26). Plenty of
older phones, tablets, and TV sticks still work fine but are locked out.
Tailscale Legacy brings the client back to Android 5.1+.

## Base version

Forked from **Tailscale Android 1.60.1** (tag
`1.60.1-t2caffeeb4-g6f7ae79a31c`, the Tailscale 1.60.1 release). That is the
newest clean tag whose `android/` module still declares `minSdkVersion 22`
with a single-module layout.

Why not 1.62.0 (the last release shipped with minSdk 22)? 1.62.0 has no
clean git tag, and by then the repo was mid-migration to a new Compose UI
(`android/` at minSdk 26 alongside the old Gio `android_legacy/` module) —
the build target is ambiguous and the Gio UI from that era is reported to
crash on some API 22 devices without OpenGL ES 3.0. 1.60.1 is the newest
unambiguous, buildable API 22 base.

## What changed vs upstream (v1.0.0)

- **App name:** "Tailscale Legacy" (was "Tailscale"), with a distinct indigo
  launcher icon so it is easy to tell apart from the official app.
- **Application ID:** `com.tailscale.legacy` (was `com.tailscale.ipn`), so
  this fork installs **alongside** the official Tailscale app and never
  overwrites it. The Java package (`namespace`) deliberately stays
  `com.tailscale.ipn` — the Go IPN code loads
  `com.tailscale.ipn.Google` by name via JNI, so the namespace must not
  change.
- **Version:** 1.0.0 (versionCode 1).
- **minSdk:** 22 (Android 5.1), unchanged from the base. targetSdk 33,
  compileSdk 33, NDK 23.1.7779620, AGP 8.1.0, Go 1.22 — all as in the base.
- **No new features, no VPN-logic changes.** The WireGuard data plane and
  the Tailscale control-plane code are untouched.

## Which APK to install

CI builds the **F-Droid flavor** (`tailscale-fdroid.apk`) — it has no Google
Play Services dependency, so it installs and runs on devices without Google
Play (old phones, Fire TV sticks, cheap TV boxes). The Play flavor
(`make tailscale-debug.apk`) also builds but needs Play Services for Google
sign-in.

## Building

You need a Go runtime, a JDK 17, and the Android SDK
(`make androidsdk` installs the components). Then:

```sh
make tailscale-fdroid.apk   # F-Droid flavor, no Google bits (recommended)
make apk                    # Play flavor debug APK
make install                # install the debug APK on a connected device
```

CI (GitHub Actions, `.github/workflows/build.yml`) builds the F-Droid debug
APK on every push to `main` and uploads it as an artifact.

## Honest limits (v1)

- **Live tailnet connectivity is NOT verified.** v1 proves the fork builds
  and installs. Connecting to a real tailnet on a real old device is
  untested — sign in and try it, and report what happens.
- **Old clients may be limited server-side.** Tailscale's control plane may
  refuse or degrade very old clients over time; if the current servers
  reject 1.60.1-era clients, this fork cannot fix that without a newer
  core (which would raise the minSdk again).
- **The UI is Gio, and Gio wants OpenGL ES 3.0.** Most Android 5.1 devices
  have GLES 3.0, but a few (notably the 2nd-gen Fire TV Stick, which only
  has GLES 2.0) may show a black screen or crash on launch. On the
  emulator, set OpenGL ES to "Renderer maximum (up to OpenGL ES 3.1)" and
  cold-boot. This is a limitation of the base version, not something v1
  changes.
- **Google Sign-In** uses the app identifier; this fork's identifier is
  `com.tailscale.legacy`, so the official Google API Console project does
  not apply. Other sign-in methods (Tailscale login, auth keys) are the
  way to connect.
- **Android 4.x is NOT supported.** minSdk is 22 (Android 5.1). Devices on
  Android 4.1–4.4 (e.g. the 1st-gen Fire TV Stick) cannot run this fork.

## License and attribution

BSD licensed, same as upstream. See [LICENSE](LICENSE). The upstream
copyright belongs to Tailscale Inc and the Tailscale authors; this fork's
changes are minimal (rebrand + applicationId + version) and carry no
additional license terms. WireGuard is a registered trademark of
Jason A. Donenfeld. "Tailscale" is a trademark of Tailscale Inc — this
unofficial fork uses the name only to describe what it is a fork of.

## Bugs

File issues here for fork-specific problems (build, branding, old-Android
issues). For the Tailscale service itself or the core client, use
[the tailscale issue tracker](https://github.com/tailscale/tailscale/issues).
