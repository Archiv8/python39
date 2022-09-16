#!/bin/bash

# Based on original PKGBUILD by Tobias Kunze <r@rixx.de>

# Disable various shellcheck rules that produce false positives in this file.
# Repository rules should be added to the .shellcheckrc file located in the
# repository root directory, see https://github.com/koalaman/shellcheck/wiki
# and https://archiv8.github.io for further information.
# shellcheck disable=SC2034,SC2154
# [ToDo]: Add files: User documentation
# [ToDo]: Add files: Tooling
# [FixMe]: Namcap warnings and errors

# Maintainer: Ross Clark <https://github.com/orgs/Archiv8/python39/discussions>
# Contributor: Ross Clark <https://github.com/orgs/Archiv8/python39/discussions>

pkgname="python39"
pkgver=3.9.14
pkgrel=2
_pybasever=3.9
_pymajver=3
pkgdesc="Major release 3.9 of the Python high-level programming language"
arch=(
  "i686"
  "x86_64"
)
license=(
  "custom"
)
url="https://www.python.org/"
depends=(
  "bzip2"
  "expat"
  "gdbm"
  "libffi"
  "libnsl"
  "libxcrypt"
  "openssl"
  "zlib"
)
makedepends=(
  "bluez-libs"
  "mpdecimal"
  "gdb"
)
optdepends=(
  "sqlite" "mpdecimal: for decimal"
  "xz: for lzma"
  "tk: for tkinter")
source=(
  "https://www.python.org/ftp/python/${pkgver}/Python-${pkgver}.tar.xz"
  mpdecimal-2.5.1.patch
)
sha512sums=(
  "691a7814cf6c7bee96d8dbb7c5c85cb11f2e999101e20491b99435cdec07c3bbd5ce43ad3d9c64f695383b79197884caa1965c4346e4525e23b09c686271e4ab"
  "58f683cbfdc6aa84c03d068c1bc2f1d8d2c17ba4f7b632c14ab1d529d8332e767354266c3815e239427497fff1a42ec2a37739ea312d24cb76a69dcf1c98c0ad"
)

provides=("python=$pkgver")

prepare() {
  cd "${srcdir}/Python-${pkgver}"

  patch -p1 -i "${srcdir}/mpdecimal-2.5.1.patch"

  # FS#23997
  sed -i -e "s|^#.* /usr/local/bin/python|#!/usr/bin/python|" Lib/cgi.py

  # Ensure that we are using the system copy of various libraries (expat, zlib and libffi),
  # rather than copies shipped in the tarball
  rm -rf Modules/expat
  rm -rf Modules/zlib
  rm -rf Modules/_ctypes/{darwin,libffi}*
  rm -rf Modules/_decimal/libmpdec
}

build() {
  cd "${srcdir}/Python-${pkgver}"
  CFLAGS="${CFLAGS} -fno-semantic-interposition"

  export ax_cv_c_float_words_bigendian=no
  ./configure --prefix=/usr \
    --enable-shared \
    --with-computed-gotos \
    --with-lto \
    --enable-ipv6 \
    --with-system-expat \
    --with-dbmliborder=gdbm:ndbm \
    --with-system-ffi \
    --with-system-libmpdec \
    --enable-loadable-sqlite-extensions \
    --without-ensurepip \
    --with-tzpath=/usr/share/zoneinfo

  make EXTRA_CFLAGS="$CFLAGS"
}

package() {
  cd "${srcdir}/Python-${pkgver}"
  # altinstall: /usr/bin/pythonX.Y but not /usr/bin/python or /usr/bin/pythonX
  make DESTDIR="${pkgdir}" altinstall maninstall

  # Split tests
  rm -r "$pkgdir"/usr/lib/python*/{test,ctypes/test,distutils/tests,idlelib/idle_test,lib2to3/tests,sqlite3/test,tkinter/test,unittest/test}

  # Avoid conflicts with the main "python" package.
  rm -f "${pkgdir}/usr/lib/libpython${_pymajver}.so"
  rm -f "${pkgdir}/usr/share/man/man1/python${_pymajver}.1"

  # Clean-up reference to build directory
  sed -i "s|$srcdir/Python-${pkgver}:||" "$pkgdir/usr/lib/python${_pybasever}/config-${_pybasever}-${CARCH}-linux-gnu/Makefile"

  # Add useful scripts FS#46146
  install -dm755 "${pkgdir}"/usr/lib/python${_pybasever}/Tools/{i18n,scripts}
  install -m755 Tools/i18n/{msgfmt,pygettext}.py "${pkgdir}"/usr/lib/python${_pybasever}/Tools/i18n/
  install -m755 Tools/scripts/{README,*py} "${pkgdir}"/usr/lib/python${_pybasever}/Tools/scripts/

  # License
  install -Dm644 LICENSE "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}
