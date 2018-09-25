liveng
======

**Next Generation Linux live distributions concepts**

| Once written onto a USB key, a common live operating system is usually made up of one ISO9660 partition, containing the kernel, the initrd, the compressed filesystem.squashfs image and the second stage bootloader, usually *isolinux* (the boot sector code is contained within the MBR of the key).
| If you need a live system which does persistence, you will find a second partition, usually an EXT4 one. This is pretty common as well.

| The aim of the liveng project is to give the Community a set of actions and best practices in order to transform a common Linux live into a live(ng) operating system which does:

    | * encrypted persistence;
    | * kernel update (on a live ISO 9660 filesystem!);
    | * UEFI, UEFI Secure Boot.
  
| As the base of liveng we have chosen Debian Strecth live.

| Mon-K Data Protection's `Secure-K OS <https://mon-k.com/products/secure-k-personal/>`_ is built according to liveng's best practices.

