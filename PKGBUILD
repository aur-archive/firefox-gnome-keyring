# Maintainer: lepokle <devel@leo.von-klenze.de>
pkgname='firefox-gnome-keyring'
pkgver=37.0.1
pkgrel=1
pkgdesc="Gnome-keyring integration for Firefox"
arch=('i686' 'x86_64')
url='http://github.com/lepokle/firefox-gnome-keyring'
license=('GPL')
depends=("firefox>=$pkgver" 'gnome-keyring')
#depends=('gnome-keyring')
makedepends=('zip' 'unzip' 'nspr>=4.10.8' 'libgnome-keyring' 'nss>=3.18')
source=("${pkgname}-1.2.tar.gz::$url/tarball/v1.2"
        "ftp://ftp.mozilla.org/pub/mozilla.org/firefox/releases/$pkgver/source/firefox-$pkgver.source.tar.bz2"
        "mozconfig")

md5sums=('eee0cd910b893edce89c56d505ba1b3f'
         '2d466d850ef9b0a71bce3b208f5f884c'
         '4c041fc20e70a754d496f6bf34960474')

_xulsrcdir="mozilla-release"
_xuldir="xulrunner"
_subdir="lepokle-firefox-gnome-keyring-ffe2844"

prepare()
{
    msg "Prepare XUL Runner..."
    cp mozconfig "$srcdir/$_xulsrcdir/.mozconfig" 
    echo "mk_add_options MOZ_OBJDIR=\"$srcdir/$_xuldir\"" >> "$srcdir/$_xulsrcdir/.mozconfig"

    # WebRTC build tries to execute "python" and expects Python 2
    # Workaround taken from chromium PKGBUILD
    mkdir -p "$srcdir/python2-path"
    ln -sf /usr/bin/python2 "$srcdir/python2-path"

    # configure script misdetects the preprocessor without an optimization level
    # https://bugs.archlinux.org/task/34644
    sed -i '/ac_cpp=/s/$CPPFLAGS/& -O2/' "$srcdir/$_xulsrcdir/configure"
}

build()
{
    msg "Build XUL Runner..."

    export PATH="$srcdir/python2-path:$PATH"
    export LDFLAGS="$LDFLAGS -Wl,-rpath,/usr/lib/xulrunner-$pkgver"
    export PYTHON="/usr/bin/python2"

    cd "$srcdir/$_xulsrcdir"
    make -f client.mk build MOZ_MAKE_FLAGS="$MAKEFLAGS"

    msg "Building extension..."

    cd "${srcdir}/${_subdir}"
    
    mkdir -p "$srcdir/$_xuldir/lib/pkgconfig"
    cat <<END > "$srcdir/$_xuldir/lib/pkgconfig/libxul.pc"
prefix=$srcdir/$_xuldir/dist

Name: libxul
Description: The Mozilla Runtime and Embedding Engine
Version: $pkgver 
Requires: nspr
Libs: -L\${prefix}/lib -lxpcomglue_s -lxul -lmozalloc
Cflags: -I\${prefix}/include
 
END

    export PKG_CONFIG_PATH="$srcdir/$_xuldir/lib/pkgconfig"
    make clean && make
}

package_firefox-gnome-keyring()
{
    cd ${srcdir}

    mkdir -p ${pkgdir}/usr/lib/firefox/browser/extensions
    unzip ${srcdir}/${_subdir}/mozilla-gnome-keyring-*.xpi -d ${pkgdir}/usr/lib/firefox/browser/extensions/{6f9d85e0-794d-11dd-ad8b-0800200c9a66}
}

