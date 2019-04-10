The [Ubuntu PPA for ZoL](https://launchpad.net/~zfs-native/+archive/stable) has an enhanced `/sbin/mountall` that recognizes ZFS and can mount ZFS at system start.

# 1.0  Installation

## 1.1 Installing the `mountall` package

Enable the PPA and install the `ubuntu-zfs` package.  Do not do anything else.

The `ubuntu-zfs` package installs a pin file that is required to keep the right `mountall` package installed.

## 1.2 Verifying the `mountall` package

Run `apt-cache policy mountall` to check that the PPA is the preferred archive for the `mountall` package.  The installed package should have "zfs" in the version string and there should be a line that begins with `900` in the output.

```
$ apt-cache policy mountall
mountall:
  Installed: 2.36-zfs1
  Candidate: 2.36-zfs1
  Version table:
 *** 2.36-zfs1 0
        900 http://ppa.launchpad.net/zfs-native/daily/ubuntu/ precise/main amd64 Packages
        100 /var/lib/dpkg/status
     2.36 0
        500 http://us.archive.ubuntu.com/ubuntu/ precise/main amd64 Packages
```

Also check that the `/sbin/mountall` binary has a `parse_zfs_list` function:

```
$ grep parse_zfs_list /sbin/mountall
Binary file /sbin/mountall matches
```

## 1.3 Intended `mountall` operation

How is ZoL supposed to work at system start?

### 1.3.1 Ubuntu 14.04 LTS (Trusty Tahr) and earlier.

1. All storage pool devices come online and appear in `/dev/disk/by-id`.
1. Upstart runs the `/etc/init/mountall.conf` part.
1. The `/sbin/mountall` utility calls the `/sbin/zfs list` command.
1. The `/sbin/zfs` utility inserts the `zfs.ko` kernel module.
1. The ZFS driver reads the `/etc/zfs/zpool.cache` file, which contains a list of disks in each zpool.  Any disk that is not ready during this step faults out of the pool.
1. The `/sbin/mountall` utility mounts all ZFS filesystems and all `/etc/fstab` entries during the same pass.
1. The `/sbin/mountall` utility issues the `local-filesystems` event for Upstart.

### 1.3.2 Ubuntu 14.10 (Utopic Unicorn) in testing.

1. Upstart runs the `/etc/init/mountall.conf` job, which is blocked by the `/etc/init/zpool-import.conf` task.
1. The `zpool-import` script reads the `/etc/zfs/zpool.cache` file through `/sbin/zdb`, waits for all pool member devices to appear, and invokes `zpool import`. This is a "non-automatic" import for the `zfs_autoimport_disable=1` module option that is likely to become the ZoL 0.6.4 default.
1. The `/sbin/mountall` utility calls `/sbin/zfs list` and mounts all ZFS filesystems and all `/etc/fstab` entries during the same pass.
1. The `/sbin/mountall` utility issues the `local-filesystems` event for Upstart.

# 2.0 Usage

## 2.1 Mount Points

By default, the `mountpoint` filesystem property determines mount points.  For example:

```
# zfs get mountpoint tank/media
NAME           PROPERTY    VALUE           SOURCE
tank/media     mountpoint  /tank/media     default

# zfs get mountpoint tank/home
NAME           PROPERTY    VALUE           SOURCE
tank/home      mountpoint  /home           local
```

Set the `mountpoint` property to change the mount point:

```
# zfs set mountpoint=/export/media tank/media
```

Changes happen at reboot if the mount point is busy.


## 2.2 /etc/fstab

Most systems should use the `mountpoint` filesystem property, but the `/etc/fstab` file can still be used as an alternative.  The `mountpoint` property must be set to `legacy` on each filesystem *before* it is added to the `/etc/fstab` file like this:

```
# zfs set mountpoint=legacy tank/media
```

The filesystem name in the `/etc/fstab` must not have a leading slash character:

```
tank/media  /export/media  zfs  defaults  0  0
```

# 3.0  Frequent Mistakes

* **Do not** put the ZFS driver in the initrd.
* **Do not** export pools at reboot or shutdown.
* ~~**Do not** set `ZFS_MOUNT=y` in the `/etc/default/zfs` file.~~ (This option is deprecated. The `/etc/init.d/zfs-mount` script will not be installed for new installations.)
* **Do not** install the `mountall` package on a vanilla Debian system.
* **Do not** put a `zfs` line in the `/etc/modules` file or otherwise load it earlier than upstart calls the `mountall` part.
* **Do not** delete the `/etc/hostid` file. It must exist.

And

* **Never** manually compile ZoL and run `make install` on a system that is using the PPA.  A local installation is incompatible with a managed installation.

Things should *just work* if the `mountall` package for ZoL is installed from the PPA.  Please send a bug report otherwise.

Most startup problems are caused by:

* Storage devices coming online after the `/etc/zfs/zpool.cache` file is checked.
* ZFS mounting after the `local-filesystems` upstart event is emitted.


# 4.0 Known Problems

## ~~4.1  `/tmp` mount point~~

_(Resolved by mountall-2.49 in Ubuntu 14.04, Trusty Tahr.)_

~~The `mountall` utility fails if a `mountpoint` property for a ZFS filesystem is the same as any mount point described in the `/lib/init/fstab` override file.  On a default system, `/tmp` is the only likely collision.~~

~~If `/tmp` must be a separate filesystem, then set `mountpoint=legacy` and put a line for it in the `/etc/fstab` file.  This bug does not happen if `/tmp` is in a ZFS root filesystem.~~

~~Changing other mount points described in the `/lib/init/fstab` file are likely to cause general system failures, even on non-ZoL systems.~~

## ~~4.2  Read-only Mounts~~

_(Resolved by mountall-2.49 in Ubuntu 14.04, Trusty Tahr.)_

~~The "ro" mount flag is not always honored.~~

## 4.3  JBOD mode on hardware RAID implementations

_The latest Ubuntu packages for Ubuntu 14.10 should fix these problems. Please try them if anything here affects you._

Some HBAs bring JBOD disks online asynchronously at power-on. Pool import fails if JBOD disks appear after the ZFS driver loads and checks the `/etc/zfs/zpool.cache` for the list of zpool members. The failure can be sporadic and happens most frequently on computers that use the mpt2sas driver.

Try these fixes in order:

1. Upgrade the HBA firmware.
1. Disable JBOD and export each disk as a single-element RAID-0 or RAID-1 volume.
1. Put a `sleep` command in the `/etc/init/mountall.conf` file like this:

```
...
sleep 60
exec mountall --daemon $force_fsck $fsck_fix
...
Please note that the 
exec mountall --daemon $force_fsck $fsck_fix 
line should be present in the unmodified /etc/init/mountall.conf file

This means "wait sixty seconds before running mountall".  Tune the number up or down according to how much time it takes for the HBA to bring JBOD disks online.

If a `sleep` command of sufficient length works, waiting for udev to create the devices may be a cleaner fix.  Save the following script as e.g. `/root/devicewait`:

####################################################################################
#!/bin/sh

waitavail() {
        date +"%D %T Checking devices..."
        count=0
        while [ true ]; do
               if [ -e ${diskdevdir}xxx1-part1 ] && [ -e ${xxx1-part9 ] && [ -e ${diskdevdir}xxx2-part1 ] && [ -e ${diskdevdir}xxx2-part9 ] && [ -e ${diskdevdir}xxx3-part1 ] && [ -e ${diskdevdir}xxx3-part9 ] && [ -e ${diskdevdir}xxx4-part1 ] && [ -e ${diskdevdir}xxx4-part9 ]
                then break;
                echo "loop"
                fi
                for file in /sys/block/sd* /sys/block/sd*/sd*; do udevadm test $file; done >/dev/null 2>&1
                printf "\r%s\r" "$(date +'%D %T Waiting on devices...')"
                sleep 1
                count=$((count+1))
                [ "$count" -gt 60 ] && break
        done
        if [ "$count" -gt 60 ]; then
                date +"%D %T Gave up checking devices."
        else
                date +"%D %T Done checking devices."
        fi
}

failsafedbus() {
        sleep 5
        /sbin/start dbus
        /sbin/start tty2
}

waitavail >/dev/tty1 2>&1
failsafedbus &
exit 0
############################################################################################
```
Make sure to `chmod +x /root/devicewait`.
Then modify `/etc/init/mountall.conf` like this:
```
...
/root/devicewait # instead of fixed sleep 60

exec mountall --daemon $force_fsck $fsck_fix
...
```
This script causes the start of mountall to be delayed until the expected scsi-* devices have been created by udev (indicating the hardware is online).  (Modify the script as necessary if you have differing device names).

So ultimately please notice that in the above /root/devicewait script, one MUST modify the values in the if statement of /root/devicewait as:
change xxx1 to your 1st ZFS drive's ID
change xxx2 to your 2nd ZFS drive's ID
change xxx3 to your 3rd ZFS drive's ID
change xxx3 to your 4th ZFS drive's ID
.......
change xxxn to your nth ZFS drive's ID

and so on for all your ZFS drives in the pool you want to automount on boot. You must include ALL the drive ID filenames associated with ALL the drives in your ZFS arrays that you want to mount on boot. Note that in this example, I am including both partitions of each disk ID in the if statement. Since I had two partitions (part1 and part9) per ZFS disk this gave a total of eight disk IDs in the if statement of /root/deviceswait, for my four ZFS drives. Now this might be redundant and one might be able to just specify the disk IDs without partitions.
In any case, the if statement of /root/deviceswait MUST test for the existence of ALL drives' IDs associated with the ZFS pool one would like to mount on boot. Note that one will have to modify the if statement of /root/deviceswait whenever disks are changed.

While I haven't tried ZIL drives, I also suggest you include them in the if statement too.
The drives' IDs are obtained by looking in your /dev/disk/by-id directory when your system is booted. You may wish to use disk utilities or other tools to positively obtain the disk IDs of the disks in your ZFS array(s). This is important if you want the script to reliably protect you from mis-mounting your ZFS arrays on boot.

In case of a REAL disk failure, one should note that the if statement will fail to break the loop and the loop will continue for 60 tries (one can set this in the [ "$count" -gt 60 ] && break in /root/devicewait).
Therefore, a true disk failure should be expected to result in a significant delay on boot when running the above (/root/devicewait).

## 4.4  eSATA, USB, Firewire, Thunderbolt, etc

ZoL does not automatically import pools on storage devices that are configured through hotplug events, particularly USB drives.  All storage devices must be "fixed disks" or otherwise fully configured during system start before the ZFS driver is loaded.

## 4.5  VMware

* Use virtio instead of an emulated HBA.  Set this option for each scsi adapter in the VMX configuration file or at the vSphere configuration panel:

```
scsi0.virtualDev = "pvscsi"
```

This option is compatible with VMware Workstation and VMware Player, even though these editions do not expose it in the configuration panel.  Note that pvscsi is generally faster than the buslogic or lsilogic emulations.

* If the `/dev/disk/by-id` aliases are missing in a VMware guest, then set this option in the VMX configuration file or at the vSphere configuration panel:

```
disk.EnableUUID = "TRUE"
```

Afterwards, export and import all pools to refresh the `/etc/zfs/zpool.cache` file.

## 4.6  LUKS

Add encrypted LUKS devices to the `/etc/initramfs-tools/conf.d/cryptroot` configuration such that all pool members are unlocked and ready before the regular system starts.

An alternative to running ZFS on LUKS is to run eCryptFS on ZFS.


# 5.0  Troubleshooting

First, refresh the `/etc/zfs/zpool.cache` file:

* Export all pools.
* Delete the `/etc/zfs/zpool.cache`. file.
* Import all pools.

Second, double-check that:

* All mount points are empty in the underlying filesystem. This requirement is inherited from Solaris.
* If the system has selinux or sudo customizations, then double check that the `/sbin/mountall` and `/sbin/zfs` commands are always invoked with full root privileges.
* All filesystems in the `/etc/fstab` file have `mountpoint=legacy` set.

Third, run `mountall --verbose` as the root user.  If this command succeeds, but automatic mounting fails during system start, then check for incompatibilities described in section 4.3 or 4.4 of this document.

Also run `mountall --debug` as the root user.


# 6.0  Bug Reporting

Please include these things with a bug report:

* The whole `mountall --verbose` transcript.
* The unmodified `/var/log/kern.log` and `/etc/fstab` files.
* The `zpool status` and `zfs get all` for affected filesystems.
* The full raw output of the `dmesg` command.