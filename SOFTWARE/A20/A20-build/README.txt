
How to make a bootable SD card for Olimex's A20-SOM, A20-OLinuXino-MICRO, A20-OLinuXino-LIME,
A20-OLinuXino-LIME2 with kernel 3.4.90 and Linux Debian file system

===============================================================================================

1. Setup of the toolchain
-------------------------
You should make sure that you have the tools needed for building the Linux Kernel. You'd have to install
them if you don’t have them already installed. To install new software you should be logged to a user
with super user rights on your Linux machine. Type in the terminal:

$ sudo su

You will be asked for your password and then your prompt will change to # which means you are now the
super user, all future commands should be run in this mode.

Next update apt-get links by typing:

# apt-get update

Next install the toolchain by typing the following:

# apt-get install gcc-4.7-arm-linux-gnueabihf ncurses-dev uboot-mkimage build-essential git

This will install: GCC compiler used to compile the kernel, the kernel config menu uboot make image
which is required to allow the SD card to boot into the Linux image, Git that allows you to download
from the github which holds source code for some of the system, Some other tools for building the kernel

Note that if you use debian may be you will need to add:

deb http://www.emdebian.org/debian squeeze main

in the file below:

/etc/apt/sources.list

You would also need a torrent client. It is needed to be able to download the file system. Any torrent
that suits your needs would do the job, as long as it can load torrent files it is fine.

Now you have all important tools to make your very own A20 kernel image!

2. Building Uboot
-----------------

First let's make the directory where we would build the A20 Linux:

# mkdir a20-olimex
# cd a20-olimex

Then let’s download the uboot sources from GitHub repository, note there are lot of branches but you
have to use sunxi branch. The u-boot is tested with the following branch:

# git rev-parse --verify HEAD 36080eb05e9a1e96d58e3168631d3cc9c612a0e3

Download u-boot sourses:

# git clone -b sunxi https://github.com/linux-sunxi/u-boot-sunxi.git

After the download you should have a new directory

# cd u-boot-sunxi/

With the following commands you have to configure the uboot for the different boards, use only the make
command for your board:

2.1 A20-OLinuXino_Lime2 board

# make A20-OLinuXino_Lime2_config ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf-

2.2 A20-SOM board

Note that there is 2 different types of A20-SOM boards. The main differences are in DDR3 memory bus
speed. 

A20-SOM up to rev.B  - DDR3 memory bus speed is 384MHz(6 layer PCB)

A20-SOM after rev.D - DDR3 memory bus speed is 480MHz(8 layer PCB)

2.2.1 For A20-SOM up to rev.B type 

# make A20-SOM_config ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf-

2.2.2 For A20-SOM after rev.D you can use the u-boot settings for A20-Lime2

# make A20-OLinuXino_Lime2_config ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf-

2.3 A20-OLinuXino-MICRO board

# make A20-OLinuXino_MICRO_config ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf-

2.4 A20-OLinuXino-LIME board

# make A20-OLinuXino_Lime_config ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf-

Now type the following command for u-boot building

# make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf-

At the end of the process you can check if everything is OK by

# ls u-boot.bin u-boot-sunxi-with-spl.bin spl/sunxi-spl.bin

spl/sunxi-spl.bin  u-boot.bin  u-boot-sunxi-with-spl.bin

If you got these files everything is complete, well done so far

# cd ..

You should be in the following directory:

/home/user/a20-olimex#

3. Building kernel sources for A20 boards
----------------------------------------------

Kernel sources for A20 are available on GitHub. Note that the following building is made with the
revision below: 

# git rev-parse --verify HEAD e37d760b363888f3a65cd6455c99a75cac70a7b8

You can download the kernel sources using the following command:

# git clone https://github.com/linux-sunxi/linux-sunxi

After the download go to the kernel directory:

# cd linux-sunxi/

3.1. How to add SPI support. If you don't need SPI then you can skip 3.1 section

download the  spi-sun7i.c file using wget command 

# wget https://raw.githubusercontent.com/OLIMEX/OLINUXINO/master/SOFTWARE/A20/A20-build/spi-sun7i.c

and copy it in drivers/spi directory

# cp spi-sun7i.c /drivers/spi 

download the patch file SPI.patch using wget command

# wget https://raw.githubusercontent.com/OLIMEX/OLINUXINO/master/SOFTWARE/A20/A20-build/SPI.patch

and apply the patch:

# patch -p0 < SPI.patch

3.2 configure the system 

Here you need from a20 configuration file – a20_olimex_defconfig . The file contains all kernel module
settings.

Download a20_olimex_defconfig  using wget command 

# wget https://raw.githubusercontent.com/OLIMEX/OLINUXINO/master/SOFTWARE/A20/A20-build/a20_olimex_defconfig

then copy a20_olimex_defconfig file to configs directory:

# cp a20_olimex_defconfig  /arch/arm/configs/

and make:

# make ARCH=arm a20_olimex_defconfig 

The result should be:

configuration written to .config

If you wish to make your changes in the kernel configuration do:

# make ARCH=arm menuconfig
The menuconfig changes a .config text file, which you can view/edit even with a text editor like vi,
nano. With this command you can add or remove different modules for the different peripherials in the
kernel. Be careful when you change these settings because the kernel can stop working properly.

Note that if you want to have SPI support then you have to enable 
Device driver > SPI support > <*>SUN7I SPI Controller 
							[*] SUN7I SPI Normal DMA mode select

Now you can continue with kernel image compiling 

# make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j4 uImage

when this finish’s you will have uImage ready and the result should be:

Kernel: arch/arm/boot/zImage is ready
UIMAGE  arch/arm/boot/uImage
Image Name:   Linux-3.4.90+
Created:      Tue Aug 19 16:23:22 2014
Image Type:   ARM Linux Kernel Image (uncompressed)
Data Size:    4596072 Bytes = 4488.35 kB = 4.38 MB
Load Address: 40008000
Entry Point:  40008000
Image arch/arm/boot/uImage is ready

The next step is kernel modules building:

# make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j4 INSTALL_MOD_PATH=out modules
# make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j4 INSTALL_MOD_PATH=out modules_install

DONE! At this point you have uboot and kernel modules.

The uImage file is located in linux-sunxi/arch/arm/boot/

The kernel modules are located in

linux-sunxi/out/lib/modules/3.x.xx

where 3.x.xx is kernel version. In our case the directory with modules is:

linux-sunxi/out/lib/modules/3.4.90+

4. Format and setup the SD-card
--------------------------------
We suggest class 10 microSD card with memory capacity greater than 2GB. We haven't tested microSD cards
bigger than 16GB.

First we have to make the correct card partitions, this is done with fdisk.

Plug SD card into your SD card reader and enter in the terminal

# ls /dev/sd
Then press two times <TAB> you will see a list of your sd devices like sda sdb sdc note that some of
these devices may be your hard disk so make sure you know which one is your sd card before you proceed
as you can damage your HDD if you choose the wrong sd-device. You can do this by unplugging your sd card
reader - pay attention which "sd" device disappears from the list.

Once you know which one is your microSD (as sda#) use it instead of the sdX name in the references below:

# fdisk /dev/sdX

then:

4.1. p

will list your partitions. If there are already partitions on your card do:

4.2. d enter 1

if you have more than one partitition press d while delete them all

4.3. create the first partition, starting from 2048

n enter p enter 1 enter enter +16M

4.4. create second partition

n enter p enter 2 enter enter enter

then list the created partitions:

p enter

if you did everything correctly on 4GB card you should see something like:

Disk /dev/sdg: 3980 MB, 3980394496 bytes
123 heads, 62 sectors/track, 1019 cylinders, total 7774208 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000

   Device Boot      Start         End      Blocks   Id  System
/dev/sdg1            2048       34815       16384   83  Linux
/dev/sdg2           34816     7774207     3869696   83  Linux


4.5. press w

write changes to sd card

Now we have to format the file system on the card (the first partition should be vfat as this is FS
which the Allwinner bootloader understands):

# mkfs.vfat /dev/sdX1

The second should be normal Linux EXT3 FS:

# mkfs.ext3 /dev/sdX2


5. Write the Uboot and u-boot-sunxi-with-spl.bin
-------------------------------------
You should be in /home/user/a20-olimex# directory 

Note that you have to write u-boot-sunxi-with-spl.bin in /dev/sdX (not sdX1 or sdX2)

Type:

# dd if=u-boot-sunxi/u-boot-sunxi-with-spl.bin of=/dev/sdX bs=1024 seek=8

6. Write kernel uImage you build to the SD-card
-----------------------------------------------
You should be in the directory below

/home/user/a20-olimex# directory 
# mkdir /mnt/sd
# mount /dev/sdX1 /mnt/sd
copy the Kernel uImage to root directory in partition 1

# cp linux-sunxi/arch/arm/boot/uImage /mnt/sd

7. Write script.bin file
-------------------------
script.bin is a file with very important configuration parameters like port GPIO assignments, DDR memory
parameters, video resolution etc, according to the A20 board you use you have to download and write
different script.bin file.

7.1 for A20-OLinuXino_Lime2 board download the script.bin file using wget command
# wget https://github.com/OLIMEX/OLINUXINO/raw/master/SOFTWARE/A20/A20-build/scripts_a20_Lime2_34_90_camera_rel_2/script.bin

7.2 for A20-SOM board download the script.bin file using wget command
# wget https://github.com/OLIMEX/OLINUXINO/raw/master/SOFTWARE/A20/A20-build/A20-SOM-3.4.90_camera_scripts_rel_3/script_a20_SOM_HDMI_720p50/script.bin

7.3 for A20-OLinuXino-MICRO board download the script.bin file using wget command
# wget https://github.com/OLIMEX/OLINUXINO/raw/master/SOFTWARE/A20/A20-build/script_a20_OLinuXino-micro_3.4.90_camera_rel_10/script.bin

7.4 for A20-OLinuXino-LIME board download the script.bin file using wget command
# wget https://github.com/OLIMEX/OLINUXINO/raw/master/SOFTWARE/A20/A20-build/script_a20_lime_3.4.90_camera_rel_3/script.bin

then copy the downloaded script.bin file to the mounted first partition of the SD card
# cp script.bin /mnt/sd
# sync
# umount /dev/sdX1

8. Debian rootfs
----------------
The Linux Kernel and Uboot are ready, now we have need from Linux distribution rootfs.

Basically the only difference between the different Linux distributions is the rootfs, so if you put
Debian rootfs you will have Debian, if you put Ubuntu rootfs it will be Ubuntu etc.

How to build one is a long topic, the good thing is that there are many already pre-built so we can just
download one and use it ready.

Now leave the kernel directory

# cd ..

Should be in the directory below

# /home/user/a20-olimex

Download debian rootfs with the file name "debian_FS_34_90_camera_A20-olimex.tgz" , which is available
only as a torrent. You would need a torrent client for it (Azureus, uTorrent, qBittorrent, etc).

The link to the torrent file is:

https://www.olimex.com/wiki/images/2/29/Debian_FS_34_90_camera_A20-olimex.torrent

Now mount the microSD card EXT3 FS partition:

# mount /dev/sdX2 /mnt/sd

and unarchive the rootfs

# tar debian_FS_34_90_camera_A20-olimex.tgz -C /mnt/sd
# ls /mnt/sd

The correct result should be:
bin   dev  home  lost+found  mnt  proc  run   selinux  sys  usr
boot  etc  lib   media       opt  root  sbin  srv      tmp  var

Now you have to replace the new generated kernel modules from
/home/user/a20-olimex/linux-sunxi/out/lib/modules/ to the new Debian file system:

# rm -rf /mnt/sd/lib/modules/*

# cp -rfv linux-sunxi/out/lib/modules/3.x.xx+/ /mnt/sd/lib/modules/

where x.xx is the kernel version

in our case:

# cp -rfv linux-sunxi/out/lib/modules/3.4.90+/ /mnt/sd/lib/modules/

replace /lib/firmware folder with the generated /linux-sunxi/out/firmware

#cp -rfv linux-sunxi/out/lib/firmware/ /mnt/sd/lib/

# sync
# umount /mnt/sdX2

at this point you have Debian on your SD card second partition and you have an SD card ready to boot
Debian on A20-SOM or A20-OLinuXino-MICRO or A20-OLinuXino-LIME or A20-OLinuXino-LIME2

Connect USB-SERIAL-CABLE-F to UEXT Tx, Rx and GND, or connect a HDMI screen. Put the SD-card apply
power according to A20 board type.You should see Uboot and then Kernel messages on the console

default username/password is : root / olimex
