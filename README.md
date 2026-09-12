# GCB — Greg's Cosmic Bootstrap

Rice a fresh [COSMIC](https://system76.com/cosmic) desktop the same way
[GLB](https://github.com/ggregoro/GLB) already handles the terminal:
apply it once, reapply it identically the next time the machine (or the
distro) changes.

GCB is the third sibling in the GLB family:

- [GLB](https://github.com/ggregoro/GLB) — terminal/shell bootstrap, any
  of apt/dnf/pacman/zypper.
- [GWB](https://github.com/ggregoro/GWB) — the Windows/PowerShell
  equivalent.
- [GHB](https://github.com/ggregoro/ghb) — Hyprland/Omarchy desktop
  bootstrap (private, currently paused).
- **GCB** — COSMIC desktop ricing, on any distro that ships it.

## What it does

GCB assumes COSMIC is already installed and running — unlike GHB, there
is no "install the desktop environment" phase, since all four target
distros already offer COSMIC as a selectable DE. GCB layers two things
on top of a stock COSMIC install:

1. **Terminal/shell** — delegated entirely to GLB (`glb restore
   default`). GCB writes none of this itself.
2. **Ricing** — GCB's own layer: the `adw-gtk3-dark` GTK3 theme,
   `Yaru-blue-dark` icons, a dock-only panel layout (top panel removed,
   everything in a full-width bottom dock), square window corners with a
   small tiling gap, and a wallpaper.

See [`docs/design/gcb-glb-boundary.md`](docs/design/gcb-glb-boundary.md)
for exactly how GCB and GLB divide the work, and
[`docs/design/ricing-layer.md`](docs/design/ricing-layer.md) for the
ricing layer's own spec.

## Target distros

- **Arch Linux** (pacman)
- **CachyOS** (pacman)
- **Pop!_OS** (apt) — COSMIC's original home
- **Fedora COSMIC Spin** (dnf)

openSUSE ships COSMIC too, but isn't a named target yet.

## Status

**Design phase — no working script yet.** This repo currently holds the
design docs and the wallpaper asset. See `CHANGELOG.md` for progress.

## License

MIT — see [`LICENSE`](LICENSE).
