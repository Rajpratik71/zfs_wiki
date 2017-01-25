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
7740|32d41fb|
7591|541a090|
7586|c443487|
7430|68cbd56|
7402|690fe64|
7345|058ac9b|
7041|33c0819|
7016|d3c2ae1|
6843|f5f087e|
6841|4254acb|
6781|15313c5|
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
6250|-      |
6220|-      |
6209|-      |
6095|f866a4ea|
5984|480f626|
5961|-      |
5815|-      |
5770|c3275b5|
5769|dd26aa5|
5768|-      |
5766|4dd1893|
5704|-      |
5693|0f7d2a4|
5410|0bf8501|
5409|b23d543|
5316|-      |
5313|ec8501e|
5219|ef56b07|
5179|3f4058c|
5149|-      |
5148|-      |
5136|e8b96c6|
4752|aa9af22|
4745|411bf20|
4698|4fcc437|
4573|10b7549|
4571|6e1b9d0|
4570|b1d13a6|
4391|78e2739|
4242|-      |
4206|2820bc4|
4188|2e7b765|
4181|-      |
4161|-      |
4072|-      |
3947|7f9d994|
3928|-      |
3747|090ff09|
3705|-      |
3606|c5b247f|
3580|-      |
3543|8dca0a9|
3512|67629d0|
3507|43a696e|
3371|-      |
3301|-      |
3258|9d81146|
3254|-      |
3246|cc92e9d|
2933|-      |
2932|!      |The feature has not been ported because Linux fundamentally handles crash dumps differently.  The commit will need to be significantly reworked for Linux.  The only benefit will be to ensure pools from other platforms with this feature set can be easily imported under Linux.
2897|fb82700|
2665|32a9872|
2130|-      |
1974|-      |
1898|-      |
1618|-      |
1337|2402458|
1126|e43b290|
763 |-      |
742 |-      |
701 |-      |
348 |-      |
243 |-      |
184 |-      |