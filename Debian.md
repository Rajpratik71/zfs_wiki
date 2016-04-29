# Debian

[DKMS][dkms] style packages are provided for Debian from the official zfsonlinux.org repository.  These packages are less actively maintained and may not be updated as new releases are tagged.  The Debian project has [announced their intention][debian-announce] to ship ZFS on Linux as part of Debian and that work is [currently ongoing][debian-itp].  Packages are available for the following configurations:

**Debian Releases:** 7.x (Wheezy), 8.x (Jessie)  
**Architectures:** amd64  

To add the repository to your system install the zfsonlinux package as shown below. This will add the /etc/apt/sources.list.d/zfsonlinux.list and /etc/apt/trusted.gpg.d/zfsonlinux.gpg files to your computer. Afterwards, you can install zfs like any other Debian package using apt-get. As new updated packages are made available they will be detected and installed as part of the standard update process.

**Location:** /etc/apt/trusted.gpg.d/zfsonlinux.gpg  
**Debian Package:** http://archive.zfsonlinux.org/debian/pool/main/z/zfsonlinux/zfsonlinux_6_all.deb  
**Download from:** [pgp.mit.edu][pubkey]  
**Download from:** [zfsonlinux.org][pubkey-zol]  
**Key ID:** 4D5843EA  
**Fingerprint:** 5EB3 5C47 B97A C11F 551D  B287 201C 3129 4D58 43EA  

```
$ su -
$ apt-get install lsb-release
$ wget http://archive.zfsonlinux.org/debian/pool/main/z/zfsonlinux/zfsonlinux_6_all.deb
$ dpkg -i zfsonlinux_6_all.deb
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

Then run apt-get to update and install ZoL:

```
$ apt-get update
$ apt-get install debian-zfs
```
[dkms]: https://en.wikipedia.org/wiki/Dynamic_Kernel_Module_Support
[pubkey]: https://pgp.mit.edu/pks/lookup?search=turbo+fredriksson&op=index
[pubkey-zol]: http://zfsonlinux.org/4D5843EA.asc
[debian-announce]: https://lists.debian.org/debian-devel-announce/2015/04/msg00006.html
[debian-itp]: https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=686447