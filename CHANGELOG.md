# Changelog

## 1.0.5 — Diagnostic-only release

- 1.0.4 made zsync's install unconditional, but the release still isn't
  producing a .zsync file. Added explicit diagnostic output right before
  the appimagetool invocation — dpkg status for the zsync package,
  `which zsyncmake`, and `zsyncmake --version` — to pin down definitively
  whether zsync is installed, whether zsyncmake is on PATH, and whether
  it actually runs on the CI runner. No functional change otherwise.

## 1.0.4 — Fix zsync still not installing in CI

- 1.0.3 added zsync to the build-dependencies apt-get install line, but
  that whole block is gated behind a check for whether cargo is already
  on PATH. In CI, a prior "Set up Rust Toolchain" step already installs
  cargo, so the guard evaluated false and the entire block — zsync
  included — was silently skipped, same as before 1.0.3. zsync is now
  installed unconditionally, ahead of the guarded block.

## 1.0.3 — Fix missing .zsync file

- The build runner never had zsync installed, so appimagetool silently
  skipped generating the .zsync file even though UPDATE_INFORMATION was
  already set in 1.0.2 — update-aware tools had nothing to delta-update
  against. zsync is now installed alongside the other build dependencies.

## 1.0.0 — Initial release

- Browse stable Ubuntu mainline kernel versions from kernel.ubuntu.com,
  with badges comparing each version against the running kernel and a
  filter field. Both the flat and amd64/ subdirectory page layouts are
  supported.
- Download the four generic amd64 packages (image-unsigned, modules,
  headers-generic, headers-all) with per-file progress, retries,
  cancellation, and SHA256 verification against the published CHECKSUMS
  file. A file that fails verification is deleted and the download
  aborts.
- Privileged install via polkit: dpkg install, then initramfs generation
  keyed on the kernel version read from .deb package metadata (never
  parsed from filenames), cross-checked against /lib/modules, then GRUB
  update. The install fails loudly if initrd.img does not appear in
  /boot, so a silently unbootable kernel cannot be produced.
- System tab lists every kernel in /boot with initrd and modules health
  checks; a window-wide warning banner appears whenever any installed
  kernel is missing its initramfs. Disk space on /boot and / is shown
  with low-space warnings.
- Old kernels can be removed from the System tab (packages purged, GRUB
  updated); the running kernel is never removable.
- AppImage-only distribution. First launch installs the privileged
  script and polkit policy to system paths via pkexec.

## 1.0.1 — Fix AppImage version metadata

- build-appimage.sh computed VERSION from Cargo.toml but never exported
  it to appimagetool, which fell back to a git commit hash. Package
  managers like Gear Lever showed "Mainline Kernel Installer (05706c)"
  instead of a real version. VERSION is now passed into appimagetool's
  environment alongside ARCH.

## 1.0.2 — Enable update checking

- Embedded UPDATE_INFORMATION in the AppImage so update-aware tools
  (Gear Lever, AppImageUpdate) can check GitHub Releases for newer
  versions and delta-update via zsync. CI now also uploads the .zsync
  file alongside the AppImage.
