# labwc-blur-pkg

Arch Linux packaging for **labwc-blur** — [labwc](https://github.com/labwc/labwc)
with `ext-background-effect-v1` (server-side blur for windows, bars, docks) —
built from the
[`ext-background-effect` branch of grigio/labwc](https://github.com/grigio/labwc/tree/ext-background-effect).

GitHub Actions builds `labwc-blur` (+ its `scenefx` dependency) automatically
on every push here, daily, and on demand.

## Install (from Releases)

Every successful CI run publishes a **release tagged `v<pkgver>-<pkgrel>`**
(e.g. [`v0.20.2.34.g1d2cb20e-4`](https://github.com/grigio/labwc-blur-pkg/releases))
— the tag embeds the labwc short-sha, so **every new commit on
`ext-background-effect` gets its own release**:

1. Open **Releases** and take the assets of the newest release
2. Install:

```sh
sudo pacman -U labwc-blur-*.pkg.tar.zst
# only if scenefx is not installed yet (it is NOT in the official repos):
sudo pacman -U scenefx-*.pkg.tar.zst
```

The same files are also attached to each run as the **`arch-packages`**
artifact (30-day retention) if you prefer Actions → Artifacts.

Notes:

* `labwc-blur` **conflicts with the official `labwc`** package (replaces it).
* Every run also ships `labwc-blur-debug` (debug symbols) and
  `scenefx-debug`.
* A rebuild of an unchanged version refreshes the existing release assets
  (`--clobber`) instead of creating a duplicate release.

## Contents

| path | what |
|---|---|
| `PKGBUILD` | `labwc-blur`, source `git+https://github.com/grigio/labwc.git#branch=ext-background-effect`, `pkgver()` from `git describe` |
| `scenefx/PKGBUILD` | SceneFX 0.5 pinned to commit `dc3cddc` (provides `wlr_scene` blur nodes), not available in the official repos |
| `.github/workflows/build.yml` | CI: `archlinux:base-devel` container, `makepkg -sCi` |

## CI triggers

* **push** to this repo, when `PKGBUILD`, `scenefx/PKGBUILD` or the workflow
  change,
* **schedule**: every 3 hours — picks up new commits pushed to
  `grigio/labwc` `ext-background-effect` and publishes their release,
* **workflow_dispatch**: manual run from the Actions tab,
* **repository_dispatch** (`build-labwc-blur`): optional push-trigger from the
  labwc repo, see the comment inside the workflow.

## Build locally

```sh
git clone git@github.com:grigio/labwc-blur-pkg.git
cd labwc-blur-pkg
cd scenefx && makepkg -sCi && cd ..   # dependency first
makepkg -sCi                          # labwc-blur
```

`makepkg -C` forces a fresh clone of the labwc source (needed once if you have
an old `src/labwc-blur` checkout that still points at a `file://` URL).
Remember: the PKGBUILD always builds what is **pushed** to the
`ext-background-effect` branch — commit and push `~/Code/labwc-blur` first.

> [!NOTE]
> The fork needs the upstream version tags for `pkgver()`:
> `git push origin --tags` from `~/Code/labwc-blur` (done once already),
> repeat after new upstream releases.
