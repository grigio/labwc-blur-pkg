# labwc-blur-pkg

Arch Linux packaging for **labwc-blur** — [labwc](https://github.com/labwc/labwc)
with `ext-background-effect-v1` (server-side blur for windows, bars, docks) —
built from the
[`ext-background-effect` branch of grigio/labwc](https://github.com/grigio/labwc/tree/ext-background-effect).

GitHub Actions builds `labwc-blur` (+ its `scenefx` dependency) automatically
on every push here, every 3 hours, and on demand.

## Install (from Releases)

Every successful CI run publishes a **release tagged `v<pkgver>-<pkgrel>`**
(e.g. [`v0.20.2.34.g1d2cb20e-4`](https://github.com/grigio/labwc-blur-pkg/releases))
— the tag embeds the labwc short-sha, so **every new commit on
`ext-background-effect` gets its own release**:

1. Open **Releases** and take the assets of the newest release
2. Install **both** packages — they ship as a pair:

```sh
sudo pacman -U scenefx-*.pkg.tar.zst labwc-blur-*.pkg.tar.zst
```

> [!IMPORTANT]
> `scenefx` is **not** optional. `labwc-blur` links against the soname of *this*
> build (`libscenefx-0.5.so.0`); any other `scenefx` (e.g. the CachyOS
> `scenefx-wlroots20-git`, soname `libscenefx-0.5.so`) leaves you with a
> session that dies at login. Answer **yes** when pacman offers to remove
> `scenefx-wlroots20-git` or the official `labwc`.

3. Verify:

```sh
ldd /usr/bin/labwc | grep scenefx   # → /usr/lib/libscenefx-0.5.so.0, no 'not found'
labwc --version
```

The same files are also attached to each run as the **`arch-packages`**
artifact (30-day retention) if you prefer Actions → Artifacts.

Notes:

* `labwc-blur` **conflicts with the official `labwc`** package (replaces it).
* Every run also ships `labwc-blur-debug` (debug symbols) and
  `scenefx-debug`.
* A rebuild of an unchanged version refreshes the existing release assets
  (`--clobber`) instead of creating a duplicate release.
* The `scenefx` package **conflicts with `scenefx-wlroots20-git`**, so a plain
  `pacman -U` of both assets swaps it out for you.

## Enabling blur

Blur is **client-driven**: labwc-blur blurs the background *behind a surface
that asks for it* with the `ext-background-effect-v1` protocol. Installing the
package alone blurs nothing.

1. **Compositor side** — the `<blur>` block is a direct child of
   `<labwc_config>` in `~/.config/labwc/rc.xml` (these are the defaults):

   ```xml
   <blur>
     <passes>3</passes>
     <radius>5</radius>
     <noise>0.02</noise>
     <brightness>0.9</brightness>
     <contrast>0.9</contrast>
     <saturation>1.1</saturation>
     <strength>1.0</strength>  <!-- 0 turns blur off globally -->
   </blur>
   ```

   All values are re-read live: `labwc --reconfigure`. Reference:
   `man labwc-config` → **BLUR**, or `/usr/share/doc/labwc/rc.xml.all`.

2. **Translucency** — an opaque window hides whatever is behind it, so the
   client needs `opacity < 1` (or a transparent background colour).

3. **A client that speaks the protocol** — `noctalia` does, so the shell
   panels are blurred out of the box (`grep -l ext_background_effect
   /usr/bin/*` finds it).

### Alacritty

Since 0.17 alacritty ships a `blur` option. `~/.config/alacritty/alacritty.toml`:

```toml
[window]
opacity = 0.9   # translucency: what shows through the terminal
blur = true     # ask the compositor to blur behind the window
```

Unknown keys are reported on startup as `Unused config key: …`, so a plain
`alacritty` run validates the file.

> [!NOTE]
> Alacritty (through winit 0.30) issues that request with **KDE's**
> `org_kde_kwin_blur_manager`, while labwc-blur serves
> `ext-background-effect-v1`. On labwc the request is therefore a **no-op**
> today — you get the transparency, not the blur (winit logs *"Blur manager
> unavailable, unable to change blur"*). It already works on KDE Wayland and
> macOS, and starts working here as soon as labwc-blur grows an
> `org_kde_kwin_blur` shim or winit learns `ext-background-effect-v1`.

## Troubleshooting

### Session fails to start (drops straight back to the greeter)

Almost always the wrong `scenefx`:

```sh
ldd /usr/bin/labwc | grep scenefx        # libscenefx-0.5.so.0 => not found
sudo pacman -Rdd scenefx-wlroots20-git   # -dd keeps labwc-blur installed
sudo pacman -U scenefx-*.pkg.tar.zst
```

If it still fails, look at what the compositor printed:

```sh
journalctl -b --no-pager | grep -iE 'labwc|greetd' | tail -30
labwc --version
```

## Contents

| path | what |
|---|---|
| `PKGBUILD` | `labwc-blur`, source `git+https://github.com/grigio/labwc.git#branch=ext-background-effect`, `pkgver()` from `git describe` |
| `scenefx/PKGBUILD` | SceneFX 0.5 pinned to commit `dc3cddc` (provides `wlr_scene` blur nodes), not available in the official repos; conflicts with the CachyOS `scenefx-wlroots20-git` |
| `.github/workflows/build.yml` | CI: `archlinux:base-devel` container, `makepkg -sCi`, then a **sanity check** (installs the built pair, verifies the `libscenefx` soname matches what `labwc` links and that every lib resolves) before uploading/publishing |

## CI triggers

* **push** to this repo, when `PKGBUILD`, `scenefx/PKGBUILD` or the workflow
  change (a `README.md`-only push does not trigger a build),
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
