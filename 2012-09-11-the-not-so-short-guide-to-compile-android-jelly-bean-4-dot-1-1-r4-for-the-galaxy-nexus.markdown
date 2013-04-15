---
layout: post
title: "The not so short guide to compile Android Jelly Bean 4.1.1_r4 for the Galaxy Nexus"
date: 2012-09-11 02:30
comments: true
external-url:
categories: Android
---

Introduction
------------

<span itemprop="description">Back to the beginning of this year, I succeded in compiling ICS AOSP
4.0.3_r1 for the Galaxy Nexus. At that time, I didn't have much time to post a full entry on the
whole compilation process (including the diﬀerent hacks needed to get a fully functional ROM).

With the Jelly Bean AOSP release and my third ROM compilation, I've succesfully compiled 4.1
back in July. Now we have 4.1.1_r4 and it's due time for me to document the process using Debian
GNU/Linux.</span>

Without the [Android Open Source Project](http://source.android.com/), nothing has been possible and
the website clearly describes how to proceed to get the code and compile it for a supported
target. I will not rewrite the provided documentation but I'd rather provide a  guide on
how I have obtained my own ROM. Feel free to review and I will update accordingly.

*DISCLAIMER: Please do not do anything described in this post if you have no idea of what you are
doing! I will not be held responsible for bricking your phone or invalidate your device warranty! Do
not forget to backup your data before doing anything since any issue may involve a factory reset!*

My ﬁrst advices are:

* Browse the AOSP website
* Read that whole post a ﬁrst time before doing anything
* Unlock your bootloader
* Flash clockworkmod
* Backup your ROM

Get the Android Open Source Project code
----------------------------------------

Follow the instructions given by the ["Downloading the Source" section](http://source.android.com/source/downloading.html) of the AOSP website.

    $ mkdir ~/foo/bar/android
    $ cd ~/foo/bar/android
    $ repo init -u https://android.googlesource.com/platform/manifest -b android-4.1.1_r4
    $ repo sync

These commands apply even to an existing Android source tree but you will need to clean up your source tree from a previous build by using:

    $ make clobber

That's the ﬁrst step to get a JRO03L rom…

Prerequisites for hardware support
----------------------------------

To get a fully functional rom, you need to get the hardware support. Unfortunately, we only have access to
the binaries distributed for the JRO03H rom [Galaxy Nexus (GSM/HSPA+) binaries for Android 4.1.1 (JRO03H)](https://developers.google.com/android/nexus/drivers#magurojro03h):

    $ mkdir ~/foo/android_prereq
    $ cd ~/foo/android_prereq
    $ export BINARIES="broadcom-maguro-jro03h-4cc54d09,imgtec-maguro-jro03h-827bcb4c,invensense-maguro-jro03h-682067a4,samsung-maguro-jro03h-0655880b"
    $ curl -# -OOOO https://dl.google.com/dl/android/aosp/{${BINARIES}}.tgz
    $ md5sum *
    b05a41ed3096c5f19ebc5b172f034e93  broadcom-maguro-jro03h-4cc54d09.tgz
    cabedd37a42a9cbe47e1702a122421e0  imgtec-maguro-jro03h-827bcb4c.tgz
    c575cc3a712ef61d2ef5eb5e08f054eb  invensense-maguro-jro03h-682067a4.tgz
    cda04a52492ee9762196248abb2834d1  samsung-maguro-jro03h-0655880b.tgz
    $ unset BINARIES

Retrieve the installation scripts:

    $ cd ~/foo/bar/android
    $ find ~/foo/android_prereq -maxdepth 1 -type f -exec tar xzvf {} \;

Finally, it is the time to extract the binaries to the source tree:

    $ for i in `ls -1 extract-*-maguro.sh`; do ./$i; done;

*DISCLAIMER: You must accept the terms of each of the licences before going further!*


Initialize the build environment
--------------------------------

In the past, I've always compiled a `userdebug` rom. Now, I would like to get a `user` rom since I'm
happy with an unthemed and odexed rom.

    $ . build/envsetup.sh
    $ lunch full_maguro-user

    ============================================
    PLATFORM_VERSION_CODENAME=REL
    PLATFORM_VERSION=4.1.1
    TARGET_PRODUCT=full_maguro
    TARGET_BUILD_VARIANT=user
    TARGET_BUILD_TYPE=release
    TARGET_BUILD_APPS=
    TARGET_ARCH=arm
    TARGET_ARCH_VARIANT=armv7-a-neon
    HOST_ARCH=x86
    HOST_OS=linux
    HOST_OS_EXTRA=Linux-3.2.0-3-amd64-x86_64-with-debian-wheezy-sid
    HOST_BUILD_TYPE=release
    BUILD_ID=JRO03L
    OUT_DIR=out
    ============================================


If you prefer a userdebug build, just do:

    $ . build/envsetup.sh
    $ lunch full_maguro-userdebug

    ============================================
    PLATFORM_VERSION_CODENAME=REL
    PLATFORM_VERSION=4.1.1
    TARGET_PRODUCT=full_maguro
    TARGET_BUILD_VARIANT=userdebug
    TARGET_BUILD_TYPE=release
    TARGET_BUILD_APPS=
    TARGET_ARCH=arm
    TARGET_ARCH_VARIANT=armv7-a-neon
    HOST_ARCH=x86
    HOST_OS=linux
    HOST_OS_EXTRA=Linux-3.2.0-3-amd64-x86_64-with-debian-wheezy-sid
    HOST_BUILD_TYPE=release
    BUILD_ID=JRO03L
    OUT_DIR=out
    ============================================


Flash the bootloader and the baseband ﬁrmware to the latest versions available
-------------------------------------------------------------------------------

Get the latest factory image "yakju" for Galaxy Nexus "maguro" (GSM/HSPA+) JR003C

    $ mkdir ~/foo/android_prereq/jro03c
    $ cd ~/foo/android_prereq/jro03c
    $ curl -#O https://dl.google.com/dl/android/aosp/yakju-jro03c-factory-3174c1e5.tgz

We will need fastboot to flash images to the device. More over, since it's the time to compile some
host utilities, we take the opportunity to get `adb` (necessary to interact with the device for
debug purposes) and the `simg2img` (usefull to convert a sparse image ﬁle to a raw image ﬁle).

    $ cd ~/foo/bar/android
    $ make fastboot adb simg2img

The resulting host binaries could be found in the `~/foo/bar/android/out/host/linux-x86/bin`
directory.

If your bootloader is not already unlocked, you will need to boot in fastboot mode:

{% blockquote AOSP Website, Booting into fastboot mode section for the maguro device %}
Press and hold both Volume Up and Volume Down, then press and hold Power.
{% endblockquote %}

We deﬁne a linux local alias to simplify the use of the brand new fastboot (using the `_aosp`
suﬃx).

    $ alias fastboot_aosp=/home/renaud/git/android_ics/out/host/linux-x86/bin/fastboot

Let's get and flash the bootloader and the baseband from the JRO03C factory to the actual device:

    $ cd ~/foo/android_prereq
    $ tar xzvf yakju-jro03c-factory-3174c1e5.tgz
    $ tar xzf yakju-jro03c-factory-3174c1e5.tgz
    $ cd yakju-jro03c
    $ fastboot_aosp flash bootloader bootloader-maguro-primelc03.img
    $ fastboot reboot-bootloader
    $ fastboot flash radio radio-maguro-i9250xxlf1.img
    $ fastboot reboot-bootloader

The bootloader now display:

    […]
    BOOTLOADER VERSION - PRIMELC03
    BASEBAND VERSION - I9250XXLF1
    […]

Getting root or injecting `su` and `Superuser` to the source tree
-----------------------------------------------------------------

Make a backup of the original `su` source:

    $ mkdir ~/android_backup
    $ cp -rf ~/foo/bar/android/system/extras/su .

Then inject [ChainsDD/su-binary](https://github.com/ChainsDD/su-binary) using my slightly modiﬁed
fork:

    $ cd ~/foo/bar/android/system/extras
    $ git clone https://github.com/nibua-r/su-binary.git su

I want the `su` binary to be compiled in the very same time of the full AOSP
build. As a consequence, I have modiﬁed the original `Android.mk` ﬁle by setting
`LOCAL_MODULE_TAGS := optional` and adding the `su` module to `PRODUCT_PACKAGES` into the
`build/target/product/core.mk` ﬁle to force the inclusion.

I want to have the [ChainsDD/Superuser](https://github.com/ChainsDD/Superuser) apk included into the
build as a system app:

    $ cd ~/foo/bar/android/packages/apps
    $ git clone https://github.com/nibua-r/Superuser.git

and add `Superuser` to `PRODUCT_PACKAGES` into the `build/target/product/core.mk`.

You will need to generate your own certiﬁcate for Superuser using:

    $ cd ~/foo/bar/android
    $ development/tools/make_key superuser '/C=US/ST=State/L=Location/O=YourOrg/OU=WhateverYouWant/CN=WhateverYouWant/emailAddress=root@example.org'
    $ mv superuser.pk8 build/target/product/security/

If `superuser.pk8` is not there, the compilation process will fail… Don't go too far away from your
computer since the Superuser compilation will require your password to get the build done.

Time to compile
---------------

You'll need to read
[the building section of the AOSP website](http://source.android.com/source/building.html) as a
prerequisite.

After setting `ccache` to ﬁt with your resources, just do:

    $ cd ~/foo/bar/android
    $ make -j4

You normally end up with flashable images located in `out/target/product/maguro/`.
The whole compilation process took a very long time on my machine… so be it and be warned!

Flash the images to the devices
-------------------------------

Assuming the `fastboot_aosp` alias is deﬁned and the battery of your device is decently charged,
just flash the whole system with:

    $ fastboot_aosp flashall

If you upgrade from a previous Android version, you should wipe your data using the `-w` switch:

    $ fastboot_aosp -w flashall

The whole flashing process took about 1 min. to ﬁnish on rebooting the device. Congrats! But wait,
this is not the end…

*If you have clockworkmod installed on your recovery partition, flashall will overwrite it with the
 nominal android recovery.*

Flash or reflash clockworkmod recovery (for later use)
------------------------------------------------------

The clockworkmod recovery enable us to backup and/or restore our ROMs and apply zip update ﬁles.

Get the [Koushik Dutta's clockworkmod recovery](http://www.clockworkmod.com/rommanager)
([direct link ATTOW](http://download2.clockworkmod.com/recoveries/recovery-clockwork-touch-6.0.1.0-maguro.img)).


Some hardware parts are not properly supported!
-----------------------------------------------

If you test the gps or the camera, you certainly are already grumbling…

Some binaries are distributed by Google, some aren't and since the famous `extract-files.sh` have
been suppressed from this release, you will have to hunt down some missing libraries. You'll need to
play with `find`/`sort`/`diff` combos to ﬁgure out which one are needed.

OK, but on which reference data? Remember the `simg2img` host compilation? Time to use it…

Some inoﬀensive linux commands:

    $ simg2img system.img system.img.raw
    $ mkdir mnt-point
    $ sudo mount -t ext4 -o loop system.img.raw mnt-point/

*Please read the following section before hunting down the missing ﬁles…*

Where are my GApps!
-------------------

The GApps are not-so-useless on Android but they are not Open Source and therefore not distributed
by Google AFAIK… Considering the situation, Google seems flexible, up to now, about the various
gapps packagings out there…

The short version: Go get a gapps update ﬁle on [Goo.im/gapps](http://goo.im/gapps/)
(`gapps-jb-20120726-signed.zip` ATTOW). Use `adb` to put that ﬁle on the device storage, reboot to
clockworkmod and apply…

You should (in not *must*) locally unzip the ﬁle to see what's into it *before flashing and before
processing the previous section…*

Conclusion
----------

That post is more a personal log than a real guide and although some parts could be automated, the
manual toying is a full learning experience!

Have fun!
