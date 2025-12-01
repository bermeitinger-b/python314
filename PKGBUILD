# Maintainer: Blair Bonnett <blair.bonnett@gmail.com>

shopt -s extglob

pkgname=python315
pkgver=3.15.0a2
pkgrel=1
_pybasever=${pkgver%.*}
_pyurlversion=${pkgver}
_pyurlversion=${_pyurlversion%rc*}
_pyurlversion=${_pyurlversion%a*}
pkgdesc="The Python programming language (3.15)"
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
  'tzdata'
  'mpdecimal'
)
makedepends=(
  'bluez-libs'
  'cosign'
  'gdb'
  'llvm'
  'llvm-bolt'
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
  "https://www.python.org/ftp/python/${_pyurlversion}/Python-${pkgver}.tar.xz"{,.sigstore}
  EXTERNALLY-MANAGED)
md5sums=('8a16a56591101a698e8d0779d41782f4'
         '0748887e4de2c35ba06406a11ca18a17'
         '7d2680a8ab9c9fa233deb71378d5a654')

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

  # Ensure that we are using the system copy of various libraries (expat, libffi, and libmpdec),
  # rather than copies shipped in the tarball
  rm -rf Modules/expat
  rm -rf Modules/zlib
  rm -rf Modules/_ctypes/{darwin,libffi}*
  rm -rf Modules/_decimal/libmpdec
}

build() {
  cd "${srcdir}/Python-${pkgver}" || exit 1

  # PGO should be done with -O3
  CFLAGS="${CFLAGS/-O2/-O3} -ffat-lto-objects"

  export CFLAGS+=" -fno-semantic-interposition"
  export CXXLAGS+=" -fno-semantic-interposition"

  # Disable bundled pip & setuptools
  # BOLT is disabled due LLVM or upstream issue
  # https://github.com/python/cpython/issues/124948
  ./configure \
    --prefix=/usr \
    --enable-ipv6 \
    --enable-loadable-sqlite-extensions \
    --enable-optimizations \
    --enable-shared \
    --with-computed-gotos \
    --with-dbmliborder=gdbm:ndbm \
    --with-lto \
    --with-system-expat \
    --with-system-libmpdec \
    --with-tzpath=/usr/share/zoneinfo \
    --without-ensurepip

  make EXTRA_CFLAGS="$CFLAGS"
}

package() {
  cd "${srcdir}/Python-${pkgver}" || exit 1

  # Hack to avoid building again
  sed -i 's/^all:.*$/all: build_all/' Makefile

  # altinstall: /usr/bin/pythonX.Y but not /usr/bin/python or /usr/bin/pythonX
  make DESTDIR="${pkgdir}" altinstall maninstall

  # Split tests
  rm -rf "$pkgdir"/usr/lib/python*/{test,ctypes/test,distutils/tests,idlelib/idle_test,lib2to3/tests,tkinter/test,unittest/test}

  # some useful "stuff" FS#46146
  install -dm755 "${pkgdir}/usr/lib/python${_pybasever}/Tools/"{i18n,scripts}
  install -m755 "Tools/i18n/"{msgfmt,pygettext}".py" "${pkgdir}/usr/lib/python${_pybasever}/Tools/i18n/"
  install -m755 "Tools/scripts/"{README,*py} "${pkgdir}/usr/lib/python${_pybasever}/Tools/scripts/"

  # PEP668
  install -Dm644 "${srcdir}/EXTERNALLY-MANAGED" -t "${pkgdir}/usr/lib/python${_pybasever}/"

  # License
  install -Dm644 LICENSE "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}
