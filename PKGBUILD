# Maintainer: Jan Alexander Steffens (heftig) <heftig@archlinux.org>

pkgname=js91
pkgver=91.11.0
pkgrel=1
pkgdesc="JavaScript interpreter and libraries - Version 91"
arch=(riscv64)
url="https://spidermonkey.dev/"
license=(MPL)
depends=(gcc-libs readline zlib sh)
makedepends=(zip autoconf2.13 python-setuptools python-psutil rust llvm clang lld)
checkdepends=(mercurial git)
options=(!lto debug)
_relver=${pkgver}esr
source=(https://archive.mozilla.org/pub/firefox/releases/$_relver/source/firefox-$_relver.source.tar.xz{,.asc}
        tests-skip-some-tests-on-rv64.patch)
sha256sums=('e59bbe92ee1ef94936ce928324253e442748d62b5777bc0846ad79ed4a2a05a4'
            'SKIP'
            '1518e134fd5448d48f960bedbe7db061d5a8b368d43db2cac7f4b1adf094c748')
validpgpkeys=('14F26682D0916CDD81E37B6D61B7B526D98F0353') # Mozilla Software Releases <release@mozilla.com>

# Make sure the duplication between bin and lib is found
COMPRESSZST+=(--long)

prepare() {
  mkdir mozbuild
  cd firefox-$pkgver

  patch -Np1 < "../tests-skip-some-tests-on-rv64.patch" 
  cat >../mozconfig <<END
ac_add_options --enable-application=js
mk_add_options MOZ_OBJDIR=${PWD@Q}/obj

ac_add_options --prefix=/usr
ac_add_options --enable-release
ac_add_options --enable-hardening
ac_add_options --enable-optimize
ac_add_options --enable-rust-simd
ac_add_options --enable-linker=bfd
ac_add_options --disable-bootstrap
ac_add_options --disable-debug
ac_add_options --disable-debug-symbols
ac_add_options --disable-jemalloc
ac_add_options --disable-strip
ac_add_options --disable-jit

# System libraries
ac_add_options --with-system-zlib
ac_add_options --without-system-icu

# Features
ac_add_options --enable-readline
ac_add_options --enable-shared-js
ac_add_options --enable-tests
ac_add_options --with-intl-api
END
}

build() {
  cd firefox-$pkgver

  export MOZ_NOSPAM=1
  export MOZBUILD_STATE_PATH="$srcdir/mozbuild"
  export MACH_USE_SYSTEM_PYTHON=1

  ## Do 3-tier PGO
  #echo "Building instrumented JS..."
  cat >.mozconfig ../mozconfig - <<END
END
  ./mach build

  #echo "Profiling instrumented JS..."
  #(
  #  local js="$PWD/obj/dist/bin/js"
  #  export LLVM_PROFILE_FILE="$PWD/js-%p-%m.profraw"

  #  cd js/src/octane
  #  "$js" run.js

  #  cd ../../../third_party/webkit/PerformanceTests/ARES-6
  #  "$js" cli.js

  #  cd ../SunSpider/sunspider-0.9.1
  #  "$js" sunspider-standalone-driver.js
  #)

  #illvm-profdata merge -o merged.profdata *.profraw

  #stat -c "Profile data found (%s bytes)" merged.profdata
  #test -s merged.profdata

  #echo "Removing instrumented JS..."
  #./mach clobber

  #echo "Building optimized JS..."
  #cat >.mozconfig ../mozconfig - <<END
#ac_add_options --enable-lto=cross
#ac_add_options --enable-profile-use=cross
#ac_add_options --with-pgo-profile-path=${PWD@Q}/merged.profdata
#END
#  ./mach build
}

check() {
  local jstests_extra_args=(
    --format=none
    --exclude-random
    --wpt=disabled
  ) jittest_extra_args=(
    --format=none
    --timeout 300
  ) jittest_test_args=(
    basic
  )

  cd firefox-$pkgver/obj
  make -C js/src check-jstests check-jit-test \
    JSTESTS_EXTRA_ARGS="${jstests_extra_args[*]}" \
    JITTEST_EXTRA_ARGS="${jittest_extra_args[*]}" \
    JITTEST_TEST_ARGS="${jittest_test_args[*]}"
}

package() {
  cd firefox-$pkgver/obj
  make DESTDIR="$pkgdir" install
  rm "$pkgdir"/usr/lib/*.ajs
  find "$pkgdir"/usr/{lib/pkgconfig,include} -type f -exec chmod -c a-x {} +
}

# vim:set sw=2 et:
