# halos-halpi-desktop-branding - Agentic Coding Guide

**LAST MODIFIED**: 2026-05-27

**Document Purpose**: Guide for AI assistants working on halos-halpi-desktop-branding.

## For Agentic Coding: Use the HaLOS Workspace

When using Claude Code or other AI assistants, work from the halos workspace repository for full context across all HaLOS repositories.

## About This Project

HALPI2-branded default desktop wallpaper for HaLOS HALPI2 desktop variants. Ships:

- **`/usr/share/halos/wallpapers/halpi.jpg`** — the HALPI2 wallpaper image (a colored treatment of Olaus Magnus's 1539 Carta Marina).
- **`/etc/xdg/pcmanfm/default/desktop-items-{0,1}.conf`** — pcmanfm system-wide desktop configuration, pointing at the wallpaper.

The two conffiles are installed at paths owned by the upstream `rpd-common` package. `debian/preinst` adds a `dpkg-divert` for each, redirecting `rpd-common`'s versions to `.upstream` so future upgrades of `rpd-common` do not clobber our default. `debian/postrm` reverses the diversions on `remove`/`purge`.

The package declares `Provides: halos-desktop-wallpaper`, `Conflicts: halos-desktop-wallpaper`, `Replaces: halos-desktop-wallpaper`. This virtual package is depended on by `halos-desktop` (in `halos-org/halos-metapackages`) and is also Provided by `halos-org/halos-desktop-branding`; only one real provider can be installed at a time. On HALPI2 desktop builds, pi-gen pre-installs this package via a conditional substage in `stage-halos-halpi2/` (gated on `STAGE_LIST` containing `stage-halos-desktop`), so `halos-desktop`'s virtual dependency is satisfied by this package and the generic `halos-desktop-branding` is not pulled in.

Both real providers ship their respective wallpaper to the same path (`/usr/share/halos/wallpapers/halpi.jpg` vs `/usr/share/halos/wallpapers/halos.jpg`). The exclusivity Conflicts on `halos-desktop-wallpaper` ensures the two `.deb` files never need to coexist on disk.

Per-user wallpaper choices in `~/.config/pcmanfm/default/desktop-items-{0,1}.conf` take precedence over the system defaults this package ships, so user agency is preserved.

## Repository Layout

```
.
├── debian/                        Debian packaging
├── docker/                        debtools container for building
├── etc/xdg/pcmanfm/default/       Conffiles (one source, the other is generated at build time)
├── usr/share/halos/wallpapers/    Asset shipped in the .deb (halpi.jpg)
├── sources/                       Affinity Photo source + color reference (LFS, not shipped)
├── .github/                       CI workflows, local actions, lefthook scripts
├── run                            Build/lint/deploy commands
├── VERSION                        Single source of truth for package version
├── .bumpversion.cfg               bumpversion config
└── lefthook.yml                   Pre-commit hooks
```

Source files (`*.afphoto`, `*.png`) under `sources/` are tracked via Git LFS and excluded from the `.deb`. Only `usr/share/halos/wallpapers/halpi.jpg` plus the two pcmanfm conffiles ship.

## Git Workflow Policy

**MANDATORY**: PRs must ALWAYS have all checks passing before merging. No exceptions.

**Branch Workflow:** Never push to main directly — always use feature branches and PRs.

**Changelog Policy**: Never edit `debian/changelog` directly. Always use `./run bumpversion` which uses `dch` for proper RFC 2822 date formatting.

**VERSION bumps**: Per release cycle, not per PR. See workspace `AGENTS.md` for the policy.

## CI / APT publishing

CI uses `hatlabs/shared-workflows` (note: NOT `halos-org/shared-workflows`). The default APT publish target is `hatlabs/apt.hatlabs.fi`, which is correct for this repo since it lives in the `hatlabs` org.

## Quick Start

```bash
# Build package
./run build-debtools  # First time only — builds the debtools container
./run build-deb

# Check quality
./run lint-deb

# Deploy to test device
./run deploy pi@halosdev.local

# Clean build artifacts
./run clean
```

## Verification on a Live Device

```bash
# Install the package
sudo apt install halos-halpi-desktop-branding

# Confirm diversions in place
dpkg-divert --list halos-halpi-desktop-branding

# Confirm our conffile is at the canonical path
dpkg -S /etc/xdg/pcmanfm/default/desktop-items-0.conf
# -> halos-halpi-desktop-branding

# Upstream's file should be at the diverted path
ls /etc/xdg/pcmanfm/default/desktop-items-0.conf.upstream
```

After a labwc session restart, the HALPI2 wallpaper should be visible on the desktop.
