=================================================================
 Building the ARM GNU (bare metal) toolchain on Linux ARMv7 host
=================================================================

You'll need at least a 32GB card or larger device to build on.  Also run::

  $ sudo apt-get autoclean

after the updates described below.

1) Raspbian jessie

Add staging to jessie repos in sources.list, I also added Linaro overlay and crosstools sources but that shouldn't matter.  Edit sources.list and add this::

  deb http://mirrordirector.raspbian.org/raspbian/ jessie-staging main contrib non-free rpi
  deb-src http://archive.raspbian.org/raspbian/ jessie-staging main contrib non-free rpi

2) Ubuntu wily or higher

Leave sources.list as-is, continue.

Now run apt-get update, apt-get upgrade, and apt-get dist-upgrade.  Reboot and install the build preqrequisites::

  $ sudo apt-get install python-software-properties build-essential realpath libxml2-utils python-tempita python2.7-dev
  $ sudo apt-get install apt-src scons p7zip-full gawk gzip perl autoconf m4 automake libtool libncurses5-dev gettext gperf dejagnu expect tcl autogen guile-1.6 flex flip bison tofrodos texinfo g++ gcc libgmp3-dev libmpfr-dev debhelper texlive texlive-extra-utils
  $ sudo apt-get install sed wget coreutils unzip texi2html texinfo docbook-utils gawk python-pysqlite2 diffstat help2man make gcc build-essential g++ desktop-file-utils chrpath libxml2-utils xmlto apache2-utils info2man libebook-tools-perl libpod-2-docbook-perl

Note that many of these are probably already installed, but we need to pick up
some rather obscure build deps for this configuration.

Use the modified build scripts and --skip args from the upstream tarball as shown below.

Rpi2/3 (Raspbian) works, but is slow and will crater with more than one job (even using a swapfile).

A Samsung (snow) chromebook running chrubuntu also works, and should take -j2. Note this should work without any swap, as wily has the buggy version of swapon:

Download the source tarball for the GNU ARM toolchain and unpack it::

  $ wget https://launchpad.net/gcc-arm-embedded/5.0/5-2016-q3-update/+download/gcc-arm-none-eabi-5_4-2016q3-20160926-src.tar.bz2
  $ tar xpf gcc-arm-none-eabi-5_4-2016q3-20160926-src.tar.bz2

.. warning:: If building on rpi or other limited platform, edit build_common.sh
             before you start - comment out JOBS=2 and uncomment JOBS=1 near the
             bottom of the file (see comments).

Now copy the ``*.sh`` files from this repo to the above unpacked source directory::

  $ cd gnu-arm-toolchain/
  $ cp ``*.sh`` ../gcc-arm-none-eabi-5_4-2016q3-20160926/
  $ cd ../gcc-arm-none-eabi-5_4-2016q3-20160926/
  
  $ time ./build-prerequisites.sh --skip_steps=mingw3
  
  real    8m37.118s
  user    6m55.460s
  sys     1m8.600s
  
  $ time ./build-toolchain.sh --skip_steps=manual,mingw32
  
  real    447m9.451s
  user    508m12.645s
  sys     103m57.105s
  
  yay!

YMMV, it will take "forever" but should work on anything with at least 1GB of RAM and 32GB of rootfs/storage.  Stick with JOBS=1 on anything with less than 2GB RAM, which should allow -j2 or even 3.
