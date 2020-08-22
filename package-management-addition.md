package management addition to rootfs
=====================================
If you build your own _flir-image-sherlock_ image, you may try to add the "Image feature" called _package-management_<br>

Add the following line to file _meta-flir-base/recipes-flir/images/flir-image-sherlock.bb_ (or in a _.bbappend_):<br>
_IMAGE_FEATURES += " package-management"_ <br>
Then<br>
_bitbake flir-image-sherlock_<br>
You should now find a large number of files to inspect below _build_ec201/tmp/work/ec201-oe-linux-gnueabi/flir-image-sherlock/1.0-r0/rootfs/var/lib/opkg_<br>
Example:<br>
~~~console
$ cat tmp/work/ec201-oe-linux-gnueabi/flir-image-sherlock/1.0-r0/rootfs/var/lib/opkg/status 
Package: shadow-securetty
Version: 4.2.1-r3
Status: install ok installed
Architecture: ec201
Installed-Size: 1848
Installed-Time: 1600073972
Auto-Installed: yes

Package: gstreamer1.0-plugins-bad-stereo
Version: 1.14.4.imx-r0
Depends: gstreamer1.0 (>= 1.14.4.imx), libc6 (>= 2.27), libglib-2.0-0 (>= 2.54.3), libgstaudio-1.0-0 (>= 1.14.4.imx)
Status: install ok installed
Architecture: cortexa7hf-neon-mx7ulp
Installed-Size: 9652
Installed-Time: 1600073994
Auto-Installed: yes

...
$
~~~

(Should you install the generated image into target, this database would be available to _opkg_ utility.<br>
Note: By doing so, you would lose additional installed opkg packages, "appkit" and "prodkit" information.)<br>


