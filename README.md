# Rockchip Debian SDK

Below is the instructions of how to build image for ROCK Pi 4.

## Get the source code

You need Git to get multiple git repositories to build the image.

Install Git if you don't have it.

    sudo apt-get update
    sudo apt-get install git

Clone the source code

    git clone --recursive https://github.com/radxa/rockchip-bsp.git

You will get 

    build  kernel  README.md  rkbin  rootfs  u-boot

## Install toolchain and other build tools

    sudo apt-get install gcc-aarch64-linux-gnu device-tree-compiler libncurses5 libncurses5-dev build-essential libssl-dev

## Build u-boot

    ./build/mk-uboot.sh rockpi4b     #For ROCK Pi 4 Mode B

The generated images will be copied to out/u-boot folder

    ls out/u-boot/
    idbloader.img  rk3399_loader_v1.12.112.bin  trust.img  uboot.img

## Build kernel

    ./build/mk-kernel.sh rockpi4b    #For ROCK Pi 4 Mode B

You will get the kernel image and dtb file

    ls out/kernel/
    Image  rockpi-4b-linux.dtb

## Make rootfs image

Building a base debian system by ubuntu-build-service from linaro.

    sudo apt-get install binfmt-support qemu-user-static
    sudo dpkg -i ubuntu-build-service/packages/*        # ignore the broken dependencies, we will fix it next step
    sudo apt-get install -f
    RELEASE=stretch TARGET=desktop ARCH=armhf ./mk-base-debian.sh

This will bootstrap a Debian stretch image, you will get a rootfs tarball named `linaro-stretch-alip-xxxx.tar.gz`. 

Building the rk-debain rootfs with debug:

    VERSION=debug ARCH=armhf ./mk-rootfs-stretch.sh  && ./mk-image.sh

This will install Rockchip specified packages and hooks on the standard Debian rootfs and generate an ext4 format rootfs image at `rootfs/linaro-rootfs.img` .

## Combine everything into one image

    build/mk-image.sh -c rk3399 -t system -r rootfs/linaro-rootfs.img

This will combine u-boot, kernel and rootfs into one image and generate GPT partition table. Output is 

    out/system.img

## Flash the image

Follow instructions [here](http://wiki.radxa.com/Rockpi4/install).

## Troubleshooting

Go to [ROCK Pi 4 FAQs](http://wiki.radxa.com/Rockpi4/FAQs)

