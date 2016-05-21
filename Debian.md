[DKMS][dkms] style packages are provided for Debian from the official zfsonlinux.org repository.

Packages from ZFS On Linux are available for the following configurations:

**Debian Releases:** 7.x (Wheezy), 8.x (Jessie), Sid (Eternal unstable)  
**Architectures:** amd64  

Although ZFS On Linux is now officially in the Debian GNU/Linux repository, we will continue to provide packages for Wheezy and Jessie (but only in the amd64 architecture). The ZoL packages differ slightly compared to the official Debian GNU/Linux packages, mostly in that it provides new, improved **sharenfs**, **sharesmb** and **shareiscsi** options.

To add the repository to your system, install the zfsonlinux package as shown below. This will add the /etc/apt/sources.list.d/zfsonlinux.list and /etc/apt/trusted.gpg.d/zfsonlinux.gpg files to your computer. Afterwards, you can install zfs like any other Debian package using apt-get. As new updated packages are made available they will be detected and installed as part of the standard update process.

**Location:** /etc/apt/trusted.gpg.d/zfsonlinux.gpg  
**Debian Package:** http://archive.zfsonlinux.org/debian/pool/main/z/zfsonlinux/zfsonlinux_8_all.deb  
**Download from:** [pgp.mit.edu][pubkey]  
**Download from:** [zfsonlinux.org][pubkey-zol]  
**Key ID:** 4D5843EA  
**Fingerprint:** 5EB3 5C47 B97A C11F 551D  B287 201C 3129 4D58 43EA  

```
$ su -
$ apt-get install lsb-release
$ wget http://archive.zfsonlinux.org/debian/pool/main/z/zfsonlinux/zfsonlinux_8_all.deb
$ dpkg -i zfsonlinux_8_all.deb
$ gpg --quiet --with-fingerprint /etc/apt/trusted.gpg.d/zfsonlinux.gpg
pub  4096R/4D5843EA 2014-09-24 Turbo Fredriksson <turbo@bayour.com>
      Key fingerprint = 5EB3 5C47 B97A C11F 551D  B287 201C 3129 4D58 43EA
uid                            Turbo Fredriksson (Turbo @ Debian GNU/Linux) <turbo@debian.org>
uid                            [jpeg image of size 1983374]
uid                            Turbo Fredriksson (Turbo @ Gmail) <fransurbo@gmail.com>
sub  4096R/9ACF3117 2014-09-24
sub  3072D/6EBBF50F 2014-09-24
sub  4096R/7DFFA34D 2014-09-24
```

If you wish to use the Debian GNU/Linux ZoL snapshot packages (pre-releases), you will need to uncomment the line(s) related to the 'dailies' repository (at the end of the /etc/apt/sources.list.d/zfsonlinux.list file) by removing the hash mark at the beginning of the line.

There is an auto builder in place to build packages two hours after a commit had been made to the *spl* and *zfs* repositories. After the two hour wait period (to catch consecutive commits) packages will then be built and uploaded to the ZoL package repositories automatically.

After changing the *sources.list* file, run apt-get to update and install ZoL:

```
$ apt-get update
$ apt-get install debian-zfs
```
[dkms]: https://en.wikipedia.org/wiki/Dynamic_Kernel_Module_Support
[pubkey]: https://pgp.mit.edu/pks/lookup?search=turbo+fredriksson&op=index
[pubkey-zol]: http://zfsonlinux.org/4D5843EA.asc
[debian-announce]: https://lists.debian.org/debian-devel-announce/2015/04/msg00006.html
[debian-itp]: https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=686447

* [[Debian GNU Linux initrd documentation]]  
* [[HOWTO install Debian GNU Linux to a Native ZFS Root Filesystem]]
* [[Dual booting OS X and Debian Jessie with ZFS root, cross mounting and full disk encryption]]