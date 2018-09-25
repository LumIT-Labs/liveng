Standard Debian live
====================

A live operating system allows booting from a removable medium, such a USB key, without the need of being installed to the hard drive.


How a standard Debian live system is made 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ 

Main components of a (Linux Debian) live operating system are:

* The Linux kernel image, usually named vmlinuz. 
* The initial RAM disk image (initrd): a RAM disk containing all the modules needed to mount the system image and some scripts and binaries to perform the task.
* The filesystem image containing all the operating system's files (kernel and initrd as well). Usually, a SquashFS compressed filesystem is used to minimize the Debian Live image size.
* The bootloader: a small piece of code crafted to boot from the chosen medium. It loads the Linux kernel and its initrd to run with an associated filesystem.

The booting in a Debian live is a two-stage process. 
First, the bootloader loads the kernel and initrd into memory, and passes the execution control to the kernel. 
After some basic initializations, the kernel extracts the initramfs archive and mounts it as a temporary root filesystem. 
The initrd contains kernel modules and userspace programs (such as the insmod tool to install kernel modules into the kernel, lvm and so on) required to initialize the physical or logical device(s) containing the real root filesystem. 
The init script on the initrd loads modules and performs other neccessary steps, such as union-mounting the compressed filesystem image with a (possibly encrypted) persistence partition or the ram disk.
At the end of this stage run-init deletes the initrd from memory, mounts the real root filesystem and passes control to the /sbin/init (Systemd) program on it.

With such setup the kernel size is kept under control by allowing most of the drivers to be compiled as modules - in an initrd-less setup the drivers neccessary for the boot-time initialization of the root device must be compiled into it. 




