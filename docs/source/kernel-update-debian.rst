Kernel update, the Debian way
=============================

So far, liveng is a Secure Boot compliant and persistent live operating system, capable of kernel update on a ISO9660 system partition.

We must now **package all the code previously given programmatically "by hand" into a couple of Debian (.deb) packages**. 
While users who simply need to improve a standard Debian live can follow all the previously described steps "by hand" at every kernel release (i.e. on every *linux-image-architecture* package release), if you want to **create a liveng-derived operating system**, you will need to **host liveng kernel-update related packages in your package repository**.

On a Debian system, any *linux-image-architecture-version* update is started by an update of the meta package *linux-image-architecture* (for example: *linux-image-amd64* on x64 machines), which depends on the real *linux-image-architecture-version* package, so updating *linux-image-architecture* will download and install *linux-image-architecture-version*.

Instead of relying on the *linux-image-architecture* meta-package for a kernel update, as distribution maintainers we need to install and update the *linux-image-liveng-architecture* meta-package, which:

    * depends on the *linux-image-liveng-common* package, which installs the *livengkupd.sh* script;
    * depends (or pre-depends) on the real liveng kernel package (*linux-image-liveng-architecture-version*).

*linux-image-liveng-architecture-version* must have a *postinst* file which mimes what `previously done <kernel-update.html>`_:

    * triggers the initrd rebuild;
    * calls the *livengkupd.sh* script with the correct kernel-version parameter. *livengkupd.sh* will then locate the second system partition and rewrite it with the new files, using xorriso.

At the end of the day, *linux-image-liveng-architecture-version* is exactly like a standard *linux-image-architecture-version* but adds a special *postinst* file. 

We are not going to give further explanations, bacause of course distributions' maintainers already well know how to create and manage Debian packages and a package repository.


