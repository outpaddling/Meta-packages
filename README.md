# Package Managers
Information and resources regarding package managers

## Background

Package managers are systems for installing, uninstalling, and upgrading
software packages in a clean, efficient, and organized manner.

Package managers install a software package along with all of its
dependencies (libraries and tools on which it depends) in one simple
command.  For example, building the MACS2 peak caller for bioinformatics
requires first installing a Python interpreter and the Cython Python-to-C
translator.  These are called *build dependencies*, and are only needed while
building the package.  They can be removed after MACS2 is installed.
Running MACS2 also requires the Numpy library for
linear algebra, a Python interface to the popular BLAS and LAPACK libraries.
This is called a *runtime dependency*, and it must remain installed along
with MACS2.

Each dependency may have build and runtime dependencies of its own, so
in total, there could be dozens to hundreds of dependencies that must
be installed, and possibly upgraded when we upgrade MACS2.  The sum
of all dependencies is called the *dependency tree*, since graphically,
it is structured like a tree.

If installing MACS 2 manually (the caveman method), one must first manually
install *an appropriate version* of every dependency.  When upgrading
MACS2, we must check all packages in the dependency tree to see if any
of them must be upgraded to support the new version of MACS2.
The entire process is a nightmare.  Nobody should be deploying software
this way in the 21st century.

Package managers take care of this for us.  They "know" what versions
of dependencies are acceptable.  They also contain patches necessary
to build or run the software on a specific operating system.  Instead
of spending days or weeks manually managing the dependency tree for
MACS2, we can install it with one simple command.  For example, to install
MACS2 on FreeBSD:

```
pkg install py39-macs2
```

To upgrade *all* of the packages on a FreeBSD system, including MACS2,
we simply do the following:

```
pkg upgrade
```

There are many package managers around today, including some that are
native to specific operating systems, such as Debian packages for
Debian Linux and derivatives like Ubuntu, FreeBSD ports for FreeBSD,
Portage for Gentoo, and Yum for Redhat Enterprise, just to name a few.

There are package managers specific to languages, such as Conda and
pip for Python, and CRAN and Bioconductor for R.  Language-specific
package managers also support packages written in other languages out
of necessity.  A python package may require a library written in C,
so the package manager must be able to install C libraries.

There are a few package managers that are neither operating system
specific nor language specific.  The only one that is fully portable to
all POSIX platforms is [pkgsrc](https:pkgsrc.org).  Pkgsrc is native
to the NetBSD operating system, but fully supports all POSIX compatible
platforms, including most Linux distributions and macOS.

## Operating Systems

I use FreeBSD for all of my work and personal computing.  FreeBSD is the
most reliable operating system I've used in my career (which began in
the 1980s), and I've used almost every popular Unix-like system around.

It also boasts the second-largest quality-controlled package collection,
behind Debian packages, according to
([https://repology.org/](https://repology.org/).  ( AUR and nix
are community repositories with a lower bar for inclusion. They do not
have the same level of quality control as Debian packages or FreeBSD ports. )

FreeBSD was the first open-source OS to fully integrate the ZFS file
system, which is highly advantageous for big-data work like bioinformatics.
Installing FreeBSD on any typical ZFS configuration is trivial.

If you don't already have a Unix-like system, I recommend giving FreeBSD
a try.  The basic installation (text-based) takes about 5 minutes.
This is all you will need for a server.

For workstations and laptops, you can install any of the popular
desktop environments in under an hour using
[https://github.com/outpaddling/desktop-installer/](https://github.com/outpaddling/desktop-installer/).
This is a different approach from systems with a graphical installer,
like Ubuntu Linux.  With FreeBSD, we first install the basic OS,
then install the desktop of our choice afterward.

If you don't have a space computer, you can also install FreeBSD under
a Virtual Machine monitor, such as Hyper-V, Parallels, Qemu/UTM, VirtualBox,
or VMWare.  I recommend VirtualBox, which is free, open-source, and
portable to many host platforms.  FreeBSD has VirtualBox guest additions
to add some conveniences such as mouse integration, screen resizing,
file sharing, etc.

## Our Package Managers

Most of the software on [https://github.com/outpaddling](https://github.com/outpaddling)
and [https://github.com/auerlab](https://github.com/auerlab) is available
in [FreeBSD Ports](https:https://www.freebsd.org/ports/)
and [pkgsrc](https:pkgsrc.org).

### FreeBSD Ports

As mentioned, I use FreeBSD for my work and development.  I also
maintain pkgsrc packages for users of other platforms such as
Linux, macOS, NetBSD, etc.

FreeBSD ports and pkgsrc are unusual among package managers in that they support
installing from both prebuilt "binary" packages, and building/installing the
same software from source.  The build/install from source process is
actually the same one used to create the binary packages, but unlike most
package managers, it is made easily accessible to the average user.

E.g., to install MACS2 on FreeBSD from binary packages, we would run:

```
pkg install py39-macs2
```

To install from source, we would run:

```
cd /usr/ports/biology/py-macs2
make install
```

Installing from source takes much longer in some cases, but is just as
easy for us as installing a binary package.  Installing from source has
three major advantages:

1. It allows the package manager to install software than cannot
be redistributed in binary for for licensing reasons, such as UCSC-userapps.

2. It allows the software to be built with non-standard options or features.
For example, if a software package has 3 optional features A, B, and C,
that must be selected at build-time, there are 7 different builds possible:
(None, A only, B only, C only, A and B, B and C, A and C).
A binary-only package manager would need to build and make available
7 different packages.  Now imagine that there are 10 optional features.
FreeBSD ports and pkgsrc offer one binary package
with default options, and let the user build all others from source.

3. Binary packages must be built to use only CPU features that virtually
all users have.  This generally means limiting the compiler to instructions
supported by hardware up to about 10 years old.  A binary package using
features only found in the latest-and-greatest CPU will not work on
older or cheaper CPUs.  Hence, binary packages won't offer optimal performance.
Building from source allows us to use all the features of our own CPU.
The binaries built won't be portable to lesser computers, but they will
be as fast as possible on this one.

    To use the latest CPU features supported by the compiler on FreeBSD,
    just add "CFLAGS+=-march-native" to the file /etc/make.conf.

### Pkgsrc

The pkgsrc package manager was derived from FreeBSD ports early on,
but in line with NetBSD's commitment to maximizing portability, it
was designed to support all POSIX platforms.

Pkgsrc does not require administrator privileges.  Any user with a normal
account on a Unix system can use pkgsrc in their own directory.

NetBSD users and users of a few other platforms for which binary packages
are supplied can install MACS2 as follows:

```
pkgin install py39-macs2
```

Users of *any* platform, or those who want an optimized, non-portable
build, can install from source as follows:

1.  First, install the pkgsrc system.  The auto-pkgsrc-setup script
    makes this easy on Linux, macOS, etc.

    curl -O http://netbsd.org/~bacon/auto-pkgsrc-setup
    chmod 755 auto-pkgsrc-setup
    ./auto-pkgsrc-setup
    
    Follow the instructions on the screen.  Default responses are
    suitable for most users.

2.  Join the pkgsrc-users email list:
    [http://netbsd.org/mailinglists/#pkgsrc-users](http://netbsd.org/mailinglists/#pkgsrc-users)
    Here you learn about pkgsrc from other users and experienced
    pkgsrc developers alike, ask questions, and eventually help other
    pkgsrc users.
    This is a fairly low-volume list, so you will not be inundated with
    irrelevant emails.

3.  Install the packages you want using the build-from-source method.
    Assuming you installed as a non-root user, chose the "current"
    pkgsrc tree, and accepted the default prefix:

    ```
    cd ~/Pkgsrc/pkgsrc/biology/py-macs2
    sbmake install
    ```
    
    It may take a while to build and install the entire dependency
    tree, but you should not have any problems.  If the build fails,
    contact the maintainer listed in the Makefile and/or post to
    pkgsrc-users.

To enable the non-portable features of your CPU during package builds,
just add "CFLAGS+=-march=native" to PREFIX/etc/mk.conf
(e.g. ~/Pkgsrc/pkg/etc/mk.conf).

## Meta-packages

Meta-ports (in FreeBSD ports) and meta-packages (in pkgsrc) are frameworks
that do not install any files of their own, but simply bundle other
ports/packages by listing them as runtime dependencies.

## rna-seq

The rna-seq meta-port in FreeBSD ports installs all software needed for
a typical RNA-Seq differential analysis.

```
pkg install rna-seq

or

cd /usr/ports/biology/rna-seq
make install
```

For pkgsrc configured using auto-pkgsrc-setup:

```
pkgin install rna-seq

or

cd ~/Pkgsrc/pkgsrc/biology/rna-seq
sbmake install
```

## atac-seq

The atac-seq meta-port in FreeBSD ports installs all software needed for
a typical ATAC-Seq differential analysis.

```
pkg install atac-seq

or

cd /usr/ports/biology/atac-seq
make install
```

For pkgsrc configured using auto-pkgsrc-setup:

```
pkgin install atac-seq

or

cd ~/Pkgsrc/pkgsrc/biology/atac-seq
sbmake install
```

