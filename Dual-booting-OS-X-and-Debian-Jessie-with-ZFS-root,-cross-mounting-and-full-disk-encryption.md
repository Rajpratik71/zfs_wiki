## Synopsis

ZFS is the only filesystem that can be mounted on the Mac with full
write support and *also* used as a Linux root partition: [FUSE for OS X](https://github.com/osxfuse/osxfuse/wiki/List-of-fuse-for-os-x-file-systems)
support for ext is constrained to ext2 with unstable write support,
XFS is read-only and BtrFS isn't mountable on OS X at all. Thus,
if you need cross-mounting on a dual-boot Mac, ZFS is the best and only
option.

As for encryption, Linux can open FileVault2 volumes with
[libfvde](https://code.google.com/p/libfvde/) and supports HFS+
out of the box, so that part of the equation is solved. The other way
round, you can't open LUKS volumes on OS X, but you *can* open
TrueCrypt containers. On Linux, TrueCrypt containers can be opened as
well, and not only with the (now abandoned) TrueCrypt software, but
natively with cryptsetup since [version 1.6](https://code.google.com/p/cryptsetup/wiki/Cryptsetup160).

So for OS X, we'll use HFS+ in a FileVault2 volume, and for Linux
we'll use ZFS in a TrueCrypt container.

Note that while Oracle has added [ZFS encryption](http://www.oracle.com/technetwork/articles/servers-storage-admin/manage-zfs-encryption-1715034.html)
as a closed-source feature in zpool version 30, their implementation is
[susceptible to watermarking attacks](http://lists.freebsd.org/pipermail/freebsd-hackers/2013-September/043340.html).
The OpenZFS community may come up with a better (incompatible) design
in the future, see [issue #494](https://github.com/zfsonlinux/zfs/issues/494).
In the meantime, the best solution is to layer ZFS on top of FileVault2,
LUKS or TrueCrypt.

Unfortunately, both TrueCrypt on OS X as well as libfvde on Linux
don't run in kernel space but plug in to FUSE. Thus, when
cross-mounting the other operating system's filesystem, the encryption
happens in user space which incurs a performance penalty. However,
that constraint only pertains to encryption of the cross-mounted filesystem.
The native filesystem is always encrypted in kernel space (using FileVault2
on OS X and dm-crypt on Linux). Also, the code to access the actual HFS+
or ZFS filesystems runs in kernel space in either case. Last not least,
TrueCrypt on OS X makes use of AESNI, so even though the program runs
in user space, the actual encryption happens in hardware (on CPUs that
support AESNI).

## Partition layout

```
$ diskutil list
/dev/disk0
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *960.2 GB   disk0
   1:                        EFI EFI                     209.7 MB   disk0s1
   2:          Apple_CoreStorage                         315.0 GB   disk0s2
   3:                 Apple_Boot Boot OS X               134.2 MB   disk0s3
   4:                 Apple_Boot Recovery HD             650.0 MB   disk0s4
   5: 7FFEC5C9-2D00-49B7-8941-3EA10A5586B7               644.2 GB   disk0s5
/dev/disk1
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:                  Apple_HFS Macintosh HD           *314.6 GB   disk1
/dev/disk2
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:                                                   *644.2 GB   disk2
```

There are five partitions here: A 200 MiB EFI partition, the FileVault2 volume
(315 GB in this case, cleartext version mounted as disk1),
a 128 MiB OS X boot partition containing the FileVault key material, a 650 MB
OS X recovery partition and finally the TrueCrypt container (644 GB in this
case, cleartext version mounted as disk2).

There's no need for a Linux boot partition, we'll just store the kernel
and initrd on the EFI partition and load it with [gummiboot](http://freedesktop.org/wiki/Software/gummiboot/). Same for swap, we can use a ZVOL for that.
This leads to a simpler and cleaner partition layout.

## Step 1: Install OS X with FileVault2

Although it's possible to convert an existing volume to FileVault2,
I recommend starting afresh: Wipe the disk, create a new partition
of type "Mac OS Extended (encrypted)" in Disk Utility, shrink it
to the desired size and add the recovery partition and an unformatted
partition for the TrueCrypt container. One benefit of this approach
is that you're not asked to escrow the FileVault2 keys with Apple,
which is the last thing you want.

## Step 2: Install TrueCrypt and OpenZFS on OS X

Download and install the following:

  1. [FUSE for OS X](http://sourceforge.net/projects/osxfuse/files/)
  2. [TrueCrypt 7.1a Mac OS X](https://truecrypt.ch/downloads/)
  3. [OpenZFS on OS X](https://openzfsonosx.org/wiki/Downloads)

When installing TrueCrypt, be sure to skip the bundled FUSE release
as it is out of date.

## Step 3: Create the encrypted zpool

We can comfortably do that on OS X, no need to fiddle around
in the Debian installer.

Start TrueCrypt and use the Volume Creation Wizard in the Tools menu
to create a "Standard TrueCrypt" volume on partition disk0s5. When it
comes to selection of a hash algorithm, steer clear of the
NSA-designed SHA-512 and resort to a hash designed by the scientific
community. Whirlpool seems like a decent option until Keccak is
more widely supported.

Mount the container in the TrueCrypt GUI and select the checkbox to not
mount any filesystem in the container.

Create the zpool and root filesystem (replace /dev/disk2 with whatever
device number the TrueCrypt container was assigned):

```
$ sudo zpool create -o ashift=12 -o feature@lz4_compress=enabled mypool /dev/disk2
$ sudo zfs create -o atime=off -o compression=lz4 mypool/root
$ sudo zpool export mypool
$ diskutil eject disk2
```

Note how you can use advanced, OpenZFS-specific features like lz4
compression on the Mac just as you would on Linux thanks to OpenZFS'
awesome cross-platform support.

Click "Dismount" in the TrueCrypt GUI to close the container.

## Step 4: Install Debian Jessie

I decided to do an offline installation without any need for Internet
access, one of the reasons being that most contemporary Macs use
Broadcom Ethernet chips which require proprietary firmware and
thus won't work out of the box in a Debian live environment.

Unfortunately, there are no live images yet for Jessie so we'll have to
use a Wheezy live image to bootstrap the installation. Download the
[Wheezy rescue disk](http://cdimage.debian.org/debian-cd/current-live/amd64/iso-hybrid/) and the [Jessie install disk 1](http://cdimage.debian.org/cdimage/jessie_di_beta_1/amd64/iso-dvd/)
and burn each of them on a DVD or USB stick.

Next, download the [Wheezy backport of cryptsetup 1.6](https://packages.debian.org/wheezy-backports/cryptsetup-bin). You need two debs, cryptsetup-bin and
libcryptsetup4. I stored these temporarily on the EFI partition where I could
access them in the live environment. (You can't mount the OS X partition in
the live environment because it's encrypted and there's no libfvde available.)

We'll also need the [Jessie gummiboot package](https://packages.debian.org/jessie/admin/gummiboot) which isn't on install disk 1.

Finally, manually download the [ZFS on Linux packages](http://archive.zfsonlinux.org) and store them on the EFI partition as well. You'll need 11 debs:
libnvpair1 libuutil1 libzfs2 libzpool2 spl zfsutils dkms spl-dkms zfs-dkms
zfs-initramfs zfsonlinux.

Boot from the Wheezy live image. Deinstall the package zfs-fuse as it only
supports zpool version 23 and conflicts with ZFS on Linux. Mount the EFI
partition and install the Wheezy backport of cryptsetup 1.6 as well as the
ZFS on Linux debs.

Open the TrueCrypt container and mount the root filesystem:

```
$ cryptsetup open --type tcrypt --allow-discards /dev/sda5 myvdev
$ zpool import -a
$ mount --bind /mypool/root /mnt
```

Insert the Jessie install disk 1 and mount it in /jessie. Locate the
[debian-archive-keyring package](https://packages.debian.org/jessie/debian-archive-keyring) on the disk and install it. (Alternatively, pass --no-check-gpg
to the debootstrap command below.) Then install the base system:

```
$ debootstrap --include=dkms,console-setup,cryptsetup,file,libc6-dev,linux-headers-amd64,linux-image-3.14-2-amd64,locales,lsb-release jessie /mnt file:///jessie/
$ echo myhostname > /mnt/etc/hostname
$ chroot /mnt dpkg-reconfigure locales
$ chroot /mnt passwd root
$ mount --bind /proc /mnt/proc
$ mount --bind /sys /mnt/sys
$ mount --bind /dev /mnt/dev
```

Add `myhostname` (or whatever hostname you've chosen) to the 127.0.0.1 entry
in /mnt/etc/hosts. Add the lines `auto lo` and `iface lo inet loopback` to
/mnt/etc/network/interfaces. Customize the installation with dpkg-reconfigure
as necessary, e.g. adjust the keyboard layout.

Finally, use `dpkg --root /mnt --install` to install the ZFS on Linux
packages. Never mind that they're intended for Wheezy, they work perfectly
fine with Jessie.

## Step 5: Install gummiboot

Add entries to /mnt/etc/fstab like this:

```
/dev/disk/by-label/EFI  /boot/efi       auto    defaults        0       1
/dev/mapper/myvdev      /               auto    defaults        0       1
```

Don't be confused by the second line, it's necessary to keep update-initramfs
happy. (Obviously /dev/mapper/myvdev isn't the actual root device but rather
contains the zpool, which in turn contains a dataset to be used as root
partition.)

Add an entry to /mnt/etc/crypttab like this, replace the dots with the id
of your drive:

```
myvdev /dev/disk/by-id/wwn-0x................-part5 none tcrypt,discard
```

The current version 0.115 of the initramfs-tools package has broken
TrueCrypt support. I've submitted a [patch](https://bugs.debian.org/cgi-bin/bugreport.cgi?msg=10;filename=tcrypt.diff;att=1;bug=748286) which you'll have
to apply manually to /usr/share/initramfs-tools/scripts/local-top/cryptroot until it gets merged by the maintainers.

Create the initrd and mount the EFI partition in preparation of the
gummiboot installation. Ignore any warnings about allegedly missing
hash modules:

```
$ chroot /mnt update-initramfs -u
$ mkdir /mnt/boot/efi
$ mount /dev/disk/by-label/EFI /mnt/boot/EFI
```

Install the gummiboot package you've downloaded earlier. Edit
/mnt/etc/default/gummiboot to look like this:

```
# Set the root device. If not specified, this is read from /proc/cmdline
GUMMIBOOT_ROOT="ZFS=mypool/root"

# Set the options to pass to the kernel command line
GUMMIBOOT_OPTIONS="ro boot=zfs rpool=mypool bootfs=mypool/root"
```

On my machine with dual GPUs I also had to add the kernel options
`nouveau.modeset=0 i915.modeset=0`, otherwise the screen would become
garbled at boot.

The current version 45-2 of the gummiboot package has a couple of bugs,
the postinst script assumes that gummiboot is already installed and does
an *update* instead of an *install*. Also, it assumes the kernel version
we want to install is that of the Wheezy live system. Just work around
the bugs like this:

```
$ chroot /mnt update-gummiboot `ls /mnt/boot/vmlinuz* | cut -d- -f2-`
$ chroot /mnt gummiboot install --path=/boot/efi --no-variables
```

We're done. Unmount everything and reboot:

```
$ umount /mnt/dev
$ umount /mnt/sys
$ umount /mnt/proc
$ umount /mnt/boot/efi
$ umount /mnt
$ zpool export mypool
$ reboot
```

Because we invoked gummiboot with `--no-variables`, the NVRAM variable
`efi-boot-device` was left untouched and the system will boot OS X by
default. To boot Linux, hold down the option key during startup and
select the disk icon labeled "EFI Boot".
