# CLAUDE.md — GCB (Greg's Cosmic Bootstrap)

Guidance for Claude Code (and contributors) working in this repo.

## What this project is

GCB rices a [COSMIC](https://system76.com/cosmic) desktop that's already
installed and running, on any distro that offers it. It is the third
sibling in the GLB family — see
[`docs/design/gcb-glb-boundary.md`](docs/design/gcb-glb-boundary.md) for
exactly how it relates to [GLB](https://github.com/ggregoro/GLB) (the
terminal/shell layer, delegated entirely) and
[GHB](https://github.com/ggregoro/ghb) (the Hyprland/Omarchy sibling,
whose "install the whole DE first" shape does **not** apply here since
COSMIC is already installable/selectable on every GCB target).

- Repo: <https://github.com/ggregoro/GCB> (public)
- License: MIT
- Language: Bash, matching GLB (no runtime dependency on Python/Node/etc.)

## Target distros

Arch, CachyOS (pacman), Pop!_OS (apt), Fedora COSMIC Spin (dnf).
openSUSE/zypper is not a target unless asked.

## Design docs

- [`docs/design/gcb-glb-boundary.md`](docs/design/gcb-glb-boundary.md) —
  the two-layer delegation model and why `glb restore --from-manifest`
  is the delivery mechanism.
- [`docs/design/ricing-layer.md`](docs/design/ricing-layer.md) — the
  actual ricing spec: packages, extras, dotfiles, the wallpaper's
  absolute-path problem, gsettings, AUR/COPR handling.

## Working principle: verify for real

Every GLB-family project (GLB itself, GHB) treats a plan as unverified
until it's been run for real on the actual target — a VM at minimum, real
hardware where it matters. Package-name research from search/documentation
is a starting point, not a conclusion; several entries in
`ricing-layer.md`'s package table are marked unverified for exactly this
reason and must be corrected against a real install before being trusted.

## Status

Design phase. No working `gcb` script yet — see `CHANGELOG.md`.
