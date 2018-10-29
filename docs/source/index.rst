liveng
======

**Next Generation Linux live distributions concepts**

A live operating system allows booting from a removable medium, such a USB key, without the need of being installed to the hard drive.


Why liveng
^^^^^^^^^^

None of the existing live operating systems provide a **kernel update feature**: the kernel and the initrd are the only components that a live operating system cannot update, because they lay outside of the data persistence partition (if any) and usually the system partition is ISO9660-formatted. This will soon lead to an outdated operating system, particularly unsafe if used as a desktop-replacement or for security-critical activities.


More features
^^^^^^^^^^^^^

Once written onto a USB key, a common live operating system is usually made up of one ISO9660 partition, containing the kernel, the initrd, the compressed filesystem.squashfs image and the second stage bootloader, usually *isolinux* (the boot sector code linking the second stage bootloader is contained within the MBR of the key). Modern lives also add a UEFI partition (some add a "fake" one). 

If you need a live system which does data persistence, you will find (or need to create) another partition, usually an EXT4 one. This is pretty common as well.

There are only a few live distibutions which support the UEFI Secure Boot (Debian lives do not), and, as stated before, no distribution is capable of updating the kernel maintaining a ISO9660 filesystem, which is the best option for a live.

The full aim of the liveng project is to give the Community a set of best practices in order to turn a common Debian Linux live into a live(ng) operating system which features:

* native **encrypted persistence**;
* kernel update (on a live ISO 9660 filesystem!);
* UEFI, with **UEFI Secure Boot compatibility**, with a real efi partition.
  
As the base of liveng we have chosen the Debian Stretch live distribution.

liveng is a `LumIT Labs <about.html>`_ project.


Table of contents
^^^^^^^^^^^^^^^^^

.. toctree::
   
   standard-live
   setup
   liveng-structure
   secure-boot
   persistence
   kernel-update
   kernel-update-debian
   kernel-update-improvements
   about
   liveng-based-systems
   license



