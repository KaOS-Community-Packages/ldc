pkgname=('ldc')
pkgver=1.20.1
_pkgcommit=96437a25c28a2a6fbeb36dc6a46b600e56021051
_dversion=2.090.1
_clangversion=9.0.1 # related to where ldc2 looks for compiler-rt sanitizers
pkgrel=1
pkgdesc="A D Compiler based on the LLVM Compiler Infrastructure including D runtime and libphobos2"
arch=('x86_64')
url="https://github.com/ldc-developers/ldc"
backup=('etc/ldc2.conf')
depends=('dmd' 'gcc' 'clang' 'llvm')
makedepends=('git' 'cmake')
license=('BSD')
options=('staticlibs')
source=(
    "git+https://github.com/ldc-developers/ldc#commit=$_pkgcommit"
    "git+https://github.com/ldc-developers/druntime.git"
    "git+https://github.com/ldc-developers/phobos.git"
    "git+https://github.com/ldc-developers/dmd-testsuite.git"
)
sha256sums=('SKIP'
            'SKIP'
            'SKIP'
            'SKIP')
prepare() {
    cd "$srcdir/ldc"

    git submodule init
    git config submodule.druntime.url "$srcdir/druntime"
    git config submodule.phobos.url "$srcdir/phobos"
    git config submodule.tests/d2/dmd-testsuite.url "$srcdir/dmd-testsuite"
    git submodule update

    # Set version used for path construction in getFullClangCompilerRTLibPath()
    sed -i "s/ldc::llvm_version_base/\"$_clangversion\"/" driver/linker-gcc.cpp
}
build() {
    cd "$srcdir/ldc"

    mkdir -p build && cd build

    cmake \
    -DCMAKE_INSTALL_PREFIX=/usr \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_SKIP_RPATH=ON \
    -DINCLUDE_INSTALL_DIR=/usr/include/dlang/ldc \
    -DBUILD_SHARED_LIBS=BOTH \
    -DBUILD_LTO_LIBS=ON \
    -DLDC_WITH_LLD=OFF \
    -DADDITIONAL_DEFAULT_LDC_SWITCHES="\"-link-defaultlib-shared\"" \
    ..
    make
}
check() {
    cd "$srcdir/ldc/build"
    make all-test-runners
}
package() {
    cd "$srcdir/ldc/build"
    make install DESTDIR="$pkgdir"

    # move bash-completion
    mkdir -p "$pkgdir/usr/share/bash-completion/completions/"
    mv "$srcdir/ldc/packaging/bash_completion.d/ldc2" "$pkgdir/usr/share/bash-completion/completions/"
    rm -rf "$pkgdir/etc/bash_completion.d"

    # symlinks
    ln -s /usr/share/bash-completion/completions/ldc2 "$pkgdir/usr/share/bash-completion/completions/ldc"
    ln -s /usr/bin/ldc2 "$pkgdir/usr/bin/ldc"
    ln -s /usr/bin/ldmd2 "$pkgdir/usr/bin/ldmd"

    # remove liblphobos files
    rm -rf "$pkgdir/usr/include"
    rm -rf "$pkgdir/usr/lib"

    # remove ldc files
    rm -rf "$pkgdir/usr/bin/"
    rm -rf "$pkgdir/etc/"

    # licenses
    install -D -m644 "$srcdir/ldc/LICENSE" "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
