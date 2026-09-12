# GCB ⇄ GLB Boundary

| | |
|---|---|
| **Status** | Draft |
| **Date** | 2026-09-12 |
| **Authors** | Greg, with Claude |
| **Applies to** | GCB (this repo) and [GLB](https://github.com/ggregoro/GLB) |
| **Model** | Two-layer delegation — GLB owns terminal/shell, GCB owns COSMIC ricing |

---

## 1. Why GCB is not another Omarchy-style overlay

GHB's shape (minimal Arch → install a whole desktop shell → thin overlay)
doesn't apply here. **COSMIC is already installable/selectable on every
GCB target** (Arch, CachyOS, Pop!_OS, Fedora COSMIC Spin) — there's no
"Phase 1: install the DE" step. GCB assumes COSMIC is already running,
same as `Cosmic-Desktop-Customization` already assumed.

That makes GCB's job narrower than GHB's: apply a terminal/shell setup,
then rice the COSMIC desktop itself. Both halves already have a proven
tool — GLB for the first, and GLB's own manifest engine (see §3) turns
out to fit the second too.

## 2. The model

```
COSMIC already running (any of the 4 target distros)
        │
        ├─▶ glb restore default          (terminal/shell layer — delegated, no GCB code)
        │
        └─▶ glb restore --from-manifest <gcb>/profile   (ricing layer — GCB's own content,
                                                           applied through GLB's engine)
        │
        └─▶ gcb-specific post-steps GLB's manifest can't express:
             - wallpaper asset placement + RON path rewrite
             - gsettings theme/icon selection
             - AUR helper / COPR enablement where a package needs one
```

`gcb` (this repo's own dispatcher) runs all three steps in order. GCB
never forks GLB, never adds a profile into GLB's own `profiles/`
directory, and never reimplements package-manager detection, dotfile
symlinking, or dry-run/undo — all of that is GLB's job, reused as-is.

## 3. The delivery mechanism: `glb restore --from-manifest`

GLB already supports applying a "profile-shaped directory living
anywhere on disk" (`lib/profile.sh`'s `glb_apply_manifest`, wired to
`glb restore --from-manifest <path>`) — packages.txt + extras.txt +
dotfiles/, same engine as a real profile: per-distro package overrides,
`github-release`/`github-release-tree` extras, symlink+backup dotfiles,
dry-run, and undo via `*.glb-backup` files.

This is exactly the "lean-profile delivery" question GHB's own boundary
doc (`ghb-glb-boundary.md`, §5, in the ghb repo) left open before GLB
dropped out of that project entirely — GLB has since grown the answer.
GCB is the first project to actually use it: GCB ships `profile/` in its
own repo (packages.txt/extras.txt/dotfiles/, same shape as a GLB
profile) and calls `glb restore --from-manifest <path-to-that-dir>`
instead of needing a profile committed to the GLB repo itself.

**Consequence — GCB's dotfiles are live, not one-time copies.** GLB's
dotfile mechanism symlinks (`ln -s`), it doesn't copy. So
`~/.config/cosmic/com.system76.CosmicPanel.Dock/v1/*` ends up symlinked
back into GCB's own checkout. COSMIC's settings daemon writes through
that symlink when the user changes something live (resizing the dock,
picking a new wallpaper via the GUI), which means **GCB's checkout is a
living, git-tracked mirror of the real config** — the same model
`Cosmic-Desktop-Customization` already uses, just via GLB's proven
symlink+backup mechanism instead of a one-time `cp -r`. This implies GCB
needs a permanent install location (mirroring GLB's own
`~/.local/share/glb`), not a run-once-and-discard checkout.

## 4. What GLB owns vs. what GCB owns

### 4.1 Owned by GLB (`glb restore default`) — GCB does not touch
| Area | Owner |
|---|---|
| Shell (bash/zsh/fish), Starship prompt, completions | GLB |
| Neovim + LazyVim | GLB |
| yazi, atuin, git-delta, eza/bat/fd/fzf/ripgrep/zoxide | GLB |
| Ghostty | GLB |
| Package-manager detection, dry-run, undo, repair/diff/export | GLB |

### 4.2 Owned by GCB (`glb restore --from-manifest <gcb>/profile` + post-steps)
| Area | Mechanism |
|---|---|
| `adw-gtk3-dark` GTK3 widget theme | `profile/packages.txt` (Arch/CachyOS, Fedora — pending real verification) or `profile/extras.txt` `github-release`/`github-release-tree` (Pop!_OS, no apt package exists) |
| `Yaru-blue-dark` icon theme | `profile/packages.txt` per-distro (AUR on Arch/CachyOS, COPR-enable step on Fedora — see `ricing-layer.md`) |
| COSMIC RON config (dock/panel layout, appearance style, `CosmicTk` icon theme) | `profile/dotfiles/.config/cosmic/**` |
| Wallpaper asset (`fmuAYkF.jpg`, "Solarized Osaka") | Shipped in the repo; **not** a plain dotfile symlink — the RON `source:` field is an absolute path, so GCB writes it itself after placing the file (see `ricing-layer.md` §wallpaper) |
| `gsettings` GTK theme/icon-theme selection | Not a file GLB can symlink — a `gcb`-run command, same as GHB's Chrome-default step wasn't GLB's job either |
| AUR helper / COPR enablement | `gcb`-run pre-step before the manifest's packages.txt can resolve |

Nothing in 4.2 collides with a file GLB's `default` profile writes.

## 5. Non-goals

- Installing or configuring COSMIC itself. GCB assumes it's already
  there, on any of the four named distros.
- Forking or vendoring GLB. GCB depends on GLB being installed
  (`~/.local/share/glb` on `PATH`) and calls it, same as any other GLB
  consumer would.
- openSUSE/zypper support, until asked — COSMIC runs there too, but it
  isn't one of the four named targets.

## 6. Consequences

- **No fork-maintenance burden**, same reasoning GHB's boundary doc gave
  for Omarchy: GLB updates itself independently, GCB's manifest is
  re-runnable on top at any time.
- **Reversibility**: `glb restore --undo` reverts GCB's dotfile changes
  the same way it reverts any GLB profile's, since undo operates on
  `*.glb-backup` files globally, not per-profile.
- **Risk surface**: real per-distro package availability for the theme/
  icon packages is unverified in three of four cases (see
  `project_gcb.md` in the memory repo) — that verification happens
  during the VM-testing phase, not assumed here.
