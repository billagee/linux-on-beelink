# Installing Debian Linux on a Beelink Mini S12 Pro

Boot from a Debian 12 netinstall USB stick.

Plug in an Ethernet cable since the Debian 12 installer doesn't (as of Feb 2025) seem to have support for the wifi device in this machine.

Set your hostname and domain name, e.g.,

```
Hostname: minik8s
Domain name: .local
```

Click on the storage device name to delete all existing partitions and reinit the partition table. 

Then select Guided partitioning and use the entire disk.

Unplug the USB stick, reboot, make sure the machine boots into the Debian partition after starting up.

Make your non-root user a sudoer:

```
usermod -aG sudo b
```

At this point, the system is missing a working iwlwifi ucode file.

So you'll see `no suitable firmware found!` errors related to `iwlwifi` in dmesg:

```
root@minik8s:/home/b# dmesg | grep AX101
[    3.197931] iwlwifi 0000:00:14.3: Detected Intel(R) Wi-Fi 6 AX101

root@minik8s:/home/b# dmesg | grep iwlwifi
[    3.196437] iwlwifi 0000:00:14.3: enabling device (0000 -> 0002)
[    3.197884] iwlwifi 0000:00:14.3: Detected crf-id 0x501, cnv-id 0x80400 wfpm id 0x80000030
[    3.197929] iwlwifi 0000:00:14.3: PCI dev 54f0/0244, rev=0x370, rfid=0x10c000
[    3.197931] iwlwifi 0000:00:14.3: Detected Intel(R) Wi-Fi 6 AX101
[    3.198028] iwlwifi 0000:00:14.3: Direct firmware load for iwlwifi-so-a0-hr-b0-89.ucode failed with error -2
[    3.198055] iwlwifi 0000:00:14.3: Direct firmware load for iwlwifi-so-a0-hr-b0-88.ucode failed with error -2
[    3.198078] iwlwifi 0000:00:14.3: Direct firmware load for iwlwifi-so-a0-hr-b0-87.ucode failed with error -2
[    3.198101] iwlwifi 0000:00:14.3: Direct firmware load for iwlwifi-so-a0-hr-b0-86.ucode failed with error -2
[    3.198124] iwlwifi 0000:00:14.3: Direct firmware load for iwlwifi-so-a0-hr-b0-85.ucode failed with error -2
[    3.198146] iwlwifi 0000:00:14.3: Direct firmware load for iwlwifi-so-a0-hr-b0-84.ucode failed with error -2
[    3.198167] iwlwifi 0000:00:14.3: Direct firmware load for iwlwifi-so-a0-hr-b0-83.ucode failed with error -2
[    3.198190] iwlwifi 0000:00:14.3: Direct firmware load for iwlwifi-so-a0-hr-b0-82.ucode failed with error -2
[    3.198212] iwlwifi 0000:00:14.3: Direct firmware load for iwlwifi-so-a0-hr-b0-81.ucode failed with error -2
[    3.198235] iwlwifi 0000:00:14.3: Direct firmware load for iwlwifi-so-a0-hr-b0-80.ucode failed with error -2
[    3.198257] iwlwifi 0000:00:14.3: Direct firmware load for iwlwifi-so-a0-hr-b0-79.ucode failed with error -2
[    3.198279] iwlwifi 0000:00:14.3: Direct firmware load for iwlwifi-so-a0-hr-b0-78.ucode failed with error -2
[    3.198301] iwlwifi 0000:00:14.3: Direct firmware load for iwlwifi-so-a0-hr-b0-77.ucode failed with error -2
[    3.198303] iwlwifi 0000:00:14.3: no suitable firmware found!
[    3.198316] iwlwifi 0000:00:14.3: minimum version required: iwlwifi-so-a0-hr-b0-77
[    3.198328] iwlwifi 0000:00:14.3: maximum version supported: iwlwifi-so-a0-hr-b0-89
[    3.198339] iwlwifi 0000:00:14.3: check git://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git
```

I'd like to get wifi working, so we'll need to download updated firmware.

Note the `maximum version supported: iwlwifi-so-a0-hr-b0-89` message in dmesg output.

So let's download that firmware file:

( See also: https://forums.debian.net/viewtopic.php?t=154203 )

```
su -
wget https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/plain/iwlwifi-so-a0-hr-b0-89.ucode
cp iwlwifi-so-a0-hr-b0-89.ucode /lib/firmware
reboot
```

Now the iwlwifi errors in dmesg should be gone, and you should see `loaded firmware version 89.1a492d28.0 so-a0-hr-b0-89.ucode`:

```
root@minik8s:/home/b# dmesg | grep iwlwifi
[    3.246715] iwlwifi 0000:00:14.3: enabling device (0000 -> 0002)
[    3.252854] iwlwifi 0000:00:14.3: Detected crf-id 0x501, cnv-id 0x80400 wfpm id 0x80000030
[    3.252895] iwlwifi 0000:00:14.3: PCI dev 54f0/0244, rev=0x370, rfid=0x10c000
[    3.252897] iwlwifi 0000:00:14.3: Detected Intel(R) Wi-Fi 6 AX101
[    3.265482] iwlwifi 0000:00:14.3: TLV_FW_FSEQ_VERSION: FSEQ Version: 0.0.2.42
[    3.265949] iwlwifi 0000:00:14.3: loaded firmware version 89.1a492d28.0 so-a0-hr-b0-89.ucode op_mode iwlmvm
```

Run `nmtui` and activate the WiFi connection. Select your wireless network and provide its passphrase.

Disable sleep since this machine needs to be up 24/7:

```
# vim /etc/systemd/sleep.conf and add:

[Sleep]
#AllowSuspend=yes
AllowSuspend=no
#AllowHibernation=yes
AllowHibernation=no
```

Then run:
```
systemctl restart systemd-logind
```
