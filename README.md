# Rockchip Debian SDK

Below is the instructions of how to build image for ROCK Pi 4.

## Get the source code

Note that here the non-root user is jack and we are at the home folder.

    $ cd ~
    $ pwd
    /home/jack

When you have a different name, like rose, you can replace jack with rose in the next parts of this guide.

Command prepended by $ means the command may be executed by an unprivileged user.  And command prepended by # means the command may be executed by an privileged user. But the symbol, $ or #, is not part of the command.

You need Git to get multiple git repositories to build the image.

Install Git if you don't have it.

    $ sudo apt-get update && sudo apt-get upgrade
    $ sudo apt-get install git

Clone the source code

    $ cd ~
    $ git clone --recursive https://github.com/radxa/rockchip-bsp.git

You will get 

    build  kernel  README.md  rkbin  rootfs  u-boot

Directories usage introductions:

* build:
    * Some script files and configuration files for building u-boot, kernel and rootfs.

* kernel:
    * kernel source code, current version is 4.4

* rkbin:
    * Prebuilt Rockchip binaries, include first stage loader and ATF(Arm Trustzone Firmware).

* rootfs:
    * Bootstrap a Debian based rootfs, support architechture armhf and arm64, support Debian Jessie and Stretch.

* u-boot:
    * u-boot as the second stage bootloader

* docker:
    * Init a ubuntu 16.04 build environment for easier building u-boot, kernel, rootfs and system.

## Build images

Two methods of building images will be shown in the next parts. One uses Docker (Part one) and the other doesn't (Part two). Just select one part and start to build your wanted images.

### Part one

#### Install Docker

    $ sudo apt-get update
    $ sudo apt-get install binfmt-support qemu-user-static
    $ sudo apt-get install \
        apt-transport-https \
        ca-certificates \
        curl \
        software-properties-common
    $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    $ sudo apt-key fingerprint 0EBFCD88

    pub   4096R/0EBFCD88 2017-02-22
          Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
    uid                  Docker Release (CE deb) <docker@docker.com>
    sub   4096R/F273FCD8 2017-02-22

    $ sudo add-apt-repository \
       "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
       $(lsb_release -cs) \
       stable"

    $ sudo apt-get update
    $ sudo apt-get install docker-ce
    $ apt-cache madison docker-ce

    docker-ce | 18.09.0~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages

    $ sudo apt-get install docker-ce=<VERSION>

Go to the docker folder.

    $ cd ~/rockchip-bsp/docker

Build a Docker image, called rockchip-radxa:1.

    $ sudo docker build -t rockchip-radxa:1 -f ./Dockerfile .

Now the Docker image, rockchip-radxa:1, is ready. You just need to build Docker image once. Everytime you want to build images, just run a Docker container.

#### Run a Docker container

    $ docker run --privileged -it -v /home/jack/rockchip-bsp:/root rockchip-radxa:1 /bin/bash

Now the Docker container should be running.

Here Docker bind mounts /home/jack/rockchip-bsp in the host to /root in the Docker container. cd /root and ls will show:

    # cd /root
    # ls
    build  kernel  README.md  rkbin  rootfs  u-boot

#### Build u-boot

    # cd /root
    # ./build/mk-uboot.sh rockpi4b     #For ROCK Pi 4 Mode B

The generated images will be copied to out/u-boot folder

    # ls out/u-boot/
    idbloader.img  rk3399_loader_v1.12.112.bin  trust.img  uboot.img

#### Build kernel

    # ./build/mk-kernel.sh rockpi4b    #For ROCK Pi 4 Mode B

You will get the kernel image and dtb file

    # ls out/kernel/
    Image  rockpi-4b-linux.dtb

#### Make rootfs image

To build 32bit rootfs:

    # export ARCH=armhf

To build 64bit rootfs:

    # export ARCH=arm64

Building a base debian system by ubuntu-build-service from linaro.

    # cd rootfs
    # dpkg -i ubuntu-build-service/packages/*        # ignore the broken dependencies, we will fix it next step
    # apt-get install -f
    # RELEASE=stretch TARGET=desktop ARCH=${ARCH} ./mk-base-debian.sh

This will bootstrap a Debian stretch image, you will get a rootfs tarball named `linaro-stretch-alip-xxxx.tar.gz`.

Building the rk-debain rootfs with debug:

    # VERSION=debug ARCH=${ARCH} ./mk-rootfs-stretch.sh  && ./mk-image.sh

This will install Rockchip specified packages and hooks on the standard Debian rootfs and generate an ext4 format rootfs image at `rootfs/linaro-rootfs.img` .

#### Combine everything into one image

    # build/mk-image.sh -c rk3399 -t system -r rootfs/linaro-rootfs.img

This will combine u-boot, kernel and rootfs into one image and generate GPT partition table. Output is

    out/system.img

#### Exit Docker

After getting all the wanted images, exit Docker;

    # exit

In the host, all generated images are in the ~/rockchip-bsp/out directory.

### Part two

When you don't want to use Docker to build images, you can try this way.

Note that if you just used Docker to build the images, then you can't wait to try the new method, there may be operational permissions issues.

#### Install toolchain and other build tools

    $ sudo apt-get install gcc-aarch64-linux-gnu device-tree-compiler libncurses5 libncurses5-dev build-essential libssl-dev mtools bc python dosfstools

#### Build u-boot/

    $ cd ~/rockchip-bsp
    $ ./build/mk-uboot.sh rockpi4b     #For ROCK Pi 4 Mode B

The generated images will be copied to out/u-boot folder

    $ ls out/u-boot/
    idbloader.img  rk3399_loader_v1.12.112.bin  trust.img  uboot.img

#### Build kernel

    $ ./build/mk-kernel.sh rockpi4b    #For ROCK Pi 4 Mode B

You will get the kernel image and dtb file

    $ ls out/kernel/
    Image  rockpi-4b-linux.dtb

#### Make rootfs image

To build 32bit rootfs:

    $ export ARCH=armhf

To build 64bit rootfs:

    $ export ARCH=arm64

Building a base debian system by ubuntu-build-service from linaro.

    $ cd rootfs
    $ sudo apt-get install binfmt-support qemu-user-static gdisk
    $ sudo dpkg -i ubuntu-build-service/packages/*        # ignore the broken dependencies, we will fix it next step
    $ sudo apt-get install -f
    $ RELEASE=stretch TARGET=desktop ARCH=${ARCH} ./mk-base-debian.sh

This will bootstrap a Debian stretch image, you will get a rootfs tarball named `linaro-stretch-alip-xxxx.tar.gz`. 

Building the rk-debain rootfs with debug:

    $ VERSION=debug ARCH=${ARCH} ./mk-rootfs-stretch.sh  && ./mk-image.sh

This will install Rockchip specified packages and hooks on the standard Debian rootfs and generate an ext4 format rootfs image at `rootfs/linaro-rootfs.img` .

#### Combine everything into one image

    $ build/mk-image.sh -c rk3399 -t system -r rootfs/linaro-rootfs.img

This will combine u-boot, kernel and rootfs into one image and generate GPT partition table. Output is 

    out/system.img

## Flash the image

For normal users, follow instructions [installation](http://wiki.radxa.com/Rockpi4/install). You will need the generated '''out/system.img''' only.

For developers, flash from USB OTG port, follow instructions [usb-installation](http://wiki.radxa.com/Rockpi4/dev/usb-install). You will need the flash helper '''rk3399_loader_xxx.bin''' and generated '''out/system.img''' files.

## Troubleshooting

Go to [ROCK Pi 4 FAQs](http://wiki.radxa.com/Rockpi4/FAQs)

