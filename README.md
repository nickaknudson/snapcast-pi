# snapcast-pi
This project uses [buildroot](https://buildroot.org) to build [snapcast](https://github.com/badaix/snapcast) for the raspberry pi. This example was tested with the raspberry pi zero, but has configurations for all raspberry pi models.

# Build Environment
For convenience the build environment has been containerized using [docker](https://www.docker.com/). To get started, simply [install docker for your machine](https://www.docker.com/products/overview#/install_the_platform).

For more information on the docker image or to build it yourself see [nickaknudson/buildroot-docker](https://github.com/nickaknudson/buildroot-docker) and [nickaknudson/buildroot](https://cloud.docker.com/app/nickaknudson/repository/docker/nickaknudson/buildroot/general).

If you do not wish to use the provided docker container then you will need to install all of the [buildroot dependencies](https://buildroot.org/downloads/manual/manual.html#requirement) yourself.

# Building
Clone this repository and change directory:

    git clone https://github.com/nickaknudson/snapcast-pi.git && cd snapcast-pi

Enter the build environment:

    docker run -it -v $(pwd):/home/buildroot/snapcast-pi -w /home/buildroot/snapcast-pi nickaknudson/buildroot /bin/bash
    
(Use the following if using a SElinux-enabled host such as Fedora, RHEL, Scientific Linux, etc.:)

    docker run -it -v $(pwd):/home/buildroot/snapcast-pi:Z -w /home/buildroot/snapcast-pi nickaknudson/buildroot /bin/bash

Clone buildroot:

    BUILDROOT_VERSION=2016.11.2
    git clone --branch $BUILDROOT_VERSION --depth=1 git://git.buildroot.net/buildroot ~/buildroot

Clone snapcast:

    git clone --depth=1 https://github.com/badaix/snapcast.git ~/snapcast

[Configure buildroot](https://git.busybox.net/buildroot/tree/board/raspberrypi/readme.txt) (this command is for raspberry pi zero, see `buildroot-external/configs` folder for other configurations):

    make O=~/buildroot-output BR2_DL_DIR=~/buildroot-dl BR2_EXTERNAL=~/snapcast/buildroot:~/snapcast-pi/buildroot-external -C ~/buildroot snapcastpi0_defconfig

Use menuconfig to add additional packages (optional):

    make O=~/buildroot-output BR2_DL_DIR=~/buildroot-dl BR2_EXTERNAL=~/snapcast/buildroot:~/snapcast-pi/buildroot-external -C ~/buildroot menuconfig

Build:

    make O=~/buildroot-output BR2_DL_DIR=~/buildroot-dl BR2_EXTERNAL=~/snapcast/buildroot:~/snapcast-pi/buildroot-external -C ~/buildroot

Just build the snapcast binary:

    make O=~/buildroot-output BR2_DL_DIR=~/buildroot-dl BR2_EXTERNAL=~/snapcast/buildroot:~/snapcast-pi/buildroot-external -C ~/buildroot package/snapcast

# Flashing
Copy the build products from the container back to the host machine:

    cp -R ~/buildroot-output/images .

Exit the build environment:

    exit

Find your SD card:

    df -h

Flash the SD card (where `dev/sdX` is from above):

    dd bs=4M if=images/sdcard.img of=/dev/sdX

# Tips
By design, docker will launch a new container every time you call `docker run`. But you can attach to old containers which will still contain all of the old build products.

First find the container that you want:

    docker ps -a

Then start it if it is stopped:

    docker start <container_id>

Then attach to it:

    docker attach <container_id>

If you want to remove old containers to free up hard disk space:

    docker containers prune
