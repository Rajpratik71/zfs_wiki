# Controlling the ZFS Buildbot

There are a number of ways to control the ZFS Buildbot at a commit level.  This page will
provide a summary on the methods to control the ZFS Buildbot and how it performs testing.
More detailed information regarding its implementation can be found at the
[ZFS Buildbot Github page](https://github.com/zfsonlinux/zfs-buildbot).

## Choosing Builders

By default, all commits in your ZFS pull request are compiled by the BUILD
builders.  Additionally, the top commit of your ZFS pull request is tested by
TEST builders.  However, there is the option to override which types of builder
should be used on a per commit basis.  In this case, you can add
`Requires-builders: <none|all|style|build|arch|distro|test|perf>` to your
commit message.  A comma separated list of options can be
provided.  Supported options are:

* `all`: This commit should be built by all available builders
* `none`: This commit should not be built by any builders
* `style`: This commit should be built by STYLE builders
* `build`: This commit should be built by all BUILD builders
* `arch`: This commit should be built by BUILD builders tagged as 'Architectures'
* `distro`: This commit should be built by BUILD builders tagged as 'Distributions'
* `test`: This commit should be built and tested by the TEST builders
* `perf`: This commit should be built and tested by the PERF builders

A couple of examples on how to use `Requires-builders:` in commit messages can be found below.

### Preventing a commit from being built and tested.
```
This is a commit message

This text is part of the commit message body.

Signed-off-by: Contributor <contributor@email.com>
Requires-builders: none
```

### Submitting a commit to STYLE and TEST builders only.
```
This is a commit message

This text is part of the commit message body.

Signed-off-by: Contributor <contributor@email.com>
Requires-builders: style test
```

## Controlling Tests with the TEST File
The ZFS Buildbot also has the ability to control which tests are executed by TEST
builders.  At the top level of the ZFS source tree, there is the [`TEST`
file](https://github.com/zfsonlinux/zfs/blob/master/TEST). Within this file,
various variables can be defined to control if and how a specific test should
run. Below is a list of variables and a brief description of what each variable
controls.

* `TEST_PREPARE_WATCHDOG` -
* `TEST_SPLAT_SKIP` -
* `TEST_SPLAT_OPTIONS` -
* `TEST_ZTEST_SKIP` -
* `TEST_ZTEST_TIMEOUT` -
* `TEST_ZTEST_DIR` -
* `TEST_ZTEST_OPTIONS` -
* `TEST_ZTEST_OPTIONS` -
* `TEST_ZTEST_OPTIONS` -
* `TEST_ZTEST_OPTIONS` -
* `TEST_ZTEST_OPTIONS` -
* `TEST_ZTEST_OPTIONS` -
* `TEST_ZTEST_OPTIONS` -
* `TEST_ZTEST_OPTIONS` -


### zimport
#TEST_ZIMPORT_SKIP="yes"
#TEST_ZIMPORT_DIR="/var/tmp/zimport"
#TEST_ZIMPORT_VERSIONS="master installed"
#TEST_ZIMPORT_POOLS="zol-0.6.1 zol-0.6.2 master installed"
#TEST_ZIMPORT_OPTIONS="-c"

### xfstests
#TEST_XFSTESTS_SKIP="yes"
#TEST_XFSTESTS_URL="https://github.com/behlendorf/xfstests/archive/"
#TEST_XFSTESTS_VER="zfs.tar.gz"
#TEST_XFSTESTS_POOL="tank"
#TEST_XFSTESTS_FS="xfstests"
#TEST_XFSTESTS_VDEV="/var/tmp/vdev"
#TEST_XFSTESTS_OPTIONS=""

### zfs-tests.sh
#TEST_ZFSTESTS_SKIP="yes"
#TEST_ZFSTESTS_DISKS="vdb vdc vdd"
#TEST_ZFSTESTS_DISKSIZE="8G"
#TEST_ZFSTESTS_RUNFILE="linux.run"

### zfsstress
#TEST_ZFSSTRESS_SKIP="yes"
#TEST_ZFSSTRESS_URL="https://github.com/nedbass/zfsstress/archive/"
#TEST_ZFSSTRESS_VER="master.tar.gz"
#TEST_ZFSSTRESS_RUNTIME=300
#TEST_ZFSSTRESS_POOL="tank"
#TEST_ZFSSTRESS_FS="fish"
#TEST_ZFSSTRESS_VDEV="/var/tmp/vdev"
#TEST_ZFSSTRESS_DIR="/$TEST_ZFSSTRESS_POOL/$TEST_ZFSSTRESS_FS"
#TEST_ZFSSTRESS_OPTIONS=""