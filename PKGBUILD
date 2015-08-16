# Maintainer: kalenz <archlinux@kalenz.fr>
# Based on lldb-svn PKGBUILD =>
# Contributor: Muhammad Tauqir Ahmad <mtahmed@uwaterloo.ca>
# Contributor: Philipp Sieweck <psi@informatik.uni-kiel.de>
# Contributor: Xavier de Gaye <xdegaye@gmail.com>
# Contributor: Michael Hansen <zrax0111 gmail com>

pkgname=lldb
_gcc_ver=4.6.2
pkgver=3.6.0
pkgrel=2
pkgdesc="The LLDB Debugger"
arch=('i686' 'x86_64')
url="http://llvm.org/"
license=('custom:University of Illinois/NCSA')
depends=('gcc-libs' 'libffi' 'python2' "gcc>=$_gcc_ver" 'libedit' 'llvm' 'clang')
makedepends=('cmake' 'swig')
provides=('lldb')
conflicts=('lldb')
source=(
  "http://llvm.org/releases/${pkgver}/llvm-${pkgver}.src.tar.xz"
  "http://llvm.org/releases/${pkgver}/cfe-${pkgver}.src.tar.xz"
  "http://llvm.org/releases/${pkgver}/lldb-${pkgver}.src.tar.xz"
)
sha256sums=(
  'b39a69e501b49e8f73ff75c9ad72313681ee58d6f430bfad4d81846fe92eb9ce'
  'be0e69378119fe26f0f2f74cffe82b7c26da840c9733fe522ed3c1b66b11082d'
  '2b1ad1d42c4ea3fa2f9dd6db7c522d86e80891659b24dbb3d0d80386d8eaf0b2'
)

build() {
  cd "$srcdir"
  msg2 "Moving Clang in the LLVM tree ..."
  mv "cfe-${pkgver}.src" "llvm-${pkgver}.src/tools/clang"
  msg2 "Moving LLDB in the LLVM tree ..."
  mv "lldb-${pkgver}.src" "llvm-${pkgver}.src/tools/lldb"

  cd "$srcdir/llvm-${pkgver}.src/tools/lldb"

  msg2 "Applying Archlinux-specific patch ..."

  sed -i -e "s|python-config|python2-config|" lib/Makefile
  sed -i -e "s|python-config|python2-config|" Makefile
  sed -i -e "s|/usr/bin/env python|&2|" scripts/Python/build-swig-Python.sh
  sed -i -e "s|/usr/bin/env python|&2|" scripts/Python/finish-swig-Python-LLDB.sh

  cd "$srcdir/llvm-${pkgver}.src"
  msg2 "Starting build ..."

  [[ -d build ]] && rm -r build
  mkdir build && cd build

  # libffi's includes are in a non-standard location :(
  _libffi_include=$(pkg-config libffi --cflags-only-I | sed 's/-I//')

  export CFLAGS="$CFLAGS -fno-tree-pre"
  export CXXFLAGS="$CXXFLAGS -fno-tree-pre"
  cmake \
    -DCMAKE_INSTALL_PREFIX=/usr \
    -DCMAKE_BUILD_TYPE=Release \
    -DLLVM_ENABLE_ASSERTIONS=OFF \
    -DLLVM_ENABLE_FFI=ON \
    -DPYTHON_EXECUTABLE=/usr/bin/python2 \
    -DFFI_INCLUDE_PATH=$_libffi_include \
    ..

  make
}

package() {
  cd "$srcdir/llvm-${pkgver}.src"

  # Install the license
  install -Dm644 tools/lldb/LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"

  cd "$srcdir/llvm-${pkgver}.src/build"

  # Install the lldb binaries
  install -Dm755 bin/lldb "$pkgdir/usr/bin/lldb"
  install -Dm755 bin/lldb-mi "$pkgdir/usr/bin/lldb-mi"
  install -Dm755 bin/lldb-platform "$pkgdir/usr/bin/lldb-platform"
  install -Dm755 bin/lldb-gdbserver "$pkgdir/usr/bin/lldb-gdbserver"

  # Install the lldb library
  install -Dm755 lib/liblldb.so "$pkgdir/usr/lib/liblldb.so"

  # Install the lldb python libraries.
  python_dir="$pkgdir/usr/lib/python2.7/site-packages"
  mkdir -p "$python_dir"
  cp -a lib/python2.7/site-packages/lldb "$python_dir"

  # Relink the _lldb.so for python
  ln -sf /usr/lib/liblldb.so "$python_dir/lldb/_lldb.so"
  chmod o+r "$python_dir/lldb/embedded_interpreter.py"
}

# vim:set sts=2 sw=2 et:
