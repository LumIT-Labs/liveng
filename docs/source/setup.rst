Set up a standard Debian live
=============================

All the steps needed to write a standard Debian Stretch live operating system image onto a USB key briefely follow.

* download the iso-hybrid live image from: https://cdimage.debian.org/debian-cd/current-live/ for your PC's architecture, i386 or amd64;
* plug the USB key into your computer running any Linux-based OS (I am using Debian Stretch for this guide as my host system);
* open a terminal emulator and get root privileges;
* use ``fdisk -l`` in order to locate the USB key’s device file, for example: /dev/sdx (change for your case). A sample output on my PC::

    Disk /dev/sdc: 29,4 GiB, 31608274944 bytes, 61734912 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0xd8915d07

    Device    Start      End  Sectors  Size  Id Type
    /dev/sdc1  2048 61734911 61732864  29,4G b  W95 FAT32

* unmount the auto-mounted devices, if any (desktop environments usually have an auto-mount feature). You can see them via the ``mount`` command and unmount them by using ``umount /dev/sdxN`` for each device file;
* finally write the image with dd; in my case the command is:: 

    dd if=/home/marco/Download/debian-live-9.5.0-amd64-gnome.iso of=/dev/sdx bs=50M. 

Be careful or you will overwrite your hard drive content! Here I replaced /dev/sdc with /dev/sdx to avoid blind copies and pastes.
Setting the bs command line options to 10MB (or more), speeds up the process::

    2365587456 bytes (2,4 GB, 2,2 GiB) copied, 31,882 s, 74,2 MB/s

Please note that you must refer to the whole disk, so do not use /dev/sdx1 but /dev/sdx, as an example.

Resulting partitioning scheme::

    Disk /dev/sdc: 29,4 GiB, 31608274944 bytes, 61734912 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0x1dc1a6b1

    Device     Boot Start     End Sectors  Size Id Type
    /dev/sdc1  *        0 4620287 4620288  2,2G  0 Empty
    /dev/sdc2        1416    2247     832  416K ef EFI (FAT-12/16/32)

Being Stretch UEFI compliant and not only BIOS compatible, you will notice two partitions out of the box here, one of which is a fake partition (an overlapping one) and to us not a good way of solving problems.

Your USB key can now boot your PC if set as the boot device and UEFI Secure Boot is disabled on the PC's firmware. 
Older PCs does not have the Secure Boot feature and will boot anyway, if supported.

You can now enjoy a stock Debian live operating system with the desktop environment of your choice, without any data persistence: upon booting, the (live-build modified) initd will union mount the compressed filesystem with the ram disk and pass that mount to the Linux kernel as /. Reboot the system and every data will be lost.

