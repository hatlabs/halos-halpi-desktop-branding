# halos-halpi-desktop-branding

HALPI2 branding and default wallpaper for HaLOS HALPI2 desktop variants. Ships a single Debian package, `halos-halpi-desktop-branding`, that overrides the upstream Raspberry Pi OS default desktop wallpaper with a HALPI2-branded one on HALPI2 hardware.

Sibling package: [`halos-org/halos-desktop-branding`](https://github.com/halos-org/halos-desktop-branding) — same mechanism, generic HaLOS artwork for non-HALPI2 HaLOS desktop builds.

## What it ships

- `/usr/share/halos/wallpapers/halpi.jpg` — the HALPI2 wallpaper image (a colored treatment of Olaus Magnus's 1539 Carta Marina).
- `/etc/xdg/pcmanfm/default/desktop-items-0.conf` and `-1.conf` — pcmanfm system-wide desktop configuration pointing at the wallpaper.

## How the override works

The two `.conf` paths are owned by the upstream `rpd-common` package. This package uses `dpkg-divert` in its `preinst` to redirect `rpd-common`'s versions to `.upstream` and ship its own at the canonical paths. Future `rpd-common` upgrades land at the diverted paths, so the HALPI2 default is preserved.

Per-user wallpaper choices in `~/.config/pcmanfm/default/desktop-items-{0,1}.conf` take precedence — user agency is preserved.

## Exclusivity with generic HaLOS branding

`halos-halpi-desktop-branding` declares `Provides: halos-desktop-wallpaper`, `Conflicts: halos-desktop-wallpaper`, `Replaces: halos-desktop-wallpaper`. The generic-HaLOS sibling does the same. Only one of the two real packages can be installed at a time; the `halos-desktop` metapackage depends on the virtual `halos-desktop-wallpaper` name.

On HALPI2 desktop builds, pi-gen pre-installs this package from a conditional substage in `stage-halos-halpi2/` (gated on the build's `STAGE_LIST` containing `stage-halos-desktop`), so when `halos-desktop` is later resolved the virtual dependency is already satisfied by this package and the generic provider is not pulled in.

## Development

```bash
./run build-debtools  # first time
./run build-deb
./run lint-deb
```

See `AGENTS.md` for the full development guide.

## Source artwork

The `sources/` directory contains the Affinity Photo source file (a layered blend of the Bayerische Staatsbibliothek Munich scan of the 1539 woodcut with a colored facsimile reference from the James Ford Bell Library via Google Arts & Culture). The source is tracked via Git LFS and excluded from the `.deb`. Install Git LFS (`brew install git-lfs && git lfs install`) before cloning if you intend to edit the artwork.

## License

Code: MIT. Wallpaper artwork: CC-BY-4.0 © Hat Labs Ltd. The underlying 1539 Carta Marina by Olaus Magnus is in the public domain; the colored facsimile lineage is attributed in `debian/copyright`.
