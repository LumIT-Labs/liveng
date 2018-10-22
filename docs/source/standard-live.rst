Standard Debian live
====================

A live operating system allows booting from a removable medium, such a USB key, without the need of being installed to the hard drive.


How a standard Debian live system is made 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ 

Main components of a (Linux Debian) live operating system are:

* The Linux kernel image, usually named vmlinuz. 
* The initial RAM disk image (`initrd <https://en.wikipedia.org/wiki/Initial_ramdisk>`_): a RAM disk containing all the modules needed to mount the root filesystem and some scripts and binaries to perform the task. *initramfs-tools* is responsible of creating such an image when updating the kernel or adding some modules to the system.
* The filesystem image containing all the operating system's files (kernel and initrd as well). Usually, a `SquashFS <https://en.wikipedia.org/wiki/SquashFS>`_ compressed filesystem is used to minimize the Debian live image size.
* The `bootloader <https://en.wikipedia.org/wiki/Booting>`_: a small piece of code crafted to boot from the chosen medium. It loads the Linux kernel and its initrd to run with an associated filesystem.

Upon booting, for BIOS-based hardware, the computer's firmware will locate and load the first-stage bootloader (within the MBR), which in turn loads its second-stage, resposible for loading the kernel and the associated initrd into memory; then the bootloader passes the execution control to the kernel. For UEFI-based hardware, there is only a binary executable bootloader file located in a small FAT partition. 

After some basic initializations, the kernel extracts the initrd archive and mounts it as a temporary root filesystem.

The initrd contains kernel modules and userspace programs (such as the tool to install kernel modules into the kernel) required to initialize the device(s) containing the real root filesystem. 

The init script on the initrd loads modules and performs other neccessary steps, such as `union-mounting <https://en.wikipedia.org/wiki/Union_mount>`_ (by the use of `unionfs <https://en.wikipedia.org/wiki/UnionFS>`_) the compressed filesystem image with a (possibly `LUKS <https://en.wikipedia.org/wiki/Linux_Unified_Key_Setup>`_ encrypted) persistence partition or with the RAM disk.

At the end of this stage the initrd is deleted from the memory, and the real root filesystem is mounted. Finally the kernel passes the control to the /sbin/init (`Systemd <https://en.wikipedia.org/wiki/Systemd>`_) program on it.

With such setup the kernel size is kept under control by allowing most of the drivers to be compiled as modules - in an initrd-less setup the drivers neccessary for the boot-time initialization of the root device must be compiled into it. 

