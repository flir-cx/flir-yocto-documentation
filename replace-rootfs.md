Replace rootfs
==============
When _bitbake flir-image-sherlock_ has been successfully run, you should find a _flir-image-sherlock-ec201.ext4_ in sub-folder _tmp/deploy/images/ec201_<br>
Size of file is typically **340 MB**<br>

### File transfer to target
(Before working with target(s) you need to establish a connect to the target as a developer. Read more about it in [USB RNDIS and shell connection](rndis.md))<br>


Generated file should now be transfered to target and stored in a location big enough to accomodate it.<br>
Normally "/tmp" is a suitable target location. <br>
"/tmp" in on a volatile ramdisk, which size is **~245 MB**.
~~~console
# cd /tmp
root@ec201-0A13DC:/tmp# df -h .
Filesystem                Size      Used Available Use% Mounted on
tmpfs                   247.3M    200.0K    247.1M   0% /tmp
root@ec201-0A13DC:/tmp# 
~~~
-> /tmp mount **is not big enough** for _flir-image-sherlock-ec201.ext4_.<br>

We need a bigger mount. <br>
Proposal is that you use "/FLIR/images"<br>
~~~console
root@ec201-0A13DC:~# df -h | grep images
/dev/mmcblk0p7            1.5G    327.1M      1.1G  23% /FLIR/images
root@ec201-0A13DC:~#
~~~

Use _scp_ or _sftp_ to transfer the .ext file to target, example:
~~~console
$ scp tmp/deploy/images/ec201/flir-image-sherlock-ec201.ext4 root@192.168.250.2:/FLIR/images
root@192.168.250.2's password:
flir-image-sherlock-ec201.ext4                100%  324MB   3.7MB/s   01:28    
$ 
~~~
Make **sure** that operation result is OK (file transfered OK)<br>

### Check active boot partition
Run the following commands from camera prompt (example):
~~~console
root@ec201-0A13DC:~# fw_printenv system_active
system_active=system2
root@ec201-0A13DC:~# cat /proc/cmdline
console=ttyLP0,115200 root=/dev/mmcblk0p3 rootwait rw ethaddr=00:40:7f:0a:13:dc wlanaddr=00:04:f3:ff:ff:fb btaddr=00:40:7f:0a:13:dd init=/sbin/preinit
root@ec201-0A13DC:~#
~~~
The uboot environment variable _system_active_ can take 2 values; _system1_ or _system2_<br>
_system1_ corresponds to _/dev/mmcblk0p2_<br>
_system2_ corresponds to _/dev/mmcblk0p3_<br>

In the above example, linux uses rootfs from _/dev/mmcblk0p3_ - _system2_ <br>
This means that _system1_, i.e _/dev/mmcblk0p2_, is the free partition, that we may rewrite.<br>
(Do not try to rewrite the active partition, undefined results. certainly not as expected, and risk of bricking)
### Programming image
use _dd_ to copy the transfered .ext4 file as raw data into the free rootfs partition (as indicated from "check active partition").<br>
**NOTE:** If you don't understand the difference between free and active partition, DO NOT PROCEED. You may harm your device and make it unusable.<br>
Assuming that you:<br>
- successfully transfered a _flir-image-sherlock-ec201.ext4_ file to _/FLIR/images_
- Found out which rootfs partition is the free one (in the example, _/dev/mmcblk0p2_ is the free partition)

You may now run the following command (from target ssh prompt, example):
~~~console
root@ec201-0A13DC:~# dd if=/FLIR/images/flir-image-sherlock-ec201.ext4 of=/dev/mmcblk0p2 bs=1M
324+0 records in
324+0 records out
339738624 bytes (324.0MB) copied, 30.257324 seconds, 10.7MB/s
root@ec201-0A13DC:~# 
~~~
(You may try _dd_ (without parameters) for a brief usage text).<br>
The binary content of flir-image-sherlock.ext4 will be written into the free rootfs partition.<br>
To check it, you may try to do a temporary mount of the free partition,<br>
Type (example):<br>
~~~console
root@ec201-0A13DC:~# mount /dev/mmcblk0p2 /mnt
root@ec201-0A13DC:~# ls /mnt
FLIR        dev         lost+found  run         usr
aufs        etc         media       sbin        var
bin         home        mnt         sys         www
boot        lib         proc        tmp
root@ec201-0A13DC:~# ls -l /mnt/boot/zImage
lrwxrwxrwx    1 root     root            29 Aug 17 12:57 /mnt/boot/zImage -> zImage-4.14.98-2.2.0+g5910884
root@ec201-0A13DC:~#
~~~
Mount should succeed. You should find files present below /mnt as in the provided example<br>
Specifically you should find a linux kernel below /mnt/boot/...<br>

### Switching active partition
When programmed and checked, you may now switch active partition,<br>
This means that you will boot the linux system from the switched partition from next time you restart. <br><br>
**Warning** - If you switch to boot from a partition that is not correct in some way, you may lose contact with your device, i.e. brick it.<br>
Do not do this unless you are confident in that you understand all the steps in this description,<br><br>
That said, to switch to boot from partition "p2" (i.e. _/dev/mmcblk0p2_), type the following command (at target ssh prompt):<br>
_fw_setenv system_active system1_<br>
(To switch active partition "p3" (i.e. _/dev/mmcblk0p3_), type command:<br>
_fw_setenv system_active system2_ )<br><br>
### Reboot to activate new partition
Now restart the device by typing command:<br>
_reboot_ <br>
Device should reboot and then use the partition defined by _system_active_<br>
When rebooted, you may check this. See "Check active boot partition" above<br>

