---
description: Provides an overview of Lambda filesystems
tags:
  - storage
---

# Filesystems

Filesystems, also known as persistent storage, allow you to store your large
datasets and the state of your instance, for example:

*  Packages installed system-wide using `apt-get`.
*  Python packages installed using `pip`.
*  [conda](../education/programming/virtual-environments-containers.md#creating-a-conda-virtual-environment)
   and
   [Python venv](../education/programming/virtual-environments-containers.md#creating-a-python-virtual-environment)
   virtual environments.

Lambda GPU Cloud filesystems have a capacity of 8 exabytes, or 8,000,000
terabytes, and you can have a total of 24 filesystems, except for filesystems
created in the Texas, USA (us-south-1) region. The capacity of filesystems
created in the Texas, USA (us-south-1) region is 10 terabytes.

## How are filesystems billed?

Persistent storage is billed per GB used per month, in increments of 1 hour.

For example, based on the price of $0.20 per GB used per month:

*  If you use 1,000 GB of your filesystem capacity for an entire month (30
   days, or 720 hours), you’ll be billed $200.00.
*  If you use 1,000 GB of your filesystem capacity for a single day (24
hours), you’ll be billed $6.67.

!!! note

    The actual price of persistent storage will be displayed when you create
    your filesystem.

## Can filesystems be accessed without an instance?

Persistent storage filesystems can't be accessed unless attached to an
instance _at the time the instance is launched_.

For this reason, _it's recommended that you keep a local copy of the files
you have saved in your persistent storage filesystems_. This can be done
using `rsync`.

!!! note

    Filesystems can't be attached to running instances and can't be mounted
    remotely, for example, using NFS.

    Moreover, filesystems can only be attached to instances in the same
    region. For example, a filesystem created in the us-west-1 (California,
    USA) region can only be attached to instances in the us-west-1 region.

    Filesystems can't be transferred from one region to another. However,
    you can copy data between filesystems using tools such as `rsync`.

    Lambda GPU Cloud currently doesn't offer block or object storage.

## Can I set a limit (quota) on my filesystem usage?

Currently, you can't set a limit (quota) on your persistent storage filesystem
usage.

You can see the usage of a persistent storage filesystem from within an
instance by running `df -h -BG`. This command will produce output similar to:

```{.text .no-copy}
Filesystem           1G-blocks  Used   Available Use% Mounted on
udev                       99G    0G         99G   0% /dev
tmpfs                      20G    1G         20G   1% /run
/dev/vda1                1357G   23G       1335G   2% /
tmpfs                      99G    0G         99G   0% /dev/shm
tmpfs                       1G    0G          1G   0% /run/lock
tmpfs                      99G    0G         99G   0% /sys/fs/cgroup
persistent-storage 8589934592G    0G 8589934592G   0% /home/ubuntu/persistent-storage
/dev/vda15                  1G    1G          1G   6% /boot/efi
/dev/loop0                  1G    1G          0G 100% /snap/core20/1822
/dev/loop1                  1G    1G          0G 100% /snap/lxd/24061
/dev/loop2                  1G    1G          0G 100% /snap/snapd/18357
tmpfs                      20G    0G         20G   0% /run/user/1000
```

In the example output, above:

* The name of the filesystem is `persistent-storage`.
* The size of the filesystem is `8589934592G` (8 exabytes).
* The available capacity of the filesystem is `8589934592G`.
* The used percentage of the filesystem is `0%`.
* The filesystem is mounted on `/home/ubuntu/persistent-storage`.

!!! note

    You can also use the Cloud API's `/file-systems` endpoint to find out your
    filesystem usage.

### Preserving the state of your system

For saving the state of your system, including:

* Packages installed system-wide using `apt-get`
* Python packages installed using `pip`
* conda environments

We recommend creating containers using Docker or other software for creating
containers.

You can also create a script that runs the commands needed to re-create your
system state. For example:

```bash
sudo apt install PACKAGE_0 PACKAGE_1 PACKAGE_2 && \
pip install PACKAGE_3 PACKAGE_4 PACKAGE_5
```

Run the script each time you start an instance.

If you only need to preserve Python packages and not packages installed
system-wide, you can
[create a Python virtual environment](../education/programming/virtual-environments-containers.md#creating-a-python-virtual-environment).

You can also create a
[conda environment](../education/programming/virtual-environments-containers.md#creating-a-conda-virtual-environment).

!!! tip

    For the highest performance when training, we recommend copying your
    dataset, containers, and virtual environments from persistent storage to
    your home directory. This can take some time but greatly increases the
    speed of training.
