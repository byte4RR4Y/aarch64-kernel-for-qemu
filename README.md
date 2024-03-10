# Compiling a kernel for QEMU with graphics support
## This repository shows you how to cross-compile(on amd64) an arm64/aarch64 Kernel for QEMU with graphics support

### 1.) We install dependencies:
     apt update -y && apt install -y git bc bison flex libssl-dev make libc6-dev libncurses5-dev crossbuild-essential-arm64
### 2.) We clone the latest availible Kernel from https://github.com/torvalds/linux and change to its directory:
     git clone --depth=1 https://github.com/torvalds/linux
     cd linux
### 3.) We make a standard defconfig for arm64
     yes "" | make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- defconfig
### 4.) We configure it to support a graphical QEMU boot with:
     scripts/config -e CONFIG_MACVLAN
     scripts/config -e CONFIG_VIRTIO_NET
     scripts/config -e CONFIG_NLMON
     scripts/config -d CONFIG_VT_HW_CONSOLE_BINDING
     scripts/config -e CONFIG_SERIAL_AMBA_PL011
     scripts/config -e CONFIG_SERIAL_AMBA_PL011_CONSOLE
     scripts/config -e CONFIG_VIRTIO_CONSOLE
     scripts/config -e CONFIG_HW_RANDOM
     scripts/config -e CONFIG_HW_RANDOM_VIRTIO
     scripts/config -e CONFIG_DRM
     scripts/config -e CONFIG_DRM_VIRTIO_GPU
     scripts/config -e CONFIG_RTC_CLASS
     scripts/config -e CONFIG_RTC_DRV_PL031
     scripts/config -e CONFIG_VIRTIO_PCI
     scripts/config -e CONFIG_VIRTIO_MMIO
     scripts/config -e CONFIG_VIRTIO_MMIO_CMDLINE_DEVICES
     scripts/config -e CONFIG_MAILBOX
     scripts/config -e CONFIG_AHCI
     scripts/config -e CONFIG_PCIEPORT
     scripts/config -e CONFIG_VIRTIO_INPUT
     scripts/config -e CONFIG_VIRT_DRIVERS
     scripts/config -e CONFIG_VIRTIO_MEM
### 5.) We cross-compile the Kernel for arm64 on a AMD64-Machine
     yes "" | make -j ${CPUS} ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- Image.gz

### This kernel supports QEMU's graphic card provided by:
     qemu-system-aarch64 -m virt \
       -kernel arch/arm64/boot/Image.gz \
       -append "root=vda1 rw" \
       -initrd YOUR_INITRD_IMAGE \
       -cpu cortex-a76 \
       -drive if=none,file=YOUR_ROOTFS_IMAGE,format=raw,id=disk
       -device virtio-blk-device,drive=disk \
       -device virtio-gpu-pci \
       -display gtk,gl=on,show-cursor=on \
       -device virtio-mouse-pci \
       -device virtio-keyboard-pci \
  
