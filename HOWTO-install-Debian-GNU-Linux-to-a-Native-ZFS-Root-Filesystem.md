---------
---------
---------
**WARNING - this instruction is outdated.**
---------
Now ZFS packages are provided by Debian [backports repository](https://backports.debian.org/Instructions/).

[See this page](https://github.com/zfsonlinux/zfs/wiki/Debian) for more info.

This instruction will be updated soon, for now you can use [beta version of HOWTO for Debian Jessie](https://github.com/zfsonlinux/zfs/issues/5536#issuecomment-269687475).

**WARNING - this instruction is outdated.**
---------
---------
---------
---------

These instructions are for Debian GNU/Linux. It is an almost exact clone of the [HOWTO install Ubuntu to a Native ZFS Root Filesystem](https://github.com/zfsonlinux/pkg-zfs/wiki/HOWTO-install-Ubuntu-to-a-Native-ZFS-Root-Filesystem) procedure with some very small differences.

### System Requirements
  * 64-bit Debian GNU/Linux Live CD.  (Not the alternate installer, and not the 32-bit installer!)
    For wheezy, have a look at [Debian GNU/Linux live cd - Wheezy](http://cdimage.debian.org/debian-cd/current-live/amd64/iso-hybrid/).
  * AMD64 or EM64T compatible computer. (ie: x86-64)
  * 8GB disk storage available.
  * 2GB memory minimum.

Computers that have less than 2GB of memory run ZFS slowly.  4GB of memory is recommended for normal performance in basic workloads.  16GB of memory is the recommended minimum for deduplication.  Enabling deduplication is a permanent change that cannot be easily reverted.

### Latest Tested And Recommended Version
* Debian GNU/Linux 7.2 Wheezy (live cd <code>debian-live-7.2-amd64-standard.iso</code>
* spl-0.6.2
* zfs-0.6.2

## Step 1: Prepare The Install Environment

1.1 Start the Debian GNU/Linux LiveCD. The tested iso will not start a graphical interface. Instead you will have a virtual terminal.

1.2 Input these commands at the terminal prompt:

	$ sudo -i
	# wget http://archive.zfsonlinux.org/debian/pool/main/z/zfsonlinux/zfsonlinux_6_all.deb
	# dpkg -i zfsonlinux_6_all.deb
	# apt-get update
	# apt-get install linux-image-amd64 debian-zfs

**Note:** Installing the image is just to make sure that you get a working kernel - ZoL won't compile on a RT kernel.

1.3 Check that the ZFS filesystem is installed and available:

	# modprobe zfs
	# dmesg | grep ZFS:
	[    5.588948] ZFS: Loaded module v0.6.2-524_gd6385c9, ZFS pool version 5000, ZFS filesystem version 5

## Step 2: Disk Partitioning

This tutorial intentionally recommends a small USB or extra disk using MBR partitioning used for the /boot filesystem.  GPT can be used instead, but beware of UEFI firmware bugs.

2.1 Run your favorite disk partitioner, like `parted` or `cfdisk`, on the boot device.  `/dev/disk/by-id/scsi-SATA_disk1` is the example device used in this document. The device to use for the ZFS root is in this example named `/dev/disk/by-id/scsi-SATA_disk2`.

2.2 Create a small MBR primary partition of **at least** 150 megabytes.  `/dev/disk/by-id/scsi-SATA_disk1-part1` is the example boot partition used in this document.

2.3 On this first small partition, set `type=83` and enable the `bootable` flag.

2.4 If this is not a USB device, create a swap partition (`type=82`) of any size you want.

The partition table should look like this:

	# fdisk -l /dev/disk/by-id/scsi-SATA_disk1
	
	Disk /dev/sda: 8589 MB, 8589934592 bytes
	255 heads, 63 sectors/track, 1044 cylinders, total 16777216 sectors
	Units = sectors of 1 * 512 = 512 bytes
	Sector size (logical/physical): 512 bytes / 512 bytes
	I/O size (minimum/optimal): 512 bytes / 512 bytes
	Disk identifier: 0x00000000

	Device    Boot      Start         End      Blocks   Id  System
	/dev/sda1   *        2048    10487807     5242880   83  Linux
	/dev/sda2        10487808    16777215     3144704   82  Linux swap / Solaris

**Remember:**  Substitute `scsi-SATA_disk1-part1` and `scsi-SATA_disk1-part2` appropriately below.

**Hints:**
* Are you doing this in a virtual machine?  Is some something in `/dev/disk/by-id` is missing?  Go read the troubleshooting section.
*  Recent GRUB releases assume that the `/boot/grub/grubenv` file is writable by the stage2 module.  Until GRUB gets a ZFS write enhancement, the GRUB modules should be installed to a separate filesystem in a separate partition that is grub-writable.
* If `/boot/grub` is in the ZFS filesystem, then GRUB will fail to boot with this message:  `error: sparse file not allowed`.  If you absolutely want only one filesystem, then remove the call to `recordfail()` in each `grub.cfg` menu stanza, and edit the `/etc/grub.d/10_linux` file to make the change permanent.
* Alternatively, if `/boot/grub` is in the zfs filesystem you can comment each line with the text `save_env` in the file `/etc/grub.d/00_header` and run update-grub.  


## Step 3: Disk Formatting

3.1  Format the small boot partition created by Step 2.2 as a filesystem that has stage1 GRUB support like this:

	# mke2fs -m 0 -L /boot/grub -j /dev/disk/by-id/scsi-SATA_disk1-part1

3.2 Create a swap area on the second partition if one was created:

	# mkswap -L swap /dev/disk/by-id/scsi-SATA_disk1-part2

3.3  Create the root pool on the second device:

	# zpool create -o ashift=12 -o altroot=/mnt -m none rpool /dev/disk/by-id/scsi-SATA_disk2
	# zfs set atime=off rpool

Always use the long /dev/disk/by-id/* aliases with ZFS.  Using the /dev/sd* device nodes directly can cause sporadic import failures, especially on systems that have more than one storage pool.

**NOTE:** It is the ZoL developers recommendation that you always enable ashift=12 to accomodate future disks.

**Hints:**
* `# ls -la /dev/disk/by-id` will list the aliases.
* The root pool can be a mirror.  For example, `zpool create -o ashift=12 rpool mirror /dev/disk/by-id/scsi-SATA_disk2 /dev/disk/by-id/scsi-SATA_disk3`. Remember that the version and ashift matter for any pool that GRUB must read, and that these things are difficult to change after pool creation.
* If you are using a mirror with a separate boot device as described above, don't forget to edit the grub.cfg file <b>on the second HD partition</b> so that the "root=" partition refers to that partition on the second HD also; otherwise, if you lose the first disk, you won't be able to boot from the second because the kernel will keep trying to mount the root partition from the first disk.
* The latest GRUB (2.01-22debian1+zfs3~wheezy) can boot from any type of RAID-Z groups. For example, `zpool create -o ashift=12 rpool raidz3 /dev/disk/by-id/scsi-SATA_disk[2-6]`.
* The pool name is arbitrary.  On systems that can automatically install to ZFS, the root pool is named "rpool" by default.  Note that system recovery is easier if you choose a unique name instead of "rpool".  Anything except "rpool" or "tank", like the hostname, would be a good choice. If you DO choose to name your pool by any other name, the grub boot option `rpool` can be used to specify this.


3.4  Create a "ROOT" filesystem in the root pool:

	# zfs create -o mountpoint=none rpool/ROOT

3.5  Create a descendant filesystem for the Debian GNU/Linux system:

	# zfs create -o mountpoint=/ rpool/ROOT/debian-1

On Solaris systems, the root filesystem is cloned and the suffix is incremented for major system changes through `pkg image-update` or `beadm`.  Similar functionality for APT is possible but currently unimplemented.

**NOTE:** if you prefer, you can name the root filesystem you like. Just use the appropriate grub boot option `bootfs`. For example: `zfs create zfspool/hostname`. This would need the following grub boot commandline: `rpool=zfspool bootfs=zfspool/hostname` (replace `hostname` with the actual hostname of the machine).

3.6   Set the `bootfs` property on the root pool.

	# zpool set bootfs=rpool/ROOT/debian-1 rpool

The boot loader uses these two properties to find and start the operating system.  These property names are *not* arbitrary, although their values are (as long as they corresponds with the reality).

3.7  Creating File Systems (not necessary):

	# zfs create -o mountpoint=/home zroot/home
	# zfs create -o mountpoint=/usr -o canmount=off zroot/usr
	# zfs create -o mountpoint=/var zroot/var
	# zfs create -o compression=lz4 -o atime=on zroot/var/mail
	# zfs create -o compression=lz4 -o setuid=off -o exec=off zroot/var/log
	# zfs create -o compression=lz4 -o setuid=off -o exec=off zroot/var/tmp
	# zfs create -o mountpoint=/tmp -o compression=lz4 -o setuid=off -o exec=off rpool/tmp

3.8  Export the pool:

	# zpool export rpool

Don't skip this step.  The system is put into an inconsistent state if this command fails or if you reboot at this point.


## Step 4:  System Installation

**Remember:** Substitute "rpool" for the name chosen in Step 3.2.

4.1  Import the pool:

	# zpool import -d /dev/disk/by-id -R /mnt rpool
	# ## alternative
	# zpool import -o altroot=/mnt rpool

4.2  Place cache pool configuration:

	# mkdir -p /mnt/etc/zfs/
	# zpool set cachefile=/mnt/etc/zfs/zpool.cache rpool

4.3  Mount the small boot filesystem for GRUB that was created in step 3.1: 

	# mkdir -p /mnt/boot/grub
	# mount /dev/disk/by-id/scsi-SATA_disk1-part1 /mnt/boot/grub

4.5  Install the minimal system (read more https://www.debian.org/releases/jessie/amd64/apds03.html.en):

	# apt-get install debootstrap
	# debootstrap wheezy /mnt

The `debootstrap` command leaves the new system in an unconfigured state.  In Step 5, we will only do the minimum amount of configuration necessary to make the new system runnable.


## Step 5:  System Configuration

5.1  Copy these files from the LiveCD environment to the new system:

	# cp /etc/hostname /mnt/etc/
	# cp /etc/hosts /mnt/etc/

5.2  The `/mnt/etc/fstab` file should be empty except for a comment.  Add this line to the `/mnt/etc/fstab` file:

	/dev/disk/by-id/scsi-SATA_disk1-part1  /boot/grub  auto  defaults  0  1

5.3  Edit the `/mnt/etc/network/interfaces` file so that it contains something like this:

	# interfaces(5) file used by ifup(8) and ifdown(8)
	auto lo
	iface lo inet loopback
	
	auto eth0
	iface eth0 inet dhcp

Customize this file if the new system is not a DHCP client on the LAN.

5.4  Make virtual filesystems in the LiveCD environment visible to the new system and `chroot` into it:

	# mount --bind /dev  /mnt/dev
	# mount --bind /proc /mnt/proc
	# mount --bind /sys  /mnt/sys
	# chroot /mnt /bin/bash --login

5.5 Install the ZoL archive support in the chroot environment in the exact same way you did in point 1.2:

	# apt-get install locales
	# locale-gen en_US.UTF-8

	# wget http://archive.zfsonlinux.org/debian/pool/main/z/zfsonlinux/zfsonlinux_6_all.deb
	# apt-get install lsb-release
	# dpkg -i zfsonlinux_6_all.deb
	# apt-get update
	# apt-get install linux-image-amd64 debian-zfs

	# apt-get install grub2-common grub-pc zfs-initramfs
	# apt-get dist-upgrade

Even if you prefer a non-English system language, always ensure that en_US.UTF-8 is available.

**Warning:** This is the second time that you must wait for the SPL and ZFS modules to compile.  *Do not* try to skip this step by copying anything from the host environment into the chroot environment.

**Note:** This should install a kernel package and its headers and dkms packages.  Double-check that you are getting these packages from the ZoL archive if you are deviating from these instructions in any way.

Choose `/dev/disk/by-id/scsi-SATA_disk1` if prompted to install the MBR loader.

Ignore warnings that are caused by the chroot environment like:

* `Can not write log, openpty() failed (/dev/pts not mounted?)`
* `df: Warning: cannot read table of mounted file systems`

5.7  Set a root password on the new system:

	# passwd root

**Note:** At this time the system have been installed on a new ZFS root system so it is clean and pristine. It would be a good idea to do a zfs snapshot here. For example: `zfs snapshot rpool/ROOT/debian-1@YYYYMMDD-cleaninstall`

## Step 6:  Cleanup and First Reboot

6.1  Exit from the `chroot` environment back to the LiveCD environment:

	# exit

6.2  Run these commands in the LiveCD environment to dismount all filesystems:

	# umount /mnt/boot/grub
	# umount /mnt/dev
	# umount /mnt/proc
	# umount /mnt/sys
	# zfs umount -a
	# zpool export rpool

The `zpool export` command must succeed without being forced or the new system will fail to start.

6.3  We're done!

	# reboot


## Caveats and Known Problems

### This is an experimental system configuration.

This document was first published in 2010 to demonstrate that the `lzfs` implementation made ZoL 0.5 feature complete.  Upstream integration efforts began in 2012, and it will be at least a few more years before this kind of configuration is even minimally supported.

Gentoo, and its derivatives, are the only Linux distributions that are currently mainlining support for a ZoL root filesystem.

### `zpool.cache` inconsistencies cause random pool import failures.

The `/etc/zfs/zpool.cache` file embedded in the initrd for each kernel image must be the same as the `/etc/zfs/zpool.cache` file in the regular system.  Run `update-initramfs -c -k all` after any `/sbin/zpool` command changes the `/etc/zfs/zpool.cache` file.

This will be a recurring problem until issue [zfsonlinux/zfs#330](https://github.com/zfsonlinux/zfs/issues/330) is resolved.

### Every upgrade can break the system.

Debian GNU/Linux based systems remove old dkms modules before installing new dkms modules.  If the system crashes or restarts during a ZoL module upgrade, which is a failure window of several minutes, then the system becomes unbootable and must be rescued.

This will be a recurring problem until issue [zfsonlinux/pkg-zfs#12](https://github.com/zfsonlinux/pkg-zfs/issues/12) is resolved.


# Troubleshooting

## (i) MPT2SAS

Most problem reports for this tutorial involve `mpt2sas` hardware that does slow asynchronous drive initialization, like some IBM M1015 or OEM-branded cards that have been flashed to the reference LSI firmware.

The basic problem is that disks on these controllers are not visible to the Linux kernel until after the regular system is started, and ZoL does not hotplug pool members.  See https://github.com/zfsonlinux/zfs/issues/330.

Most LSI cards are perfectly compatible with ZoL, but there is no known fix if your card has this glitch.  Please use different equipment until the `mpt2sas` incompatibility is diagnosed and fixed, or donate an affected part if you want solution sooner.

## (ii) Areca

Systems that require the `arcsas` blob driver should add it to the `/etc/initramfs-tools/modules` file and run `update-initramfs -c -k all`.

Upgrade or downgrade the Areca driver if something like `RIP: 0010:[<ffffffff8101b316>]  [<ffffffff8101b316>] native_read_tsc+0x6/0x20` appears anywhere in kernel log.  ZoL is unstable on systems that emit this error message.


## (iii) GRUB Installation

Verify that the ZoL repository for the ZFS enhanced GRUB is installed:

	# rgrep -E '^deb .*archive.zfsonlinux.org' `find /etc/apt/sources.list*`
	/etc/apt/sources.list.d/zfsonlinux.list:deb http://archive.zfsonlinux.org/debian wheezy main
	/etc/apt/sources.list.d/zfsonlinux.list:deb http://archive.zfsonlinux.org/debian wheezy main

If not, install the ZoL repository trust package like you did in point 1.2 and 5.5:

	# wget http://archive.zfsonlinux.org/debian/pool/main/z/zfsonlinux/zfsonlinux_3%7Ewheezy_all.deb
	# dpkg -i zfsonlinux_3~wheezy_all.deb
	# apt-get update

Reinstall the `zfs-grub` package, which is an alias for a patched `grub-common` package:

	# apt-get install --reinstall zfs-grub

Afterwards, this should happen:

	# apt-cache search zfs-grub
	grub-common - GRand Unified Bootloader (common files)
	grub-pc - GRand Unified Bootloader, version 2 (PC/BIOS version)
	grub-pc-bin - GRand Unified Bootloader, version 2 (PC/BIOS binaries)
	
	# apt-cache show zfs-grub
	N: Can't select versions from package 'zfs-grub' as it is purely virtual
	N: No packages found
	
	# apt-cache policy grub-common zfs-grub
	grub-common:
	  Installed: 2.01-22debian1+zfs3~wheezy
	  Candidate: 2.01-22debian1+zfs3~wheezy
	  Version table:
	 *** 2.01-22debian1+zfs3~wheezy 0
	       1001 http://archive.zfsonlinux.org/debian/ wheezy/main amd64 Packages
	        100 /var/lib/dpkg/status
	     1.99-27+deb7u2 0
	        500 http://ftp.us.debian.org/debian/ wheezy/main amd64 Packages
	zfs-grub:
	  Installed: (none)
	  Candidate: (none)
	  Version table:

For safety, grub modules are never updated by the packaging system after initial installation. Manually refresh them by doing this:

	# cp /usr/lib/grub/i386-pc/*.mod /boot/grub/i386-pc/

If the problem persists, then open a bug report and attach the entire output of those `apt-get` commands.

GRUB packages in the ZoL repository are compiled against the stable distribution.

Note that GRUB does not currently dereference symbolic links in a ZFS filesystem, so you cannot use the  `/vmlinux` or `/initrd.img` symlinks as GRUB command arguments.

## (iv) VMware

* Set `disk.EnableUUID = "TRUE"` in the vmx file or vsphere configuration.  Doing this ensures that `/dev/disk` aliases are created in the guest.

## (v) QEMU/KVM/XEN

* In the `/etc/default/grub` file,  enable the `GRUB_TERMINAL=console` line and remove the `splash` option from the `GRUB_CMDLINE_LINUX_DEFAULT` line.  Plymouth can cause boot errors in these virtual environments that are difficult to diagnose.

* Set a unique serial number on each virtual disk.  (eg: `-drive if=none,id=disk1,file=disk1.qcow2,serial=1234567890`)

## (vi) Kernel Parameters

The `zfs-initramfs` package requires that `boot=zfs` always be on the kernel command line.  If the `boot=zfs` parameter is not set, then the init process skips the ZFS routine entirely.  This behavior is for safety; it makes the casual installation of the zfs-initramfs package unlikely to break a working system.

ZFS properties can be overridden on the the kernel command line with `rpool` and `bootfs` arguments.  For example, at the GRUB prompt:

`linux /ROOT/debian-1/@/boot/vmlinuz-3.0.0-15-generic boot=zfs rpool=AltPool bootfs=AltPool/ROOT/foobar-3`

## (vii) System Recovery

If the system randomly fails to import the root filesystem pool, then do this at the initramfs recovery prompt:

	# zpool export rpool
	: now export all other pools too
	# zpool import -d /dev/disk/by-id -f -N rpool
	: now import all other pools too
	# mount -t zfs -o zfsutil rpool/ROOT/debian-1 /root
	: do not mount any other filesystem
	# cp /etc/zfs/zpool.cache /root/etc/zfs/zpool.cache
	# exit

This refreshes the `/etc/zfs/zpool.cache` file.  The `zpool` command emits spurious error messages regarding missing or corrupt vdevs if the `zpool.cache` file is stale or otherwise incorrect.