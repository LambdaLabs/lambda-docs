# Using the nvidia-bug-report.log file to troubleshoot your system

NVIDIA provides a script that generates a log file that you can use to
troubleshoot issues with NVIDIA GPUs. This log file has comprehensive
information about your system, including information about individual devices,
configuration of NVIDIA drivers, system journals, and more.

## Generate the log file

To generate the log file, log in as the root user or use `sudo`, then run the
following command:

```bash
sudo nvidia-bug-report.sh
```

This script generates a zipped file called nvidia-bug-report.log.gz in the
current directory. To verify that the script ran successfully, run `ls -la
nvidia*` and look for a row similar to the following:

```bash
-rw-r--r-- 1 root   root   207757 Aug 28 20:11 nvidia-bug-report.log.gz
```

After you generate the log archive file, you can expand it and open the log in a text editor.

## Troubleshoot the log file

The log file is comprehensive, as it collects information from various sources.
The following are suggestions on where to start looking, depending on the issue
you are seeing. You might see output in the log file from the same or a related
Linux command you run on your system, or both.

### Use the check-nvidia-bug-report shell script

To make the NVIDIA log report easier to use, Lambda provides a shell script,
`check-nvidia-bug-report.sh`, that parses and summarizes the report. This script
scans the report for:

* Xid errors
* Thermal slowdown messages
* Segfaults
* CPU throttling and bad CPU errors
* Hardware errors
* “Fallen off the bus” errors
* RmInit failures

You can find this script in the [lambda-public-tools GitHub
repository](https://github.com/lambdal-support/lambda-public-tools). To use the
script, first clone the repository:

```bash
git clone https://github.com/lambdal-support/lambda-public-tools.git
```

After you generate the NVIDIA log file, run the Lambda script. The following
example assumes the Lambda script and the NVIDIA log file are in the same
directory:

```bash
cd lambda-public-tools/check-nvidia-bug-report
./check-nvidia-bug-report.sh
```

### Verify hardware with dmidecode

If you prefer to investigate the log file on your own, a good place to start is
to check that all the hardware reported by the BIOS is installed, available, and
seen by the system. Use `dmidecode`, or search for _dmidecode_ in the log file.

```bash
/sbin/dmidecode
```

You should see output similar to the following:

```bash

# dmidecode 3.3
Getting SMBIOS data from sysfs.
SMBIOS 2.8 present.
53 structures occupying 3036 bytes.
Table at 0x7FFFF420.

Handle 0x0000, DMI type 0, 24 bytes
BIOS Information
Vendor: SeaBIOS
Version: 1.13.0-1ubuntu1.1
Release Date: 04/01/2014
Address: 0xE8000
Runtime Size: 96 kB
ROM Size: 64 kB
Characteristics:
BIOS characteristics not supported
Targeted content distribution is supported
BIOS Revision: 0.0

Handle 0x0100, DMI type 1, 27 bytes
System Information
Manufacturer: QEMU
Product Name: Standard PC (Q35 + ICH9, 2009)
Version: pc-q35-8.0
Serial Number: Not Specified
UUID: 55c8d550-3188-4747-9711-89067cca4646
Wake-up Type: Power Switch
SKU Number: Not Specified
Family: Not Specified

Handle 0x0300, DMI type 3, 22 bytes
Chassis Information
Manufacturer: QEMU
Type: Other
Lock: Not Present
Version: pc-q35-8.0
Serial Number: Not Specified
Asset Tag: Not Specified
Boot-up State: Safe
Power Supply State: Safe
Thermal State: Safe
Security Status: Unknown
OEM Information: 0x00000000
Height: Unspecified
Number Of Power Cords: Unspecified
Contained Elements: 0
SKU Number: Not Specified

Handle 0x0400, DMI type 4, 42 bytes
Processor Information
Socket Designation: CPU 0
Type: Central Processor
Family: Other
Manufacturer: QEMU
ID: A6 06 06 00 FF FB 8B 0F
Version: pc-q35-8.0
Voltage: Unknown
External Clock: Unknown
Max Speed: 2000 MHz
Current Speed: 2000 MHz
Status: Populated, Enabled
Upgrade: Other
L1 Cache Handle: Not Provided
L2 Cache Handle: Not Provided
L3 Cache Handle: Not Provided
Serial Number: Not Specified
Asset Tag: Not Specified
Part Number: Not Specified
Core Count: 1
Core Enabled: 1
Thread Count: 1
Characteristics: None

Handle 0x0401, DMI type 4, 42 bytes
Processor Information
Socket Designation: CPU 1
Type: Central Processor
Family: Other
…
```

### Check for Xid or SXid errors

An Xid message is an NVIDIA error report that prints to the kernel log or event
log. Xid messages indicate that a general GPU error occurred, typically due to
the driver programming the GPU incorrectly or by corruption of the commands sent
to the GPU. The messages may indicate a hardware problem, an NVIDIA software
problem, or an application problem. To understand the Xid message, read the
[NVIDIA
documentation](https://docs.nvidia.com/deploy/xid-errors/index.html#xid-error-listing)
and review these [common Xid
errors](https://docs.nvidia.com/deploy/xid-errors/index.html#common-xid-errors).

NVIDIA drivers for NVSwitch report error conditions relating to NVSwitch
hardware in kernel logs through a mechanism similar to Xids. SXid (or switch
Xids) are errors relating to the NVIDIA switch hardware; they appear in kernel
logs similar to Xids. For more information about SXids, read appendixes D.4
through D.7 in the [NVIDIA
documentation](https://docs.nvidia.com/datacenter/tesla/pdf/fabric-manager-user-guide.pdf).

Search the log file for Xid or SXid and see what errors are associated with
them, or run `dmesg` as root:

```bash
sudo dmesg | grep SXid
```

You should see output like the following:

```bash
[   60.006338] kernel: nvidia-nvswitch3: SXid (PCI:0000:0a:00.0): 22013, Non-fatal, Link 55 Minion Link DLREQ interrupt
[   60.008040] kernel: nvidia-nvswitch3: SXid (PCI:0000:0a:00.0): 22013, Severity 0 Engine instance 55 Sub-engine instance 00
[   60.010058] kernel: nvidia-nvswitch3: SXid (PCI:0000:0a:00.0): 22013, Data {0x00000037, 0x00000037, 0x00000000, 0x00000037, 0x80005302, 0x00000000, 0x00000000, 0x00000000, 0x00000000}
[   60.013283] kernel: nvidia-nvswitch3: SXid (PCI:0000:0a:00.0): 22013, Non-fatal, Link 54 Minion Link DLREQ interrupt
[   60.015379] kernel: nvidia-nvswitch3: SXid (PCI:0000:0a:00.0): 22013, Severity 0 Engine instance 54 Sub-engine instance 00
[   60.017388] kernel: nvidia-nvswitch3: SXid (PCI:0000:0a:00.0): 22013, Data {0x00000036, 0x00000036, 0x00000000, 0x00000036, 0x80005302, 0x00000000, 0x00000000, 0x00000000, 0x00000000}
[   60.020455] kernel: nvidia-nvswitch3: SXid (PCI:0000:0a:00.0): 22013, Non-fatal, Link 51 Minion Link DLREQ interrupt
[   60.022557] kernel: nvidia-nvswitch3: SXid (PCI:0000:0a:00.0): 22013, Severity 0 Engine instance 51 Sub-engine instance 00
[   60.024563] kernel: nvidia-nvswitch3: SXid (PCI:0000:0a:00.0): 22013, Data {0x00000033, 0x00000033, 0x00000000, 0x00000033, 0x80005302, 0x00000000, 0x00000000, 0x00000000, 0x00000000}
[   60.027627] kernel: nvidia-nvswitch3: SXid (PCI:0000:0a:00.0): 22013, Non-fatal, Link 50 Minion Link DLREQ interrupt
[   60.029737] kernel: nvidia-nvswitch3: SXid (PCI:0000:0a:00.0): 22013, Severity 0 Engine instance 50 Sub-engine instance 00
[   60.031772] kernel: nvidia-nvswitch3: SXid (PCI:0000:0a:00.0): 22013, Data {0x00000032, 0x00000032, 0x00000000, 0x00000032, 0x80005302, 0x00000000, 0x00000000, 0x00000000, 0x00000000}
```

### Detect the GPU

To make sure your system can see the GPUs, run `nvidia-smi`:

```bash
nvidia-smi
```

The output below is for a single GPU system:

```bash
Wed Aug 28 20:01:47 2024
+---------------------------------------------------------------------------------------+
| NVIDIA-SMI 535.129.03             Driver Version: 535.129.03   CUDA Version: 12.2     |
|-----------------------------------------+----------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
|                                         |                      |               MIG M. |
|=========================================+======================+======================|
|   0  NVIDIA A10                     On  | 00000000:08:00.0 Off |                    0 |
|  0%   26C    P8               8W / 150W |      4MiB / 23028MiB |      0%      Default |
|                                         |                      |                  N/A |
+-----------------------------------------+----------------------+----------------------+

+---------------------------------------------------------------------------------------+
| Processes:                                                                            |
|  GPU   GI   CI        PID   Type   Process name                            GPU Memory |
|        ID   ID                                                             Usage      |
|=======================================================================================|
|  No running processes found                                                           |
+---------------------------------------------------------------------------------------+
```

### Check disk usage

You can use `df -h` to see disk usage on your nodes. Running low on disk space
could affect performance of your system.

```bash
df -h
```

The output should look something like the following:

```bash
Filesystem                            Size  Used Avail Use% Mounted on
tmpfs                                  23G  1.2M   23G   1% /run
/dev/vda1                             1.4T   25G  1.4T   2% /
tmpfs                                 112G     0  112G   0% /dev/shm
tmpfs                                 5.0M     0  5.0M   0% /run/lock
ae17b7ce-8c3b-464d-9f7c-325bc1080289  8.0E     0  8.0E   0% /home/ubuntu/lambda-pete-fs1
/dev/vda15                            105M  6.1M   99M   6% /boot/efi
```

### Check the systemd journal for errors

To check the `systemd` journal, either search for `journalctl` entries in the
log file or run the `journalctl` command. You should see output similar to the
following:

```bash
Aug 28 19:18:40 lambda-node jupyter[897]: [W 2024-08-28 19:18:40.342 ServerApp] ServerApp.token config is deprecated in 2.0. Use IdentityProvider.token.
Aug 28 19:18:40 lambda-node jupyter[897]: [W 2024-08-28 19:18:40.342 ServerApp] ServerApp.allow_password_change config is deprecated in 2.0. Use PasswordIde>
Aug 28 19:18:40 lambda-node jupyter[897]: [I 2024-08-28 19:18:40.349 ServerApp] Package jupyterlab took 0.0000s to import
Aug 28 19:18:40 lambda-node jupyter[897]: [I 2024-08-28 19:18:40.371 ServerApp] Package jupyter_collaboration took 0.0215s to import
Aug 28 19:18:40 lambda-node jupyter[897]: [I 2024-08-28 19:18:40.382 ServerApp] Package jupyter_lsp took 0.0103s to import
Aug 28 19:18:40 lambda-node jupyter[897]: [W 2024-08-28 19:18:40.382 ServerApp] A `_jupyter_server_extension_points` function was not found in jupyter_lsp. >
Aug 28 19:18:40 lambda-node jupyter[897]: [I 2024-08-28 19:18:40.382 ServerApp] Package jupyter_server_fileid took 0.0000s to import
Aug 28 19:18:40 lambda-node jupyter[897]: [I 2024-08-28 19:18:40.387 ServerApp] Package jupyter_server_terminals took 0.0047s to import
…
```

### Check Your NVLink topology

To confirm that your NVLink topology is correct, search the log file for
`nvidia-smi nvlink --status` output or run the following command:

```bash
nvidia-smi nvlink --status
```

This command checks the status of each NVLink connection for each GPU. The
output shows information about each NVLink, including the utilization and active
or inactive status. It’s similar to the following truncated output for an
eight-GPU system:

```bash
GPU 0: NVIDIA H100 80GB HBM3 (UUID: GPU-4f495419-b618-a303-a09b-8246fed175ac)
Link 0: 26.562 GB/s
Link 1: 26.562 GB/s
Link 2: 26.562 GB/s
Link 3: 26.562 GB/s
Link 4: 26.562 GB/s
Link 5: 26.562 GB/s
Link 6: 26.562 GB/s
Link 7: 26.562 GB/s
Link 8: 26.562 GB/s
Link 9: 26.562 GB/s
Link 10: 26.562 GB/s
Link 11: 26.562 GB/s
Link 12: 26.562 GB/s
Link 13: 26.562 GB/s
Link 14: 26.562 GB/s
Link 15: 26.562 GB/s
Link 16: 26.562 GB/s
Link 17: 26.562 GB/s
GPU 1: NVIDIA H100 80GB HBM3 (UUID: GPU-fe4b42e7-b7ad-49be-6765-66a18c186dc2)
Link 0: 26.562 GB/s
Link 1: 26.562 GB/s
Link 2: 26.562 GB/s

...

GPU 7: NVIDIA H100 80GB HBM3 (UUID: GPU-2d4d1957-9704-9732-3c2d-628238913816)
Link 0: 26.562 GB/s
Link 1: 26.562 GB/s
Link 2: 26.562 GB/s
Link 3: 26.562 GB/s
Link 4: 26.562 GB/s
Link 5: 26.562 GB/s
Link 6: 26.562 GB/s
Link 7: 26.562 GB/s
Link 8: 26.562 GB/s
Link 9: 26.562 GB/s
Link 10: 26.562 GB/s
Link 11: 26.562 GB/s
Link 12: 26.562 GB/s
Link 13: 26.562 GB/s
Link 14: 26.562 GB/s
Link 15: 26.562 GB/s
Link 16: 26.562 GB/s
Link 17: 26.562 GB/s
```

### Show specific output

The `nvidia-smi` command can return a wealth of content. You can fine-tune your
results to isolate a specific issue by using the `-d` (display) option along
with the `-q` (query) option. For example, if you suspect a memory issue, you
can choose to display only memory-related output:

```bash
nvidia-smi -q -d MEMORY
```

The output looks similar to the following:

```bash
==============NVSMI LOG==============

Timestamp                                 : Thu Aug 29 16:45:11 2024
Driver Version                            : 535.129.03
CUDA Version                              : 12.2

Attached GPUs                             : 1
GPU 00000000:08:00.0
    FB Memory Usage
        Total                             : 23028 MiB
        Reserved                          : 512 MiB
        Used                              : 4 MiB
        Free                              : 22511 MiB
    BAR1 Memory Usage
        Total                             : 32768 MiB
        Used                              : 1 MiB
        Free                              : 32767 MiB
    Conf Compute Protected Memory Usage
        Total                             : 0 MiB
        Used                              : 0 MiB
        Free                              : 0 MiB
```

The output you can display includes:

* `MEMORY`: displays memory usage in the system.
* `UTILIZATION`: displays GPU, memory, and encode/decode utilization rates, including sampling data with maximum, minimum, and average.
* `ECC`: displays error correction code mode and errors.
* `TEMPERATURE`: displays temperature data for the GPU.
* `POWER`: displays power readings, including sampling data with maximum, minimum, and average.
* `CLOCK`: displays data for all the clocks in the GPU, including sampling data with maximum, minimum, and average.
* `COMPUTE`: displays the compute mode for the GPU.
* `PIDS`: displays running processes.
* `PERFORMANCE`: displays performance information for the GPU.
* `SUPPORTED_CLOCKS`: displays the supported frequencies for the GPU clocks.
* `PAGE_RETIREMENT`: when ECC is enabled, this option displays any framebuffer pages that have been dynamically retired.
* `ACCOUNTING`: displays which processes are subject to accounting, how many processes are subject to accounting, and whether accounting mode is enabled.

You can specify multiple options by separating them with a comma. For example,
the following command displays information about both memory and power usage:

```bash
nvidia-smi -q -d MEMORY,POWER
```

The output looks similar to the following:

```bash
==============NVSMI LOG==============

Timestamp                                 : Thu Aug 29 16:54:52 2024
Driver Version                            : 535.129.03
CUDA Version                              : 12.2

Attached GPUs                             : 1
GPU 00000000:08:00.0
    FB Memory Usage
        Total                             : 23028 MiB
        Reserved                          : 512 MiB
        Used                              : 4 MiB
        Free                              : 22511 MiB
    BAR1 Memory Usage
        Total                             : 32768 MiB
        Used                              : 1 MiB
        Free                              : 32767 MiB
    Conf Compute Protected Memory Usage
        Total                             : 0 MiB
        Used                              : 0 MiB
        Free                              : 0 MiB
    GPU Power Readings
        Power Draw                        : 8.97 W
        Current Power Limit               : 150.00 W
        Requested Power Limit             : 150.00 W
        Default Power Limit               : 150.00 W
        Min Power Limit                   : 100.00 W
        Max Power Limit                   : 150.00 W
    Power Samples
        Duration                          : 117.96 sec
        Number of Samples                 : 119
        Max                               : 16.00 W
        Min                               : 9.12 W
        Avg                               : 9.74 W
    Module Power Readings
        Power Draw                        : N/A
        Current Power Limit               : N/A
        Requested Power Limit             : N/A
        Default Power Limit               : N/A
        Min Power Limit                   : N/A
        Max Power Limit                   : N/A
```

## Contact Lambda

If you can’t discover the cause for the issue you are experiencing, [contact
Lambda Support](https://support.lambdalabs.com/hc/en-us/requests/new) and
generate then upload the [Lambda bug
report](https://docs.lambdalabs.com/software/troubleshooting-and-debugging#generate-a-lambda-bug-report),
which includes data from the NVIDIA bug report. For example:

```bash
wget -nv -O - https://raw.githubusercontent.com/lambdal-support/lambda-public-tools/main/lambda-bug-report.sh | bash -
```

!!! warning

    You can run this script only on Lambda workstations and On-Demand Cloud instances. Do **not** run it on a cluster.

## Other resources

NVIDIA has a wealth of resources on their processes and tools. For more information, read the following:

* [NVIDIA GPU debug guidelines](https://docs.nvidia.com/deploy/gpu-debug-guidelines/index.html#abstract)
* [Useful nvidia-smi queries](https://nvidia.custhelp.com/app/answers/detail/a\_id/3751/\~/useful-nvidia-smi-queries)
* [nvidia-smi.txt](https://developer.download.nvidia.com/compute/DCGM/docs/nvidia-smi-367.38.pdf)
