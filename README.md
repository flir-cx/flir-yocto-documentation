flir-cx
=======

Overview
--------
flir-cx is a github project that contains the open source distribution for
the firmware in [FLIR C5](https://www.flir.eu/products/c5/) infrared camera 
(and its variants).

The FLIR C5 product is based upon a FLIR lepton module for the IR image generation, It also has a visual camera module. Both these image modules are connected to a (FLIR designed) small computer board based upon [i.MX7ULP](https://www.nxp.com/products/processors-and-microcontrollers/arm-processors/i-mx-applications-processors/i-mx-7-processors/i-mx-7ulp-family-ultra-low-power-with-graphics:i.MX7ULP)

Software running on this FLIR board (called ec201) is divided into a M4 part
(closed source) and a A7 part.
The cortex A7 runs linux. 

This linux distribution is based upon yocto. 
FLIR has added its own yocto layers to the yocto platform
One layer called `meta-flir-base` contains FLIR additions to open source recipes.

(A 2:nd layer `meta-flir-internal` exists that contains FLIR private recipes)

It is fully possible to build and install a working linux platform without the
private meta-flir-internal layer. 
How to do this is described on these page(s).

On top of the linux platform a closed source FLIR application executes.
It provides a graphical user interface based upon the Qt framework.
This FLIR application is what the normal FLIR C5 user interacts with using the LCD with its touch screen.

Limitations
-----------
The source code in flir-cx is provided 'as is'. It represents the used code in delivered FLIR C5 products. 
Updates to these git repos will be made if/when new versions of the software are developed (within a reasonable delay in time.)

Not all source for code used within FLIR C5 is present, only what is considered 'open'
Closed source licenses apply to:
- Code running within the Cortex M4 part of i.MX7
- A few parts of the linux rootfs platform 
- Camera application code

These parts are distributed in binary form (at factory) and are installed into the FLIR C5 non-volatile memory.

Open source for FLIR C5 is accessed using the [meta-flir-openmanifest](https://github.com/flir-cx/flir-yocto-openmanifest.git) repo (and applying proper commands). 
Anyone is allowed to download, study and possibly build artifacts from the provided source.

To be able to install and run the built artifacts on a target device (i.e. FLIR C5), the specific device needs to be 'unlocked'. (see below)

Download
--------
See [meta-flir-openmanifest](https://github.com/flir-cx/flir-yocto-openmanifest.git)

Building toolchain, individual packages or full target images
-------------------------------------------------------------
Provided that source is downloaded locally to a computer suitable for yocto
development, it is possible to build code for a FLIR C5.

FLIR yocto distribution is tested to build on an [Ubuntu](https://ubuntu.com/) host, ubuntu 14 or ubuntu 16.
Ubuntu 18 will most likely run into build problem(s), and is not recommended directly. Ubuntu 20 is untested.
Other linux distributions (that supports yocto development) might work, but is not tested. Other operating systems than linux (Windows, Apple OS...) could work using the docker build environment presented below. Not tested.

#### local host based build<br>
(Works with ubuntu 14, 16. Possibly with other linux distributions as well)
- Make sure that your local host has all the necessary additional .deb (or .rpm) packages installed (see relevant yocto documentation, yocto 2.5 - "Sumo")
- Create a local build environment and download code as described by [meta-flir-openmanifest](https://github.com/flir-cx/flir-yocto-openmanifest.git)
- Proceed as stated below "common procedure"
<br>

#### docker based build<br>
To isolate host distribution dependencies, a [docker](https://en.wikipedia.org/wiki/Docker_(software)) image is provided (part of this documentation repository)<br>
See [flir-yocto-builder docker](https://github.com/flir-cx/flir-yocto-documentation/blob/master/docker/flir-yocto-builder/README.md) for usage.<br>
This image is based upon ubuntu 14 and has all necessary, additional .deb packages (needed for a yocto build) preinstalled.

To use it for building:
- Install docker environmant on your local host
- Build a *flir-yocto-builder* container as described by link above
- Create a local build environment and download code as described by [meta-flir-openmanifest](https://github.com/flir-cx/flir-yocto-openmanifest.git)
- Start *flir-yocto-builder* container while in root of local build environment as described by link above
- Proceed as stated by "common procedures"

#### common procedure
First, the local build environment needs to be initiated.
Run (while in root of local build environment):
~~~console
MACHINE=ec201 source ./flir-setup-release.sh -b build_ec201
~~~
(creates sub folder *build_ec201* if not present before and initiates it. First time you need to approve a license agreement. "current working directory" will also be set to _build_ec201_ )

Now it is possible to build code for the "ec201" using *bitbake* commands. Implicitely needed artifacts will be built when needed.<br>
Main build target would be *flir-image-sherlock*. (i.e. *bitbake flir-image-sherlock*) This recipe will build almost all components, including the linux kernel and a standard populated rootfs. 
Note that the disk space requirements for a build is quite large.<br>
**70 GB** additional space is a **minimum** (!).<br>
Other targets than *flir-image-sherlock* might be built. Study the yocto source tree.

Target connection
-----------------
To download and execute generated code on a FLIR cx target, A USB connection is recommended.<br>
Wi-Fi is also supported, See [WiFi backdoor to target device](https://github.com/flir-cx/flir-yocto-documentation/blob/master/backdoor.md)<br>

#### Warning:
Note: To access the FLIR C5 device as a developer, you will need to _**unlock**_ the device. Even if possible, this is **NOT** recommended.<br>
You will be on your own, product warranty will be limited.<br>
FLIR cannot take responsibility for the software quality anymore if uncontrolled software has been installed onto the device. Even if nothing is installed, this will be impossible to tell if the device has been unlocked.

#### Connection:
If you still want to connect to your device as a developer, please read more in [USB RNDIS and shell connection](https://github.com/flir-cx/flir-yocto-documentation/blob/master/rndis.md)<br>

Target software disk layout
---------------------------
The "disk"on the ec201 board is actually a 4 MB eMMC.
This "disk" is partitioned as:

~~~console
eMMC layout:
-------------------------------------------------------------------------------------------------------------------------------------
| u-boot | u-boot env | recovery | rootfs1 | rootfs2 | rootfsrw | apps, /FLIR/usr | data, /FLIR/system | storage, /FLIR/images      |
-------------------------------------------------------------------------------------------------------------------------------------
                        ^                                                                               ^
                        |                                                                               |
                       mmcblk0 ...                                                                     mmcblk7 
		       
root@ec201-0A13DC:~# parted /dev/mmcblk0 print
Model: MMC 004GA0 (sd/mmc)
Disk /dev/mmcblk0: 3959MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system  Name      Flags
 1      2097kB  44.0MB  41.9MB  fat16        recovery  msftdata
 2      44.0MB  581MB   537MB   ext4         rootfs1   msftdata
 3      581MB   1118MB  537MB   ext4         rootfs2   msftdata
 4      1118MB  1252MB  134MB   ext4         rootfsrw  msftdata
 5      1252MB  1789MB  537MB   ext4         apps      msftdata
 6      1789MB  2326MB  537MB   ext4         data      msftdata
 7      2326MB  3959MB  1634MB  ext4         storage   msftdata

root@ec201-0A13DC:~#
root@ec201-0A13DC:~# mount | grep -e mmc -e " / "
/dev/mmcblk0p4 on /aufs/rwfs type ext4 (rw,relatime,data=ordered)
/dev/mmcblk0p3 on /aufs/rofs type ext4 (ro,relatime,data=ordered)
none on / type overlay (rw,relatime,lowerdir=/tmp/aufs/rofs,upperdir=/tmp/aufs/rwfs/upper,workdir=/tmp/aufs/rwfs/workdir)
/dev/mmcblk0p5 on /FLIR/usr type ext4 (rw,noatime,data=ordered)
/dev/mmcblk0p6 on /FLIR/system type ext4 (rw,noatime,data=ordered)
/dev/mmcblk0p7 on /FLIR/images type ext4 (rw,relatime,data=ordered)
/dev/mmcblk0p7 on /srv/sftp type ext4 (rw,relatime,data=ordered)
root@ec201-0A13DC:~# 
~~~
Note that the root filesystem ("none on / ") is mounted as an overlay file system.<br>
Partition 2 and 3 contains 2 versions of a readonly .ext4 image (from flir-image-sherlock). <br>
On top of this there is a "overlay" "rw" file system that contains changes to the base image.<br>
/FLIR mount points contains FLIR camera application + data (closed source).<br>
Please do not touch content within these folders deliberately, except content below /FLIR/images that could be considered "open"

Installable software components
-------------------------------
From within yocto, it is possible to build "all" the open source content intended for ec201.<br>

All software pre-populated into _flir-image-sherlock_ is rebuildable using https://github.com/flir-cx/flir-yocto-openmanifest <br>
It is possible to replace single or multiple package(s), or even replace the complete rootfs (and/or u-boot bootloader) in your device using the techniques described in this document collection.<br>

Specifically this is true for GPLv3/LGPLv3 licensed packages.<br>
(Typically _umtp_responder_, _bash_ shell and _Qt libraries_.)<br>

To find out which packages are really installed into _flir-image-sherlock_ (and thus replaceable), generate your own package database using information as described by: [package management addition to rootfs](https://github.com/flir-cx/flir-yocto-documentation/blob/master/package-management-addition.md).<br>

#### rootfs
As indicated, build of a "flir-image-sherlock" will generate an artifact (_flir-image-sherlock.ext4_) that could be used to replace the complete base image within rootfs1/rootfs2.
This image file will contain a linux kernel, device tree and a populated rootfs.

See [replace-rootfs](https://github.com/flir-cx/flir-yocto-documentation/blob/master/replace-rootfs.md) for details.<br>
However, replacing the complete rootfs is **NOT** recommended (even if it technically has been tested at FLIR and "should" work).
- You will lose the non-public parts of the rootfs.<br>
- If your generated binary .ext4 file is "bad", or if installation fails for some reason, it is likely that you will brick your device.

#### u-boot
Will be generated implicitely by running _bitbake flir-image-sherlock_ or by explicitely running: _bitbake u-boot_<br>
See [replace-u-boot](https://github.com/flir-cx/flir-yocto-documentation/blob/master/replace-u-boot.md) for details.<br>
However, replacing u-boot is **NOT** recommended (even if it technically has been tested at FLIR and "should" work).
- If your generated binary u-boot.imx file is "bad", wrong file used or if installation fails for some reason, it is likely that you will brick your device.

#### rootfs installable packages
rootfs as defined by _flir-image-sherlock_ contains a large number of various packages.<br> For details, see source files in _meta-flir-base_<br>
The file _meta-flir-base/conf/distro/flir.conf_ declares: _PACKAGE_CLASSES = "package_ipk"_ as opposed to "rpm").<br>
This means that the packages that are built within the flir-yocto environment will generate components in the format ".ipk".<br>


_flir-image-sherlock_ (among other packages) contains a package called _okpg_.<br>
opkg is a utility to manage installation/removal of (.opk/.ipk) packages in runtime.<br>
(Brief background could be found for instance [here](https://en.wikipedia.org/wiki/Opkg).)<br><br>
With opkg, package installation state is kept in a "database" inside the rootfs at _/var/lib/opkg_ (text based).<br>
Even if would be possible to pre-populate this database with all packages installed into _flir-image-sherlock_ at build time, FLIR selected to skip this prepopulation (!).<br>
The reason is that this "database" will take a significant part of the disk space. For _flir-image-sherlock_ the required space for a populated opkg database would be around 7 MB.<br>

If you connect to a (unlocked, not manipulated) FLIR C5 camera and try command _opkg list_ you would typically find 2 installed packages;
~~~console
root@ec201-0A13DC:~# opkg list
appkit - 1.0.5-r280860
prodkit - 1.0.3.15-r280286
root@ec201-0A13DC:~#
~~~
These 2 packages are added on top of the installed rootfs, thus visible.<br>
(Occupied disk space for the opkg database is now ~60K.)

To add/check what a pre-populated opkg database would look like, you may want to look at [package management addition to rootfs](https://github.com/flir-cx/flir-yocto-documentation/blob/master/package-management-addition.md).<br>

##### Installing/replacing packages in device
See [Working with packages](https://github.com/flir-cx/flir-yocto-documentation/blob/master/working-with-packages.md) about building and installing ipk packages to your target.<br>

#### Non open source parts of FLIR C5
Note that the "camera application" that makes the FLIR C5 work as an infrared camera is **NOT** open source.<br>
You are not allowed to try to manipulate any files within folders _/FLIR/usr_ and/or _/FLIR/system_ (_/FLIR/images_ is OK).<br>
Failure to follow this guideline will be a license violation to the "flir application".<br>
That said, feel free to manipulate other files in the root file system (at your own responsibility) or just use the provided information for educational purposes.


