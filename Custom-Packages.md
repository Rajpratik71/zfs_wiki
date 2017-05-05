The following instructions assume you are building from an official [release tarball][release] or directly from the [git repository][git]. Most users should not need to do this and should preferentially use the distribution packages. As a general rule the distribution packages will be more tightly integrated, widely tested, and better supported. However, if your distribution of choice doesn't provide packages or you're a developer and want to roll your own here's the way to do it.

The first thing to be aware of is that the build system is capable of generating several different types of packages. Which type of package you choose depends on what's supported on your platform and exactly what your needs are.

* **DKMS** packages contain only the source code and scripts for rebuilding the kernel modules. When the DKMS package is installed kernel modules will be built for all available kernels. Additionally, when the kernel is upgraded new kernels modules will be automatically built for that kernel. This is particularly convenient for desktop systems which receive frequent kernel updates. The downside is that because the DKMS packages build the kernel modules from source a full development environment is required which may not be appropriate for large deployments.

* **kmods** packages are binary kernel modules which are compiled against a specific version of the kernel. This means that if you update the kernel you must recompile and install a new kmod package. If you don't frequently update your kernel or if your managing a large number of systems then kmod packages are a good choice.

* **kABI-tracking kmod** packages are similar to standard binary kmods and may be used with Enterprise Linux distributions like RHEL / CentOS.  These distributions provide a stable kABI which allows the same binary modules to be used with new versions of the distribution provided kernel. 

By default the build system will generate user packages and both DKMS and kmod style kernel packages if possible. The user packages can be used with either set of kernel packages and do not need to be rebuilt when the kernel is updated. You can also streamline the build process by building only the DKMS or kmod packages as shown below.

Be aware that when building directly from a git repository you must first run the *autogen.sh* script to create the *configure* script. This will require installing the GNU autotools packages for your distribution.  In addition you must install all the necessary development tools and headers for your distribution.

For Debian and Ubuntu:

```
$ sudo apt-get install build-essential autoconf libtool gawk alien fakeroot gdebi linux-headers-$(uname -r)
$ sudo apt-get install zlib1g-dev uuid-dev libattr1-dev libblkid-dev libselinux-dev libudev-dev parted lsscsi wget ksh
```

For RHEL and CentOS:

```
$ sudo yum groupinstall "Development Tools" parted lsscsi wget ksh
$ sudo yum install kernel-devel zlib-devel libattr-devel libuuid-devel libblkid-devel libselinux-devel libudev-devel
```

For Fedora:

```
sudo dnf groupinstall "C Development Tools and Libraries"
sudo dnf install kernel-devel zlib-devel libuuid-devel libattr-devel libblkid-devel libselinux-devel libudev-devel
```

If you want to use the official released tarballs, then use the following commands to fetch and prepare the source.

```
$ wget http://archive.zfsonlinux.org/downloads/zfsonlinux/spl/spl-x.y.z.tar.gz
$ wget http://archive.zfsonlinux.org/downloads/zfsonlinux/zfs/zfs-x.y.z.tar.gz

$ tar -xzf spl-x.y.z.tar.gz
$ tar -xzf zfs-x.y.z.tar.gz
```
	
If instead you would like to use the git version, you can clone it from Github and prepare the source like this.

```
$ git clone https://github.com/zfsonlinux/spl.git
$ git clone https://github.com/zfsonlinux/zfs.git

$ cd spl
$ ./autogen.sh
$ cd ../zfs
$ ./autogen.sh
$ cd ..
```

Once the source has been prepared you'll need to decide what kind of packages you're building and jump the to appropriate section below.  Note that not all package types are supported for all platforms.

## DKMS (rpm-only)

Building rpm-based DKMS and user packages can be done as follows.  However, be aware that the build system currently does not support building deb-based DKMS packages.


```
$ cd spl
$ ./configure
$ make pkg-utils rpm-dkms
```

Install the spl packages.  They are required dependencies to build zfs in the next step.

For RHEL, Centos and Fedora:

```
$ sudo yum localinstall *.<arch>.rpm
```

Next change to the zfs directory and repeat the same process.

```
$ cd ../zfs
$ ./configure
$ make pkg-utils rpm-dkms
```

For RHEL, Centos and Fedora:

```
$ sudo yum localinstall *.<arch>.rpm
```

## kmod (rpm and deb)

The key thing to know when building a kmod package is that a specific Linux kernel must be specified. At configure time the build system will make an educated guess as to which kernel you want to build against. However, if configure is unable to locate your kernel development headers, or you want to build against a different kernel, you must specify the exact path with the *--with-linux* and *--with-linux-obj* options.

```
$ cd spl
$ ./configure
$ make pkg-utils pkg-kmod
```

Install the spl packages.  They are required dependencies to build zfs in the next step.

For Debian and Ubuntu:

```
$ for file in *.deb; do sudo gdebi -q --non-interactive $file; done
```

For RHEL, Centos, and Fedora:

```
$ sudo yum localinstall *.<arch>.rpm
```

Next change to the zfs directory and repeat the same process.

```
$ cd ../zfs
$ ./configure
$ make pkg-utils pkg-kmod
```

For Debian and Ubuntu:

```
$ for file in *.deb; do sudo gdebi -q --non-interactive $file; done
```

For RHEL, Centos, and Fedora:

```
$ sudo yum localinstall *.<arch>.rpm
```

## kABI-tracking kmod (rpm-only)

The process for building kABI-tracking kmods is almost identical to for building normal kmods.  However, it will only produce binaries which can be used by multiple kernels if the distribution supports a stable kABI.  Enterprise Linux distributions such as RHEL and CentOS provide this but faster moving distributions like Fedora do not.  The build system also does not support building kABI-tracking deb packages.  In order to request kABI-tracking package the *--with-spec=redhat* option must be passed to configure.

```
$ cd spl
$ ./configure --with-spec=redhat
$ make pkg-utils pkg-kmod
```

Install the spl packages.  They are required dependencies to build zfs in the next step.

For RHEL and Centos:

```
$ sudo yum localinstall *.<arch>.rpm
```

Next change to the zfs directory and repeat the same process.

```
$ cd ../zfs
$ ./configure --with-spec=redhat
$ make pkg-utils pkg-kmod
```

For RHEL and Centos:

```
$ sudo yum localinstall *.<arch>.rpm
```

[release]: https://github.com/zfsonlinux/zfs/release
[git]: https://github.com/zfsonlinux/zfs