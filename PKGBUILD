# Maintainer: Blair Bonnett <blair.bonnett@gmail.com>

shopt -s extglob

pkgbase=python
pkgname=(python python-tests)
pkgver=3.14.0
pkgrel=1
_pybasever=${pkgver%.*}
pkgdesc="The Python programming language (3.14)"
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
  "https://www.python.org/ftp/python/${pkgver%rc*}/Python-${pkgver}.tar.xz"{,.sigstore}
  EXTERNALLY-MANAGED)
md5sums=('41389edaf9c643263cbed9b5ed307df8'
         '5851da095040130aeded817ff21d9003'
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

package_python() {
  optdepends=(
    'python-setuptools: for building Python packages using tooling that is usually bundled with Python'
    'python-pip: for installing Python packages using tooling that is usually bundled with Python'
    'python-pipx: for installing Python software not packaged on Arch Linux'
    'sqlite: for a default database integration'
    'xz: for lzma'
    'tk: for tkinter'
  )
  provides=('python313' 'python-externally-managed')
  replaces=('python313' 'python-externally-managed')

  cd "${srcdir}/Python-${pkgver}" || exit 1

  # Hack to avoid building again
  sed -i 's/^all:.*$/all: build_all/' Makefile

  # PGO should be done with -O3
  CFLAGS="${CFLAGS/-O2/-O3}"

  make DESTDIR="${pkgdir}" EXTRA_CFLAGS="$CFLAGS" install

  # Why are these not done by default...
  ln -s "python3" "${pkgdir}/usr/bin/python"
  ln -s "python3-config" "${pkgdir}/usr/bin/python-config"
  ln -s "idle3" "${pkgdir}/usr/bin/idle"
  ln -s "pydoc3" "${pkgdir}/usr/bin/pydoc"
  ln -s "python${_pybasever}.1" "${pkgdir}/usr/share/man/man1/python.1"

  # some useful "stuff" FS#46146
  install -dm755 "${pkgdir}/usr/lib/python${_pybasever}/Tools/"{i18n,scripts}
  install -m755 "Tools/i18n/"{msgfmt,pygettext}".py" "${pkgdir}/usr/lib/python${_pybasever}/Tools/i18n/"
  install -m755 "Tools/scripts/"{README,*py} "${pkgdir}/usr/lib/python${_pybasever}/Tools/scripts/"

  # PEP668
  install -Dm644 "${srcdir}/EXTERNALLY-MANAGED" -t "${pkgdir}/usr/lib/python${_pybasever}/"

  # Split tests
  cd "${pkgdir}/usr/lib/python${_pybasever}/" || exit 1
  rm -r {test,idlelib/idle_test}
}

package_python-tests() {
  pkgdesc="Regression tests packages for Python"
  depends=('python313')

  cd "${srcdir}/Python-${pkgver}" || exit 1

  make DESTDIR="${pkgdir}" EXTRA_CFLAGS="$CFLAGS" libinstall
  cd "${pkgdir}/usr/lib/python${_pybasever}/" || exit 1
  rm -r !(test)
}
