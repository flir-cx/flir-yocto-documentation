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
Anyone are allowed to download, study and possibly build artifacts from the provided source.

To be able to install and run the built artifacts on a target device (i.e. FLIR C5), the device needs to be 'unlocked'.

Download
--------
See [meta-flir-openmanifest](https://github.com/flir-cx/flir-yocto-openmanifest.git)

Building toolchain, individual packages or full target images
-------------------------------------------------------------
Provided that source is downloaded locally to a computer suitable for yocto
development, it is possible to build code for a FLIR C5.

FLIR yocto distribution is tested to build on an [Ubuntu](https://ubuntu.com/) host, ubuntu 14 or ubuntu 16.
Ubuntu 18 will most likely run into build problem(s), and is not recommended directly. Ubuntu 20 us untested.
Other linux distributions (that supports yocto development) might work, but is not tested. Other operating systems than linux (Windows, Apple OS...) could work using the docker build environment presented below. Not tested.

#### local host based build<br>
(Works with ubuntu 14, 16. Possibly with other linux distributions as well)
- Make sure that your local host has all the necessary additional .deb (or .rpm) packages installed (see yocto documentation)
- Create a local build environment and download code as described by [meta-flir-openmanifest](https://github.com/flir-cx/flir-yocto-openmanifest.git)
- Proceed as stated below "common procedure"
<br>

#### docker based build<br>
To isolate host distribution dependencies, a [docker](https://en.wikipedia.org/wiki/Docker_(software)) image is provided (part of this documentation repository)<br>
See [flir-yocto-builder docker](https://github.com/flir-cx/flir-yocto-documentation/blob/prework/docker/flir-yocto-builder/README.md) for usage.<br>
This image is based upon ubuntu 16 and has all necessary, additional .deb packages (needed for a yocto build) preinstalled.

To use it for building:
- Install docker environmant on your local host
- Build a *flir-yocto-builder* container as described by link above
- Create a local build environment and download code as described by [meta-flir-openmanifest](https://github.com/flir-cx/flir-yocto-openmanifest.git)
- Start *flir-yocto-builder* container as described by link above
- Proceed similar as for true local build

#### common procedure
First, the local build environment needs to be started.
Run:
~~~console
MACHINE=ec201 source ./flir-setup-release.sh -b build_ec201
~~~
(creates sub folder *build_ec201* if not present before and initiates it. First time you need to approve a license agreement. "current working directory" will also be set to build_ec201 )

Now it is possible to build code for the "ec201" using *bitbake* cxommand. Implicitely needed artifacts will be built when needed.<br> 
Note that the disk space requirements for a build is quite **large**.<br>
**60 GB** additional space is a **minimum** (!). 
