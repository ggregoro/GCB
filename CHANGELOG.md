# GCB Project Changelog

All notable changes to the GCB project will be documented in this file.

---

## [Unreleased]

### Design
- Project started: COSMIC desktop ricing, built on GLB, targeting Arch,
  CachyOS, Pop!_OS, and the Fedora COSMIC Spin.
- Wallpaper asset ("Solarized Osaka", 3440x2160) added.
- Architecture decided: two-layer delegation — GLB owns the
  terminal/shell layer (`glb restore default`), GCB owns a ricing layer
  applied through GLB's own `glb restore --from-manifest` engine. See
  `docs/design/gcb-glb-boundary.md`.
- Ricing layer spec drafted (`docs/design/ricing-layer.md`): `adw-gtk3-dark`
  GTK3 theme, `Yaru-blue-dark` icons, dock-only panel layout carried over
  from `Cosmic-Desktop-Customization` unchanged, wallpaper delivery
  handled outside GLB's plain dotfile symlinking (the RON config's
  `source:` field is an absolute path GLB can't template).
- Per-distro theme/icon package availability researched (not yet
  verified on real hardware): confirmed official on Arch/CachyOS;
  Pop!_OS has no apt package for `adw-gtk3-dark` (needs GLB's
  `github-release` extras method instead); Fedora's icon-theme story is
  COPR-only with unconfirmed variant coverage — the weakest link of the
  four targets.
