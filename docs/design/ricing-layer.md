# GCB ricing layer — spec

| | |
|---|---|
| **Status** | Draft, pre-implementation — package availability below is
researched, not yet verified on real hardware/VMs (see §6) |
| **Related** | [`gcb-glb-boundary.md`](gcb-glb-boundary.md) |

`gcb` runs once, on a machine that already has COSMIC running (Arch,
CachyOS, Pop!_OS, or the Fedora COSMIC Spin) and GLB already installed.
It is idempotent and re-runnable, same expectation as GLB itself.

## What changed from `Cosmic-Desktop-Customization`

That repo rices Yaru theme + Yaru icons + a Milky Way wallpaper, applied
by hand (`cp -r`, then a manually-edited absolute wallpaper path). GCB
modernizes both the **content** (the newer `adw-gtk3-dark` +
`Yaru-blue-dark` + "Solarized Osaka" combo from Greg's screenshots) and
the **delivery** (GLB's symlink+backup manifest engine instead of a
one-time copy — see `gcb-glb-boundary.md` §3). The dock-only panel
layout (top panel removed, full-width bottom dock) carries over
unchanged — Greg's explicit call, 2026-09-12.

## Step 1 — preconditions (checked, fail loudly)

- Running under COSMIC: `echo $XDG_CURRENT_DESKTOP` mentions `COSMIC`,
  or `cosmic-comp` process present.
- `glb` on `PATH` (GLB already installed) — if missing, point at
  <https://github.com/ggregoro/GLB>'s own `install.sh`, don't install it
  for the user.
- Per-distro package-source prerequisites (see Step 2) — an AUR helper
  on Arch/CachyOS, `dnf copr` available on Fedora.

## Step 2 — theme/icon packages

**Not yet real-verified per distro — this table is the researched
starting point (2026-09-12), to be corrected against actual installs
during VM testing:**

| Distro | `adw-gtk3-dark` | `Yaru-blue-dark` icons |
|---|---|---|
| Arch / CachyOS (pacman) | ✅ official `extra/adw-gtk-theme` | ⚠️ AUR `yaru-icon-theme` — needs confirming this (not the separate third-party `yaru-colors-icon-theme`) actually contains the `Yaru-blue-dark` folder |
| Pop!_OS (apt) | ❌ no apt package anywhere — deliver via GLB's `extras.txt` `github-release`/`github-release-tree` method against `lassekongo83/adw-gtk3`'s release tarball, same pattern GLB already uses for yazi/atuin/fastfetch | ⚠️ official `yaru-theme-icon` exists but `Yaru-blue-dark` folder inclusion is unconfirmed — a live Ubuntu bug report exists about missing Yaru variants on recent releases |
| Fedora COSMIC Spin (dnf) | ⚠️ `adw-gtk3-theme` search-indicated as official; not independently confirmed (Fedora's own package pages are unreachable from the dev sandbox) | ❌ no official package at all — COPR only (`david35mm/yaru-theme` or `mivanovic/yaru-theme`), variant coverage unverified. **Weakest link of the four targets.** |

`profile/packages.txt` carries the confirmed entries; anything ❌/⚠️
goes in `profile/extras.txt` or gets a documented manual fallback,
mirroring how GLB itself handles distros without a clean native package
(see GLB's own `_GLB_PACKAGE_MANUAL_HINT` pattern).

## Step 3 — COSMIC RON config

`profile/dotfiles/.config/cosmic/**` — symlinked into place by GLB's
manifest engine (see boundary doc §3). Carries over from
`Cosmic-Desktop-Customization`, updated for the new theme:

- `com.system76.CosmicTk/v1/icon_theme` → `Yaru-blue-dark`
- `com.system76.CosmicPanel/v1/entries` — top panel removed (kept as-is
  from the old repo, per Greg's "keep the dock-only layout for now")
- `com.system76.CosmicPanel.Dock/v1/*` — full-width, persistent,
  `XS`-sized bottom dock (kept as-is)
- `com.system76.CosmicTheme.{Dark,Light}{,.Builder}/v2/{corner_radii,gaps}`
  — square corners, small tiling gap (kept as-is)
- **Not carried over as a plain dotfile**: `com.system76.CosmicBackground/v1/all`
  — see Step 4, the absolute-path problem.

## Step 4 — wallpaper

The RON file's `source:` field is a **hardcoded absolute path**, and
GLB's dotfile symlinking has no templating/variable-substitution (it's a
plain `ln -s`, confirmed by reading `lib/profile.sh`). Shipping the RON
file as a normal dotfile would bake in a path that only exists on
whichever machine authored it — exactly the bug `gcb` is meant to fix
relative to the old repo's manual "hand-edit the source: path" step.

So this file is **not** in `profile/dotfiles/`. Instead, `gcb` itself:

1. Installs the wallpaper asset (`fmuAYkF.jpg`, "Solarized Osaka",
   3440x2160) to a fixed path under GCB's own install location (e.g.
   `~/.local/share/gcb/wallpaper.jpg`) — a location `gcb` controls and
   can always find, independent of $HOME layout quirks.
2. Writes `~/.config/cosmic/com.system76.CosmicBackground/v1/all` with
   `source:` pointed at that fixed path (backing up any existing file
   first, same `*.gcb-backup` convention GLB uses for its own dotfiles,
   for consistency).

COSMIC's settings daemon picks up the change live, no reload needed
(confirmed behavior, per both the old repo's README and GLB's own
fresh-VM notes about COSMIC's live config watching).

## Step 5 — GTK theme/icon selection

Not a file — `dconf`/`gsettings` state, same as the old repo already
documented:

```sh
gsettings set org.gnome.desktop.interface gtk-theme 'adw-gtk3-dark'
gsettings set org.gnome.desktop.interface icon-theme 'Yaru-blue-dark'
```

Scripted by `gcb`, not a manual README step (an improvement over the old
repo, which left this to the user by hand).

## Step 6 — AUR helper / COPR handling

- Arch/CachyOS: requires an AUR helper (`yay` or similar) already
  present — same expectation the old repo already documented. `gcb`
  checks for one and warns loudly with exact next-step instructions if
  missing (skipping just the AUR-dependent icon package, not aborting
  the whole run), rather than installing one on the user's behalf — an
  AUR-helper choice is personal.
- Fedora: if `Yaru-blue-dark` icons end up COPR-only after real
  verification, `gcb` runs `dnf copr enable <repo>` itself (a one-line,
  reversible, well-understood step) rather than punting to a manual
  step — but flags it clearly since enabling a COPR adds a new package
  source the user should know about.

## Verification (post-run, once built)

Mirrors GLB's own fresh-VM verification discipline
(`docs/fresh-vm-verification.md` in the GLB repo) — run for real on each
of the four target distros, not just parsed/reviewed:

- `adw-gtk3-dark` actually applies to a legacy GTK3 app (COSMIC's own
  native apps won't visibly change — confirmed they don't use GTK
  theming at all).
- `Yaru-blue-dark` icons show on legacy/Flatpak GTK apps and in
  `com.system76.CosmicTk`'s native icon set.
- Wallpaper renders correctly, survives a reboot, and a second `gcb` run
  is a no-op (no unexpected `*.gcb-backup.*` files).
- Dock/panel layout matches the old repo's (unchanged, per Greg's call).
- A second run of the whole `gcb` script is idempotent end to end.
