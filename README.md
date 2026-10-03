# ps4-flatpak-mesa

Builds a Flatpak GL runtime **Mesa patched for the PS4 (Liverpool) GPU**, so Flatpak apps
(specifically **Sober / Roblox**) render correctly on a jailbroken PS4.

## Why

Upstream Mesa does not know the PS4's Liverpool/Gladius GPU. Without the PS4 patches it
mis-identifies the chip as KAVERI, applies the wrong raster config, and hits a broken
RADV fast-clear path — producing the well-known "3D geometry/textures corrupted, UI fine"
behaviour (see vinegarhq/sober#2490).

The host system uses a patched Mesa (`mesa-ps4`), which is why the desktop is fine, but the
Flatpak sandbox always uses its own bundled Mesa — hence the corruption inside Sober.

## What it does

* builds Mesa `26.2.2` inside `org.freedesktop.Sdk//25.08` (so the produced `.so` files are
  ABI-compatible with the Flatpak runtime's LLVM 21.1 / glibc),
* applies `ps4-mesa.patch` (from DionKill/ps4-video-archlinux) which:
  * adds `CHIPSET(0x9923, LIVERPOOL)` / `GLADIUS` recognition,
  * supplies the correct `raster_config` for LIVERPOOL/GLADIUS,
  * fixes `FAST_CLEAR_ELIMINATE` predication in RADV,
* uploads the install tree as a build artifact.

## Deploy to the PS4

1. Download the `mesa-ps4-flatpak` artifact and copy `mesa-ps4-flatpak.tar.gz` to the PS4.
2. Back up the current GL runtime, then overlay the new Mesa into it:

```bash
RT=$(ls -d /var/lib/flatpak/runtime/org.freedesktop.Platform.GL.default/x86_64/25.08/*/files | head -1)
cp -a "$RT" /root/gl-runtime-backup
tar -xzf mesa-ps4-flatpak.tar.gz -C "$RT"
```

3. Restart Sober.

## Deploy as a proper extension (optional)

```bash
flatpak --user install ./repo org.freedesktop.Platform.GL.ps4//25.08
flatpak override --user --env=FLATPAK_GL_DRIVERS=ps4 org.vinegarhq.Sober
```
