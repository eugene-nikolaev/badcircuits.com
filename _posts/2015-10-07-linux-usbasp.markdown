---
layout: post
title:  Setting up permissions for USBasp AVR programmer in Linux Mint 17
date:   2015-10-07 11:20:50
categories: avr usbasp linux
---

![Linux File Permission coding](/assets/2015/10/linux-permissions.jpg)
<br>
<br>
Once I tried to upload firmware to attiny13. I used Arduino IDE (on Linux Mint 17.2) and [USBasp](https://rover.ebay.com/rover/1/711-53200-19255-0/1?icep_id=114&ipn=icep&toolid=20004&campid=5338190330&mpre=http%3A%2F%2Fwww.ebay.com%2Fitm%2FUSBASP-USBISP-AVR-Programmer-Adapter-10-Pin-USB-Cable-ATMEGA8-ATMEGA128-Arduino-%2F141924793771) programmer.
But an error encountered:

```
avrdude: Warning: cannot open USB device: Permission denied
avrdude: error: could not find USB device with vid=0x16c0 pid=0x5dc
```

It was obvious that my user did not have permissions to use that usb device.
Setting up recipes was found in web but not from the first try.

That's why I want to share it too.
At first, disconnect the USBasp and look which linux device was related to it.
Grep by **16c0** (that was a **vid** from the message error above): 

```
$ lsusb | grep 16c0
Bus 003 Device 024: ID 16c0:05dc Van Ooijen Technische Informatica shared ID for use with libusb
```

Looking for the **Bus 003** (you may have different number):

```
$ ls -la /dev/bus/usb/003/ | grep 024
crw-rw-r-- 1 root root 189, 279 Oct  7 11:07 024
```

Please note — a device numbered 024 has owner and group: root root.
Our task here is to make this device to belong to a group which includes your user.

Create a `usbasp.rules` file (anywhere) with the following content:

```
ATTR{idVendor}=="16c0", ATTR{idProduct}=="05dc", MODE:="0664", GROUP:="<your group>"
```

The easiest way is to specify a group with the same name as your user's name. If you want another one, you may use a `groups` command, which groups does the user belong to.

Then copy the file:

```
sudo cp usbasp.rules /etc/udev/rules.d/
```

**udev** applies change automatically, you have nothing to restart manually.
<br>
Disconnect and connect again the programmer.
Look at the bus (I repeat: you may have another number):

```
$ ls -la /dev/bus/usb/003/
total 0
drwxr-xr-x 2 root root       180 Oct  7 11:15 .
drwxr-xr-x 8 root root       160 Oct  7 06:32 ..
crw-rw-r-- 1 root root  189, 256 Oct  7 06:32 001
crw-rw-r-- 1 root root  189, 257 Oct  7 06:32 002
crw-rw-r-- 1 root root  189, 258 Oct  7 06:32 003
crw-rw-r-- 1 root root  189, 259 Oct  7 06:32 004
crw-rw-r-- 1 root root  189, 260 Oct  7 06:32 005
crw-rw-r-- 1 root root  189, 261 Oct  7 06:32 006
crw-rw-r-- 1 root wayne 189, 280 Oct  7 11:15 025
```

As you can see, the programmer was bound to a device with a new number `025` and now it belongs to a group of my *wayne* user.
Now Arduino IDE can upload firmwares.

P.S. Linux does not have some default permissions to upload firmwares to arduino in a standard way.
USB-to-Serial adapter of arduino is working in scope of `dialout` group.
You should add your user to that group and everything will work:

```
sudo adduser <your user> dialout
```