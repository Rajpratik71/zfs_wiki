Only [DKMS][dkms] style packages can be provided for Fedora from the official zfsonlinux.org repository.  This is because Fedora is a fast moving distribution which does not provide a stable kABI. These packages track the official ZFS on Linux tags and are updated as new versions are released.  Packages are available for the following configurations:

**Fedora Releases:** 25, 26, 27
**Architectures:** x86_64  

To simplify installation a zfs-release package is provided which includes a zfs.repo configuration file and the ZFS on Linux public signing key.  All official ZFS on Linux packages are signed using this key, and by default both yum and dnf will verify a package's signature before allowing it be to installed.  Users are strongly encouraged to verify the authenticity of the ZFS on Linux public key using the fingerprint listed here.

**Location:** /etc/pki/rpm-gpg/RPM-GPG-KEY-zfsonlinux  
**Fedora 25 Package:** http://download.zfsonlinux.org/fedora/zfs-release.fc25.noarch.rpm  
**Fedora 26 Package:** http://download.zfsonlinux.org/fedora/zfs-release.fc26.noarch.rpm  
**Fedora 27 Package:** http://download.zfsonlinux.org/fedora/zfs-release.fc27.noarch.rpm  
**Download from:** [pgp.mit.edu][pubkey]  
**Fingerprint:** C93A FFFD 9F3F 7B03 C310  CEB6 A9D5 A1C0 F14A B620

```
$ sudo dnf install http://download.zfsonlinux.org/fedora/zfs-release$(rpm -E %dist).noarch.rpm
$ gpg --quiet --with-fingerprint /etc/pki/rpm-gpg/RPM-GPG-KEY-zfsonlinux
pub  2048R/F14AB620 2013-03-21 ZFS on Linux <zfs@zfsonlinux.org>
    Key fingerprint = C93A FFFD 9F3F 7B03 C310  CEB6 A9D5 A1C0 F14A B620
    sub  2048R/99685629 2013-03-21
```

The ZFS on Linux packages should be installed with dnf on Fedora. Note that it is important to make sure that the matching *kernel-devel* package is installed for the running kernel since DKMS requires it to build ZFS.

```
$ sudo dnf install kernel-devel zfs
```

**Systemd Update:**

When upgrading to the zfs-0.7.4 release it's recommended that users manually reset the zfs systemd presets.  Failure to do so can result in the pool not automatically importing when the system is rebooted.

```
systemctl preset zfs-import-cache zfs-import-scan zfs-import.target zfs-mount zfs-share zfs-zed zfs.target
```

**Repository Renamed:**

The repository has been renamed `http://download.zfsonlinux.org/` from `http://archive.zfsonlinux.org/`.  An updated zfs-release package was added to the legacy repository to move users to the new repository.  Users which modifed the `/etc/yum.repos.d/zfs.repo` must manually replace this file with the updated `/etc/yum.repos.d/zfs.repo.rpmsave`.

## Testing Repositories

In addition to the primary *zfs* repository a *zfs-testing* repository is available. This repository, which is disabled by default, contains the latest version of ZFS on Linux which is under active development. These packages are made available in order to get feedback from users regarding the functionality and stability of upcoming releases. These packages **should not** be used on production systems. Packages from the testing repository can be installed as follows.

```
$ sudo dnf --enablerepo=zfs-testing install kernel-devel zfs 
```

[dkms]: https://en.wikipedia.org/wiki/Dynamic_Kernel_Module_Support
[pubkey]: http://pgp.mit.edu/pks/lookup?search=0xF14AB620&op=index&fingerprint=on