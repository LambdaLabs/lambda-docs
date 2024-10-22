# Configuring Software RAID

Software RAID (redundant array of independent disks) provides fast and resilient storage for your machine learning data. This document shows you how to configure software RAID in your cluster using [`mdadm`](https://linux.die.net/man/8/mdadm).

1. Install new drives as needed, then power on the machine.
2. Check that the drives are present with `lsblk`. Your output should look similar to the following:

```bash
ubuntu@ubuntu:~$ lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
nvme1n1  259:1   0 1.8T  0 disk
nvme3n1  259:2   0 1.8T  0 disk
nvme2n1  259:3   0 1.8T  0 disk
...
```

3. Use `parted` to partition and format the drives.&#x20;

!!! warning

    Before running this step, ensure you back up **all** your data on this drive. Formatting the drive makes your data unrecoverable.

```bash
ubuntu@ubuntu:~$ sudo parted /dev/nvme1n1
GNU Parted 3.4
Using /dev/nvme1n1
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) mklabel gpt
(parted) mkpart primary ext4 0% 100%
(parted) set 1 raid on
(parted) print
Model: MSI M480 PRO 2TB (nvme)
Disk /dev/nvme1n1: 2000GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name     Flags
   1    1049kB  2000GB  2000GB  ext4         primary  raid

(parted) quit
Information: You may need to update /etc/fstab.
```

4. Repeat the previous step for every drive listed in step 2.
5. Use `mdadm` to create the RAID array with the new drives.&#x20;

!!! note

    If mdadm is not on the system already, you can install it by running:

    `ubuntu@ubuntu:~$ sudo apt update && sudo apt install mdadm`

This example uses RAID level 5. To use a different [RAID level](https://en.wikipedia.org/wiki/Standard\_RAID\_levels) (see below), set `--level` to the desired RAID level. Your output should look similar to the following:

```bash
ubuntu@ubuntu:~$ sudo mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
```

6. Format the RAID array:

```bash
ubuntu@ubuntu:~$ sudo mkfs.ext4 /dev/md0
```

7. Update the `mdadm` configuration file so that the software RAID persists through reboots:

```bash
ubuntu@ubuntu:~$ sudo mdadm --detail --scan >> /etc/mdadm/mdadm.conf
ubuntu@ubuntu:~$ sudo update-initramfs -u
```

8. Create a mount point and mount the array:

```bash
ubuntu@ubuntu:~$ sudo mkdir -p /mnt/raid5 sudo mount /dev/md0 /mnt/raid5
```

9. Get the the block ID for your RAID array, then add a line to your `/etc/fstab` so it mounts at boot:

```bash
ubuntu@ubuntu:~$ sudo blkid /dev/md0
#Use a text editor to edit /etc/fstab and add the following line
UUID-your-uuid-here /mnt/raid5 ext4 defaults,nofail 0 2
```

## Understanding RAID Levels

There are a number of RAID levels, each performing a slightly different function. RAID 5, used in the steps above, provides a good balance between performance and availability. Many sources online — [Wikipedia](https://en.wikipedia.org/wiki/Standard\_RAID\_levels) for example — provide more information about RAID levels.
