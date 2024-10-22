---
description: Learn how to start using Lambda Public Cloud on-demand instances.
tags:
  - 1-click clusters
  - automation
  - on-demand cloud
---

# Getting started

## Can my data be recovered once I've terminated my instance?

!!! warning

    We cannot recover your data once you've terminated your instance! Before
    terminating an instance, make sure to back up all data that you want to
    keep.

    If you want to save data even after you terminate your instance, create a
    [persistent storage file system
    :octicons-link-external-16:](https://lambdalabs.com/blog/persistent-storage-beta/){target="_blank"}.

!!! note

    The persistent storage file system must be attached to your instance before
    you start your instance. The file system cannot be attached to your instance
    after you start your instance.

    When you create a file system, a directory with the name of your file system
    is created in your home directory. For example, if the name of your file
    system is **PERSISTENT-FILE-SYSTEM**, the directory is created at
    `/home/ubuntu/PERSISTENT-FILE-SYSTEM`. **Data not stored in this directory
    is erased once you terminate your instance and cannot be recovered.**

## Can I pause my instance instead of terminating it?

It currently isn't possible to pause (suspend) your instance rather than
terminating it. But, this feature is in the works.

Until this feature is implemented, you can use persistent storage file systems
to imitate some of the benefits of being able to pause your instance.

## Do you support Kubernetes (K8s)?

Yes. You can install and use Kubernetes, also known as K8s, on on-demand
instances and [1-Click Clusters
:octicons-link-external-16:](https://lambdalabs.com/service/gpu-cloud/1-click-clusters){target=_blank}.

Additionally, Managed Kubernetes and Pre-Installed Kubernetes are available for
1-Click Clusters and [Reserved Cloud
:octicons-link-external-16:](https://lambdalabs.com/service/gpu-cloud/reserved){target="_blank"}.

Managed Kubernetes includes:

* Kubernetes installation and upgrades.
* Control plane maintenance and high-availability.
* [NVIDIA GPU Operator
  :octicons-link-external-16:](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/index.html){target="_blank"}
  installed and configured to deploy and manage NVIDIA GPUs in a Kubernetes
  cluster.
* Detecting node failures, node pool adjustment and failed hardware replacement.
* Gathering chassis and cluster metrics and proactive monitoring.

See our [:material-file-pdf-box: Managed Kubernetes Product
Outline](../../assets/docs/Lambda_Kubernetes_One_Pager.pdf){target="_blank"} to learn
more.

## Why can't my program find the NVIDIA cuDNN library?

Unfortunately, the NVIDIA cuDNN license limits how cuDNN can be used on our
instances.

On our instances, cuDNN can only be used by the PyTorch® framework and
TensorFlow library installed as part of Lambda Stack.

Other software, including PyTorch and TensorFlow installed outside of Lambda
Stack, won't be able to find and use the cuDNN library installed on our
instances.

!!! tip

    Software outside of Lambda Stack usually looks for the cuDNN library files
    in `/usr/lib/x86_64-linux-gnu`. However, on our instances, the cuDNN library
    files are in `/usr/lib/python3/dist-packages/tensorflow`.

    Creating symbolic links, or "symlinks," for the cuDNN library files might
    allow your program to find the cuDNN library on our instances.

    Run the following command to create symlinks for the cuDNN library files:

    ```bash
    for cudnn_so in /usr/lib/python3/dist-packages/tensorflow/libcudnn*; do
      sudo ln -s "$cudnn_so" /usr/lib/x86_64-linux-gnu/
    done
    ```

## How do I open Jupyter Notebook on my instance?

To open Jupyter Notebook on your instance:

1. In the [GPU instances dashboard
   :octicons-link-external-16:](https://cloud.lambdalabs.com/instances){target="_blank"},
   find the row for your instance.
2. Click **Launch** in the **Cloud IDE** column.

!!! tip

    Watch Lambda's [GPU Cloud Tutorial with Jupyter
    Notebook :octicons-link-external-16:](https://www.youtube.com/watch?v=CKxR6ClKstU){target="_blank"} video on YouTube to
    learn more about using Jupyter Notebook on Lambda GPU Cloud instances.

## How do I upgrade Python?

!!! warning

    Upgrading Python, that is, replacing the preinstalled Python version with a
    newer version, will break your instance.

    Instead of upgrading Python, you should install your desired version of
    Python alongside the preinstalled version, and use your desired version in a
    virtual environment.

To install another version of Python alongside the preinstalled version:

1. Run the following command:

      ```bash
      sudo apt -y update && sudo apt -y install pythonVERSION-full
      ```

      Replace `VERSION` with the Python version you want to install, for
      example, `3.13`. Make sure `-full` is appended to the Python version,
      otherwise, you won't have the `venv` module needed to create Python
      virtual environments.

      As a complete example, if you want to install Python version 3.13, run:

      ```bash
      sudo apt -y update && sudo apt -y install python3.13-full
      ```

1. Run the following command to create a Python virtual environment:

      ```bash
      pythonVERSION -m venv VENV-NAME
      ```

      Replace `VERSION` with the Python version you installed in the previous
      step. Replace `VENV-NAME` with the name you want to give your virtual
      environment.

1. Run the following command to activate the virtual environment:

      ```bash
      source VENV-NAME/bin/activate
      ```

      Replace `VENV-NAME` with the name you gave your virtual environment.

      As a complete example, if you want to create a virtual environment named
      `my-virtual-environment` using Python version 3.13 (installed in the example
      in the previous step), run:

      ```bash
      python3.13 -m venv my-virtual-environment
      source my-virtual-environment/bin/activate
      ```

1. Run the following command to confirm that your virtual environment is using your desired Python
   version:

      ```bash
      python --version
      ```

## Can I upgrade to the latest Ubuntu release?

!!! warning

    **Do not run `sudo do-release-upgrade` or try to upgrade to the latest
    Ubuntu release.** Doing so will break [Jupyter
    Notebook](#how-do-i-open-jupyter-notebook-on-my-instance) and unless you
    have [SSH access](#) to your instance, [Lambda Support
    :octicons-link-external-16:](https://lambdalabs.com/support){target="_blank"}
    won't be able to help you recover your data.

    Jupyter Notebook on our instances is configured and tested for the
    preinstalled version of Python. Upgrading to the latest Ubuntu release will
    replace the preinstalled version of Python and make Jupyter Notebook
    inaccessible.

## Is it possible to use more than one SSH key?

It's possible to allow more than one SSH key to access your instance. To do so,
you need to add public keys to `~/.ssh/authorized_keys`. You can do this with
the `echo` command.

!!! tip

    You can also [import SSH keys from GitHub](#).

!!! note

    This FAQ assumes that you've already generated another SSH key pair, that
    is, a private key and a public key.

    Public keys look like this:

    ```{.text .no-copy}
    ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIK5HIO+OQSyFjz0clkvg+48YAihYMo5J7AGKiq+9Alg8 user@hostname
    ```

SSH into your instance as you normally do and run:

```bash
echo 'PUBLIC-KEY' >> ~/.ssh/authorized_keys
```

Replace `PUBLIC-KEY` with the public key you want to add to your instance.
**Make sure to keep the single quotes (`' '`).**

You should now be able to log into your instance using the SSH key you just
added.

!!! tip

    You can make sure the public key has been added by running:

    ```bash
    cat ~/.ssh/authorized_keys
    ```

    The last line of output should be the public key you just added.

## What SSH key formats are supported?

You can add SSH keys in the following formats using the [dashboard](#) or the
[Cloud API](#):

* OpenSSH (the format `ssh-keygen` uses by default when generating keys)
* RFC4716 (the format PuTTYgen uses when you save a public key)
* PKCS8
* PEM

!!! note

    *   OpenSSH keys look like:

        ```{.text .no-copy}
        ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIK5HIO+OQSyFjz0clkvg+48YAihYMo5J7AGKiq+9Alg8 foo@bar
        ```

    *   RFC4716 keys begin with:

        ```{.text .no-copy}
        ---- BEGIN SSH2 PUBLIC KEY ----
        ```

    *   PKCS8 keys begin with:

        ```{.text .no-copy}
        -----BEGIN PUBLIC KEY-----
        ```

    *   PEM keys begin with, for example:

        ```{.text .no-copy}
        -----BEGIN RSA PUBLIC KEY-----
        ```

## How long does it take for instances to launch?

Single-GPU instances usually take 3-5 minutes to launch.

Multi-GPU instances usually take 10-15 minutes to launch.

!!! note

    [Jupyter Notebook](#how-do-i-open-jupyter-notebook-on-my-instance) and
    [Demos](#) can take a few minutes after an instance launches to become
    accessible.

!!! note

    Billing starts the moment an instance begins booting.

## What network bandwidth does Lambda GPU Cloud provide?

!!! note

    Some sites limit transfer speeds. This is known as _bandwidth throttling_.

    **Lambda GPU Cloud doesn't limit your transfer speeds** but can't control
    other sites' use of bandwidth throttling.

    Further, **real-world network bandwidth depends on a variety of factors**,
    including the total number of connections opened by your applications and
    overall network utilization.

### Utah, USA region (us-west-3)

The bandwidth between instances in our Utah, USA region (us-west-3) can be up to
200 Gbps.

The total bandwidth from this region to the Internet can be up to 20 Gbps.

### Texas, USA region (us-south-1)

The bandwidth between instances in our Texas, USA region (us-south-1) can be up
to 200 Gbps.

The total bandwidth from this region to the Internet can be up to 20 Gbps.

!!! note

    We're in the process of testing the network bandwidth in our other regions.

## How do I learn my instance's private IP address and other info?

You can [learn your instance's private IP
address](getting-started.md#learn-your-instances-private-ip-address) with the
`ip` command.

You can [learn what ports are open on your
instance](getting-started.md#learn-what-ports-on-your-instance-are-publicly-accessible)
with the `nmap` command.

### Learn your instance's private IP address

To learn your instance's private IP address, SSH into your instance and run:

```bash
ip -4 -br addr show enp5s0
```

The above command will output, for example:

```
enp5s0           UP             10.19.51.71/20
```

In the above example, the instance's private IP address is **10.19.51.71**.

!!! tip

    If you want your instance's private IP address and only that address, run
    the following command instead:

    ```bash
    ip -4 -br addr show enp5s0 | grep -Eo '(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)'
    ```

    The above command will output, for example:

    ```{.text .no-copy}
    10.19.51.71
    ```

### Learn what ports on your instance are publicly accessible

You can use Nmap to learn what ports on your instance are publicly accessible,
that is, reachable over the Internet.

!!! note

    The instructions, below, assume you're running Ubuntu on your computer.

First, install Nmap on your computer (not on your instance) by running:

```bash
sudo apt install -y nmap
```

Next, run:

```bash
nmap -Pn INSTANCE-IP-ADDRESS
```

Replace `INSTANCE-IP-ADDRESS` with your instance's IP address, which you can
get from the [Cloud dashboard :octicons-link-external-16:](https://cloud.lambdalabs.com/instances){target="_blank"}.

The command will output, for example:

```{.text .no-copy}
Starting Nmap 7.80 ( https://nmap.org ) at 2023-01-11 13:22 PST
Nmap scan report for 129.159.46.35
Host is up (0.041s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE
22/tcp open  ssh

Nmap done: 1 IP address (1 host up) scanned in 6.42 seconds
```

In the above example, TCP port 22 (SSH) is publicly accessible.

!!! note

    If `nmap` doesn’t show TCP/22 (SSH) or any other ports open, your:

    * Instance might be terminated. Check the [GPU Instances dashboard
      :octicons-link-external-16:](https://cloud.lambdalabs.com/instances){target="_blank"}
      to find out.
    * Firewall rules might be blocking incoming connections to your instance.

!!! note

    `nmap -Pn INSTANCE-IP-ADDRESS` only scans the 1,000 most common TCP ports.

## How do I close my account?

To close your Lambda GPU Cloud account:

1. Back up all of your data on your instances as well as in your persistent
   storage file systems.

      !!! tip

       [You can use rsync to back up your data](#).

2. Terminate all of your instances from the [Cloud dashboard](#) or using the
   [Cloud API](#).

3. Delete all of your persistent storage file systems.

4. In the [Cloud dashboard](#), under
   [**Settings** :octicons-link-external-16:](https://cloud.lambdalabs.com/settings){target="_blank"}, click **Close my
   account**. Carefully read the warning in the dialog box that appears. To
   proceed with closing your account, type in **close account**, then click
   **Close account**.
