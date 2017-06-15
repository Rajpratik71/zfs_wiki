# Buildbot Options

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
At the top level of the ZFS source tree, there is the [`TEST`
file](https://github.com/zfsonlinux/zfs/blob/master/TEST) which contains variables
that control if and how a specific test should run. Below is a list of each variable
and a brief description of what each variable controls.

* `TEST_PREPARE_WATCHDOG` - Enables the Linux kernel watchdog
* `TEST_SPLAT_SKIP` - Determines if `splat` testing is skipped
* `TEST_SPLAT_OPTIONS` - Command line options to provide to `splat`
* `TEST_ZTEST_SKIP` - Determines if `ztest` testing is skipped
* `TEST_ZTEST_TIMEOUT` - The length of time `ztest` should run
* `TEST_ZTEST_DIR` - Directory where `ztest` will create vdevs
* `TEST_ZIMPORT_SKIP` - Determines if `zimport` testing is skipped
* `TEST_ZIMPORT_DIR` - Directory used during `zimport`
* `TEST_ZIMPORT_VERSIONS` - Source versions to test
* `TEST_ZIMPORT_POOLS` - Names of the pools for `zimport` to use for testing
* `TEST_ZIMPORT_OPTIONS` - Command line options to provide to `zimport`
* `TEST_XFSTESTS_SKIP` - Determines if `xfstest` testing is skipped
* `TEST_XFSTESTS_URL` - URL to download `xfstest` from
* `TEST_XFSTESTS_VER` - Name of the tarball to download from `TEST_XFSTESTS_URL`
* `TEST_XFSTESTS_POOL` - Name of pool to create and used by `xfstest`
* `TEST_XFSTESTS_FS` - Name of dataset for use by `xfstest`
* `TEST_XFSTESTS_VDEV` - Name of the vdev used by `xfstest`
* `TEST_XFSTESTS_OPTIONS` - Command line options to provide to `xfstest`
* `TEST_ZFSTESTS_SKIP` - Determines if `zfs-tests` testing is skipped
* `TEST_ZFSTESTS_DISKS` - Space delimited list of disks that `zfs-tests` is allowed to use
* `TEST_ZFSTESTS_DISKSIZE` - File size of file based vdevs used by `zfs-tests`
* `TEST_ZFSTESTS_RUNFILE` - The runfile to use when running `zfs-tests`
* `TEST_ZFSSTRESS_SKIP` - Determines if `zfsstress` testing is skipped
* `TEST_ZFSSTRESS_URL` - URL to download `zfsstress` from
* `TEST_ZFSSTRESS_VER` - Name of the tarball to download from `TEST_ZFSSTRESS_URL`
* `TEST_ZFSSTRESS_RUNTIME` - Duration to run `runstress.sh`
* `TEST_ZFSSTRESS_POOL` - Name of pool to create and use for `zfsstress` testing
* `TEST_ZFSSTRESS_FS` - Name of dataset for use during `zfsstress` tests
* `TEST_ZFSSTRESS_VDEV` - Directory to store vdevs for use during `zfsstress` tests
* `TEST_ZFSSTRESS_DIR` - Unused
* `TEST_ZFSSTRESS_OPTIONS` - Command line options to provide to `runstress.sh`