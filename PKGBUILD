# Maintainer: Antonio Rojas < arojas@archlinux.org >
# Contributor: Thomas Dziedzic < gostrc at gmail >
# Contributor: Osman Ugus <ugus11@yahoo.com>
# Contributor: Stefan Husmann <stefan-husmann@t-online.de>
# Special thanks to Nareto for moving the compile from the .install to the PKGBUILD

# use external ATLAS, comment out to compile internal version (warning: very long)
# If you compile the internal version you should disable CPU freq scaling
# USE_EXTERNAL_ATLAS=1

# build the documentation by default, comment out to skip (saves ~300MB of disk space)
BUILD_DOCS=1

pkgname=sage-mathematics-devel
sagename=sage # change to allow installation alongside stable version
pkgver=6.5.beta1
pkgrel=1
pkgdesc='Open Source Mathematics Software, a viable free alternative to Magma, Maple, Mathematica, and Matlab. Development version'
url='http://www.sagemath.org/download-latest.html'
arch=('i686' 'x86_64')
license=('GPL')
conflicts=('sage-mathematics')
provides=('sage-mathematics')
depends=()
[[ $USE_EXTERNAL_ATLAS ]] && depends+=('atlas-lapack')
makedepends=('gcc-fortran' 'desktop-file-utils')
optdepends=('ffmpeg: to show animations' 'imagemagick: to show animations and some LaTeX output in the notebook'
  'texlive-core: to view LaTeX output in the notebook and to use SageTeX'
  'openssh: to use the notebook in secure mode')
install="$pkgname.install"
source=("http://sage.sagedev.org/home/release/sage-$pkgver.tar.gz"
  'sage-devel-notebook.desktop')
noextract=()
md5sums=('7b32e990c5b3fcd14a6d38058e6f7f0b'
         '14ecef9ebd59af84c05feeb2009faa8c')

build() {
  cd sage-$pkgver

  # unset CFLAGS
  # unset CXXFLAGS
  # unset LDFLAGS

  # don't mess with user's .sage dir during build
  export DOT_SAGE='/tmp/.sage'

  # parallel build
  export MAKE="make -j$(nproc)"

  # use archlinux's fortran
  export FC='/usr/bin/gfortran'

  # use external ATLAS
  [[ $USE_EXTERNAL_ATLAS ]] && export SAGE_ATLAS_LIB='/usr/lib'

  # disable building with debugging support
  export SAGE_DEBUG='no'

  # enable fat binaries (disables processor specific optimizations)
  # comment out if you're only building it for yourself
  # export SAGE_FAT_BINARY='yes'
  
  if [[ $BUILD_DOCS ]]; then
    make
  else
    make build
  fi
}

check() {
  cd sage-$pkgver

  make test || /bin/true

  # uncomment if you want to run all the tests (warning: very long)
  # make testlong || /bin/true
}

package() {
  cd sage-$pkgver

  # remove logs and source packages
  rm -fr logs tmp upstream

  # make install is experimental
  # make install DESTDIR=$pkgdir/opt/  
  install -d $pkgdir/opt/$sagename
  cp -r * $pkgdir/opt/$sagename

  # install .desktop file
  desktop-file-install $srcdir/sage-devel-notebook.desktop --dir $pkgdir/usr/share/applications 

  # install scripts
  install -d $pkgdir/usr/bin
  ./sage -c "install_scripts('$pkgdir/usr/bin', ignore_existing=True)"

  # rename scripts to avoid conflicts
  for _i in $(ls $pkgdir/usr/bin); do
    mv $pkgdir/usr/bin/$_i $pkgdir/usr/bin/$sagename-$_i
  done

  # create link to main binary
  ln -s /opt/$sagename/sage $pkgdir/usr/bin/$sagename

  # copy license file to its proper location (keep it in /opt/sage so that license() works)
  install -d $pkgdir/usr/share/licenses/sage-mathematics-devel
  cp $pkgdir/opt/$sagename/COPYING.txt $pkgdir/usr/share/licenses/sage-mathematics-devel

  # move SageTeX files to a more appropriate directory (will conflict with sage-mathematics)
  mv $pkgdir/opt/$sagename/local/share/texmf $pkgdir/usr/share

  # remove build logs
  rm -r $pkgdir/opt/$sagename/spkg/logs
}
