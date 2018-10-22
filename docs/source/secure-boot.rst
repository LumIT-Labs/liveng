UEFI Secure Boot
================

We previously `deployed <uefi.html>`_ a USB key with a GPT and protective MBR partitioning scheme, with three partitions, two for the system and one for the UEFI compliance. We also used and installed GRUB as the bootloader instead of syslinux/isolinux.

While UEFI compatibility is assured by the efi FAT partition and by the GRUB binary image inside it, Secure Boot machines won't natively run the live system, because there is no signed bootloader yet.

In brief, while `UEFI Secure Boot <https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface#Secure_boot>`_ is enabled, the hardware restricts the execution of code at boot time only to binary code signed by trusted keys. Trusted keys belong to a chain of trust with a root key stored on firmware and referred to as the Platform Key. The UEFI firmware verifies that the boot loader image has a cryptographically valid signature before running it. 

Easiest way to *bypass the problem* is to use the `Linux Foundation's preloader <https://blog.hansenpartnership.com/linux-foundation-secure-boot-system-released/>`_.

Idea is simple: the preloader binary is signed with a (Microsoft) key and it is therefore suitable for being loaded by every UEFI Secure Boot firmware.

When run, the preloader tries to launch *loader.efi*, a chained executable, which will be the "real" bootloader - GRUB in our case. 

Preloader uses a whitelist mechanism called Machine Owner Key list. If the hash of the launching binary is in the MokList, the preloader will execute it, if not, it will launch a key management utility (the `HashTool <https://askubuntu.com/questions/594747/how-to-use-the-linux-foundations-preloader>`_) which allows enrolling the hash or key by the user::

    ┌──────────────────────────────────────────────────────────────────────────────┐
    │                            Failed to start loader                            │
    │                                                                              │
    │          It should be called loader.efi (in the current directory)           │
    │                     Please enrol its hash and try again                      │
    │                                                                              │
    │                I will now execute HashTool for you to do this                │
    │                                                                              │
    │                                                                              │
    │                                     ┌────┐                                   │
    │                                     │ OK │                                   │
    │                                     └────┘                                   │  
    │                                                                              │
    │                                                                              │
    └──────────────────────────────────────────────────────────────────────────────┘

    ┌──────────────────────────────────────────────────────────────────────────────┐
    │                                Select Binary                                 │
    │                                                                              │
    │               The Selected Binary will have its hash Enrolled                │
    │            This means it will Subsequently Boot with no prompting            │
    │    Remember to make sure it is a genuine binary before Enroling its hash     │
    │                                                                              │
    │                                                                              │
    │                            ┌─────────────────────┐                           │
    │                            │     Enroll Hash     │                           │
    │                            │ Reboot to UEFI Menu │                           │
    │                            │    Reboot System    │                           │
    │                            │        Exit         │                           │
    │                            └─────────────────────┘                           │
    │                                                                              │
    │                                                                              │
    └──────────────────────────────────────────────────────────────────────────────┘

    ┌──────────────────────────────────────────────────────────────────────────────┐
    │                                Select Binary                                 │
    │                                                                              │
    │               The Selected Binary will have its hash Enrolled                │
    │            This means it will Subsequently Boot with no prompting            │
    │    Remember to make sure it is a genuine binary before Enroling its hash     │
    │                                                                              │
    │                                                                              │
    │                                ┌──────────────┐                              │
    │                                │     ../      │                              │
    │                                │  loader.efi  │                              │
    │                                │ HashTool.efi │                              │
    │                                │ bootx64.efi  │                              │
    │                                └──────────────┘                              │
    │                                                                              │
    │                                                                              │
    └──────────────────────────────────────────────────────────────────────────────┘

    ┌──────────────────────────────────────────────────────────────────────────────┐
    │                      Enroll this hash into MOK database?                     │
    │                                                                              │
    │                               File: \loader.efi                              │
    │     Hash: 8D1B74227CB2EE6B23B829595B761BAA34D171337F70D44ABF542D5318BDBA08   │
    │                                                                              │
    │                                                                              │
    │                                                                              │
    │                                                                              │
    │                                     ┌─────┐                                  │
    │                                     │ No  │                                  │
    │                                     │ Yes │                                  │
    │                                     └─────┘                                  │
    │                                                                              │
    │                                                                              │
    └──────────────────────────────────────────────────────────────────────────────┘


In practice
^^^^^^^^^^^

Setting up UEFI Secure Boot compatibility is now easy. 
We download the Linux Foundation preloader and HashTool, rename bootx64.efi as loader.efi, then move the Linux Foundation's files into the efi folder of the USB key (efi partition)::

    cd /tmp
    wget https://blog.hansenpartnership.com/wp-uploads/2013/PreLoader.efi
    wget https://blog.hansenpartnership.com/wp-uploads/2013/HashTool.efi

    mount ${device}3 /mnt/   
    mv /mnt/efi/boot/bootx64.efi /mnt/efi/boot/loader.efi

    mv PreLoader.efi /mnt/efi/boot/bootx64.efi
    mv HashTool.efi /mnt/efi/boot/

    umount /mnt

liveng system is now capable of UEFI Secure Boot the clean way. 



