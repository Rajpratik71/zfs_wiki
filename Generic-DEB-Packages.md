# Generic DEB Packages

The following instructions assume you are building from an official [release tarball][release] or directly from the [git repository][git]. Most users should not need to do this and should preferentially use the [[Debian]] or [[Ubuntu]] packages. As a general rule the distribution packages will be more tightly integrated, widely tested, and better supported. However, if your distribution of choice doesn't provide packages or you're a developer and want to roll your own here's the way to do it.

The first thing to be aware of is that the build system is capable of generating several different types of packages. Which type of package you choose depends on what's supported on your platform and exactly what your needs are.

* **DKMS** packages contain only the source code and scripts for rebuilding the kernel modules. When the DKMS package is installed kernel modules will be built for all available kernels. Additionally, when the kernel is upgraded new kernels modules will be automatically built for that kernel. This is particularly convenient for desktop systems which receive frequent kernel updates. The downside is that because the DKMS packages build the kernel modules from source a full development environment is required which may not be appropriate for large deployments.

* **kmods** packages are binary kernel modules which are compiled against a specific version of the kernel. This means that if you update the kernel you must recompile and install a new kmod package. If you don't frequently update your kernel or if your managing a large number of systems then kmod packages are a good choice.

* **kABI-tracking kmod** packages are similar to standard binary kmods and may be used with Enterprise Linux distributions like RHEL / CentOS.  These distributions provide a stable kABI which allows the same binary modules to be used with new versions of the distribution provided kernel. 

By default the build system will generate user packages and both DKMS and kmod style kernel packages if possible. The user packages can be used with either set of kernel packages and do not need to be rebuilt when the kernel is updated. You can also streamline the build process by building only the DKMS or kmod packages as shown below.

Be aware that when building directly from a git repository you must first run the *autogen.sh* script to create the *configure* script. This will require installing the GNU autotools packages for your distribution.  In addition you must install all the necessary development tools and headers for your distribution.

For Debian and Ubuntu:

```
$ sudo apt-get install build-essential gawk alien fakeroot gdebi linux-headers-$(uname -r)
$ sudo apt-get install zlib1g-dev uuid-dev libblkid-dev libselinux-dev libudev-dev parted lsscsi wget ksh
```

## DKMS

The build system does not support building DKMS deb packages.

## kmod

The key thing to know when building a kmod package is that a specific Linux kernel must be specified. At configure time the build system will make an educated guess as to which kernel you want to build against. However, if configure is unable to locate your kernel development headers, or you want to build against a different kernel, you must specify the exact path with the *--with-linux* and *--with-linux-obj* options.

If you want to use the official released tarballs, use the following commands:

```
$ wget http://archive.zfsonlinux.org/downloads/zfsonlinux/spl/spl-x.y.z.tar.gz
$ wget http://archive.zfsonlinux.org/downloads/zfsonlinux/zfs/zfs-x.y.z.tar.gz

$ tar -xzf spl-x.y.z.tar.gz
$ tar -xzf zfs-x.y.z.tar.gz
```
	
If instead you would like to use the git version, you can clone it like this from github.

```
$ git clone https://github.com/zfsonlinux/spl.git
$ git clone https://github.com/zfsonlinux/zfs.git

$ cd spl
$ ./autogen.sh
$ cd ../zfs
$ ./autogen.sh
$ cd ..
```

After you have the source you'll need to configure, compile, and install the packages.  These steps are the same regardless of if you downloaded a tarball or the git repository.  Start off with the spl directory.

```
$ cd spl
$ ./configure
$ make pkg

#  Install the spl packages, they are required to build zfs.
$ for file in *.deb; do sudo gdebi -q --non-interactive $file; done
```

Then move on to the zfs directory.

```
$ cd ../zfs
$ ./configure
$ make pkg

#  Install the zfs packages.
$ for file in *.deb; do sudo gdebi -q --non-interactive $file; done
```

## kABI-tracking kmod

The build system does not support building kABI-tracking deb packages.

[release]: https://github.com/zfsonlinux/zfs/release
[git]: https://github.com/zfsonlinux/zfs