Kernel update
=============

So far, liveng is a Secure Boot capable and persistent (with or without AES encryption) live operating system.
The partitioning scheme we `used <liveng-structure.html>`_ is made up of two system partitions, one containing all the "interesting" files (kenel, initrd, filesystem.squashfs) and the other one containing kernel and initrd only.

The bootloader is instructed to boot by default from the second system partition, because that one always contains the most updated kernel and initrd files: at every kernel update, this small partition will be overwritten, and we are now covering how to do the rewrite in great detail. 

(First) System partition files are kept at their default state and can be useful in case of recovery or when a complete persistence reset is performed (a fallback boot menu has been added as well).
 

Prerequisites
^^^^^^^^^^^^^

If you opted for having a liveng system with an encrypted data persistence partition, because we are starting from a stock Debian live image which lacks cryptsetup files in its initrd, we need to check that cryptsetup won't break the initrd when updating the kernel (`see why <persistence.html#adding-cryptsetup-to-liveng-create-a-modified-initrd>`_). On a running liveng operating system, do::

    apt-get install -y cryptsetup
    sed -i 's/#CRYPTSETUP=/CRYPTSETUP=y/g' /etc/cryptsetup-initramfs/conf-hook

If you opted for a liveng without encryption you can skip the above steps.

Everyone must finally install xorriso onto the system::

    apt-get install -y xorriso


Manually updating the kernel
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When normally updating the kernel on a live operating system, only the content of / is modified (and persisted if on a persistent live), but the content of the outside-kernel and ouside-initrd files to which the bootloader links remains unchanged (and untouched).

In addition, the initrd build isn't performed while on a ISO9660 live operating system, so we have to force it. 

We start by creating an update script::

    #!/bin/bash

    kernelRelease=$1

    if [ ! -z $kernelRelease ]; then
        livengBootDevice=/dev/$(mount | grep persistence | head -1 | awk -F'[0-9/]' '{print $4}')
        if [ ! -z $livengBootDevice ]; then
            livengSecondSystemPartition=${livengBootDevice}2
            livengSecondSystemPartitionUUID=$(blkid -s UUID ${livengSecondSystemPartition} | awk -F\" '{print $2}')

            cd /boot
            if [ -f vmlinuz-${kernelRelease} ] && [ -f initrd.img-${kernelRelease} ]; then
                mkdir -p temp/live
                cp -a vmlinuz-${kernelRelease} temp/live/vmlinuz
                cp -a initrd.img-${kernelRelease} temp/live/initrd.img

                cd temp
                echo "Writing $livengSecondSystemPartition (UUID: $livengSecondSystemPartitionUUID)"
                umount $livengSecondSystemPartition

                if xorrisofs -volid LIVENG_SYSTEM2 --modification-date=$(echo $livengSecondSystemPartitionUUID | sed -e 's#-##g') -o $livengSecondSystemPartition . ; then
                    exit 0
                fi
            fi
        fi
    fi
 
    exit 1

We can name the script as *livengkupd.sh* and move it to /sbin/ with root:root, 750 permissions. 

The script must be called by passing it the release of the initrd (= kernel) release we are updating. It will locate the second system partition and rewrite it with the new files, using xorriso and maintaining its old UUID.

As example, we will now update the liveng kernel from the one currently running::

    uname -a

    Linux debian 4.9.0-7-amd64 #1 SMP Debian 4.9.110-1 (2018-07-05) x86_64 GNU/Linux

to the one available in the Debian Backports repository::

    echo "deb http://deb.debian.org/debian/ stretch-backports main contrib non-free" > /etc/apt/sources.list.d/backports.list
    apt-get update

    apt-cache search linux | grep image
    apt-get install -y linux-image-4.17.0-0.bpo.3-amd64   

Note that, as stated before, the update won't trigger the initrd rebuild::

    etc/kernel/postinst.d/initramfs-tools:
    I: update-initramfs is disabled (live system is running on read-only media).

In fact the new initrd is missing::

    cd /boot
    ls -al

    -rw-r--r-- 1 root root 23504103 Jul 14 16:58 initrd.img-4.9.0-7-amd64

    -rw-r--r-- 1 root root  5068656 Aug 27 08:20 vmlinuz-4.17.0-0.bpo.3-amd64
    -rw-r--r-- 1 root root  4224800 Jul  5 01:29 vmlinuz-4.9.0-7-amd64

So me must manually force its build::

    cd /boot
    mkinitramfs -o initrd.img-4.17.0-0.bpo.3-amd64 4.17.0-0.bpo.3-amd64

The task will add a new file::

    rw-r--r-- 1 root root 27882891 Oct  3 08:41 initrd.img-4.17.0-0.bpo.3-amd64

We now only need to call the *livengkupd* script::

    /sbin/livengkupd.sh 4.17.0-0.bpo.3-amd64

which will rewrite the second system partition::

    Writing /dev/sdb2 (UUID: 2018-10-02-14-46-02-00)
    xorriso 1.4.6 : RockRidge filesystem manipulator, libburnia project.

    Drive current: -outdev 'stdio:/dev/sdb2'
    Media current: stdio file, overwriteable
    Media status : is blank
    Media summary: 0 sessions, 0 data blocks, 0 data,  256m free
    Added to ISO image: directory '/'='/boot/temp'
    xorriso : UPDATE : 3 files added in 1 seconds
    xorriso : UPDATE : 3 files added in 1 seconds
    xorriso : UPDATE : Thank you for being patient. Working since 0 seconds.
    xorriso : UPDATE : Thank you for being patient. Working since 1 seconds.
    ISO image produced: 16273 sectors
    Written to medium : 16273 sectors at LBA 0
    Writing to 'stdio:/dev/sdb2' completed successfully.

If we reboot and give::

    uname -a

we can see the new running kernel::

   Linux debian 4.17.0-0.bpo.3-amd64 #1 SMP Debian 4.17.17-1~bpo9+1 (2018-08-27) x86_64 GNU/Linux

proving that now **liveng is capable of kernel update** on a ISO9660 system partition!!

