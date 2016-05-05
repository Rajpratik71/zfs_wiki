[kABI-tracking kmod][kmod] or [DKMS][dkms] style packages are provided for RHEL / CentOS based distributions from the official zfsonlinux.org repository. These packages track the official ZFS on Linux tags and are updated as new versions are released.  Packages are available for the following configurations:

**EL Releases:** 6.x, 7.x  
**Architectures:** x86_64  

To simplify installation a zfs-release package is provided which includes a zfs.repo configuration file and the ZFS on Linux public signing key.  All official ZFS on Linux packages are signed using this key, and by default yum will verify a package's signature before allowing it be to installed.  Users are strongly encouraged to verify the authenticity of the ZFS on Linux public key using the fingerprint listed here.

**Location:** /etc/pki/rpm-gpg/RPM-GPG-KEY-zfsonlinux  
**EL6 Package:** http://archive.zfsonlinux.org/epel/zfs-release.el6.noarch.rpm  
**EL7 Package:** http://archive.zfsonlinux.org/epel/zfs-release.el7.noarch.rpm  
**Download from:** [pgp.mit.edu][pubkey]  
**Fingerprint:** C93A FFFD 9F3F 7B03 C310  CEB6 A9D5 A1C0 F14A B620

```
$ sudo yum localinstall --nogpgcheck http://archive.zfsonlinux.org/epel/zfs-release$(rpm -E %dist).noarch.rpm
$ gpg --quiet --with-fingerprint /etc/pki/rpm-gpg/RPM-GPG-KEY-zfsonlinux
pub  2048R/F14AB620 2013-03-21 ZFS on Linux <zfs@zfsonlinux.org>
    Key fingerprint = C93A FFFD 9F3F 7B03 C310  CEB6 A9D5 A1C0 F14A B620
    sub  2048R/99685629 2013-03-21
```

After installing the zfs-release package and verifying the public key users can opt to install ether the kABI-tracking kmod or DKMS style packages.  For most users the kABI-tracking kmod packages are recommended in order to avoid needing to rebuild ZFS for every kernel update.  DKMS packages are recommended for users running a non-distribution kernel or for users who wish to apply local customizations to ZFS on Linux.

## kABI-tracking kmod

By default the zfs-release package is configured to install DKMS style packages so they will work with a wide range of kernels.  In order to install the kABI-tracking kmods the *baseurl* in the */etc/yum.repos.d/zfs.repo* file must be updated as shown.  Keep in mind that the kABI-tracking kmods are only verified to work with the distribution provided kernel.

```diff
# /etc/yum.repos.d/zfs.repo
 [zfs]
 name=ZFS on Linux for EL<6|7>
-baseurl=http://archive.zfsonlinux.org/epel/<6|7>/$basearch/
+baseurl=http://archive.zfsonlinux.org/epel/<6|7>/kmod/$basearch/
 enabled=1
```

The ZFS on Linux packages can now be installed using yum.

```
$ sudo yum install zfs
```

## DKMS

To install DKMS style packages issue the following yum command.  Note that it is important to make sure that the matching *kernel-devel* package is installed for the running kernel since DKMS requires it to build ZFS.

```
$ sudo yum install kernel-devel zfs
```

## Testing Repositories

In addition to the primary *zfs* repository a *zfs-testing* repository is available. This repository, which is disabled by default, contains the latest version of ZFS on Linux which is under active development. These packages are made available in order to get feedback from users regarding the functionality and stability of upcoming releases. These packages **should not** be used on production systems. Packages from the testing repository can be installed as follows.

```
$ sudo yum --enablerepo=zfs-testing install kernel-devel zfs 
```

[kmod]: http://elrepoproject.blogspot.com/2016/02/kabi-tracking-kmod-packages.html
[dkms]: https://en.wikipedia.org/wiki/Dynamic_Kernel_Module_Support
[pubkey]: http://pgp.mit.edu/pks/lookup?search=0xF14AB620&op=index&fingerprint=on