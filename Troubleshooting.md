# DRAFT
This page contains tips for troubleshooting ZFS on Linux and what info developers might want for bug triage.
## About Log Files
Log files can be very useful for troubleshooting. In some cases, interesting information is stored in multiple log files that are correlated to system events.

Pro tip: logging infrastructure tools like _elasticsearch_, _fluentd_, _influxdb_, or _splunk_ can simplify log analysis and event correlation.

### Generic Kernel Log
Typically, Linux kernel log messages are available from `dmesg -T`, `/var/log/syslog`, or where kernel log messages are sent (eg by `rsyslogd`).

### ZFS Kernel Module Debug Messages 
The ZFS kernel modules use an internal log buffer for detailed logging information.
This log information is available in the pseudo file `/proc/spl/kstat/zfs/dbgmsg` for ZFS builds where ZFS module parameter [zfs_dbgmsg_enable = 1](https://github.com/zfsonlinux/zfs/wiki/ZFS-on-Linux-Module-Parameters#zfs_dbgmsg_enable)

## Unkillable Process
Symptom: `zfs` or `zpool` command appear hung, does not return, and is not killable

Likely cause: kernel thread hung or panic

Log files of interest: [Generic Kernel Log](#generic-kernel-log), [ZFS Kernel Module Debug Messages](#zfs-kernel-module-debug-messages)

Important information: if a kernel thread is stuck, then a backtrace of the stuck thread can be in the logs.

# DRAFT
