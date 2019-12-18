## SNU 2019-F Computer Network Project

#### Build Guide

Refer to: [OpenWrt official build guide](https://openwrt.org/docs/guide-developer/quickstart-build-images)


#### Menuconfig Settings (project specific)

* Target
  - Target System: Atheros AR7xxx/AR9xxx
  - Subtarget: Generic
  - Target Profile (TP-Link Archer C7 v4)

* Advanced Options
  - [*] Build the OpenWrt Image Builder
  - [*]   Include package repositories
  - [*] Build the OpenWrt SDK
  - [*] Package the OpenWrt-based Toolchain

* Kernel modules -> Wireless Drivers
  - {*} kmod-ath
  - <M> kmod-ath9k
  - [*]   Support for Ubiquiti UniFi Outdoor+ access point
  - -*- kmod-cfg80211
  - -*- kmod-mac80211
  - (-- disable all Ath10k related modules --)

For convenience, we included our config file: `./.config.project`
Rename to `./.config` after initial config to apply settings!
(Also make sure to backup old config file)


#### Working with submodule

Motivation: need to maintain atheros driver changes

Changed target directory `build_dir/target-mips_24kc_musl/linux-ar71xx_generic/backports-2017-11-01/drivers/net/wireless/ath` to a submodule.

The submodule is maintained at [https://github.com/nhosj2/ath.git](https://github.com/nhosj2/ath.git).

After initial build,

```
$ git submodule init
$ git submodule update
```

This should update target directory `/build_dir/ ... /ath` to match the remote repository.


#### Replacing stock firmware with OpenWrt

1. Prepare your custom-built firmware binary, or [Download pre-built OpenWrt image](http://downloads.openwrt.org/releases/18.06.5/targets/ar71xx/generic/openwrt-18.06.5-ar71xx-generic-archer-c7-v4-squashfs-factory.bin)
    * Rename it to something shorter like factory.bin.

2. From the stock web interface, find the "Manual Upgrade" under Advanced \> System Tools \> Firmware Upgrade.
    * The web interface is available at the gateway address. For example, http://192.168.1.1/
    * If the "Manual Upgrade" item doesn't exist, you may need to update your firmware to the latest version.

3. Select your prepared binary. The process should proceed automatically.
    * May take up to 5~10 minutes.
    * The router should reboot when done. (Indicated by the flashing of all leds)

4. Log into your router running OpenWrt by ssh
    * `ssh root@192.168.1.1`. If prompted for password, try "toor".
    * After initial login, set password by typing `passwd`.

5. Install LuCI & Reboot
    * **Check if LuCI is already installed & running at http://192.168.1.1/cgi-bin/luci**. If so, skip this step.
  ```
  root@OpenWrt# vi /etc/opkg.conf
  (Uncomment everything)

  root@OpenWrt# opkg update
  root@OpenWrt# opkg install luci
  root@OpenWrt# reboot
  (Will disconnect. Reconnect after boot)

  root@OpenWrt# /etc/init.d/uhttpd start
  root@OpenWrt# /etc/init.d/uhttpd enable
  ```

6. By default, wifi is disabled. Configure & enable via LuCI.

7. Profit!!


#### Recovery from Brick

**This solution is TP-Link Archer C7 family specific**

###### Install TFTP Server.

1. Prepare a device (preferably Ubuntu) with an ethernet port.

2. Install the following packages:
```
$ sudo apt install xinetd tftpd tftp
```

3. Create **/etc/xinetd.d/tftp** with the following content:
```
service tftp
{
    protocol = udp
    port = 69
    socket_type = dgram
    wait = yes
    user = nobody
    server = /usr/sbin/in.tftpd
    server_args = /tftpboot
    disable = no
}
```

4. Create the directory specified in the above `server\_args`: **/tftpboot**.
```
$ sudo mkdir /tftpboot
$ sudo chmod -R 777 /tftpboot
$ sudo chown -R nobody /tftpboot
```

5. Download and put the stock firmware in that directory:
    * For Archer C7 v4, .zip download available from [here](https://www.tp-link.com/kr/support/download/archer-c7/v4/#Firmware).
    * Extract and rename firmware to **ArcherC7v4\_tp\_recovery.bin**, and then copy to /tftpboot.

Now the TFTP Server is ready!
But before actually starting the service, proceed to the following section.

##### Connection Setup

1. Unplug all ethernet cables from your router and turn it off. **Do not unplug power cable!**

2. Disconnect your laptop/pc from any wireless/wired connection.

3. Grab an ethernet cable and plug one end to the **"Ethernet 1"** port of your router. Connect the opposite end to your laptop/pc.

4. From your laptop/pc, configure wired connection settings:
```
IPv4    : 192.168.0.66
Netmask : 255.255.255.0
Gateway : 192.168.0.1
```

##### Recovery

1. Turn on your router. At the same time, press and hold the **reset** button for about 5 seconds.

2. Two led indicators should light up: power and transfer (the one with two arrows pointing opposite directions).

3. Sit tight for ~5 minutes. When the flashing is done, the router should reboot itself.


---

  _______                     ________        __
 |       |.-----.-----.-----.|  |  |  |.----.|  |_
 |   -   ||  _  |  -__|     ||  |  |  ||   _||   _|
 |_______||   __|_____|__|__||________||__|  |____|
          |__| W I R E L E S S   F R E E D O M
 -----------------------------------------------------

This is the buildsystem for the OpenWrt Linux distribution.

To build your own firmware you need a Linux, BSD or MacOSX system (case
sensitive filesystem required). Cygwin is unsupported because of the lack
of a case sensitive file system.

You need gcc, binutils, bzip2, flex, python, perl, make, find, grep, diff,
unzip, gawk, getopt, subversion, libz-dev and libc headers installed.

1. Run "./scripts/feeds update -a" to obtain all the latest package definitions
defined in feeds.conf / feeds.conf.default

2. Run "./scripts/feeds install -a" to install symlinks for all obtained
packages into package/feeds/

3. Run "make menuconfig" to select your preferred configuration for the
toolchain, target system & firmware packages.

4. Run "make" to build your firmware. This will download all sources, build
the cross-compile toolchain and then cross-compile the Linux kernel & all
chosen applications for your target system.

Sunshine!
	Your OpenWrt Community
	http://www.openwrt.org


