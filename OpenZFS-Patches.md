# OpenZFS Patches

The ZFS on Linux project is an adaptation of the upstream [OpenZFS repository][openzfs-repo] designed to work in a Linux environment.  This upstream repository acts as a location where new features, bug fixes, and performance improvements from all the OpenZFS platforms can be integrated.  Each platform is responsible for tracking the OpenZFS repository and merging the relevant improvements back in to their release.

For the ZFS on Linux project this tracking is managed through an [[OpenZFS tracking]] page.  The page is updated regularly and shows a list of OpenZFS commits and their status in regard to the ZFS on Linux master branch.  The status of each commit is color coded with a red circle indicating that it needs to be ported to ZFS on Linux.

This page describes the process of applying outstanding OpenZFS commits to ZFS on Linux and submitting those changes for inclusion.  As a developer this is a great way to familiarize yourself with ZFS on Linux and to begin quickly making a valuable contribution to the project.  The following guide assumes you have a [github account][github-account], are familiar with git, and are used to developing in a Linux environment.

## Porting OpenZFS changes to ZFS on Linux

### Setup the Environment

**Clone the source.** Start by making a local clone of the [spl][spl-repo] and [zfs][zfs-repo] repositories.

```
$ git clone -o zfsonlinux https://github.com/zfsonlinux/spl.git
$ git clone -o zfsonlinux https://github.com/zfsonlinux/zfs.git
```

**Add remote repositories.** Using the GitHub web interface [fork][github-fork] the [zfs][zfs-repo] repository in to your personal GitHub account.  Add your new zfs fork and the [openzfs][openzfs-repo] repository as remotes and then fetch both repositories.  The OpenZFS repository is large and the initial fetch may take some time over a slow connection.

```
$ cd zfs 
$ git remote add your-github-account git@github.com:your-github-account/zfs.git
$ git remote add openzfs https://github.com/openzfs/openzfs.git
$ git fetch --all
```

**Build the source.** Compile the spl and zfs master branches.  These branches are always kept stable and this is a useful verification that you have a full build environment installed and all the required dependencies are available.  This may also speed up the compile time latter for small patches where incremental builds are an option.

```
$ cd ../spl
$ sh autogen.sh && ./configure --enable-debug && make -s -j$(nproc)
$
$ cd ../zfs
$ sh autogen.sh && ./configure --enable-debug && make -s -j$(nproc)
```

### Porting a Patch

**Pick a patch.**  Consult the [[OpenZFS tracking]] page and select a patch which has not yet been applied.  For your first patch you will want to select a small patch to familiarize yourself with the process.  For the purposes of this example [OpenZFS 5669][openzfs-5669] is used.

**Create a new branch.**  It is important to create a new branch for every commit you port to ZFS on Linux.  This will allow you to easily submit your work as a GitHub pull request and it makes it possible to work on multiple OpenZFS changes concurrently.  All development branches need to be based off of the zfs master branch and it's helpful to name the branches after the issue you're working on.

```
$ git checkout -b openzfs-5669 master
```

**Generate a patch.**  One of the first things you'll notice about the ZFS on Linux repository is that it is laid out differently than the OpenZFS repository.  Organizationally it is much flatter, this is possible because it only contains the code for OpenZFS not an entire OS.  That means that in order to apply a patch from OpenZFS the path names in the patch must be changed.  A script called zfs2zol-patch.sed has been provided to perform this translation.  Use the `git format-patch` command and this script to generate a patch.

```
$ git format-patch --stdout c423721^..c423721 | ./scripts/zfs2zol-patch.sed >openzfs-5669.diff
```

**Apply the patch.**  In many cases the generated patch will apply cleanly to the repository.  However, it's important to keep in mind the zfs2zol-patch.sed script only translates the paths.  There are often additional reasons why a patch might not apply.  In some cases hunks of the patch may not be applicable to Linux and should be dropped.  In other cases a patch may depend on other changes which must be applied first.  The changes may also conflict with Linux specific modifications.  In all of these cases the patch will need to be manually modified to apply cleanly while preserving the its original intent.

```
$ git am ./openzfs-5669.diff
```

**Update the commit message.** By using `git format-patch` to generate the patch and then `git am` to apply it the original comment and authorship will be preserved.  Due to the formatting of the OpenZFS commit you may find that the entire commit comment has been squashed in to the subject line.  Use `git commit --amend` to fix the commit and update it to match the form used for all changes applied from OpenZFS.  The following information should be included:

  * A short subject line of the form: "OpenZFS issue - short description".
  * The original patch authorship should be preserved.
  * The OpenZFS commit message.
  * The following tags:
    * **Reviewed by:** All OpenZFS reviewers from the original patch.
    * **Approved by:** All OpenZFS reviewers from the original patch.
    * **Ported-by:** Your name and email address.
    * **OpenZFS-issue:** https ://www.illumos.org/issues/issue
    * **OpenZFS-commit:** https ://github.com/openzfs/openzfs/commit/hash
  * **Porting Notes:** An optional section describing any changes required when porting.

For example:

```
Author: Xin Li <delphij@freebsd.org>
Date:   Wed May 27 16:10:16 2015 +0200

    OpenZFS 5669 - altroot not set in zpool create
    
    5669 altroot not set in zpool create when specified with -o
    Reviewed by: Matthew Ahrens <mahrens@delphix.com>
    Reviewed by: George Wilson <george@delphix.com>
    Approved by: Dan McDonald <danmcd@omniti.com>
    Ported-by: Brian Behlendorf <behlendorf1@llnl.gov>
    
    OpenZFS-issue: https://www.illumos.org/issues/5669
    OpenZFS-commit: https://github.com/openzfs/openzfs/commit/c423721   
```

### Testing a Patch

**Build the source.**  Verify the patched source compiles without errors and all warnings are resolved.

```
$ make -s -j$(nproc)
```

**Run the style checker.**  Verify the patched source passes the style checker, the command should return without printing any output.

```
$ make cstyle
```

**Open a Pull Request.**  When your patch builds cleanly and passes the style checks [open a new pull request][github-pr].  The pull request will be queued for [automated testing][buildbot].  As part of the testing the change is built for a wide range of Linux distributions and a battery of functional and stress tests are run to detect regressions.

```
$ git push your-github-account openzfs-5669
```

**Fix any issues.**  Testing takes approximately 2 hours to fully complete and the results are posted in the GitHub [pull request][openzfs-pr].  All the tests are expected to pass and you should investigate and resolve any test failures.  The [test scripts][buildbot-scripts] are all available and designed to run locally in order reproduce an issue.  Once you've resolved the issue force update the pull request to trigger a new round of testing.  Iterate until all the tests are passing.

```
# Fix issue, amend commit, force update branch.
$ git commit --amend
$ git push --force your-github-account openzfs-5669
```

### Merging the Patch

**Review.**  Lastly one of the ZFS on Linux maintainers will make a final review of the patch and may request additional changes.  Once the maintainer is happy with the final version of the patch they will add their signed-off-by, merge it to the master branch, mark it complete on the tracking page, and thank you for your contribution to the project!

## Porting ZFS on Linux changes to OpenZFS

Often an issue will be first fixed in ZFS on Linux or a new feature developed.  Changes which are not Linux specific should be submitted upstream to the OpenZFS GitHub repository for review.  The process for this is described in the [OpenZFS README][openzfs-repo].

[github-account]: https://help.github.com/articles/signing-up-for-a-new-github-account/
[github-pr]: https://help.github.com/articles/creating-a-pull-request/
[github-fork]: https://help.github.com/articles/fork-a-repo/
[buildbot]: https://github.com/zfsonlinux/zfs-buildbot/
[buildbot-scripts]: https://github.com/zfsonlinux/zfs-buildbot/tree/master/scripts
[spl-repo]: https://github.com/zfsonlinux/spl
[zfs-repo]: https://github.com/zfsonlinux/zfs
[openzfs-repo]: https://github.com/openzfs/openzfs/
[openzfs-5669]: https://github.com/openzfs/openzfs/commit/c423721
[openzfs-pr]: https://github.com/zfsonlinux/zfs/pull/4594