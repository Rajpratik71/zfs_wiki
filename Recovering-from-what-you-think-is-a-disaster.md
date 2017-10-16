I'm going to assume you got to this point because your in trouble...  don't panic, you can probably recover!

ZFS is not supposed to be corruptible, but bugs in code can cause corruption (for example issue 6414  https://github.com/zfsonlinux/zfs/issues/6414).  What is amazing is the fundamental design of ZFS (COW) means you have a very very good chance of recover, however the tools to assist you are lacking.  The tools that are around are also very very scary. 

To start the process, I'd recommend
* make sure zfs is unmounted and not imported
* you have root access to the system
* you have the ability to copy a large amount of data to some other form of storage (USB? network file system?)

The first part of the process is making sure you don't write any data to the disks that you can't back out of.  Take raw dd's of the drives

    dd if=/dev/sda bs=1024K | pv > /path/to/storage

this means at the very least, you have a copy of each drive, which you should be able to use in the worst case scenario.

Next, setup a writable snapshot of each disk using dm.  This allows us to "write" to the devices without affecting the data on the physical drive (writes go to a loop-back mounted device).

    dd if=/dev/zero bs=1024K count=10000 of=/path/to/tempfile-sda
    losetup -f /path/to/tempfile-sda
    echo 0 $(blockdev --getsz /dev/sda) snapshot /dev/sda /dev/loop0 p 4 | dmseup create snapsda
  
after this, you have a writable /dev/mapper/snapsda where writes go to /path/to/tempfile-sda rather than to the physical disk.

Now, you can try to import the pool and fix your file system... more to come.