# Maintainer: Michael Hansen <zrax0111 gmail com>

pkgbase=swift-language
pkgname=(swift swift-lldb)
_swiftver=3.1-RELEASE
pkgver=${_swiftver//-RELEASE/}
pkgrel=1
pkgdesc="The Swift programming language and debugger"
arch=('i686' 'x86_64')
url="http://swift.org/"
license=('apache')
depends=('python2' 'libutil-linux' 'icu' 'libbsd' 'libedit' 'libxml2'
         'sqlite' 'ncurses' 'libkqueue')
makedepends=('git' 'cmake' 'ninja' 'swig' 'clang>=3.9' 'python2-six' 'perl'
             'python2-sphinx' 'python2-requests')

source=(
    "swift-${_swiftver}.tar.gz::https://github.com/apple/swift/archive/swift-${_swiftver}.tar.gz"
    "swift-llvm-${_swiftver}.tar.gz::https://github.com/apple/swift-llvm/archive/swift-${_swiftver}.tar.gz"
    "swift-clang-${_swiftver}.tar.gz::https://github.com/apple/swift-clang/archive/swift-${_swiftver}.tar.gz"
    "swift-lldb-${_swiftver}.tar.gz::https://github.com/apple/swift-lldb/archive/swift-${_swiftver}.tar.gz"
    "swift-cmark-${_swiftver}.tar.gz::https://github.com/apple/swift-cmark/archive/swift-${_swiftver}.tar.gz"
    "swift-llbuild-${_swiftver}.tar.gz::https://github.com/apple/swift-llbuild/archive/swift-${_swiftver}.tar.gz"
    "swift-package-manager-${_swiftver}.tar.gz::https://github.com/apple/swift-package-manager/archive/swift-${_swiftver}.tar.gz"
    "swift-corelibs-xctest-${_swiftver}.tar.gz::https://github.com/apple/swift-corelibs-xctest/archive/swift-${_swiftver}.tar.gz"
    "swift-corelibs-foundation-${_swiftver}.tar.gz::https://github.com/apple/swift-corelibs-foundation/archive/swift-${_swiftver}.tar.gz"
    "swift-corelibs-libdispatch-${_swiftver}.tar.gz::https://github.com/apple/swift-corelibs-libdispatch/archive/swift-${_swiftver}.tar.gz"
    "swift-integration-tests-${_swiftver}.tar.gz::https://github.com/apple/swift-integration-tests/archive/swift-${_swiftver}.tar.gz"
    "block_include.patch"
)
sha256sums=('bc8f4fc1cb5e9cddcdca4208dc5db89696d6ab507e739d498519a0262bd453c0'
            '5f99110ac0fcd70b7fabf02989cfd0e7f1f1b6368b80d69f1506ce1fdc38c83e'
            'bb4543904e82f433a6a65612c9c4d8218dc5358f8097318f4f7fd6af145dd1f5'
            'eb10951a920b16fefccec73f919c9fc222973164b6d922a1084770b9bb822958'
            'f0906c6048cdc93c85106090a878dea7ca3b6d862091f82fe8073e273d3fc011'
            '578c0d28fc74df52c77dd6c1bfc91e45f0d9d2349e82855ae2f9715d1b25ac36'
            '54e66ff2fbe06011207e07a75807b9c0d317355c655c1d74580411e705f2b824'
            'fb52ec66d67b2c2005fd19a7a7f23cb8a6a077c9595d1514001b26690239814b'
            '82866426aae326fb910787080ad1a994061e0da410ee5bd0a2911b9fec603e53'
            '859742ac51832918e4400f06414ac33a3c65584681626e7e90a1b2b8eb514b1b'
            '81677333d13bbac8b324db8c223917475e0dc229026b12e4be425d9d2899fb28'
            'e89716c7a5bc047eddac22e7633804108c0e3c6d543c3005dec5e9b9334fb1c3')

prepare() {
    # Use python2 where appropriate
    find "$srcdir" -type f -print0 | \
         xargs -0 sed -i 's|/usr/bin/env python$|&2|'
    find "$srcdir/swift-lldb-swift-${_swiftver}" -name Makefile -print0 | \
         xargs -0 sed -i 's|python-config|python2-config|g'
    sed -i '/^cmake_minimum_required/a set(Python_ADDITIONAL_VERSIONS 2.7)' \
         "$srcdir/swift-swift-${_swiftver}/CMakeLists.txt"
    sed -i '/^cmake_minimum_required/a set(Python_ADDITIONAL_VERSIONS 2.7)' \
         "$srcdir/swift-lldb-swift-${_swiftver}/CMakeLists.txt"
    sed -i 's/\<python\>/&2/' \
         "$srcdir/swift-swift-${_swiftver}/utils/build-script-impl"

    # Use directory names which build-script expects
    for sdir in llvm clang lldb cmark llbuild; do
        ln -sf swift-${sdir}-swift-${_swiftver} ${sdir}
    done
    for sdir in corelibs-xctest corelibs-foundation corelibs-libdispatch \
                integration-tests
    do
        ln -sf swift-${sdir}-swift-${_swiftver} swift-${sdir}
    done
    ln -sf swift-swift-${_swiftver} swift
    ln -sf swift-package-manager-swift-${_swiftver} swiftpm

    # Fix some broken includes in foundation lib
    ( cd "${srcdir}/swift-corelibs-foundation" && \
        patch -p1 -i "${srcdir}/block_include.patch" )
}

build() {
    cd "$srcdir/swift"

    export SWIFT_SOURCE_ROOT="$srcdir"
    export LDFLAGS='-ldl -lpthread'
    export PATH="$PATH:/usr/bin/core_perl"
    _cpus="$(lscpu --parse=CPU | grep -v '^#' | wc -l)"
    utils/build-script -R -j "$_cpus" \
        --lldb --llbuild --swiftpm --xctest --foundation --libdispatch

    # Fix the lldb swig binding's import path (matches Arch LLDB package)
    # Need to do this before check(), since the test suite uses the lldb
    # python bindings directly from the build dir
    sed -i "/import_module('_lldb')/s/_lldb/lldb.&/" \
        "${srcdir}/build/Ninja-ReleaseAssert/lldb-linux-${CARCH}/lib/python2.7/site-packages/lldb/__init__.py"
}

check() {
    cd "$srcdir/swift"

    export SWIFT_SOURCE_ROOT="$srcdir"
    utils/build-script -R -t
}

package_swift() {
    pkgdesc='The Swift programming language compiler and tools'
    provides=('swift-language')
    conflicts=('swift-language-git' 'swift-git' 'swift-bin')
    optdepends=('swift-lldb: Swift REPL and debugger')

    cd "$srcdir/build/Ninja-ReleaseAssert"

    install -dm755 "$pkgdir/usr/bin"
    install -dm755 "$pkgdir/usr/lib/swift"

    # Swift's components don't provide an install target :(
    # These are based roughly on what's included in the binary release packages
    (
        cd swift-linux-$CARCH
        install -m755 bin/swift bin/swift-{demangle,ide-test} "$pkgdir/usr/bin"
        ln -s swift "$pkgdir/usr/bin/swiftc"
        ln -s swift "$pkgdir/usr/bin/swift-autolink-extract"

        install -dm755 "$pkgdir/usr/share/man/man1"
        install -m644 docs/tools/swift.1 "$pkgdir/usr/share/man/man1"

        umask 0022
        cp -rL lib/swift/{clang,linux,shims} "$pkgdir/usr/lib/swift/"
    )
    (
        cd llbuild-linux-$CARCH
        install -m755 bin/swift-build-tool "$pkgdir/usr/bin"
    )
    (
        cd swiftpm-linux-$CARCH
        install -m755 release/swift-{build,test,package} "$pkgdir/usr/bin"

        install -dm755 "$pkgdir/usr/lib/swift/pm"
        install -m755 lib/swift/pm/libPackageDescription.so "$pkgdir/usr/lib/swift/pm"
        install -m644 lib/swift/pm/PackageDescription.swiftmodule "$pkgdir/usr/lib/swift/pm"
    )
    (
        cd xctest-linux-$CARCH
        install -m755 libXCTest.so "$pkgdir/usr/lib/swift/linux/"
        install -m644 XCTest.swiftdoc "$pkgdir/usr/lib/swift/linux/$CARCH"
        install -m644 XCTest.swiftmodule "$pkgdir/usr/lib/swift/linux/$CARCH"
    )
    (
        cd foundation-linux-$CARCH
        install -m755 Foundation/libFoundation.so "$pkgdir/usr/lib/swift/linux/"
        install -m644 Foundation/Foundation.swiftdoc "$pkgdir/usr/lib/swift/linux/$CARCH"
        install -m644 Foundation/Foundation.swiftmodule "$pkgdir/usr/lib/swift/linux/$CARCH"

        umask 0022
        cp -r Foundation/usr/lib/swift/CoreFoundation "$pkgdir/usr/lib/swift/"
    )
    (
        cd libdispatch-linux-$CARCH
        make install DESTDIR="$pkgdir"
    )

    # License file
    install -dm755 "$pkgdir/usr/share/licenses/swift"
    install -m644 "$srcdir/swift/LICENSE.txt" "$pkgdir/usr/share/licenses/swift"
}

package_swift-lldb() {
    pkgdesc='The Swift programming language debugger (LLDB) and REPL'
    depends=('swift' 'python2-six')
    provides=('lldb')
    conflicts=('lldb')
    options=('!strip')  # Don't strip repl_swift -- we need its symbols

    cd "$srcdir/build/Ninja-ReleaseAssert"

    # Install swift-capable lldb
    (
        cd lldb-linux-$CARCH
        DESTDIR="$pkgdir" ninja install
    )

    # This should be provided from python2-six
    rm "$pkgdir/usr/lib/python2.7/site-packages/six.py"
    rm "$pkgdir/usr/lib/python2.7/site-packages/six.pyc"
}
