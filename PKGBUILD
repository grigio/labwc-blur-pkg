# Maintainer: Arch build of the labwc-blur fork (ext-background-effect-v1)
#
# Fork of the official Arch 'labwc' package (Peter Jung <ptr1337@archlinux.org>)
# building https://github.com/grigio/labwc branch ext-background-effect.
# The same PKGBUILD is used locally and by GitHub Actions (see
# .github/workflows/build.yml); the source is always the branch tip on GitHub,
# so local changes must be pushed before `makepkg` picks them up.
# Installing this package replaces the official 'labwc' package.
#
# Needs the local 'scenefx' package (see ./scenefx/PKGBUILD), which is not
# available in the Arch repositories.

pkgname=labwc-blur
pkgver=0.20.2.34.g1d2cb20e
pkgrel=5
pkgdesc='stacking wayland compositor with look and feel from openbox (fork with ext-background-effect-v1 support)'
url="https://github.com/grigio/labwc"
arch=('x86_64')
license=('GPL-2.0-only')
depends=(
  cairo
  glib2
  glibc
  libinput
  libpng
  librsvg
  libsfdo
  libwlroots-0.20.so
  libxcb
  libxkbcommon
  libxml2
  pango
  pixman
  # Versioned, not the bare virtual name: scenefx-wlroots20-git (CachyOS)
  # also provides 'scenefx' but ships SONAME libscenefx-0.5.so, while this
  # binary needs libscenefx-0.5.so.0 -> the session would fail to start.
  scenefx-0.5
  seatd
  ttf-font
  wayland
)
makedepends=(
  git
  meson
  scdoc
  wayland-protocols
  xorg-xwayland
)
optdepends=(
  "bemenu: default launcher via Alt+F3"
  "xorg-xwayland: X11 application support"
)
provides=("labwc=${pkgver}")
conflicts=('labwc')
source=(
  "labwc-blur::git+https://github.com/grigio/labwc.git#branch=ext-background-effect"
)
b2sums=('SKIP')

pkgver() {
  # Needs the version tags on the fork: GitHub does not copy tags when
  # labwc/labwc is forked, so `git push origin --tags` from the local
  # checkout must be run once (and after new upstream releases), otherwise
  # git describe fails and makepkg aborts with "pkgver is not allowed to be empty".
  cd labwc-blur
  git describe --tags --long | sed 's/-/./g'
}

build() {
  arch-meson -Dman-pages=enabled "$pkgname" build
  meson compile -C build
}

package() {
  meson install -C build --destdir "$pkgdir"
}
