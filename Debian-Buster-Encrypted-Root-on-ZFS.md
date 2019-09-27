This experimental guide has been made official at [[Debian Buster Root on ZFS]].

If you have an existing system installed from the experimental guide, adjust your sources:

    # vi /etc/apt/sources.list.d/buster-backports.list
    deb http://deb.debian.org/debian buster-backports main contrib
    deb-src http://deb.debian.org/debian buster-backports main contrib
    
    # vi /etc/apt/preferences.d/90_zfs
    Package: libnvpair1linux libuutil1linux libzfs2linux libzpool2linux zfs-dkms zfs-initramfs zfs-test zfsutils-linux zfs-zed
    Pin: release n=buster-backports
    Pin-Priority: 990

This will allow you to upgrade from the locally-built packages to the official buster-backports packages.

You should set a root password before upgrading:

    # passwd

Apply updates:

    # apt update
    # apt dist-upgrade

Patch zfs-initramfs:

    # apt install curl
    # curl https://github.com/zfsonlinux/zfs/commit/f335b8f.patch | \
      patch /usr/share/initramfs-tools/scripts/zfs
    # update-initramfs -u -k all

The patch fixes zfs-initramfs's Plymouth support for prompting for encryption passphrases. It is included in 0.8.2, so by the time of the next package update, the patch should not need to be reapplied.

Reboot:

    # reboot

If the bpool fails to import, then enter the rescue shell (which requires a root password) and run:

    # zpool import -f bpool
    # zpool export bpool
    # reboot
