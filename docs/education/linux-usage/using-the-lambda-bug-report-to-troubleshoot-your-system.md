---
description: Learn how to use the Lambda bug report to troubleshoot your system.
tags:
  - troubleshooting
---

# Using the Lambda bug report to troubleshoot your system

The Lambda bug report helps simplify the process of troubleshooting by collecting system information for you into one place. This article helps you utilize the `lambda-bug-report.log` file to troubleshoot common issues.

!!! warning

    The `lambda-bug-report.sh` is intended for use on [Vector](https://lambdalabs.com/gpu-workstations/vector), [Scalar](https://lambdalabs.com/products/scalar), [Hyperplane](https://lambdalabs.com/deep-learning/servers/hyperplane), and [On-Demand](https://lambdalabs.com/service/gpu-cloud) products only. Do not run this script on a cluster as it installs packages that may cause unintended outcomes.


To generate a report, run:

```bash
wget -nv -O - https://raw.githubusercontent.com/lambdal-support/lambda-public-tools/main/lambda-bug-report.sh | bash -
```

This command creates a file named `lambda-bug-report.tar.gz` in your current directory that contains the Lambda bug report.

## Understanding the bug report log file

The log file is organized by system components, making it easier to locate
relevant information. It includes outputs from various commands, scripts, and
logs that you might otherwise have to collect manually.

While the log file is extensive, certain sections are more commonly referenced
for troubleshooting. Outlined below are the key directories and files you should
consider examining when troubleshooting.

### Baseboard Management Controller (BMC) information

The `bmc-info` folder collects sensor information and error history for systems with a BMC. This information is useful if you suspect hardware malfunctions or want to check the health status of your system components.

!!! note

    Only [Hyperplane](https://lambdalabs.com/deep-learning/servers/hyperplane), [Scalar](https://lambdalabs.com/products/scalar), [Vector](https://lambdalabs.com/gpu-workstations/vector), and [Vector Pro](https://lambdalabs.com/gpu-workstations/vector-pro) products have a BMC. The [Vector One](https://lambdalabs.com/gpu-workstations/vector-one) does not include a BMC, so these files, while present in the bug report run on a Vector One, do not contain Intelligent Platform Management Interface (IPMI) data.


Files:

`ipmi-elist.txt, ipmi-sdr.txt`

### Drives and storage

This section provides information about disk usage, mounted filesystems, RAID
configurations, and disk I/O statistics. Look here when experiencing issues
related to storage, such as disk failures, insufficient disk space, or mounting
errors.

Files:

`df.txt, fstab.txt, iostat.txt, lsblk.txt, mdadm-conf.txt, mdadm-scan.txt, mdstat.txt, mounts.txt`

### GPU memory errors

The following files are logs of GPU memory errors, including error-correcting
code (ECC) errors and remapped memory regions. They are relevant if you’re
encountering GPU-related issues like crashes during computation or suspect
faulty GPU memory.

Files:

`ecc-errors.txt, remapped-memory.txt, uncorrected-ecc_errors.txt`

### Grub

The `grub` folder stores the GRUB bootloader configuration files and boot
command-line parameters. These files can be helpful when troubleshooting boot
issues or modifying boot parameters for kernel debugging.

Files:

`grub.d/50-cloudimg-settings.cfg, grub.d/init-select.cfg, grub.txt, proc_cmdline.txt`

### Hardware list

This section lists all recognized hardware devices on the system. It’s useful to
verify if all hardware components are detected by the system or to identify
missing devices.

Files:

`hw-list.txt`

### Networking

You can find network configuration and status here, including IP addresses,
firewall settings, and active network connections. This section is helpful when
facing network connectivity issues, firewall problems, or to check network
configurations.

Files:

`ip-addr.txt, iptables.txt, netplan.txt, resolvectl-status.txt, ss.txt, ufw-status.txt`

### NVIDIA bug report and SMI

The `nvidia-bug-report.log` and `nvidia-smi.txt` files contain detailed
information about NVIDIA GPU drivers and hardware status. Use this information
for diagnosing GPU-related issues, driver problems, or performance bottlenecks
involving NVIDIA GPUs. For tips on how to best use the NVIDIA bug report file,
see [Using the NVIDIA bug report to troubleshoot your
system](using-the-nvidia-bug-report.log-file-to-troubleshoot-your-system.md).

Files:

`nvidia-bug-report.log, nvidia-smi.txt`

### Repositories and packages

This section lists installed packages, their sources, and repository
configurations. Use these files to identify software conflicts, check package
versions, or verify repository settings.

Files:

`dpkg.txt, listd-repos.txt, pip-list.txt, sources-list.txt`

### Sensors

The `sensors.txt` file contains internal thermal sensor readings. This
information is helpful when investigating overheating issues or thermal
throttling.

Files:

`sensors.txt`

### System logs

This section aggregates various system logs that record events and errors from
different system components. It provides a broad overview of system events,
kernel messages, package installation history, and error tracking.

Files:

`apt-history.log, dmesg, dmesg-errors.txt, dpkg.log, journalctl.txt, kern.log, syslog`

### Systemctl services

The `systemctl-services.txt` file contains the status of `systemd` services.
Check this section to verify essential services are either running or have
failed.

Files:

`systemctl-services.txt`

### Top

Top captures a snapshot of system processes and resource usage at the time the
bug report was generated. This section helps identify processes consuming
excessive CPU or memory resources that may lead to performance degradation.

Files:

`top.txt`

## Bug report folder hierarchy

The files in the Lambda bug report are organized into the following folders:

```
├── bmc-info
    ├── ipmi-elist.txt
    └── ipmi-sdr.txt
├── drives-and-storage
    ├── df.txt
    ├── fstab.txt
    ├── iostat.txt
    ├── lsblk.txt
    ├── mdadm-conf.txt
    ├── mdadm-scan.txt
    ├── mdstat.txt
    └── mounts.txt
├── gpu-memory-errors
    ├── ecc-errors.txt
    ├── remapped-memory.txt
    └── uncorrected-ecc_errors.txt
├── grub
    ├── grub.d
    │   ├── 50-cloudimg-settings.cfg
    │   └── init-select.cfg
    ├── grub.txt
    └── proc_cmdline.txt
├── hibernation-settings.txt
├── hw-list.txt
├── ibstat.txt
├── lsmod.txt
├── networking
    ├── ip-addr.txt
    ├── iptables.txt
    ├── netplan.txt
    ├── resolvectl-status.txt
    ├── ss.txt
    └── ufw-status.txt
├── nvidia-bug-report.log
├── nvidia-smi.txt
├── repos-and-packages
    ├── dpkg.txt
    ├── listd-repos.txt
    ├── pip-list.txt
    └── sources-list.txt
├── sensors.txt
├── sysctl-all.txt
├── system-logs
    ├── apt-history.log
    ├── dmesg
    ├── dmesg-errors.txt
    ├── dpkg.log
    ├── journalctl.txt
    ├── kern.log
    └── syslog
├── systemctl-services.txt
└── top.txt
```

## Contact Lambda

If you can’t discover the cause for the issue you are experiencing, [contact
Lambda Support](https://support.lambdalabs.com/hc/en-us/requests/new) and
provide us with your [Lambda bug
report](https://docs.lambdalabs.com/software/troubleshooting-and-debugging#generate-a-lambda-bug-report).

## Other resources

* [Using the NVIDIA bug report log file to troubleshoot your system](https://docs.lambdalabs.com/software/using-the-nvidia-bug-report.log-file-to-troubleshoot-your-system)
* [Troubleshooting and debugging](troubleshooting-and-debugging.md)
