# Creating a bootable ESXi 8.0 USB stick

Use this doc plus Etcher to flash the USB stick:

https://virtuallywired.io/2022/10/28/create-a-bootable-esxi-8-usb-installer-on-macos/

```
diskutil list # and get disk number of usb stick, e.g. /dev/disk4

❯ sudo fdisk -e /dev/disk4

fdisk: could not open MBR file /usr/standalone/i386/boot0: No such file or directory
The signature for this MBR is invalid.

Would you like to initialize the partition table? [y] y

fdisk:*1> f 1
Partition 1 marked active.

fdisk:*1> write
Writing MBR at offset 0.

fdisk: 1> quit
```

