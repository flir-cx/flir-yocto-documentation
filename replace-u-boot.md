Replace u-boot
==============
When _bitbake flir-image-sherlock_ and/or _bitbake u-boot_ has been successfully run, you should find a _u-boot.imx_ in sub-folder _tmp/deploy/images/ec201_<br>
File is a symlink to _u-boot-ec201-2018.03-r0.imx_, file size is around 400K <br>

### File transfer to target
(Before working with target(s) you need to establish a connect to the target as a developer. Read more about it in [USB RNDIS and shell connection](rndis.md))<br>

The generated file should now be transfered to target. 
Typically "/tmp" is a suitable target location. <br>
("/tmp" in on a volatile ramdisk, which size is **~245 MB**).
~~~console
# cd /tmp
root@ec201-0A13DC:/tmp# df -h .
Filesystem                Size      Used Available Use% Mounted on
tmpfs                   247.3M    200.0K    247.1M   0% /tmp
root@ec201-0A13DC:/tmp# 
~~~
-> /tmp mount **is big enough** for _u-boot.imx_.<br>

Use _scp_ or _sftp_ to transfer the .imx file to target,<br>
example:
~~~console
$ scp tmp/deploy/images/ec201/u-boot.imx root@192.168.250.2:/tmp
root@192.168.250.2's password: 
u-boot.imx                                    100%  411KB 411.0KB/s   00:00    
$ 
~~~
Make **sure** that operation result is OK (file transfered OK)<br>

### Check u-boot file 
There is no obvious way to check that the transfered u-boot.imx file is a
bootable binary. -><br>
**AVOID programming of u-boot unless you know what you are doing,**<br>
(and willing to take the risk of bricking your device).

However, you may do a simple confidence check
(search for welcome message string in binary):
~~~console
root@ec201-0A13DC:~# strings /tmp/u-boot.imx | grep " 2018"
U-Boot 2018.03 - 31a58b9-dirty (Aug 14 2020 - 14:59:16 +0000)
U-Boot 2018.03 - 31a58b9-dirty (Aug 14 2020 - 14:59:16 +0000)
root@ec201-0A13DC:~# 
~~~
If you find this type of text, it is most likely that your u-boot.imx really is a real "u-boot" file, and that you may proceed (on your own risk of course).

### Unlock mmc boot partition
Normally the "boot" partition of the eMMC is write protected to avoid unintentinal (bad) writes that will make the device unbootable.<br>
To be able to perform an intentional reprogram of the boot partition the following procedure is necessary:
~~~console
root@ec201-0A13DC:~# echo 0 >/sys/class/block/mmcblk0boot0/force_ro
root@ec201-0A13DC:~#
~~~

### Programming image
use _dd_ to copy the transfered .imx file as raw data into the unlocked mmc boot partition _/dev/mmcblk0boot0_<br>

Run the following command (from target ssh prompt, example):
~~~console
root@ec201-0A13DC:~# dd if=/tmp/u-boot.imx of=/dev/mmcblk0boot0 seek=2
822+0 records in
822+0 records out
420864 bytes (411.0KB) copied, 0.115582 seconds, 3.5MB/s
root@ec201-0A13DC:~# 
~~~
Note the "seek=2" - **MOST important**<br> 
(You may try _dd_ (without parameters) for a brief usage text).<br>
The binary content of u-boot.imx will be written into the boot partition.<br><br>
To check it, you may try to grep for the ID string<br>
Example:<br>
~~~console
root@ec201-0A13DC:~# strings /dev/mmcblk0boot0 | grep " 2018"
U-Boot 2018.03 - 31a58b9-dirty (Aug 14 2020 - 14:59:16 +0000)
U-Boot 2018.03 - 31a58b9-dirty (Aug 14 2020 - 14:59:16 +0000)
root@ec201-0A13DC:~#
~~~
If you do not get a result similar to the above, **DO NOT REBOOT, try to reprogram another file**

### Reboot to activate new u-boot
Theoretically you may want to relock the boot partition by performing _echo 1 >/sys/class/block/mmcblk0boot0/force_ro_<br>
Do this if you like.<br>
(However, write protecting will happen automatically upon reset.)<br><br>
Now restart the device by typing command:<br>
_reboot_ <br>
Device should reboot and use your new u-boot if successfully programmed.<br>
You may check it by running the **strings...** command from prompt.


