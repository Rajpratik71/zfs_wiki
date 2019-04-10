Offical ZFS on Linux [DKMS](https://en.wikipedia.org/wiki/Dynamic_Kernel_Module_Support) style packages are available from the [Debian GNU/Linux repository](https://tracker.debian.org/pkg/zfs-linux) for the following configurations.  The packages previously hosted at archive.zfsonlinux.org will not be updated and are not recommended for new installations.

**Debian Releases:** Jessie, Stretch, and newer (testing, sid)
**Architectures:** amd64

# Table of contents
- [Installation](#installation)
- [Jessie to Stretch update](#jessie-to-stretch-update)
- [Related Links](#related-links)

## Installation
For Debian Stretch, ZFS packages are included in the [contrib repository](https://packages.debian.org/source/stretch/zfs-linux). Newer ZFS packages are provided by the [backports repository](https://backports.debian.org/Instructions/).

If you want to boot from ZFS, see [[Debian Stretch Root on ZFS]] instead.

Add the backports repository:

    # vi /etc/apt/sources.list.d/stretch-backports.list
    deb http://deb.debian.org/debian stretch-backports main contrib
    deb-src http://deb.debian.org/debian stretch-backports main contrib

    # vi /etc/apt/preferences.d/90_zfs
    Package: libnvpair1linux libuutil1linux libzfs2linux libzpool2linux spl-dkms zfs-dkms zfs-test zfsutils-linux zfsutils-linux-dev zfs-zed
    Pin: release n=stretch-backports
    Pin-Priority: 990

Update the list of packages:

    # apt update

Install the kernel headers and other dependencies:

    # apt install --yes dpkg-dev linux-headers-$(uname -r) linux-image-amd64

Install the zfs packages:

    # apt-get install zfs-dkms zfsutils-linux

## Jessie to Stretch update
From Debian Stretch packages are included in Debian official `contrib` repository. Steps to reinstall packages:

1) Remove old packages:
```
# apt remove debian-zfs libzfs2 zfs-dkms zfsonlinux zfsutils libzpool2 libzfs2linux libnvpair1linux libnvpair1 libuutil1 libuutil1linux  libzpool2linux zfs-zed zfsutils-linux spl-dkms
```
2) Remove ZFSonLinux repository.

3) [Install new packages](#installation), as described above.

## Related Links
- [[Debian GNU Linux initrd documentation]]
- [[Debian Stretch Root on ZFS]]
- [[Dual booting OS X and Debian Jessie with ZFS root, cross mounting and full disk encryption]]
