# Maintainer: Anatol Pomozov
# Maintainer: Eli Schwartz <eschwartz@archlinux.org>

_target=aarch64-linux-gnu
_tmpgccpath=/opt/gcc/stage1/
pkgname=$_target-glibc
pkgver=2.38
pkgrel=1
pkgdesc="GNU C Library ARM64 target"
arch=(any)
url='https://www.gnu.org/software/libc/'
license=('GPL' 'LGPL')
depends=(linux-api-headers)
makedepends=(python)
options=(!buildflags !strip staticlibs)
source=(https://ftp.gnu.org/gnu/libc/glibc-$pkgver.tar.xz{,.sig}
	git+http://github.com/celesrenata/glibc-2.38-aur.git{,.sig})
sha256sums=('fb82998998b2b29965467bc1b69d152e9c307d2cf301c9eafb4555b770ef3fd2'
            '2e4e058e845283ed06805bbac61e5e3c6c07298779ab5beff49f121c795a536a')
validpgpkeys=(7273542B39962DF7B299931416792B4EA25340F8  # "Carlos O'Donell <carlos@systemhalted.org>"
              BC7C7372637EC10C57D7AA6579C43DFBF1CF2187) # Siddhesh Poyarekar

prepare() {
  mkdir -p gcc13-build
}

build() {
  echo "testing if gcc13 already built by us"
  if ! test -f "/opt/gcc/stage1/bin/gcc"; then
    ./$srcdir/gcc13.build
  fi
  cd glibc-build

  echo 'slibdir=/lib' >> configparms
  echo 'rtlddir=/lib' >> configparms
  echo 'sbindir=/bin' >> configparms
  echo 'rootsbindir=/bin' >> configparms

  # remove hardening options for building libraries
  export CFLAGS="-U_FORTIFY_SOURCE -mlittle-endian -O2"
  export CPPFLAGS="-U_FORTIFY_SOURCE -O2"
  unset LD_LIBRARY_PATH

  export BUILD_CC=${_tmpgccpath}/bin/gcc
  export CC=${_tmpgccpath}/bin/${_target}-gcc
  export CXX=${_tmpgccpath}/bin/${_target}-g++
  export AR=${_tmpgccpath}/bin/${_target}-ar
  export RANLIB=${_tmpgccpath}/bin/${_target}-ranlib

  ../glibc-$pkgver/configure \
      --prefix=/usr \
      --target=$_target \
      --host=$_target \
      --build=$CHOST \
      --includedir=/include \
      --libdir=/lib \
      --libexecdir=/lib \
      --with-headers=/usr/$_target/include \
      --with-bugurl=https://bugs.archlinux.org/ \
      --enable-add-ons \
      --enable-obsolete-rpc \
      --enable-kernel=2.6.32 \
      --enable-bind-now \
      --disable-profile \
      --enable-stackguard-randomization \
      --enable-lock-elision \
      --enable-multi-arch \
      --disable-werror

  echo 'build-programs=no' >> configparms
  make
}

package() {
  cd glibc-build

  make install_root="$pkgdir"/usr/$_target install

  rm -r "$pkgdir"/usr/$_target/{etc,usr/share,var}
}

