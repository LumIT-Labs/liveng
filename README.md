# liveng

Next Generation Linux live distributions concepts

A live operating system allows booting from a removable medium, such a USB key, without the need of being installed to the hard drive.

None of the existing live operating systems provide a kernel update feature: the kernel and the initrd are the only components that a live operating system cannot update, because they lay outside of the data persistence partition (if any) and usually the system partition is ISO9660-formatted. This will soon lead to an outdated operating system, particularly unsafe if used as a desktop-replacement or for security-critical activities.

The aim of the liveng project is to give the Community a set of best practices in order to turn a common (Debian Stretch) Linux live into a live(ng) operating system which features:

* native encrypted persistence;
* kernel update (on a live ISO 9660 filesystem!);
* UEFI, with UEFI Secure Boot compatibility, with a real efi partition.

This Github repository hosts all the source documentation files for Read the Docs, see https://liveng.readthedocs.io.
