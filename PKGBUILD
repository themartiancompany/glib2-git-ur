# SPDX-License-Identifier: AGPL-3.0
#
# Maintainer: Limao Luo <luolimao+AUR@gmail.com>
# Contributor: Yichao Yu <yyc1992@gmail.com>
# Contributor: kfgz <kfgz@interia.pl>
# Contributor: Pellegrino Prevete (dvorak) <pellegrinoprevete@gmail.com>
# Contributor: Truocolo <truocolo@aol.com>

_checks="false"
_docs="false"
_libmount="enabled"
_py='python'
_pkg="glib"
_pkgname="${_pkg}2"
pkgbase="${_pkgname}-git"
pkgname=(
  "${pkgbase}"
)
[[ "${_docs}" == "true" ]] && \
  pkgname+=(
    "${_pkgname}-docs-git"
  )
pkgver=2.78.0.r212.g0a6e19f
pkgrel=1
pkgdesc="Low Level Core Library"
arch=(
  'x86_64'
  'i686'
  'pentium4'
  'arm'
  'armv7h'
  'aarch64'
  'powerpc'
)
url="https://wiki.gnome.org/Projects/GLib"
license=(
  LGPL2.1
)
depends=(
  pcre
  libffi
  util-linux-libs
  zlib
  libsysprof-capture
)
makedepends=(
  dbus
  gettext
  git
  libelf
  meson
  shared-mime-info
  "${_py}"
  util-linux
)
[[ "${_docs}" == "true" ]] && \
  makedepends+=(
    'gi-docgen'
    'gtk-doc'
    "${_py}-docutils"
)
checkdepends=(
  desktop-file-utils
)
optdepends=(
  "${_py}: gdbus-codegen, ${_pkg}-genmarshal, ${_pkg}-mkenums, gtester-report"
  "libelf: gresource inspection tool"
)
conflicts=(
  "${_pkgname}"
  "${_pkg}")
provides=(
  "${_pkgname}=${pkgver}"
  "${_pkg}=${pkgver}"
)
options=(
  debug
  staticlibs)
_http="https://gitlab.gnome.org"
_ns="GNOME"
_url="${_http}/${_ns}/${_pkg}"
_local="file://${HOME}/${_pkg}"
_local_gvdb="file://${HOME}/gvdb"
source=(
  # "git+${_url}.git"
  "git+${_local}"
  # "git+${_http}/${_ns}/gvdb.git"
  "git+${_local_gvdb}"
  "0001-noisy-${_pkg}-compile-schemas.patch"
  "${_pkg}-compile-schemas.hook"  
  gio-querymodules.hook
)
sha512sums=(
  'SKIP'
  'SKIP'
  'ddbf4a8eaf60e943a10a1ad67f2de078143558df8cc06e8009da87d8068af0cf8c66f443474b8b2848239c003e6210ff9ceb1ba5ffda1b95b80687adbf813722'
  'c04fe25afc217c295b5ce4034733cec046126482d00fb8d0299e4815ac57129dd3f1c9ac824b9386d208a4f113e9dae682ea5b72f75387ed6b6b96a9cbbee8ca'
  '5afd6f275c8fff16df3e685818f2e7989b39ffb3b8f5fc261a5a6d54a9b28ef53af62f3bf5067cf87cb74691572f85730cbc508691956ae048a0f3ecc1a0a39c')

pkgver() {
  cd \
    "${_pkg}"
  git \
    describe \
    --long \
    --tags \
    --abbrev=7 | \
    sed \
      's/\([^-]*-g\)/r\1/;s/-/./g'
}

_prefix="$( \
  dirname \
    "$( \
      dirname \
        "$( \
          command \
            -v \
            "cc")")")"

prepare() {
  cd \
    "${srcdir}/${_pkg}"
    # Suppress noise from glib-compile-schemas.hook
  patch \
    -Np1 \
    -i \
    ../"0001-noisy-${_pkg}-compile-schemas.patch"

  git \
    submodule \
    init
  git \
    submodule \
    set-url \
    subprojects/gvdb \
    "${srcdir}/gvdb"
  git \
    -c protocol.file.allow=always \
    submodule \
      update
}

_meson_options=(
  --default-library both
  -D "${_pkg}"_debug="${_checks}"
  -D documentation="${_docs}"
  -D libmount="${_libmount}"
  -D man-pages="${_docs}"
  -D selinux=disabled
  -D sysprof="${_checks}"
)

_flags() {
  _cflags=()
  local \
    _bin \
    _include
  _bin="$( \
    cc \
     -v 2>&1 |
     grep \
       "InstalledDir" | \
       awk '{print $2}')"
  if [[ "${_bin}" == "" ]]; then
    _bin="$( \
      dirname \
      "$(command \
          -v \
          gcc)")"
  fi
  if [[ "${_bin}" != "" ]]; then
    _include="$( \
      dirname \
        "${_bin}")/include"
    _cflags+=" -I${_include}"
  fi
  _cflags+=(
    "${CFLAGS}"
    "-ffat-lto-objects"
    "-g3")
}

build () {
  # Produce more debug info: GLib has a lot of useful macros
  CXXFLAGS+=" -g3"

  # use fat LTO objects for static libraries
  CXXFLAGS+=" -ffat-lto-objects"

  _flags

  CC="gcc" \
  CXX="g++" \
  CFLAGS="${_cflags[@]}" \
    arch-meson \
      "${_pkg}" \
      build \
      "${_meson_options[@]}"
  CC="gcc" \
  CXX="g++" \
  CFLAGS="${_cflags[@]}" \
    meson \
      compile \
      -C build
}

check() {
  meson \
    test \
    -C build
}

package_glib2-git() {
  local \
    _codegen
  _codegen="/usr/share/glib-2.0/codegen"
  provides+=(
    libgio-2.0.so
    "lib${_pkg}-2.0.so=${pkgver}"
    libgmodule-2.0.so
    libgobject-2.0.so
    libgthread-2.0.so)
  depends+=(
    libffi.so
  )
  [[ "libmount" == "enabled" ]] && \
    depends+=(
      libmount.so
    )
  DESTDIR="${pkgdir}" \
    meson \
      install \
      -C build
  for _f  \
    in \
      find \
        "${pkgdir}/usr/lib/pkgconfig"; do
    sed \
      -i
      "s%prefix=/usr%prefix=${_prefix}%"
      "${_f}"
  done
  if [[ " ${_meson_options[*]} " =~ ' documentation=true ' ]]; then
    mv \
      "${pkgdir}/usr/share/gtk-doc" \
      "${srcdir}"
  fi
  install \
    -Dt \
    "${pkgdir}/usr/share/libalpm/hooks" \
    -m644 \
    "${srcdir}/"*".hook"

  "${_py}" \
    -m compileall \
    -d "${_codegen}" \
    "${pkgdir}${_codegen}"
}

package_glib2-docs-git() {
  pkgdesc="Documentation for GLib"
  depends=()
  optdepends=()
  provides=(
    "${_pkgname}-docs"
    "${_pkg}-docs"
  )
  conflicts=(
    "${_pkgname}-docs"
    "${_pkg}-docs"
  )
  license+=(
    custom)

  if [[ " ${_meson_options[*]} " =~ ' documentation=true ' ]]; then
    mkdir \
      -p \
      "${pkgdir}/usr/share"
    mv \
      gtk-doc \
      "${pkgdir}/usr/share"

    install \
      -Dt "${pkgdir}/usr/share/licenses/glib2-docs" \
      -m644 glib/docs/reference/COPYING
  fi
}

# vim:set sw=2 sts=-1 et:
