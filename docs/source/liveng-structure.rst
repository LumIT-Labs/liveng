liveng structure
================

We have previously `deployed <setup.html>`_ a stock Debian live with the use of dd, preserving all the structures contained within the downloaded `ISO image <https://en.wikipedia.org/wiki/ISO_image>`_, resulting in a live operating system without proper UEFI or Secure Boot support and without the wows of a kernel update (on a ISO9660 filesystem) feature. 

In order to build a liveng operating system, we must now restart from scratch, however, and keep only a bunch of the old files - only the the **kernel**, the **initrd** and the **filesystem.squashfs** are really needed. 

For future reference, the `device file <https://en.wikipedia.org/wiki/Device_file>`_ corresponding to the whole USB key (please plug the USB key in and use ``fdisk -l`` as stated previously) and the path of the downloaded Debian live image are kept in two variables, together with the size of the image file itself in a third one::

    device="/dev/sdx"
    imageFile="/path/to/debian-live-x.y.z-arch-desktop.iso"
    imageFileSize=$(du -m ${imageFile} | awk '{print $1}')

As done before, I'm trying avoiding some hard drive wiping (and some heart attacks) by changing the device file name to */dev/sdx* for the whole liveng documentation, please remember to change it for your case. 

We are also tracking the name of the files we'll need, which are contained inside the ISO image::

    mount ${imageFile} /mnt

    configFile=$(ls /mnt/live/ | awk '{print $1}' | grep config)       
    initrdFile=$(ls /mnt/live/ | awk '{print $1}' | grep initrd)  
    vmlinuzFile=$(ls /mnt/live/ | awk '{print $1}' | grep vmlinuz)  
    systemMapFile=$(ls /mnt/live/ | awk '{print $1}' | grep System)  

    umount /mnt

We will unomunt now every possible mountpoint that you or your desktop environment may have enstablished with the key's partitions, and wipe all the contained `filesystems <https://en.wikipedia.org/wiki/File_system>`_' key structures::

    cd /tmp
    
    # Unmount.
    for i in $(mount | grep ${device} | awk '{print $1}'); do umount $i; done

    # Wipe.
    wipefs -af ${device}


GUID partition table
^^^^^^^^^^^^^^^^^^^^

Now we are creating a new empty `GUID partition table <https://en.wikipedia.org/wiki/GUID_Partition_Table>`_ (GPT for its friends); this operation deletes all the exisiting partitions and creates a new `protective MBR <https://en.wikipedia.org/wiki/GUID_Partition_Table#Protective_MBR_(LBA_0)>`_ (for backward compatibility with BIOS hardware)::

    printf "o\nY\nw\nY\n" | gdisk ${device} && sync

You can launch ``gdisk ${device}`` and type all the options by hand in order to see what the previous command actually does.

Resulting partitioning scheme (with ``fdisk -l``)::

    Disk /dev/sdb: 29,4 GiB, 31608274944 bytes, 61734912 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: gpt
    Disk identifier: 1D0FE347-D269-4E57-843C-AA5DAEA2A7A1

No partitions available so far. So good.


System partition
^^^^^^^^^^^^^^^^

Now it's the turn of manually creating the first partition onto the USB key; we will always use gdisk, for the older fdisk is not GPT compliant. 
The first partition will host most of the files contained within the downloaded Debian live image, so we are creating it of the same size as the image, which is consistent with our scope::

   printf "n\n\n\n+${imageFileSize}M\n8300\nw\nY\n" | gdisk ${device} && sync

Previous line means::

    Command (? for help): n
    Partition number (1-128, default 1): 
    First sector (34-61734878, default = 2048) or {+-}size{KMGTP}: 
    Last sector (2048-61734878, default = 61734878) or {+-}size{KMGTP}: +2300M
    Current type is 'Linux filesystem'
    Hex code or GUID (L to show codes, Enter = 8300): 
    Changed type of partition to 'Linux filesystem'

    Command (? for help): w
    Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
    PARTITIONS!!
    Do you want to proceed? (Y/N): Y
    OK; writing new GUID partition table (GPT) to /dev/sdb.
    The operation has completed successfully.

Resulting partitioning scheme (with ``fdisk -l``)::

    Disk /dev/sdb: 29,4 GiB, 31608274944 bytes, 61734912 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: gpt
    Disk identifier: 1D0FE347-D269-4E57-843C-AA5DAEA2A7A1

    Device     Start     End Sectors  Size Type
    /dev/sdb1   2048 4712447 4710400  2,3G Linux filesystem

One unformatted partition has been created. 

With the next step, xorriso will help us writing the files to the new newely created USB's partition, maintaining the ISO9660 filesystem (rtfm of xorriso for more)::

    xorriso -indev ${imageFile} -boot_image any discard -overwrite on -volid LIVENG_SYSTEM -move live/${configFile} live/config -move live/${initrdFile} live/initrd.img -move live/${vmlinuzFile} live/vmlinuz -move live/${systemMapFile} live/System.map -rm_r .disk boot d-i dists isolinux pool -- -outdev stdio:${device}1 -blank as_needed  

As the result::

    ISO image produced: 1081069 sectors
    Written to medium : 1081248 sectors at LBA 32
    Writing to 'stdio:/dev/sdb1' completed successfully.

Having a look that everything is as expected::

    mount ${device}1 /mnt/
    ls /mnt/live/

Will display::

    config  filesystem.squashfs  initrd.img  System.map  vmlinuz

As you may have noted, all the files have been renamed, in order for us to ease the configuration of GRUB.
Now we can unmount the partition::
  
    umount /mnt


Second system partition
^^^^^^^^^^^^^^^^^^^^^^^

With the same technique used before, we are now adding a second system partition to the key, which will contain the **kernel and the initrd files only**.
The bootloader will be then instructed to **boot from this partition**, because the second system partition will always contain the **most updated kernel and initrd** files.

**At every kernel update, this small partition will be overwritten**, with the use of xorriso, by the postinst script of the liveng kernel package.
(First) System partition files are kept at their default state and can be useful in case of recovery or when a complete persistence reset is performed.

Creating the second partition::

    printf "n\n\n\n+256M\n8300\nw\nY\n" | gdisk ${device}

``fdisk -l``::

    Device       Start     End Sectors  Size Type
    /dev/sdc1     2048 4624383 4622336  2,2G Linux filesystem
    /dev/sdc2  4624384 5148671  524288  256M Linux filesystem

With the next step, xorriso will write the kernel and initrd files into the second system partition (ISO9660)::

    xorriso -indev ${imageFile} -boot_image any discard -overwrite on -volid 'LIVENG_SYSTEM2' -move live/${configFile} live/config -move live/${initrdFile} live/initrd.img -move live/${vmlinuzFile} live/vmlinuz -move live/${systemMapFile} live/System.map -rm_r .disk boot d-i dists isolinux pool live/filesystem.squashfs -- -outdev stdio:${device}2 -blank as_needed

Having a look that everything is as expected::

    mount ${device}2 /mnt/
    ls /mnt/live/
    umount /mnt

Will display::

    config    initrd.img    System.map    vmlinuz


BRUB bootloader (UEFI)
^^^^^^^^^^^^^^^^^^^^^^

The GPT partitioning scheme has been set up and the system partitions have been created. 
We are now adding the UEFI partition, which must be FAT (guess who introduced the UEFI Secure Boot...).

Every standard UEFI firmware must look into the /efi/boot/ folder of the FAT partition of the booting device for a file named *boot{arch}.efi*, so we are just creating the folder on the USB drive and copy a GRUB UEFI-compliant bootloader binary to this location.

But first we need to generate our own GRUB bootloader image with some modules included. The following command will generate the GRUB image. *grub-efi-amd64** amd64 Debian packages are needed - novices are advised to download the *bootx64.efi* hosted at `liveng Github repository <https://github.com/LumIT-Labs/liveng>`_ (**docs/grub-bin/**) instead of creating a new one::
 
    grub-mkimage -o bootx64.efi -p /efi/boot -O x86_64-efi \
        fat iso9660 part_gpt part_msdos \
        normal boot linux configfile loopback chain keylayouts \
        efifwsetup efi_gop efi_uga jpeg png \
        ls search search_label search_fs_uuid search_fs_file \
        gfxterm gfxterm_background gfxterm_menu test all_video loadenv \
        exfat ext2 ntfs udf password password_pbkdf2 pbkdf2 linuxefi

We are ready to create the third (UEFI, FAT) partition, sized 32MB, onto the key::

    printf "n\n\n\n+32M\nef00\nw\nY\n" | gdisk ${device} && sync
    mkfs.vfat -n "UEFI Boot" ${device}3

Resulting partitioning scheme (with ``fdisk -l``)::

    Device       Start     End Sectors  Size Type
    /dev/sdc1     2048 4624383 4622336  2,2G Linux filesystem
    /dev/sdc2  4624384 5148671  524288  256M Linux filesystem
    /dev/sdc3  5148672 5214207   65536   32M EFI System

Now we will copy the GRUB binary to the UEFI partition, as stated before::

    mount ${device}3 /mnt/  
    mkdir -p /mnt/efi/boot
    cp /path/to/bootx64.efi /mnt/efi/boot  

and setup the GRUB config file so that GRUB will be able to locate the kernel, initrd and filesystem.squashfs files upon booting.
Kernel and initrd files will be loaded from the second system partition and **the filesystem.squashfs will be loaded from the first system partition** (thanks to the *fromiso* live-boot's directive)::

        cat > /mnt/efi/boot/grub.cfg <<EOF
    menuentry 'liveng standard boot' --unrestricted {         
        insmod iso9660
        search --no-floppy --set=root --hint-efi=hd0,gpt2 --fs-uuid $(blkid -s UUID ${device}2 | awk -F\" '{print $2}')
        linux /live/vmlinuz initrd=/live/initrd.img fromiso=$(blkid -s UUID ${device}1 | awk -F\" '{print $2}') boot=live live-noconfig persistence
        initrd /live/initrd.img
    }
    EOF

We also add a **fallback boot**, which loads all the files from the original (first) system partition, just in case something goes wrong::

        cat >> /mnt/efi/boot/grub.cfg <<EOF
    menuentry 'liveng fallback boot' --unrestricted {         
        insmod iso9660
        search --no-floppy --set=root --hint-efi=hd0,gpt1 --fs-uuid $(blkid -s UUID ${device}1 | awk -F\" '{print $2}')
        linux /live/vmlinuz initrd=/live/initrd.img fromiso=$(blkid -s UUID ${device}1 | awk -F\" '{print $2}') boot=live live-noconfig persistence liveng-fallback
        initrd /live/initrd.img
    }
    EOF

    umount /mnt

The ``$(blkid -s UUID ${device}x | awk -F\" '{print $2}')`` command gets the UUID of the *${device}x* partition, given during the partition creation.

We will use the ``persistence`` directive later on; if no persistence partition is found, the (live-boot modified) initrd will union-mount the filesystem.squashfs with the RAM disk.
The ``liveng-fallback`` will be of use later on, too.


GRUB bootloader (BIOS)
^^^^^^^^^^^^^^^^^^^^^^
The GPT partitioning scheme we previously set up also contains a protective MBR, which assures BIOS compatibility and the possibility to create more than 4 primary partitions.

We will now set up GRUB for the BIOS compatibility - as said, unlike UEFI, BIOS needs a first-stage bootloader (the boot sector code) contained within the MBR of the key::

    mount ${device}3 /mnt/  
    cp -R /path/to/grub-bios /mnt/boot  

Where *grub-bios* is a folder containing "standard" GRUB binaries for BIOS. You can download a compressed archive of it from the liveng Github repository (**docs/grub-bin/**).

grub.cfg::

        cat > /mnt/boot/grub/grub.cfg <<EOF
    menuentry 'liveng standard boot' --unrestricted {         
        insmod iso9660
        search --no-floppy --set=root --hint-efi=hd0,gpt2 --fs-uuid $(blkid -s UUID ${device}2 | awk -F\" '{print $2}')
        linux /live/vmlinuz initrd=/live/initrd.img fromiso=$(blkid -s UUID ${device}1 | awk -F\" '{print $2}') boot=live live-noconfig persistence
        initrd /live/initrd.img
    }
    EOF

We also add a **fallback boot**, which loads all the files from the original (first) system partition, just in case something goes wrong::

        cat >> /mnt/boot/grub/grub.cfg <<EOF
    menuentry 'liveng fallback boot' --unrestricted {         
        insmod iso9660
        search --no-floppy --set=root --hint-efi=hd0,gpt1 --fs-uuid $(blkid -s UUID ${device}1 | awk -F\" '{print $2}')
        linux /live/vmlinuz initrd=/live/initrd.img fromiso=$(blkid -s UUID ${device}1 | awk -F\" '{print $2}') boot=live live-noconfig persistence liveng-fallback
        initrd /live/initrd.img
    }
    EOF

Finally, install first-stage GRUB::

    grub-install --root-directory=/mnt ${device} --force

    umount /mnt


So far, so good
^^^^^^^^^^^^^^^

So far, we have set up a USB key with a GPT and protective MBR partitioning scheme, with three partitions, two for the system and one for the UEFI compliance. We also used and installed GRUB as the bootloader instead of syslinux/isolinux - this will give us much more flexibility. 

We will go back to kernel update later on.

Live system is still non-persistent, please do a couple of UEFI (non-Secure Boot) and BIOS test bootings to verify all is set up correctly...

