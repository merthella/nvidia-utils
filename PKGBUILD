# Maintainer: Sven-Hendrik Haase <svenstaro@archlinux.org>
# Maintainer: Peter Jung <ptr1337@archlinux.org>
# Contributor: James Rayner <iphitus@gmail.com>
# Contributor: Vasiliy Stelmachenok <ventureo@yandex.ru>
# Contributor: Thomas Baechler <thomas@archlinux.org>

pkgbase=nvidia-utils
pkgname=('nvidia-utils' 'opencl-nvidia' 'nvidia-open-dkms')  # "nvidia-dkms" yerine "nvidia-open-dkms" kullan
pkgver=570.124.04
pkgrel=1
arch=('x86_64')
url="http://www.nvidia.com/"
license=('custom')
options=('!strip')
makedepends=('patchelf')
_pkg="NVIDIA-Linux-x86_64-${pkgver}"
source=('nvidia-drm-outputclass.conf'
        'nvidia-utils.sysusers'
        'nvidia.rules'
        'systemd-homed-override.conf'
        'systemd-suspend-override.conf'
        'nvidia-sleep.conf'
        "https://us.download.nvidia.com/XFree86/Linux-x86_64/${pkgver}/${_pkg}.run"
        "$pkgname-$pkgver.tar.gz::https://github.com/merthella/open-gpu-kernel-modules/archive/refs/tags/${pkgver}.tar.gz"
        0001-Enable-atomic-kernel-modesetting-by-default.patch
        0003-Add-IBT-support.patch)

sha512sums=('de7116c09f282a27920a1382df84aa86f559e537664bb30689605177ce37dc5067748acf9afd66a3269a6e323461356592fdfc624c86523bf105ff8fe47d3770'
            '1bcf2c6ee71686c0d32625e746ec8c0f7cf42fc63c76c3076ff2526b2661e8b9e9f76eaa2c4b213c7cc437a6f06006cc07672c4974d7f4515b2de2fd7c47a891'
            'f8f071f5a46c1a5ce5188e104b017808d752e61c0c20de1466feb5d693c0b55a5586314411e78cc2ab9c0e16e2c67afdd358da94c0c75df1f8233f54c280762c'
            'a0183adce78e40853edf7e6b73867e7a8ea5dabac8e8164e42781f64d5232fbe869f850ab0697c3718ebced5cde760d0e807c05da50a982071dfe1157c31d6b8'
            '55def6319f6abb1a4ccd28a89cd60f1933d155c10ba775b8dfa60a2dc5696b4b472c14b252dc0891f956e70264be87c3d5d4271e929a4fc4b1a68a6902814cee'
            'c7fea39d11565f05a507d3aded4e9ea506ef9dbebf313e0fc8d6ebc526af3f9d6dec78af9d6c4456c056310f98911c638706bccdd9926d07f492615569430455'
            '35fb1938a08586d902eb290aee04c5c6b570cc647354e49880e3c8715654ac1ab8c8cbe08e5b3c542fd9b2c36f7683454221b11625754980a6138a58c6363b40'
            '7a1b66770b62b784650bb62b9f4543ab873b95b5c42bf5e688eb7a5a6133322b143f61bb34cbd7fd112030069f0f3058946e31c38f883d3e1c8bb31033eb464b'
            '0bb89b9037f0baa9aae1ff8e70c9c93896f03fd0cc380eea4b0dc094a6991c3ad6738c9fbbaa42d8b5a544f77dc91c0e6401b1501c5970c576d5efbc0de8dd34'
            '42f621179d4fd9bf608f0d84b9019f5a5fdf5d92d68d22ce9b9a9add1cad1c90dcb3764db68e0b9bc7e902bb6b955c59563ea6d4f39f2e39a340387e4d5deb82')


create_links() {
    # create soname links
    find "$pkgdir" -type f -name '*.so*' ! -path '*xorg/*' -print0 | while read -d $'\0' _lib; do
        _soname=$(dirname "${_lib}")/$(readelf -d "${_lib}" | grep -Po 'SONAME.*: \[\K[^]]*' || true)
        _base=$(echo ${_soname} | sed -r 's/(.*)\.so.*/\1.so/')
        [[ -e "${_soname}" ]] || ln -s $(basename "${_lib}") "${_soname}"
        [[ -e "${_base}" ]] || ln -s $(basename "${_soname}") "${_base}"
    done
}

prepare() {
    sh "${_pkg}.run" --extract-only
    cd "${_pkg}"
    bsdtar -xf nvidia-persistenced-init.tar.bz2

    # Enable modeset by default
    # This avoids various issue, when Simplefb is used
    # https://gitlab.archlinux.org/archlinux/packaging/packages/nvidia-utils/-/issues/14
    # https://github.com/rpmfusion/nvidia-kmod/blob/master/make_modeset_default.patch
    patch -Np1 < "$srcdir"/0001-Enable-atomic-kernel-modesetting-by-default.patch -d "${srcdir}/${_pkg}/kernel"

    cd kernel

    sed -i "s/__VERSION_STRING/${pkgver}/" dkms.conf
    sed -i 's/__JOBS/`nproc`/' dkms.conf
    sed -i 's/__DKMS_MODULES//' dkms.conf
    sed -i '$iBUILT_MODULE_NAME[0]="nvidia"\
DEST_MODULE_LOCATION[0]="/kernel/drivers/video"\
BUILT_MODULE_NAME[1]="nvidia-uvm"\
DEST_MODULE_LOCATION[1]="/kernel/drivers/video"\
BUILT_MODULE_NAME[2]="nvidia-modeset"\
DEST_MODULE_LOCATION[2]="/kernel/drivers/video"\
BUILT_MODULE_NAME[3]="nvidia-drm"\
DEST_MODULE_LOCATION[3]="/kernel/drivers/video"\
BUILT_MODULE_NAME[4]="nvidia-peermem"\
DEST_MODULE_LOCATION[4]="/kernel/drivers/video"' dkms.conf

    # Gift for linux-rt guys
    sed -i 's/NV_EXCLUDE_BUILD_MODULES/IGNORE_PREEMPT_RT_PRESENCE=1 NV_EXCLUDE_BUILD_MODULES/' dkms.conf

    cd "$srcdir"/open-gpu-kernel-modules-${pkgver}

    # Enable modeset and fbdev as default
    # This avoids various issue, when Simplefb is used
    # https://gitlab.archlinux.org/archlinux/packaging/packages/nvidia-utils/-/issues/14
    # https://github.com/rpmfusion/nvidia-kmod/blob/master/make_modeset_default.patch
    patch -Np1 < "$srcdir"/0001-Enable-atomic-kernel-modesetting-by-default.patch -d "${srcdir}/open-gpu-kernel-modules-${pkgver}/kernel-open"

    # Fix for https://bugs.archlinux.org/task/74886
    patch -Np1 --no-backup-if-mismatch -i "$srcdir"/0003-Add-IBT-support.patch

    # Attempt to make this reproducible
    sed -i "s/^HOSTNAME.*/HOSTNAME = echo archlinux"/ utils.mk
    sed -i "s/^WHOAMI.*/WHOAMI = echo archlinux-builder"/ utils.mk
    sed -i "s/^DATE.*/DATE = date -r version.mk"/ utils.mk

    sed -i "s/__VERSION_STRING/${pkgver}/" kernel-open/dkms.conf
    sed -i 's/__JOBS/`nproc`/' kernel-open/dkms.conf
    sed -i 's/__EXCLUDE_MODULES//' kernel-open/dkms.conf
    sed -i 's/__DKMS_MODULES//' kernel-open/dkms.conf
    sed -i '$i\
BUILT_MODULE_NAME[0]="nvidia"\
BUILT_MODULE_LOCATION[0]="kernel-open"\
DEST_MODULE_LOCATION[0]="/kernel/drivers/video"\
BUILT_MODULE_NAME[1]="nvidia-uvm"\
BUILT_MODULE_LOCATION[1]="kernel-open"\
DEST_MODULE_LOCATION[1]="/kernel/drivers/video"\
BUILT_MODULE_NAME[2]="nvidia-modeset"\
BUILT_MODULE_LOCATION[2]="kernel-open"\
DEST_MODULE_LOCATION[2]="/kernel/drivers/video"\
BUILT_MODULE_NAME[3]="nvidia-drm"\
BUILT_MODULE_LOCATION[3]="kernel-open"\
DEST_MODULE_LOCATION[3]="/kernel/drivers/video"\
BUILT_MODULE_NAME[4]="nvidia-peermem"\
BUILT_MODULE_LOCATION[4]="kernel-open"\
DEST_MODULE_LOCATION[4]="/kernel/drivers/video"' kernel-open/dkms.conf

    # Gift for linux-rt guys
    sed -i 's/NV_EXCLUDE_BUILD_MODULES/IGNORE_PREEMPT_RT_PRESENCE=1 NV_EXCLUDE_BUILD_MODULES/' kernel-open/dkms.conf

    # Clean version for later copying for DKMS
    cp -r ../open-gpu-kernel-modules-${pkgver} "$srcdir"/open-gpu-kernel-modules-dkms
}

package_opencl-nvidia() {
    pkgdesc="OpenCL implemention for NVIDIA"
    depends=('zlib')
    optdepends=('opencl-headers: headers necessary for OpenCL development')
    provides=('opencl-driver')
    cd "${_pkg}"

    # OpenCL
    install -Dm644 nvidia.icd "${pkgdir}/etc/OpenCL/vendors/nvidia.icd"
    install -Dm755 "libnvidia-opencl.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-opencl.so.${pkgver}"

    create_links

    mkdir -p "${pkgdir}/usr/share/licenses"
    ln -s nvidia-utils "${pkgdir}/usr/share/licenses/opencl-nvidia"
}

package_nvidia-dkms() {
    pkgdesc="NVIDIA kernel modules - module sources"
    depends=('dkms' "nvidia-utils=$pkgver" 'libglvnd')
    provides=('NVIDIA-MODULE' 'nvidia')
    conflicts=('NVIDIA-MODULE' 'nvidia')

    cd ${_pkg}

    install -dm 755 "${pkgdir}"/usr/src
    cp -dr --no-preserve='ownership' kernel "${pkgdir}/usr/src/nvidia-${pkgver}"

    install -Dm644 "${srcdir}/${_pkg}/LICENSE" "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}

package_nvidia-utils() {
    pkgdesc="NVIDIA drivers utilities"
    depends=('libglvnd' 'egl-wayland' 'egl-gbm' 'egl-x11')
    optdepends=('nvidia-settings: configuration tool'
                'xorg-server: Xorg support'
                'xorg-server-devel: nvidia-xconfig'
                'opencl-nvidia: OpenCL support')
    conflicts=('nvidia-libgl')
    provides=('vulkan-driver' 'opengl-driver' 'nvidia-libgl')
    replaces=('nvidia-libgl')
    install="${pkgname}.install"

    cd "${_pkg}"

    # Check http://us.download.nvidia.com/XFree86/Linux-x86_64/${pkgver}/README/installedcomponents.html
    # for hints on what needs to be installed where.

    # X driver
    install -Dm755 nvidia_drv.so "${pkgdir}/usr/lib/xorg/modules/drivers/nvidia_drv.so"

    # Wayland/GBM
    mkdir -p "${pkgdir}/usr/lib/gbm"
    ln -sr "${pkgdir}/usr/lib/libnvidia-allocator.so.${pkgver}" "${pkgdir}/usr/lib/gbm/nvidia-drm_gbm.so"

    # firmware
    install -Dm644 -t "${pkgdir}/usr/lib/firmware/nvidia/${pkgver}/" firmware/*.bin

    # GLX extension module for X
    install -Dm755 "libglxserver_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/nvidia/xorg/libglxserver_nvidia.so.${pkgver}"
    # Ensure that X finds glx
    ln -s "libglxserver_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/nvidia/xorg/libglxserver_nvidia.so.1"
    ln -s "libglxserver_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/nvidia/xorg/libglxserver_nvidia.so"

    install -Dm755 "libGLX_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/libGLX_nvidia.so.${pkgver}"

    # OpenGL libraries
    install -Dm755 "libEGL_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/libEGL_nvidia.so.${pkgver}"
    install -Dm755 "libGLESv1_CM_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/libGLESv1_CM_nvidia.so.${pkgver}"
    install -Dm755 "libGLESv2_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/libGLESv2_nvidia.so.${pkgver}"
    install -Dm644 "10_nvidia.json" "${pkgdir}/usr/share/glvnd/egl_vendor.d/10_nvidia.json"

    # OpenGL core library
    install -Dm755 "libnvidia-glcore.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-glcore.so.${pkgver}"
    install -Dm755 "libnvidia-eglcore.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-eglcore.so.${pkgver}"
    install -Dm755 "libnvidia-glsi.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-glsi.so.${pkgver}"

    # misc
    install -Dm755 "libnvidia-api.so.1" "${pkgdir}/usr/lib/libnvidia-api.so.1"
    install -Dm755 "libnvidia-fbc.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-fbc.so.${pkgver}"
    install -Dm755 "libnvidia-encode.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-encode.so.${pkgver}"
    install -Dm755 "libnvidia-cfg.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-cfg.so.${pkgver}"
    install -Dm755 "libnvidia-ml.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-ml.so.${pkgver}"
    install -Dm755 "libnvidia-glvkspirv.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-glvkspirv.so.${pkgver}"
    install -Dm755 "libnvidia-allocator.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-allocator.so.${pkgver}"
    install -Dm755 "libnvidia-gpucomp.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-gpucomp.so.${pkgver}"

    # Vulkan ICD
    install -Dm644 "nvidia_icd.json" "${pkgdir}/usr/share/vulkan/icd.d/nvidia_icd.json"
    install -Dm644 "nvidia_layers.json" "${pkgdir}/usr/share/vulkan/implicit_layer.d/nvidia_layers.json"

    # VulkanSC
    install -D -m755 nvidia-pcc -t "${pkgdir}/usr/bin"
    install -D -m755 "libnvidia-vksc-core.so.${pkgver}" -t "${pkgdir}/usr/lib"
    install -D -m644 nvidia_icd_vksc.json -t "${pkgdir}/usr/share/vulkansc/icd.d"

    # VDPAU
    install -Dm755 "libvdpau_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/vdpau/libvdpau_nvidia.so.${pkgver}"

    # nvidia-tls library
    install -Dm755 "libnvidia-tls.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-tls.so.${pkgver}"

    # CUDA
    install -Dm755 "libcuda.so.${pkgver}" "${pkgdir}/usr/lib/libcuda.so.${pkgver}"
    install -Dm755 "libnvcuvid.so.${pkgver}" "${pkgdir}/usr/lib/libnvcuvid.so.${pkgver}"
    install -Dm755 "libcudadebugger.so.${pkgver}" "${pkgdir}/usr/lib/libcudadebugger.so.${pkgver}"

    # NVVM Compiler library loaded by the CUDA driver to do JIT link-time-optimization
    install -Dm644 "libnvidia-nvvm.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-nvvm.so.${pkgver}"

    # PTX JIT Compiler (Parallel Thread Execution (PTX) is a pseudo-assembly language for CUDA)
    install -Dm755 "libnvidia-ptxjitcompiler.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-ptxjitcompiler.so.${pkgver}"

    # raytracing
    install -Dm755 "nvoptix.bin" "${pkgdir}/usr/share/nvidia/nvoptix.bin"
    install -Dm755 "libnvoptix.so.${pkgver}" "${pkgdir}/usr/lib/libnvoptix.so.${pkgver}"
    install -Dm755 "libnvidia-rtcore.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-rtcore.so.${pkgver}"

    # NGX
    install -Dm755 nvidia-ngx-updater "${pkgdir}/usr/bin/nvidia-ngx-updater"
    install -Dm755 "libnvidia-ngx.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-ngx.so.${pkgver}"
    install -Dm755 _nvngx.dll "${pkgdir}/usr/lib/nvidia/wine/_nvngx.dll"
    install -Dm755 nvngx.dll "${pkgdir}/usr/lib/nvidia/wine/nvngx.dll"
    install -Dm755 nvngx_dlssg.dll "${pkgdir}/usr/lib/nvidia/wine/nvngx_dlssg.dll"

    # Optical flow
    install -Dm755 "libnvidia-opticalflow.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-opticalflow.so.${pkgver}"

    # Cryptography library wrapper
    ls libnvidia-pkcs*
    ls *openssl*
    install -Dm755 "libnvidia-pkcs11.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-pkcs11.so.${pkgver}"
    install -Dm755 "libnvidia-pkcs11-openssl3.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-pkcs11-openssl3.so.${pkgver}"

    # Sandboxhelper
    install -Dm755 "libnvidia-sandboxutils.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-sandboxutils.so.${pkgver}"

    # Debug
    install -Dm755 nvidia-debugdump "${pkgdir}/usr/bin/nvidia-debugdump"

    # nvidia-xconfig
    install -Dm755 nvidia-xconfig "${pkgdir}/usr/bin/nvidia-xconfig"
    install -Dm644 nvidia-xconfig.1.gz "${pkgdir}/usr/share/man/man1/nvidia-xconfig.1.gz"

    # nvidia-bug-report
    install -Dm755 nvidia-bug-report.sh "${pkgdir}/usr/bin/nvidia-bug-report.sh"

    # nvidia-smi
    install -Dm755 nvidia-smi "${pkgdir}/usr/bin/nvidia-smi"
    install -Dm644 nvidia-smi.1.gz "${pkgdir}/usr/share/man/man1/nvidia-smi.1.gz"

    # nvidia-cuda-mps
    install -Dm755 nvidia-cuda-mps-server "${pkgdir}/usr/bin/nvidia-cuda-mps-server"
    install -Dm755 nvidia-cuda-mps-control "${pkgdir}/usr/bin/nvidia-cuda-mps-control"
    install -Dm644 nvidia-cuda-mps-control.1.gz "${pkgdir}/usr/share/man/man1/nvidia-cuda-mps-control.1.gz"

    # nvidia-modprobe
    # This should be removed if nvidia fixed their uvm module!
    install -Dm4755 nvidia-modprobe "${pkgdir}/usr/bin/nvidia-modprobe"
    install -Dm644 nvidia-modprobe.1.gz "${pkgdir}/usr/share/man/man1/nvidia-modprobe.1.gz"

    # nvidia-persistenced
    install -Dm755 nvidia-persistenced "${pkgdir}/usr/bin/nvidia-persistenced"
    install -Dm644 nvidia-persistenced.1.gz "${pkgdir}/usr/share/man/man1/nvidia-persistenced.1.gz"
    install -Dm644 nvidia-persistenced-init/systemd/nvidia-persistenced.service.template "${pkgdir}/usr/lib/systemd/system/nvidia-persistenced.service"
    sed -i 's/__USER__/nvidia-persistenced/' "${pkgdir}/usr/lib/systemd/system/nvidia-persistenced.service"

    # application profiles
    install -Dm644 nvidia-application-profiles-${pkgver}-rc "${pkgdir}/usr/share/nvidia/nvidia-application-profiles-${pkgver}-rc"
    install -Dm644 nvidia-application-profiles-${pkgver}-key-documentation "${pkgdir}/usr/share/nvidia/nvidia-application-profiles-${pkgver}-key-documentation"

    install -Dm644 LICENSE "${pkgdir}/usr/share/licenses/nvidia-utils/LICENSE"
    install -Dm644 README.txt "${pkgdir}/usr/share/doc/nvidia/README"
    install -Dm644 NVIDIA_Changelog "${pkgdir}/usr/share/doc/nvidia/NVIDIA_Changelog"
    cp -r html "${pkgdir}/usr/share/doc/nvidia/"
    ln -s nvidia "${pkgdir}/usr/share/doc/nvidia-utils"

    # new power management support
    install -Dm644 systemd/system/*.service -t "${pkgdir}/usr/lib/systemd/system"
    install -Dm755 systemd/system-sleep/nvidia "${pkgdir}/usr/lib/systemd/system-sleep/nvidia"
    install -Dm755 systemd/nvidia-sleep.sh "${pkgdir}/usr/bin/nvidia-sleep.sh"
    install -Dm755 nvidia-powerd "${pkgdir}/usr/bin/nvidia-powerd"
    install -Dm644 nvidia-dbus.conf "${pkgdir}"/usr/share/dbus-1/system.d/nvidia-dbus.conf
    install -Dm644 "${srcdir}"/systemd-homed-override.conf "${pkgdir}"/usr/lib/systemd/system/systemd-homed.service.d/10-nvidia-no-freeze-session.conf
    install -Dm644 "${srcdir}"/systemd-suspend-override.conf "${pkgdir}"/usr/lib/systemd/system/systemd-suspend.service.d/10-nvidia-no-freeze-session.conf
    install -Dm644 "${srcdir}"/systemd-suspend-override.conf "${pkgdir}"/usr/lib/systemd/system/systemd-suspend-then-hibernate.service.d/10-nvidia-no-freeze-session.conf
    install -Dm644 "${srcdir}"/systemd-suspend-override.conf "${pkgdir}"/usr/lib/systemd/system/systemd-hibernate.service.d/10-nvidia-no-freeze-session.conf
    install -Dm644 "${srcdir}"/systemd-suspend-override.conf "${pkgdir}"/usr/lib/systemd/system/systemd-hybrid-sleep.service.d/10-nvidia-no-freeze-session.conf

    # distro specific files must be installed in /usr/share/X11/xorg.conf.d
    install -Dm644 "${srcdir}/nvidia-drm-outputclass.conf" "${pkgdir}/usr/share/X11/xorg.conf.d/10-nvidia-drm-outputclass.conf"

    install -Dm644 "${srcdir}/nvidia-utils.sysusers" "${pkgdir}/usr/lib/sysusers.d/$pkgname.conf"

    install -Dm644 "${srcdir}/nvidia.rules" "$pkgdir"/usr/lib/udev/rules.d/60-nvidia.rules

    echo "blacklist nouveau" | install -Dm644 /dev/stdin "${pkgdir}/usr/lib/modprobe.d/${pkgname}.conf"
    echo "nvidia-uvm" | install -Dm644 /dev/stdin "${pkgdir}/usr/lib/modules-load.d/${pkgname}.conf"

    # Enable PreserveVideoMemoryAllocations and TemporaryFilePath
    # Fixes Wayland Sleep, when restoring the session
    install -Dm644 "${srcdir}/nvidia-sleep.conf" "${pkgdir}/usr/lib/modprobe.d/nvidia-sleep.conf"

    # Lists NVIDIA driver files for container runtimes like nvidia-container-toolkit
    install -Dm644 sandboxutils-filelist.json "${pkgdir}/usr/share/nvidia/files.d/sandboxutils-filelist.json"

    create_links
}

package_nvidia-open-dkms() {
  pkgdesc="NVIDIA open kernel modules - module sources"
  depends+=('dkms')
  license=('MIT AND GPL-2.0-only')
  conflicts=('nvidia-open' 'NVIDIA-MODULE')
  provides=('nvidia-open' 'NVIDIA-MODULE')

  install -dm 755 "${pkgdir}"/usr/src
  # cp -dr --no-preserve='ownership' kernel-open "${pkgdir}/usr/src/nvidia-$pkgver"
  cp -dr --no-preserve='ownership' open-gpu-kernel-modules-dkms "${pkgdir}/usr/src/nvidia-$pkgver"
  mv "${pkgdir}/usr/src/nvidia-$pkgver/kernel-open/dkms.conf" "${pkgdir}/usr/src/nvidia-$pkgver/dkms.conf"

  install -Dm644 open-gpu-kernel-modules-${pkgver}/COPYING "$pkgdir"/usr/share/licenses/${pkgname}/LICENSE
}
