---
title: "Flashing an image onto BeagleBone eMMC via USB Boot"
date: 2019-05-04T01-46-00+5:30
categories:
  - blog
tags:
  - BeagleBone Black
  - NetConsole
  - Linux
---
I bought a new BeagleBone Black (BBB). This board has multiple boot options. It has an onboard eMMC connected to the MMC1 port of the processor and the SD card reader is connected to the MMC0 port. It can also boot over USB0 and UART0. 

I went through the steps in [getting started page](https://beagleboard.org/getting-started), which basically told me to falsh the onboard eMMC with the image on the SD Card. This is where things got unstuck. The board should have taken around 45 mins to burn the image onto the onbard eMMC but it never seemed to complete. I very hesistantly powered off the BBB. On powering it back all the 4 LEDS all switched on and nothing happened. I feared that I had bricked my BeagleBone. Time to debug.

I connected an 3.3V FTDI USB to TTL cable to the serial debugger port of my BBB (https://elinux.org/Beagleboard:BeagleBone_Black_Serial). Started minicom in a terminal and booted my BBB using SD Card. This is what I saw:
```
[ 2407.629902] print_req_error: I/O error, dev mmcblk0, sector 0
[ 2407.661558] print_req_error: I/O error, dev mmcblk0, sector 1
[ 2407.695584] print_req_error: I/O error, dev mmcblk0, sector 2
[ 2407.729435] print_req_error: I/O error, dev mmcblk0, sector 3
[ 2407.763374] print_req_error: I/O error, dev mmcblk0, sector 4
[ 2407.773240] print_req_error: I/O error, dev mmcblk0, sector 5
[ 2407.779552] print_req_error: I/O error, dev mmcblk0, sector 6
[ 2407.814017] print_req_error: I/O error, dev mmcblk0, sector 7
[ 2407.819861] Buffer I/O error on dev mmcblk0, logical block 0, async page read
[ 2407.880598] print_req_error: I/O error, dev mmcblk0, sector 0
[ 2407.914496] print_req_error: I/O error, dev mmcblk0, sector 1
[ 2408.088370] Buffer I/O error on dev mmcblk0, logical block 0, async page read
[ 2408.320663] Buffer I/O error on dev mmcblk0, logical block 0, async page read
[ 2408.733474] Buffer I/O error on dev mmcblk0, logical block 1940464, async page read
```
Booting with internal eMMC was also of no use. I tried several images and several SD cards, nothing worked. So, I needed another way to boot up my BBB. A great source for imformation regarding BBB is obviously the [System Reference Manual](https://github.com/beagleboard/beaglebone-black/wiki/System-Reference-Manual). But it said `Software to support USB and serial boot modes is not provided by beagleboard.org. Please contact TI for support of this feature.`, which thankfully turned out to be incorrect. There have been 2 GSoCs to tackle the issue of flashing the onboard eMMC over USB:
1. In 2013, which resulted in [BBBlfs](https://github.com/ungureanuvladvictor/BBBlfs)
1. In 2018, which resulted in [BeagleBoot](https://github.com/ravikp7/BeagleBoot). This youtube [video](https://www.youtube.com/watch?v=5JYfh2_0x8s&ab_channel=RaviKumarPrasad) provides a good description of BBlfs and BeagleBoot.

I used BBBlfs as it was just a simple CLI tool. I build the tool and first tried flashing an older image from [armhf](http://www.armhf.com/boards/beaglebone-black/)
```
$ git clone https://github.com/ungureanuvladvictor/BBBlfs.git
$ cd BBBlfs
$ sudo apt-get install -qq build-essential autoconf automake pkg-config libusb-1.0-0-dev
$ ./autogen.sh && ./configure && make
$ cd bin
$ ./flash_script.sh debian-wheezy-7.5-rootfs-3.14.4.1-bone-armhf.com.tar.xz 

We are flashing this all mighty BeagleBone Black with the image from debian-wheezy-7.5-rootfs-3.14.4.1-bone-armhf.com.tar.xz!
Please do not insert any USB Sticks or mount external hdd during the procedure.

When the BeagleBone Black is connected in USB Boot mode press [yY].y

Putting the BeagleBone Black into flashing mode!

libusb: error [udev_hotplug_event] ignoring udev action bind
libusb: error [udev_hotplug_event] ignoring udev action bind
SPL has started!

libusb: error [udev_hotplug_event] ignoring udev action bind
U-Boot has started! Sending now the FIT image!

Waiting for the BeagleBone Black to be mounted............
Are you sure the BeagleBone Black is mounted at /dev/sdd?[yY]

umount: /dev/sdd2: not mounted.
Flashing now, be patient. It will take ~5 minutes!

0+42570 records in
0+42570 records out
358082560 bytes (358 MB, 341 MiB) copied, 59.0367 s, 6.1 MB/s

Resizing partitons now, just as a saefty measure if you flash 2GB image on 4GB board!
No partition is defined yet!
Could not delete partition 94825061478169
2: unknown command
e2fsck 1.44.1 (24-Mar-2018)
ext2fs_open2: Bad magic number in super-block
e2fsck: Superblock invalid, trying backup blocks...
e2fsck: Bad magic number in super-block while trying to open /dev/sdd2

The superblock could not be read or does not describe a valid ext2/ext3/ext4
filesystem.  If the device is valid and it really contains an ext2/ext3/ext4
filesystem (and not swap or ufs or something else), then the superblock
is corrupt, and you might try running e2fsck with an alternate superblock:
    e2fsck -b 8193 <device>
 or
    e2fsck -b 32768 <device>

resize2fs 1.44.1 (24-Mar-2018)
resize2fs: Bad magic number in super-block while trying to open /dev/sdd2
Couldn't find valid filesystem superblock.

Please remove power from your board and plug it again. You will boot in the new OS!

```
Booting up the BBB gave me nothing. The errors atleast gave me some ting to debug. I found an [issue](https://github.com/ungureanuvladvictor/BBBlfs/issues/29) in BBBlfs that solved my problems. I copied flash_script.sh to flash_script2.sh, made changes as per the pull request given in the above issue. Downloaded the latest image and it WORKED!!!

```
$ ./flash_script2.sh bone-debian-10.3-iot-armhf-2020-04-06-4gb.img.xz 

We are flashing this all mighty BeagleBone Black with the image from bone-debian-10.3-iot-armhf-2020-04-06-4gb.img.xz!
Please do not insert any USB Sticks or mount external hdd during the procedure.

When the BeagleBone Black is connected in USB Boot mode press [yY].y

Putting the BeagleBone Black into flashing mode!

libusb: error [udev_hotplug_event] ignoring udev action bind
SPL has started!

libusb: error [udev_hotplug_event] ignoring udev action bind
U-Boot has started! Sending now the FIT image!

Waiting for the BeagleBone Black to be mounted............
Are you sure the BeagleBone Black is mounted at /dev/sdc?[yY]y
umount: /dev/sdc1: not mounted.
Flashing now, be patient. It will take ~5 minutes!

0+390389 records in
0+390389 records out
3774873600 bytes (3.8 GB, 3.5 GiB) copied, 481.824 s, 7.8 MB/s

Resizing partitons now, just as a saefty measure if you flash 2GB image on 4GB board!
Partition #1 contains a ext4 signature.
e2fsck 1.44.1 (24-Mar-2018)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
rootfs: 86442/230144 files (0.0% non-contiguous), 504296/920576 blocks
resize2fs 1.44.1 (24-Mar-2018)
Resizing the filesystem on /dev/sdc1 to 932864 (4k) blocks.
The filesystem on /dev/sdc1 is now 932864 (4k) blocks long.


Please remove power from your board and plug it again. You will boot in the new OS!
```

Returning back to the SD card issue, the same SD Card passes the e2fsck  on my workstation:
```
$ sudo e2fsck /dev/sdc1
e2fsck 1.44.1 (24-Mar-2018)
rootfs: clean, 86442/230144 files, 504296/920576 blocks
```
But it fails on the BeagleBone Black. I seriously suspect that the SD card reader on my BBB is broken. I will continue to look into this, but this is all for now.


Sources:  
(https://nmglug.org/beaglebone-black-recovery/)
(https://www.reddit.com/r/BeagleBone/comments/1qe8zt/tutorial_beaglebone_black_how_to_boot_from_the/)
(https://www.reddit.com/r/BeagleBone/comments/6b1gib/did_i_just_brick_my_beaglebone_black/)
(https://www.element14.com/community/thread/54521/l/boot-bbb-from-usb0)
(https://processors.wiki.ti.com/index.php/AM335x_board_bringup_tips)
(http://ungureanuvladvictor.github.io/BBBlfs/)
(https://elinux.org/images/b/b2/Netconsole.pdf)

