WiFi backdoor to target device
==============================
Be aware of that any change to files/packages in the FLIR C5 exposes you to the **risk of losing contact** with the device, or even **make it unusable** (bricked).<br>
<br>
One obviuous risk is that USB may stop working.<br>
Possibly only that the USB mode is changed - RNDIS not working anymore<br>.
Possibly that there is **NO USB at all (!)**<br>


To lower the risk (should you still want to manipulate the device) it is a good idea to establish a secondary channel for communication.<br>
One obvious such channel is a wifi connection.<br>

Provided that the graphical FLIR camera application is running, you should be able to enable WiFi and connect to a network router.<br>
Enter "Settings", Press "Connections", Select "Wi-Fi", connect to network.<br>
Select a network that you have the access code to.<br>
When connected, you may click on the network name and check the IP-adress of the device (on the Wi-Fi network).<br>


Connect to the same Wi-Fi network on your host computer.<br>
Try: _ssh root@\<camera wifi ip\>_<br>
(example: _ssh root@192.168.5.22_).<br>
Login using your password set after you unlocked the device
(see [USB RNDIS and shell connection](https://github.com/flir-cx/flir-yocto-documentation/blob/prework/rndis.md) ).<br>
Make sure that this gives you a prompt, and that you are able to communicate with the device through ssh on wifi connection<br>


Note: established secondary communication backdoor **is no guarantee** that this will continue to work as you manipulate the device.<br>
**You are still on your own** should you do changes to the device.<br><br>

### Switching usb mode, restart usb
Let's assume that you lost USB.<br>
(One effective way of doing this is by removing package umtp_responder while usbmode is _RNDIS_MTP_, or _MTP_).<br><br>
If you are still able to connect the target using ssh on wifi, you may be able to fix this.<br>
There is a command _usbfn_<br>
(Use _usbfn --help_ to show usage.)<br>
_usbfn_ without parameter shows current usbmode (or at least what is expected).<br>
Try _usbfn RNDIS_ to set usbmode to RNDIS (no composite mode).<br>
Then try to ping the usb rndis ip number from host to check if you restored the connection,<br>

You may also try _systemctl restart gadget_ to restart usb using current usbfn setting.