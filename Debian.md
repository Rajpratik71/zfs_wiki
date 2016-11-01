Offical ZFS on Linux [DKMS][dkms] style packages are available from the [Debian GNU/Linux repository](https://tracker.debian.org/pkg/zfs-linux) for the following configurations.  The packages previously hosted at archive.zfsonlinux.org will not be updated and are not recommended for new installations.

**Debian Releases:** Jessie and newer (testing, sid)  
**Architectures:** amd64  

## Installation
For Debian Jessie, ZFS packages are provided by [backports](https://backports.debian.org/Instructions/).

Add jessie-backports  repository (ZFS packages are in `contrib` area):

	# echo "deb http://ftp.debian.org/debian jessie-backports main contrib" >> /etc/apt/sources.list.d/backports.list
	# apt update

Install kernel headers:

	# apt install linux-headers-$(uname -r)

Install zfs packages:

	# apt-get install -t jessie-backports zfs-dkms

If you want to boot from ZFS, you'll need `zfs-initramfs` package too:

	# apt-get install -t jessie-backports zfs-initramfs

[dkms]: https://en.wikipedia.org/wiki/Dynamic_Kernel_Module_Support
[debian-announce]: https://lists.debian.org/debian-devel-announce/2015/04/msg00006.html
[debian-itp]: https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=686447

* [[Debian GNU Linux initrd documentation]]  
* [[HOWTO install Debian GNU Linux to a Native ZFS Root Filesystem]]
* [[Dual booting OS X and Debian Jessie with ZFS root, cross mounting and full disk encryption]]