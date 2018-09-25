liveng
======

**Next Generation Linux live distributions concepts**

A live operating system allows booting from a removable medium, such a USB key, without the need of being installed to the hard drive.

Once written onto a USB key, a common live operating system is usually made up of one ISO9660 partition, containing the kernel, the initrd, the compressed filesystem.squashfs image and the second stage bootloader, usually *isolinux* (the boot sector code linking the second stage bootloader is contained within the MBR of the key). Modern lives also add a UEFI partition (some add a "fake" one).

If you need a live system which does data persistence, you will find (or need to create) another partition, usually an EXT4 one. This is pretty common as well.

There are a few live distibutions which support the UEFI Secure Boot (Debian lives do not), but no distribution is capable of updating the kernel maintaining a ISO9660 filesystem, which is the best option for a live.

The aim of the liveng project is to give the Community a set of best practices in order to transform a common Debian Linux live into a live(ng) operating system which does:

* encrypted persistence natively;
* kernel update (on a live ISO 9660 filesystem!);
* UEFI, with UEFI Secure Boot compatibility with a real efi partition.
  
As the base of liveng we have chosen the Debian Stretch live distribution.

Mon-K Data Protection's `Secure-K OS <https://mon-k.com/products/secure-k-personal/>`_ is built according to liveng's best practices.


Table of contents
^^^^^^^^^^^^^^^^^

.. toctree::
   
   standard-live
   setup
