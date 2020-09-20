Working with packages
=====================

#### yocto generated packages
When you successfully _bitbake flir-image-sherlock_ (or "bitbake" with other recipes), you will find generated ".ipk" installable package files below _build_ec201/tmp/deploy/ipk_<br>
Below here there is an additional subdirectory level defining "classes" of packages.<br>

| Folder name            | targeted content                                  |
|------------------------|---------------------------------------------------|
| all                    | all architectures<br>                             |
| cortexa7hf-neon        | arm packages (non ec201 specific)                 |
| cortexa7hf-neon-mx7ulp | arm packages (non ec201 specific, mx7ulp specific)|
| ec201                  | arm packages, ec201 specific                      |

As an example, _flir-image-sherlock_ content is defined by file _meta-flir-base/recipes-flir/images/flir-image-sherlock.bb_<br>
This _.bb_ file contains (among other things) addition of a package called _umtp-responder_<br>
(umtp-responder is a software service that implements "usb media transfer protocol" i.e. file access using USB).<br>
The package .ipk installer for _umtp-responder_ could be found below _build_ec201/tmp/deploy/ipk/cortexa7hf-neon_:
~~~console
$ grep umtp-responder sources/meta-flir-base/recipes-flir/images/flir-image-sherlock.bb
...
    udev-extraconf \
    umtp-responder \
    ${WEB_PACKAGES} \
    videorender-service \
...
$
$ find build_ec201/tmp/deploy/ipk/ -name "umtp-responder_*"
build_ec201/tmp/deploy/ipk/cortexa7hf-neon/umtp-responder_1.0-r1_cortexa7hf-neon.ipk
$
~~~
If you look more closely in the deploy folder, you will find siblings of this package;  _umtp-responder-**dbg**\_1.0-r1_cortexa7hf-neon.ipk_ and _umtp-responder-**dev**\_1.0-r1_cortexa7hf-neon.ipk_ <br>
If you want to know more about the content, you may take a closer look at the "work" folder that bitbake uses to build (in this case) _umtp-responder_:<br>
_build_ec201/tmp/work/cortexa7hf-neon-oe-linux-gnueabi/umtp-responder/1.0-r1/_<br>
Here you will find a sub folder; _packages-split_ where the content of the various generated packages could be examined.<br>

~~~console
$ tree tmp/work/cortexa7hf-neon-oe-linux-gnueabi/umtp-responder/1.0-r1/packages-split/umtp-responder
tmp/work/cortexa7hf-neon-oe-linux-gnueabi/umtp-responder/1.0-r1/packages-split/umtp-responder
├── etc
│   └── umtprd
│       └── umtprd.conf
└── sbin
    └── umtprd

3 directories, 2 files
$
~~~

##### So how do I find the work folder for a package?
To find out where the work folder is for a specific package, you may run a special bitbake command in your yocto build shell; _bitbake \<recipe\> **-c devshell**_<br>
A subshell is then opened inside the work folder of that recipe. Use _pwd_ to find out the location.<br>

#### overlay rootfs understanding
Before installing any packages into your FLIR C5 device, you should know something on the file system you are installing packages into.<br>

The FLIR C5 uses a overlayfs type of rootfs. This must be taken into account in some aspects if/when you try to install or remove your own built packages.

The read-only "underlay" contains the file system defined by _flir-image-sherlock_<br>

(In a factory delivered FLIR C5 camera, the _flir-image-sherlock_ image based upon code in _meta-flir-base_ layer is extended with stuff from a non public _meta-flir-internal_ layer.<br>
Typically this is about automated software updates, certificates and cloud login/sync.<br>
Should you choose to install your own _flir-image-sherlock_ .ext4 image from your yocto build environment, you will lose the stuff from _meta-flir-internal_ )<br>
But the FLIR C5 camera should still work.)<br>

On top of the "underlay" there is an "overlay" mounted as read-write. Initially this is an empty ext4 file system partition.<br>
All changes to the mounted rootfs will be seamlessly written as files into this overlay.<br>

You should be able to access the "underlay" and "overlay" data directly at mount points _/aufs/rofs_ + _/aufs/rwfs/upper_. 
~~~console
root@ec201-0A13DC:~# mount | grep aufs | grep -v sftp
/dev/mmcblk0p4 on /aufs/rwfs type ext4 (rw,relatime,data=ordered)
/dev/mmcblk0p2 on /aufs/rofs type ext4 (ro,relatime,data=ordered)
none on / type overlay (rw,relatime,lowerdir=/tmp/aufs/rofs,upperdir=/tmp/aufs/rwfs/upper,workdir=/tmp/aufs/rwfs/workdir)
root@ec201-0A13DC:~#
~~~
(The /aufs/rofs mount will toggle between /dev/mmcblk0p2 and /dev/mmcblk0p3  if/when you install a new version of the "underlay" - _flir-image-sherlock_ ).

##### What is the impact of a overlayfs?
1. If you write a new file somewhere into the combined file system, the file will actually be written into the _overlay_. It will be present within the combined file system.<br>
2. If you delete a "new" file, it will be deleted from the _overlay_, and will not be seen anymore in the combined file system.<br>
3. If you update a file that is already present in the _underlay_, the changed version of the file will be stored in the _overlay_.<br>
The changed version is the file seen in the combined file system.
4. If you delete a file that also exist in the _underlay_ (regardless if updated, and thus also present in _overlay_), a special file with the same name will be written into the overlay to mark that the file is not there.<br>
The file does not show up anymore within the combined file system.<br>
The mark is visible within the overlay.<br>
A file with the same name as the deleted will exist. If using _ls -l_ you will see that the file is a "character special", no access rights flag, owned by root.
~~~console
root@ec201-0A13DC:/etc# rm /etc/os-release 
root@ec201-0A13DC:/etc# ls /etc/os-release
ls: /etc/os-release: No such file or directory
root@ec201-0A13DC:/etc# ls -l /aufs/rwfs/upper/etc/os-release
c---------    1 root     root        0,   0 Sep 17 15:17 /aufs/rwfs/upper/etc/os-release
root@ec201-0A13DC:/etc# ls -l /aufs/rofs/etc/os-release
-rw-r--r--    1 root     root           339 Sep  9 13:50 /aufs/rofs/etc/os-release
root@ec201-0A13DC:/etc# 
~~~
(Not recommended to delete files like this).<br>

5. To restore a deleted file that also exist in _underlay_ it could just be copied from _underlay_ into the combined file system.<br>
The file will then appear in both _underlay_ and _overlay_ (and in the combined file system).<br>
~~~console
root@ec201-0A13DC:/etc# rm /etc/os-release 
root@ec201-0A13DC:/etc# 
root@ec201-0A13DC:/etc# ls -l /etc/os-release
ls: /etc/os-release: No such file or directory
root@ec201-0A13DC:/etc# cp /aufs/rofs/etc/os-release /etc/os-release
root@ec201-0A13DC:/etc# ls -l /etc/os-release
-rw-r--r--    1 root     root           339 Sep 17 15:24 /etc/os-release
root@ec201-0A13DC:/etc# ls -l /aufs/rwfs/upper/etc/os-release
-rw-r--r--    1 root     root           339 Sep 17 15:24 /aufs/rwfs/upper/etc/os-release
root@ec201-0A13DC:/etc#
~~~

6. To restore a deleted file that also exist in _underlay_ it is also possible to delete the special marker file within _overlay_.<br>
However, this requires a remount of the file system. The combined file system will not be notified if there are changes done "under the hood".<br>
Should you try this method, you need to reboot the device after doing explicit changes into the _overlay_.<br>
~~~console
root@ec201-0A13DC:/etc# rm /etc/os-release 
root@ec201-0A13DC:/etc# ls -l /etc/os-release
ls: /etc/os-release: No such file or directory
root@ec201-0A13DC:/etc# ls -l /aufs/rwfs/upper/etc/os-release
c---------    1 root     root        0,   0 Sep 17 15:31 /aufs/rwfs/upper/etc/os-release
root@ec201-0A13DC:/etc# rm /aufs/rwfs/upper/etc/os-release
root@ec201-0A13DC:/etc# ls -l /etc/os-release
ls: /etc/os-release: No such file or directory
root@ec201-0A13DC:/etc# reboot
...
root@ec201-0A13DC:~# ls -l /etc/os-release 
-rw-r--r--    1 root     root           339 Sep  9 13:50 /etc/os-release
root@ec201-0A13DC:~# ls -l /aufs/rwfs/upper/etc/os-release 
ls: /aufs/rwfs/upper/etc/os-release: No such file or directory
root@ec201-0A13DC:~# 
~~~


## Installing .ipk packages

**Before** trying to install/change any packages, please check out [WiFi backdoor to target device](backdoor.md)<br>

#### Listing installed packages
Command _opkg list_ will list the known packages installed.<br>
~~~console
root@ec201-0A13DC:~# opkg list
appkit - 1.0.5-r280860
prodkit - 1.0.3.15-r280286
root@ec201-0A13DC:~#
~~~
As stated previously, all pre-populated packages in _flir-image-sherlock_ are missing here ([even if installed](README.md#rootfs-installable-packages)).<br>

#### File transfer to target
To install a package, start by transfering its .ipk file to the target.<br>
You may use _/tmp_ as the folder to store it (temporarily) in the target.<br>
_/tmp_ should be big enough (~245 MB).<br>
Use _scp_ or _sftp_ to transfer the .imx file to target.<br>
example:<br>
~~~console
$ scp tmp/deploy/ipk/cortexa7hf-neon/umtp-responder_1.0-r1_cortexa7hf-neon.ipk root@192.168.250.2:/tmp
root@192.168.250.2's password: 
umtp-responder_1.0-r1_cortexa7hf-neon.ipk                  100%   27KB   3.8MB/s   00:00    
$ 
~~~
Make **sure** that operation result is OK (file transfered OK)<br>

#### Package installation
To install a package from a .ipk file, use _opkg install \<.ipk file name\>_<br>
Note that we need to use _opkg_ flag _--force-depends_ as opkg database information about dependent packages is "missing".<br>
_libc6_ **is** present.<br>

~~~console
root@ec201-0A13DC:~# opkg list
appkit - 1.0.5-r280860
prodkit - 1.0.3.15-r280286
root@ec201-0A13DC:~# opkg install /tmp/umtp-responder_1.0-r1_cortexa7hf-neon.ipk 

Collected errors:
 * Solver encountered 1 problem(s):
 * Problem 1/1:
 *   - nothing provides libc6 >= 2.27 needed by umtp-responder-1.0-r1.cortexa7hf-neon
 * 
 * Solution 1:
 *   - do not ask to install umtp-responder-1.0-r1.cortexa7hf-neon

root@ec201-0A13DC:~# opkg install /tmp/umtp-responder_1.0-r1_cortexa7hf-neon.ipk  --force-depends
Installing umtp-responder (1.0) on root
Configuring umtp-responder.
root@ec201-0A13DC:~# opkg list
appkit - 1.0.5-r280860
prodkit - 1.0.3.15-r280286
umtp-responder - 1.0-r1
root@ec201-0A13DC:~# 
~~~

(for the specific example, the replaced _umtp-reponder_ package needs reboot, or running command _systemctl restart gadget_ to activate).<br>

.ipk packages built from yocto will (normally) install into the rootfs (in FLIR C5, a combined overlayfs file system).<br>
Should you install a package not present already in the underlay, the installed files will then be written into the _overlay_ only.<br>

_overlay_ partition size is smaller than _underlay_<br>
Using package installation, you have to take care about _overlay_ disk space.<br>
Before installing a package, check available _overlay_ space:
~~~console
root@ec201-0A13DC:~# df -h /aufs/rwfs/      
Filesystem                Size      Used Available Use% Mounted on
/dev/mmcblk0p4          120.0M     11.1M     99.9M  10% /aufs/rwfs
root@ec201-0A13DC:~# 
~~~

#### Package file listing
You may check which files are installed by a package by typing command _opkg files \<package name\>_<br>
Example:
~~~console
root@ec201-0A13DC:~# opkg files umtp-responder
Package umtp-responder (1.0-r1) is installed on root and has the following files:
/etc/
/sbin/umtprd
/etc/umtprd/umtprd.conf
/sbin/
/etc/umtprd/
root@ec201-0A13DC:~# 
~~~

#### Package removal
Installed (known) packages could be removed using _opkg remove \<package name\>_<br>

Should you remove a installed replacement package it will be properly removed (not existent in the device anymore).<br>
An important note:<br>
Files from the removed package will appear as missing in the combined overlayfs rootfs even if there are original corresponding files within the _underlay_. <br>

-> **Do not** remove replacement packages using _opkg remove_ 

Only additional installation packages (not existent in _underlay_) should be removed using  _opkg remove_.<br>

##### Example of additional package
_ldd_ is a small utility that shows dynamic dependencies for a executable. Let us install it, test it and remove it.<br>
~~~console
root@ec201-0A13DC:~# ldd /sbin/umtprd 
-sh: ldd: command not found
root@ec201-0A13DC:~# opkg install --force-depends /tmp/ldd_2.27-r0_cortexa7hf-neon.ipk 
Installing ldd (2.27) on root
Configuring ldd.
root@ec201-0A13DC:~# ldd /sbin/umtprd 
	linux-vdso.so.1 (0x7ed45000)
	libpthread.so.0 => /lib/libpthread.so.0 (0x76f22000)
	libc.so.6 => /lib/libc.so.6 (0x76de2000)
	/lib/ld-linux-armhf.so.3 (0x76f4b000)
root@ec201-0A13DC:~# opkg files ldd
Package ldd (2.27-r0) is installed on root and has the following files:
/usr/
/usr/bin/
/usr/bin/ldd
root@ec201-0A13DC:~# ls -l /aufs/rwfs/upper/usr/bin     
-rwxr-xr-x    1 root     root          5334 Aug 14 14:38 ldd
root@ec201-0A13DC:~# ls -l /aufs/rofs/usr/bin/ldd
ls: /aufs/rofs/usr/bin/ldd: No such file or directory
root@ec201-0A13DC:~# opkg remove ldd
Removing ldd (2.27) from root...
root@ec201-0A13DC:~# ldd /sbin/umtprd 
-sh: ldd: command not found
root@ec201-0A13DC:~# ls -l /aufs/rwfs/upper/usr/bin
root@ec201-0A13DC:~# 
~~~

#### Package removal of replaced packages (restore)
As stated earlier, it is not a good idea to just remove a package that you installed to try out your own variant of that particular package (example; _umtp-responder_).<br>
So, how should you do this instead?

1. Check which files are installed (=replaced) in the package
~~~console
root@ec201-0A13DC:~# opkg files umtp-responder
Package umtp-responder (1.0-r1) is installed on root and has the following files:
/etc/
/sbin/umtprd
/etc/umtprd/umtprd.conf
/sbin/
/etc/umtprd/
root@ec201-0A13DC:~# 
~~~
2. Now, remove the package using _opkg remove \<package name\>_
~~~console
root@ec201-0A13DC:~# opkg list umtp-responder
umtp-responder - 1.0-r1
root@ec201-0A13DC:~# opkg remove umtp-responder
Removing umtp-responder (1.0) from root...
root@ec201-0A13DC:~# opkg list umtp-responder
root@ec201-0A13DC:~# 
~~~
3. Remove all "deleted" markers within the _overlay_ corresponding to the list from "1.". (check first)
~~~console
root@ec201-0A13DC:~# cd /aufs/rwfs/upper/
root@ec201-0A13DC:/aufs/rwfs/upper# ls -dl etc
drwxr-xr-x   11 root     root          1024 Sep 18 10:57 etc
root@ec201-0A13DC:/aufs/rwfs/upper# ls -dl etc/umtprd 
c---------    1 root     root        0,   0 Sep 18 10:57 etc/umtprd
root@ec201-0A13DC:/aufs/rwfs/upper# rm etc/umtprd
root@ec201-0A13DC:/aufs/rwfs/upper# ls -dl sbin      
drwxr-xr-x    2 root     root          1024 Sep 18 10:57 sbin
root@ec201-0A13DC:/aufs/rwfs/upper# ls -dl sbin/umtprd 
c---------    1 root     root        0,   0 Sep 18 10:57 sbin/umtprd
root@ec201-0A13DC:/aufs/rwfs/upper# rm sbin/umtprd 
root@ec201-0A13DC:/aufs/rwfs/upper# 
~~~
(note that _/aufs/rwfs/upper/etc_ and _/aufs/rwfs/upper/sbin_ are real files/directories. We leave these)<br>

4. Reboot<br>
We need to reboot to activate updated _underlay_
5. Check that device is working after reboot