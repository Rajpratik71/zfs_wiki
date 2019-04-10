Offical ZFS on Linux [DKMS][dkms] style packages are available from the [Debian GNU/Linux repository](https://tracker.debian.org/pkg/zfs-linux) for the following configurations.  The packages previously hosted at archive.zfsonlinux.org will not be updated and are not recommended for new installations.

**Debian Releases:** Jessie, Stretch, and newer (testing, sid)
**Architectures:** amd64

# Table of contents
- [Installation](#installation)
- [Jessie to Stretch update](#jessie-to-stretch-update)
- [[Debian GNU Linux initrd documentation]]
- [[Debian Stretch Root on ZFS]]
- [[Dual booting OS X and Debian Jessie with ZFS root, cross mounting and full disk encryption]]

## Installation
For Debian Stretch, ZFS packages are included in the [contrib repository](https://packages.debian.org/source/stretch/zfs-linux). Newer ZFS packages are provided by [backports](https://backports.debian.org/Instructions/).

Add the backports repository:

        # echo "deb http://deb.debian.org/debian stretch-backports main contrib" > /etc/apt/sources.list.d/stretch-backports.list

Update the list of packages:

        # apt update

Install kernel headers and other dependencies:

        # apt install linux-headers-$(uname -r) linux-headers-amd64 dkms build-essential libelf-dev

Install zfs packages:

        # apt-get install -t stretch-backports zfs-dkms zfsutils-linux

If you want to boot from ZFS, you'll need `zfs-initramfs` package too:

        # apt-get install -t stretch-backports zfs-initramfs

[dkms]: https://en.wikipedia.org/wiki/Dynamic_Kernel_Module_Support

## Jessie to Stretch update
From Debian Stretch packages are included in Debian official `contrib` repository. Steps to reinstall packages:

1) Remove old packages:
```
# apt remove debian-zfs libzfs2 zfs-dkms zfsonlinux zfsutils libzpool2 libzfs2linux libnvpair1linux libnvpair1 libuutil1 libuutil1linux  libzpool2linux zfs-zed zfsutils-linux spl-dkms
```
2) Remove ZFSonLinux repository.

3) [Install new packages](#installation), as described above.
