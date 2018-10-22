Kernel update improvements
==========================

So far, liveng is a Secure Boot compliant and persistent (with or without AES encryption) live operating system, capable of kernel update on a ISO9660 system partition with a proper Debian package-driven procedure.

A liveng system is finally quite ready, we just need to provide a way-out for the system in case something goes wrong when kernel-updating (for example if a power failure occurs when xorriso is writing).
 

Handle kernel update failures
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When normally updating the kernel on a Debian live operating system, only the content of / is modified (and persisted if on a persistent live), but the content of the outside-kernel and ouside-initrd files to which the bootloader links remains unchanged. On liveng, the postinst script of *linux-image-liveng-architecture-version* is responsible to force the initrd update (`update-initramfs <kernel-update.html#manually-updating-the-kernel>`_, usually undone being the filesystem a readonly ISO9660), while */sbin/livengkupd.sh* locates the second system partition and rewrites it with the new files, using xorriso. Task will take very few seconds::

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

but any power failure during this phase can be disastrous leaving the system in an unbootable state; this is why liveng comes with a second, `fallback <liveng-structure.html#brub-bootloader-uefi>`_, GRUB boot parameter. 

We need a "handler" for this, which may be an ad-hoc Systemd unit file or some simple lines of code in the (deprecated but always useful) */etc/rc.local* init script::

    # Rewrite kernel partition if started in fallback boot.
    if cat /proc/cmdline | grep -q liveng-fallback; then
        ( /usr/bin/dpkg --configure -a && /usr/bin/apt-get install --reinstall linux-image-liveng-architecture) &
    fi

Change `linux-image-liveng-architecture <kernel-update-debian.html>`_ according to your system's architecture (for example: *linux-image-liveng-amd64*).

Above code will "fix" the state of apt/dpkg (remember, we left it in an unconsistent state caused by a power failure) and will reinstall the liveng kernel package, which in its turn will re-trigger the initrd build and re-launch */sbin/livengkupd.sh*. 

