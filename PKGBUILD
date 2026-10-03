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
pkgrel=4
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
  scenefx
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
