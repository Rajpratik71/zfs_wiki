Commit exceptions used to explicitly reference a given Linux commit.
These exceptions are useful for a variety of reasons.

**This page is used to generate [OpenZFS Tracking](http://build.zfsonlinux.org/openzfs-tracking.html) page.**

####Format:
- `<openzfs issue>|<->|<comment>` - 
The OpenZFS commit isn't applicable to Linux, or 
the OpenZFS -> ZFS on Linux commit matching is unable to associate
the related commits due to lack of information (denoted by a -).
- `<openzfs issue>|<commit>|<comment>` - 
The fix was merged to Linux prior to their being an OpenZFS issue.
- `<openzfs issue>|!|<comment>` - 
The commit is applicable but not applied for the reason described in the comment.

OpenZFS issue id | status/ZFS commit | comment
---|---|---
7779|-      |The change isn't relevant, `zfs_ctldir.c` was rewritten for Linux.
7740|32d41fb|
7730|e24e62a|
7591|541a090|
7586|c443487|
7542|-      |The Linux libshare code differs significantly from the upstream OpenZFS code.  Since this change doesn't address a Linux specific issue it doesn't need to be ported.  The eventual plan is to retire all of the existing libshare code and use the ZED to more flexibly control filesystem sharing.
7430|68cbd56|
7402|690fe64|
7345|058ac9b|
7278|-      |Dynamic ARC tuning is handled slightly differently under Linux and this case is covered by arc_tuning_update()
7164|b1b85c87|
7041|33c0819|
7016|d3c2ae1|
6914|-      |Under Linux the arc_meta_limit can be tuned with the zfs_arc_meta_limit_percent module option.
6843|f5f087e|
6841|4254acb|
6781|15313c5|
6765|-      |Under Linux POSIX ACLs are used and largely handled by the VFS.  This change can be ported to minimize code drift but are not required.  
6764|-      |Under Linux POSIX ACLs are used and largely handled by the VFS.  This change can be ported to minimize code drift but are not required.  
6763|-      |Under Linux POSIX ACLs are used and largely handled by the VFS.  This change can be ported to minimize code drift but are not required.  
6762|-      |Under Linux POSIX ACLs are used and largely handled by the VFS.  This change can be ported to minimize code drift but are not required.  
6494|-      |The `vdev_disk.c` and `vdev_file.c` files have been reworked extensively for Linux.  The proposed changes are not needed.
6434|472e7c6|
6421|ca0bf58|
6418|131cc95|
6391|ee06391|
6390|85802aa|
6388|0de7c55|
6386|485c581|
6385|f3ad9cd|
6368|2024041|
6346|058ac9b|
6334|1a04bab|
6290|017da6 |
6250|-      |Linux handles crash dumps in a fundamentally different way than Illumos.  The proposed changes are not needed.
6220|-      |The b_thawed debug code was unused under Linux and removed.
6209|-      |The Linux user space mutex implementation is based on phtread primitives. 
6095|f866a4ea|
5984|480f626|
5961|22872ff|
5815|-      |This patch could be adapted if needed use equivalent Linux functionality.
5770|c3275b5|
5769|dd26aa5|
5768|-      |The change isn't relevant, `zfs_ctldir.c` was rewritten for Linux.
5766|4dd1893|
5693|0f7d2a4|
5410|0bf8501|
5409|b23d543|
5379|-      |This particular issue never impacted Linux due to the need for a modified zfs_putpage() implementation.
5316|-      |The illumos idmap facility isn't available under Linux.  This patch could still be applied to minimize code delta or all HAVE_IDMAP chunks could be removed on Linux for better readability.
5313|ec8501e|
5219|ef56b07|
5179|3f4058c|
5149|-      |Equivalent Linux functionality is provided by the `zvol_max_discard_blocks` module option.
5148|-      |Discards are handled differently under Linux, there is no DKIOCFREE ioctl.
5136|e8b96c6|
5120|!      |This patch should eventually be adopted.  However, since we have no control over which version of grub is shipped by a distribution it's safest to defer until newer versions of grub are part of the default install. See #5688 ZoL PR.
4752|aa9af22|
4745|411bf20|
4698|4fcc437|
4573|10b7549|
4571|6e1b9d0|
4570|b1d13a6|
4391|78e2739|
4242|-      |Neither vnodes or their associated events exist under Linux.
4206|2820bc4|
4188|2e7b765|
4161|-      |The Linux user space reader/writer implementation is based on phtread primitives.
4128|!      |The ldi_ev_register_callbacks() interface doesn't exist under Linux.  It may be possible to receive similar notifications via the scsi error handlers or possibly a different interface.
4072|-      |None of the illumos build system is used under Linux.
3947|7f9d994|
3928|-      |Neither vnodes or their associated events exist under Linux.
3747|090ff09|
3705|-      |The Linux implementation uses the lz4 workspace kmem cache to resolve the stack issue.
3606|c5b247f|
3580|-      |Linux provides generic ioctl handlers get/set block device information.
3543|8dca0a9|
3512|67629d0|
3507|43a696e|
3301|-      |The Linux implementation of `vdev_disk.c` does not include this comment.
3258|9d81146|
3254|-      |The `aclmode` property cannot be supported under Linux.
3246|cc92e9d|
2933|-      |None of the illumos build system is used under Linux.
2932|!      |The feature has not been ported because Linux fundamentally handles crash dumps differently.  The commit will need to be significantly reworked for Linux.  The only benefit will be to ensure pools from other platforms with this feature set can be easily imported under Linux.
2897|fb82700|
2665|32a9872|
2130|460a021|
1974|-      |This change was entirely replaced in the ARC restructuring.
1898|-      |The zfs_putpage() function was rewritten to properly integrate with the Linux VM.
1618|ca67b33|
1337|2402458|
1126|e43b290|
763 |3cee226|
742 |-      |The `aclmode` property cannot be supported under Linux.
701 |460a021|
348 |-      |The Linux implementation of `vdev_disk.c` must have this differently.
243 |-      |Manual updates have been made separately for Linux.
184 |-      |The zfs_putpage() function was rewritten to properly integrate with the Linux VM.