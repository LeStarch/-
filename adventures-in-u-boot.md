# Adventures in U-Boot

This will document attempts to use UBoot to boot the BeagleBoneBlack. Why? ...because the built-in timeout for uboot is 0 seconds.

## What is U-Boot?

Das U-Boot (U-Boot) is a bootloader.  https://github.com/u-boot/u-boot

Configuration seems to be done before building.  This can be done in two ways:

1. Applying patches
2. Editing configuration files.

## Setting Up U-Boot for a Board

To do this, I am loosly following this guide here: https://forum.digikey.com/t/debian-getting-started-with-the-beaglebone-black/

First, clone the above repository.  Then look for patches for your particular board.  BeagleBoneBlack its own U-Boot fork: https://git.beagleboard.org/beagleboard/u-boot

One can compare the trees there.


