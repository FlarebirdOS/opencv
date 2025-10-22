pkgname=(
    opencv
    opencv-samples
    python-opencv
    # opencv-cuda
    # python-opencv-cuda
)
pkgbase=opencv
pkgver=4.12.0
pkgrel=1
pkgdesc="Open Source Computer Vision Library"
arch=('x86_64')
url="https://opencv.org"
license=('Apache-2.0')
depends=(
    'abseil-cpp'
    'cblas'
    'ffmpeg'
    'freetype2'
    'gcc-libs'
    'glib2'
    'glibc'
    'gst-plugins-base'
    'gst-plugins-base-libs'
    'gstreamer'
    'harfbuzz'
    'lapack'
    'libdc1394'
    'libglvnd'
    'libjpeg-turbo'
    'libjxl'
    'libpng'
    'libtiff'
    'libwebp'
    'onetbb'
    'openexr'
    'openjpeg'
    'protobuf'
    'verdict'
    'zlib'
)
makedepends=(
    'cmake'
    'cuda'
    'cudnn'
    'eigen'
    'fmt'
    'git'
    'glew'
    'hdf5'
    'lapacke'
    'mesa'
    'nlohmann-json'
    'openmpi'
    'pugixml'
    'python-numpy'
    'python-setuptools'
    'qt6-qt5compat'
    'vtk'
)
options=('!lto')
source=(git+ssh://git@github.com/opencv/opencv#tag=${pkgver}
    git+ssh://git@github.com/opencv/opencv_contrib#tag=${pkgver}
    vtk9.patch
    fix-cuda-flags.patch
    opencv-4.12.0-update_proxy.patch
    opencv_contrib-4.12.0-update_proxy.patch)
sha256sums=(84e35af82626500e922560ea7d04aeea1dfc86e0dc53445fe748325c76e511f8
    97f1ee43d9177b2da4bcc5cccbb962cf5a8a0cab04c2a304cebefda6b8b34595
    f35a2d4ea0d6212c7798659e59eda2cb0b5bc858360f7ce9c696c77d3029668e
    95472ecfc2693c606f0dd50be2f012b4d683b7b0a313f51484da4537ab8b2bfe
    6b23509f51896caea0bc9f4e68c444bf01c69d4666edb605a26ffc4e3d205784
    e8964df7475ef9c942b46a0c38e094e8d310bec7200fdff41f6f2f1425ff993e)

prepare() {
    pushd ${pkgbase}

        git apply -3 ${srcdir}/${pkgname}-${pkgver}-update_proxy.patch

        # Fix configuring with CMake version 4
        git cherry-pick -n cb8030809e0278d02fa335cc1f5ec7c3c17548e0

        # Support CUDA 13
        git cherry-pick -n df23b593c52e0e18095a3d2e772fe13083e2ebe7 \
                           42f89347005dcd14a671d30a52a046e45303833b \
                           3278820f5df20a6d4170b7374a9b396250e36641

        # Support Eigen 5
        git cherry-pick -n 353b4ddf52db48ba85d2efaa33310afa0eb73a72

        # Support FFmpeg 8
        git cherry-pick -n 90c444abd387ffa70b2e72a34922903a2f0f4f5a

        # Don't require all vtk optdepends
        patch -p1 < ${srcdir}/vtk9.patch

        # https://github.com/opencv/opencv/issues/27223
        # https://bugreports.qt.io/browse/QTBUG-134774
        sed -i 's/add_definitions(${Qt${QT_VERSION_MAJOR}${dt_dep}_DEFINITIONS})/link_libraries(${Qt${QT_VERSION_MAJOR}${dt_dep}})/' modules/highgui/CMakeLists.txt

        # OpenCV passes all CXXFLAGS to nvcc through -Xcompiler, which does not work for '-Wp,something' flags
        # We remove the -Xcompiler and pass our CXXFLAGS through cmake's CUDAFLAGS
        patch -p1 < ${srcdir}/fix-cuda-flags.patch
    popd

    pushd opencv_contrib
        git apply -3 ${srcdir}/opencv_contrib-${pkgver}-update_proxy.patch

        # Support CUDA 13
        git cherry-pick -n 9a9b173cd178e7c07a98896a009c2a2021a6b247
    popd
}

build() {
    cd ${pkgbase}

    local cmake_args=(
        -D CMAKE_BUILD_TYPE=Release
        -D CMAKE_INSTALL_PREFIX=/usr
        -D CMAKE_INSTALL_LIBDIR=lib64
        -D WITH_OPENCL=ON
        -D WITH_OPENGL=ON
        -D OpenGL_GL_PREFERENCE=LEGACY
        -D CMAKE_CXX_STANDARD=17
        -D WITH_TBB=ON
        -D WITH_VULKAN=ON
        -D WITH_QT=ON
        -D WITH_JPEGXL=ON
        -D BUILD_TESTS=OFF
        -D BUILD_PERF_TESTS=OFF
        -D BUILD_EXAMPLES=ON
        -D BUILD_PROTOBUF=OFF
        -D PROTOBUF_UPDATE_FILES=ON
        -D INSTALL_C_EXAMPLES=ON
        -D INSTALL_PYTHON_EXAMPLES=ON
        -D CMAKE_INSTALL_PREFIX=/usr
        -D CPU_BASELINE_DISABLE=SSE3
        -D CPU_BASELINE_REQUIRE=SSE2
        -D OPENCV_EXTRA_MODULES_PATH=${srcdir}/opencv_contrib/modules
        -D OPENCV_SKIP_PYTHON_LOADER=ON
        # cmake's FindLAPACK doesn't add cblas to LAPACK_LIBRARIES, so we need to specify them manually
        -D LAPACK_LIBRARIES="/usr/lib64/liblapack.so;/usr/lib64/libblas.so;/usr/lib64/libcblas.so"
        -D LAPACK_CBLAS_H=/usr/include/cblas.h
        -D LAPACK_LAPACKE_H=/usr/include/lapacke.h
        -D OPENCV_GENERATE_PKGCONFIG=ON
        -D OPENCV_ENABLE_NONFREE=ON
        -D OPENCV_GENERATE_SETUPVARS=OFF
        -D EIGEN_INCLUDE_PATH=/usr/include/eigen3
        -D protobuf_MODULE_COMPATIBLE=ON
        -D HDF5_NO_FIND_PACKAGE_CONFIG_FILE=ON
        -D BUILD_opencv_java=OFF
        -D BUILD_opencv_java_bindings_generator=OFF
        -D PYTHON3_PACKAGES_PATH=$(python3 -c "import sysconfig; print(sysconfig.get_paths()['purelib'])")
    )

    cmake -B flarebird-build            \
          -D BUILD_WITH_DEBUG_INFO=ON   \
          "${cmake_args[@]}"

    cmake --build flarebird-build

#     # In general, we want to list all real archs (sm_XX) and the latest virtual arch (compute_XX) for future PTX compatibility.
#     # Valid values can be discovered from nvcc --help
#     local cuda_archs="75;80;86;87;88;89;90;100;103;110;120;121;121-virtual"
#
#     # Avoid nvcc intercepting -Werror=format-security: Value 'format-security' is not defined for option 'Werror'
#     CUDAFLAGS="${CXXFLAGS/-Werror=format-security/-Xcompiler -Werror=format-security} -fno-lto --threads 0" \
#     CFLAGS="${CFLAGS} -fno-lto" CXXFLAGS="${CXXFLAGS} -fno-lto" LDFLAGS="${LDFLAGS} -fno-lto" \
#     cmake -B flarebird-build-cuda                       \
#           -D BUILD_WITH_DEBUG_INFO=OFF                  \
#           -D WITH_CUDA=ON                               \
#           -D WITH_CUDNN=ON                              \
#           -D ENABLE_CUDA_FIRST_CLASS_LANGUAGE=ON        \
#           -D CMAKE_CUDA_ARCHITECTURES="${cuda_archs}"   \
#           "${cmake_args[@]}"
#
#     cmake --build flarebird-build-cuda --verbose

}

package_opencv() {
    cd ${pkgbase}

    DESTDIR=${pkgdir} cmake --install flarebird-build

    # separate samples package
    _pick samples ${pkgdir}/usr/share/opencv4/samples

    # Split Python bindings
    rm -r ${pkgdir}/usr/lib64/python3*
}

package_opencv-samples() {
    pkgdesc+=" (samples)"
    depends=('opencv')

    mv samples/* ${pkgdir}
}

package_python-opencv() {
    pkgdesc='Python bindings for OpenCV'
    depends=(
        'fmt'
        'glew'
        'hdf5'
        'jsoncpp'
        'opencv'
        'openmpi'
        'pugixml'
        'python-numpy'
        'qt6-qtbase'
        'vtk'
    )

    cd ${pkgbase}

    DESTDIR=${pkgdir} cmake --install flarebird-build/modules/python3
}

package_opencv-cuda() {
    pkgdesc+=" (with CUDA support)"
    depends+=('cudnn')

    cd ${pkgbase}

    DESTDIR=${pkgdir} cmake --install flarebird-build-cuda

    # Split samples
    rm -r ${pkgdir}/usr/share/opencv4/samples

    # Add java symlinks expected by some binary blobs
    ln -sr ${pkgdir}/usr/share/java/{opencv4/opencv-${pkgver//./},opencv}.jar
    ln -sr ${pkgdir}/usr/lib64/{libopencv_java${pkgver//./},libopencv_java}.so

    # Split Python bindings
    rm -r ${pkgdir}/usr/lib64/python3*
}

package_python-opencv-cuda() {
    pkgdesc="Python bindings for OpenCV (with CUDA support)"
    depends=(
        'fmt'
        'glew'
        'hdf5'
        'jsoncpp'
        'opencv-cuda'
        'openmpi'
        'pugixml'
        'python-numpy'
        'qt6-qtbase'
        'vtk'
    )

    cd ${pkgbase}

    DESTDIR=${pkgdir} cmake --install flarebird-build-cuda/modules/python3
}
