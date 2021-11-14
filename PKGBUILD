#!/bin/hint/bash
# Maintainer : bartus <arch-user-repoᘓbartus.33mail.com>
# Contributor: Randy Heydon <randy dot heydon at clockworklab dot net>
# Contributor: saxonbeta <saxonbeta at gmail __com

_fragment=${FRAGMENT:-#branch=devel}

# Configuration.
if (( DISABLE_TRILINOS ))
  then _CMAKE_FLAGS+=(-DWITH_Trilinos=OFF)
  else depends+=('trilinos'); _CMAKE_FLAGS+=(-DWITH_Trilinos=ON)
fi
_CMAKE_FLAGS+=(
        -DCMAKE_INSTALL_PREFIX=/usr \
        -DELMER_INSTALL_LIB_DIR=/usr/lib \
        -DWITH_CONTRIB=ON \
        -DWITH_ELMERGUI=ON \
        -DWITH_ELMERGUILOGGER=OFF \
        -DWITH_ELMERGUITESTER=OFF \
        -DWITH_ElmerIce=ON \
        -DPHDF5HL_LIBRARY=/usr/lib/libhdf5_hl.so \
        -DWITH_LUA=ON \
        -DWITH_MATC=OFF \
        -DWITH_MPI=ON \
        -DWITH_OpenMP=ON \
        -DWITH_QT5=ON \
        -DWITH_Mumps=ON \
        -DWITH_Hypre=ON \
        -DWITH_OCC=ON \
        -DWITH_VTK=ON \
        -DNN_INCLUDE_DIR=/usr/include \
        -DHYPRE_INCLUDE_DIR=/usr/include/hypre \
        -DWITH_ScatteredDataInterpolator=ON \
        -DWITH_PARAVIEW=ON
)

pkgname=elmerfem-git
_pkgname=elmerfem
pkgver=9.0.r423.gb943299d
pkgrel=1
pkgdesc="A finite element software for multiphysical problems"
arch=('x86_64')
url="http://www.elmerfem.org"
license=('GPL')
depends+=('arpack' 'blas' 'qt5-script' 'python-pyqt5' 'qwt' 'netcdf-fortran-openmpi' 'paraview-opt' 'mumps-par' 'oce' 'vtk' 'hypre' 'mmg' 'libnn-git' 'libcsa-git' 'scalapack')
depends+=('tbb' 'openmpi')
depends+=('freetype2' 'qt5-base' 'fmt' 'glew' 'pugixml' 'libxcursor')
makedepends=('git' 'gcc-fortran' 'cmake')
provides=('elmerfem')
conflicts=('elmerfem')
options=(!emptydirs !staticlibs)

source=("git+https://github.com/ElmerCSC/elmerfem.git${_fragment}"
        "$_pkgname.desktop"
        "arpack.patch"
        "mmg_properties_list.patch"
        "vtk9.cmake.patch"
        "vtk9.1.patch"
        "FindMMG.patch")

sha256sums=('SKIP'
            'f4b39389e5f258c7860b8d7a6b171fb54bf849dc772f640ac5e7a12c7a384aca'
            '04e73a99d7e8d501a2c7c5211a83257137a30a8b1b5c2f7c7ff6304e0e0a6da9'
            'ce9fc22cad15d8b7b33ca4d17b70e3bf946d50f847a60619aab06d971e8e22ad'
            '282c32e2e0bc82fbc96d3597a661c1b238114cba247a13f8dc9103a51f3e194b'
            'f995a09fb8999ccb965c7dfe1de602e99a630b84f6ca260b2bdfece1372aeceb'
            '40158498b071acd8a6ba2d760d3fabada26d989bdc931914f133009a607e6700')

pkgver() {
  git -C "${srcdir}/${_pkgname}" describe --long --tag| sed -r 's/^release-//;s/([^-]*-g)/r\1/;s/-/./g'
}

prepare() {
  cd "$srcdir/$_pkgname"
  sed -i 's/1 depth/1 ${depth}/g' fem/tests/CMakeLists.txt
  sed -i 's/FALSE/false/g' ElmerGUI/Application/vtkpost/matc.cpp
  git apply -v "${srcdir}"/{arpack,mmg_properties_list,vtk9{.cmake,.1}}.patch
  git apply -v "$srcdir"/FindMMG.patch

  sed 's/${MMG_LIBRARY}/Mmg::libmmg3d_so/g' -i fem/src/CMakeLists.txt
  sed 's/${MMG_LIBRARY}/Mmg::libmmg2d_so/g' -i elmerice/Solvers/MeshAdaptation_2D/CMakeLists.txt
}

build() {
  export FFLAGS+=" -fallow-argument-mismatch"
  cmake -S "${srcdir}"/$_pkgname -B build \
        "${_CMAKE_FLAGS[@]}"
  make -C build all
}

check() {
  cd "$srcdir/build"
  export PATH=$PATH:$PWD/fem/src
  ctest $(grep -oP -- "-j\s*[0-9]+" <<< "${MAKEFLAGS}") || true
}

package() {
  make -C build DESTDIR="$pkgdir" install
  cd "$pkgdir/usr"

  # Remove unecessary libraries
  rm -rf -- lib/{*.a,ElmerGUI}

  #Create directories
  install -dv share/applications
  install -dv share/pixmaps
  install -dv share/licenses/$_pkgname

  #Icon and desktop files
  install -D -m644 "$srcdir/$_pkgname/ElmerGUI/Application/images/logo.png" share/pixmaps/$_pkgname.png
  install -D -m644 "$srcdir/$_pkgname.desktop" share/applications

  #Clean up and move stuff in place
  cp share/ElmerGUI/edf-extra/* share/ElmerGUI/edf
  mv share/ElmerGUI/license_texts/GPL_EXCEPTION share/licenses/$_pkgname
  rm share/ElmerGUI/license_texts/*
}
# vim:set sw=2 ts=2 et:
