# Maintainer: Blair Bonnett <blair.bonnett@gmail.com>

pkgname=python314
pkgver=3.14.0rc1
pkgrel=1
_pyver=3.14.0
_pybasever=3.14
_pymajver=3
pkgdesc="Major release 3.14 of the Python high-level programming language"
arch=('x86_64')
license=('PSF-2.0')
url="https://www.python.org/"

depends=(
  'bzip2'
  'expat'
  'gdbm'
  'libffi'
  'libnsl'
  'libxcrypt'
  'openssl'
  'zlib'
)
makedepends=(
  'bluez-libs'
  'cosign'
  'gdb'
  'mpdecimal'
  'sqlite'
  'tk'
)
optdepends=(
  'mpdecimal: for decimal'
  'sqlite'
  'tk: for tkinter'
  'xz: for lzma'
)
options=(!emptydirs)

source=(
  "https://www.python.org/ftp/python/${_pyver}/Python-${pkgver}.tar.xz"{,.sigstore}
)
md5sums=('48c4518c06dcb675c24276c56f69b9fd'
         '5a7ea086189014e3602fa225b179f7b1')

verify() {
  cosign verify-blob \
    --new-bundle-format \
    --certificate-oidc-issuer 'https://github.com/login/oauth' \
    --certificate-identity 'hugo@python.org' \
    --bundle ./Python-${pkgver}.tar.xz.sigstore \
    ./Python-${pkgver}.tar.xz
}

prepare() {
  cd "${srcdir}/Python-${pkgver}" || exit 1

  # Ensure that we are using the system copy of various libraries (expat, zlib and libffi),
  # rather than copies shipped in the tarball
  rm -rf Modules/expat
  rm -rf Modules/zlib
  rm -rf Modules/_ctypes/{darwin,libffi}*
  rm -rf Modules/_decimal/libmpdec
}

build() {
  cd "${srcdir}/Python-${pkgver}" || exit 1

  CFLAGS="${CFLAGS} -fno-semantic-interposition -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer"
  ./configure ax_cv_c_float_words_bigendian=no \
              --prefix=/usr \
              --enable-shared \
              --with-computed-gotos \
              --with-lto \
              --enable-ipv6 \
              --with-system-expat \
              --with-dbmliborder=gdbm:ndbm \
              --with-system-libmpdec \
              --enable-loadable-sqlite-extensions \
              --without-ensurepip \
              --with-tzpath=/usr/share/zoneinfo \
              --enable-optimizations

  make EXTRA_CFLAGS="$CFLAGS"
}

package() {
  cd "${srcdir}/Python-${pkgver}" || exit 1
  # altinstall: /usr/bin/pythonX.Y but not /usr/bin/python or /usr/bin/pythonX
  make DESTDIR="${pkgdir}" altinstall maninstall

  # Split tests
  rm -r "$pkgdir"/usr/lib/python*/{test,idlelib/idle_test}

  # Avoid conflicts with the main 'python' package.
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
