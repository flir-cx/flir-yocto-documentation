USB RNDIS and shell connection
==============================

Overview
--------

### FLIR Cx USB modes
FLIR Cx (i.e. FLIR c5) normally has its "usb mode" set to _MTP_ or possibly 
_MTP+UVC_.
(MTP = _Media Transfer Protocol_, UVC = _USB Video Class_, i.e. a webcam)<br>

As a hidden feature, aimed for development and/or service, it is possible to change the USB mode to _RNDIS_ (usb networking, see below).<br>

### Optional RNDIS mode
USB RNDIS is a way to use (tcp/)ip over USB. If/when the device exposes a USB gadget interface for USB RNDIS, on a linux host, a network adapter will be automatically created.<br>
On a windows host, it will be possible to install a "RNDIS driver".

Depending on linux host flavour or windows installation, it will then be
possible to assign a ip number for this interface. <br>
After this it should be possible to access device tcp/ip services, typically sftp and/or ssh.<br>
Note that RNDIS as such does not let you login to the device.<br>
All produced FLIR devices has their own individual passwords.

### How to switch camera to RNDIS
Switching USB mode (to RNDIS) requires some special steps.
The one described here involves performing a request for a "unlock tool" for
the specific camera from FLIR.<br>
Contact FLIR service<br>
You should then receive a special installer file, able of "unlocking" the FLIR Cx specific serial number.<br>
Typical file name; _FLIR_oss_unlocker_1.2_894001234.run_ 

#### Unlock caveat
**Note:** Although it is possible to unlock a (FLIR Cx) device for development, it is **NOT** recommended.<br>
By unlocking the device, **warranty will be limited**, even if no additional steps to change software content is taken.<br>
Although it should be possible to re-lock the device by a certified FLIR service center, this will come at a charge.

#### unlock tool installation
If you still want to unlock the (FLIR cx) device, here follows a description on how to do this:
- Connect camera to a (linux) PC host using USB cable
- Open _FLIR camera_ file system in a file browser
- Browse to camera folder _FLIR Camera - Images - doupdate_ 
- Drag / drop your received unlocker file ( _FLIR_oss_unlocker_1.2\_\<serial\>.run_) into this folder (from another file browser instance)<br>
  The unlocker file will now automatically be installed.<br>
  (There is a small instruction file in this location if you like to read more about this)
- Note that by performing this installation of the unlocker file, you accept the license that comes with it.<br> (in essence: Do not deliberately wipe its install traces, do not try to mess up the FLIR camera application)<br> 
  
After successful installation of unlocker file, _RNDIS (+MTP)_ will then be set as USB mode in the FLIR C5 unit (until a factory default is performed).<br>

Also, the root password is set to **0pened** (character "zero" instead of letter capital "O", followed by "pened") - valid until device is rebooted.<br>

#### Host computer ip setup

When _unlocker_ file has been installed, it should now be possible to login as _root_ to the device using ssh.<br>
First, we need to establish a ip network to the camera device also on the host side.<br>
For linux hosts, better run host command _dmesg_ to check that a RNDIS device is registered.<br>
(usb0 should be the interface name on ubuntu/linuxmint. Other linux distributions might give another name to the usb network adapter.)<br>
~~~console
...
$ dmesg
...
[719813.884563] usb 3-9: new high-speed USB device number 22 using xhci_hcd
[719813.903217] usb 3-9: New USB device found, idVendor=09cb, idProduct=100c
[719813.903226] usb 3-9: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[719813.903231] usb 3-9: Product: FLIR Camera
[719813.903236] usb 3-9: Manufacturer: FLIR Systems
[719813.913325] rndis_host 3-9:1.0 usb0: register 'rndis_host' at usb-0000:00:14.0-9, RNDIS device, 12:40:7f:0d:26:ac
$ 
$ ifconfig
...
usb0      Link encap:Ethernet  HWaddr 12:40:7f:0d:26:ac  
          inet addr:192.168.250.1  Bcast:192.168.250.255  Mask:255.255.255.0
          inet6 addr: fe80::1040:7fff:fe0d:26ac/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:31 errors:0 dropped:0 overruns:0 frame:0
          TX packets:74 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:3692 (3.6 KB)  TX bytes:16413 (16.4 KB)
...
$ 
~~~
Target will assign itself the ip number 192.168.250.2.<br>
In this example, a host interface was automatically assigned a suitable ip number on the same subnet (192.168.250.1)<br>
Automatical assignment depends on other host settings, linux distribution/version dependent.<br><br>
For **ubuntu 14** the following is true:<br>
Edit host file _/etc/network/interfaces_ to include:
~~~console
allow-hotplug usb0

mapping hotplug
	script grep
        map usb0

iface usb0 inet static
      address 192.168.250.1
      netmask 255.255.255.0
      network 192.168.250.0
      broadcast 192.168.250.255
      up iptables -I INPUT 1 -s 192.168.250.2 -j ACCEPT
~~~
Efter save, 
run (on your linux host)  _sudo service network-manager restart_ <br>
Possibly disconnect/reinsert USB cable too
(You only need to do this edit once if successful)

For other distributions / version of host linux, another procedure might be needed to set ip 192.168.250.1 on the host interface.<br><br>
For **ubuntu 20** the following "should work":<br>
(Ubuntu 20.04 uses netplan as a default network manager, see for instance [ubuntu_20-04_network_configuration](https://linuxhint.com/ubuntu_20-04_network_configuration/) )<br>

Edit the .yaml file in /etc/netplan/ to look similar to:
~~~console
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: NetworkManager
  ethernets:
   usb0:
    dhcp4: no
    addresses:
    - 192.168.250.1/24
    gateway4: 192.168.250.2
~~~
(note the indentation...)<br>
Run: _sudo netplan try_ to check that is syntactially correct, and if so, that networking still works. You may then run _sudo netplan apply_ to finalize.<br>
#### unlock finalization

On the host, make sure that you have ssh (for instance _openssh-client_) installed locally

- Check ip connection<br>
  _ping 192.168.250.2_
- connect to target using ssh;<br>
  _ssh root@192.168.250.2_
- You are expected to get a login prompt;<br>
  _root@192.168.250.2's password:_<br>
  Enter: _0pened_<br>
  You are now supposed to get a shell prompt, typically;<br>
  _root@ec201-0D26AC:~#_
  (hex digits are device MAC adress dependent)
- Note that the root password ( _0pened_) is temporary and will be restored
  to device specific at reboot unless you now take some action<br>
  (it is always possible to reinstall the unlocker file).
- Setting own password:<br>
  _umount /etc/shadow_<br>
  _passwd_<br>
  (enter own root password - twice<br> 
  You should get a success message)<br>
  _sync_<br>
  _reboot_<br>


#### Check connection
After reboot, check that you are able to login using ssh to root@192.168.250.2 and **your own** root password every time.

#### Restore USB RNDIS mode
Avoid changing USB mode.<br>
If you do this by mistake, here follows some notes about how to restore to RNDIS...<br><br>
You may try the [WiFi backup-connection to target device](backup-connection.md), and change usbmode using command usbfn (and follow instructions).<br>

As an alternative, if MTP is still working as USB mode, you could try to install _unlocker_ file again to get a working RNDIS connection<br>
Then login and run command _reboot_ to get a permanent USB RNDIS again.
