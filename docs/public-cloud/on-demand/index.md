# Overview

On-Demand Cloud (ODC) provides on-demand access to Linux-based, GPU-backed
virtual machine instances.

## Instance types

ODC offers a variety of predefined instance types to support different workload
requirements. Available GPUs include the state-of-the-art NVIDIA H100 Tensor
Core GPU, NVIDIA A100 Tensor Core GPU, and NVIDIA A10 Tensor Core GPU, as well
as several earlier models. Each instance you create is tied to a specific
geographical region. For a list of available regions, see the
[Regions](#regions) section below.

Select instance types are backed by GPUs that feature NVIDIA SXM. SXM offers
improved bandwidth between the NVIDIA GPUs in a single physical server.

!!! warning

    Lambda prohibits cryptocurrency mining on ODC instances.

As of October 2024, ODC offers the following instance types:

| GPU        | Number of GPUs | VRAM   | vCPU cores | RAM       | Root volume size |
|------------|----------------|--------|------------|-----------|------------------|
| H100 SXM   | 8              | 80 GB  | 208        | 1800 GiB  | 26 TiB           |
|            | 8              | 80 GB  | 208        | 1800 GiB  | 22 TiB           |
|            | 4              | 80 GB  | 104        | 900 GiB   | 11 TiB           |
|            | 2              | 80 GB  | 52         | 450 GiB   | 5.5 TiB          |
|            | 1              | 80 GB  | 26         | 225 GiB   | 2.75 TiB         |
| H100 PCIe  | 1              | 80 GB  | 26         | 225 GiB   | 1 TiB            |
| A100 SXM   | 8              | 80 GB  | 240        | 1800 GiB  | 19.5 TiB         |
|            | 8              | 40 GB  | 124        | 1800 GiB  | 5.8 TiB          |
|            | 1              | 40 GB  | 30         | 220 GiB   | 512 GiB          |
| A100 PCIe  | 4              | 40 GB  | 120        | 900 GiB   | 1 TiB            |
|            | 2              | 40 GB  | 60         | 450 GiB   | 1 TiB            |
|            | 1              | 40 GB  | 30         | 225 GiB   | 512 GiB          |
| A10        | 1              | 24 GB  | 30         | 226 GiB   | 1.3 TiB          |
| A6000      | 4              | 48 GB  | 56         | 400 GiB   | 1 TiB            |
|            | 2              | 48 GB  | 28         | 200 GiB   | 1 TiB            |
|            | 1              | 48 GB  | 14         | 100 GiB   | 512 GiB          |
| Tesla V100 | 8              | 16 GB  | 88         | 448 GiB   | 5.8 TiB          |
| RTX 6000   | 1              | 24 GB  | 14         | 46 GiB    | 512 GiB          |

## Storage

### Root volume

Each new instance comes with a root volume of a predefined size. The specific
volume size depends on which instance type you choose. To see which volume sizes
are associated with each instance type, check out the instance type table above.

### Filesystems

When you create a new instance, you're prompted to attach a **filesystem**. A
filesystem is a unit of networked persistent storage you can connect to your
instance. Filesystems are typically several orders of magnitude larger than your
root volume, and are an ideal place to store both your instance state and your
large datasets.

To use a filesystem with your instance, you must attach it during the instance
creation process. The filesystem must also reside in the same region as your
instance.

For more information about filesystems, see [Filesystems](../filesystems.md).

#### Mount point

When you mount a filesystem to your instance, the filesystem is available at:

```markup
/home/ubuntu/<FILESYSTEM_NAME>
```

#### Billing

Filesystems are billed per GB used per month, in increments of one hour. If you
delete an instance but not its associated filesystem, you'll continue to be
billed for the filesystem. For more details on filesystem pricing, see
[Filesystems](../filesystems.md).

#### Size limits

ODC filesystem size limits vary by region. In general, filesystems have a
capacity of 8 EB (8,000,000 TB), and you can have a total of 24 filesystems. In
the Texas, USA (us-south-1) region, filesystem size is limited to 10 TB.

## Network

### Connection options

You connect to your instance through a standard SSH connection. For information
on creating and managing SSH keys, see [Dashboard > Add, generate, and delete
SSH keys](dashboard.md#add-generate-and-delete-ssh-keys) and [Cloud API >
Managing SSH keys](../cloud-api.md#managing-ssh-keys).

### Firewall defaults

You can define inbound TCP and UDP firewall rules on the [Firewall
page](https://cloud.lambdalabs.com/firewall) in the Lambda Cloud dashboard. By
default, only port 22 (SSH) is open.

ODC allows ICMP traffic by default, as many network diagnostic tools rely on
ICMP to determine where connectivity issues are occurring. If you'd prefer to
restrict ICMP traffic, you can do so on the Firewall page.

### Regions

Lambda resources are hosted in multiple locations worldwide. Not every instance
type will be available in every region.

| Region           | Physical location  |
| ---------------- | ------------------ |
| asia-northeast-1 | Tokyo, Japan       |
| asia-northeast-2 | Osaka, Japan       |
| asia-south-1     | India              |
| europe-central-1 | Germany            |
| me-west-1        | Israel             |
| us-east-1        | Virginia, USA      |
| us-east-2        | Washington DC, USA |
| us-midwest-1     | Illinois, USA      |
| us-south-1       | Texas, USA         |
| us-south-2       | North Texas, USA   |
| us-south-3       | Central Texas, USA |
| us-west-1        | California, USA    |
| us-west-2        | Arizona, USA       |
| us-west-3        | Utah, USA          |

## Instance management

### Dashboard

In addition to the firewall settings mentioned earlier, you can manage your
instances, filesystems, SSH keys, API keys, and more through the Lambda Cloud
dashboard. For more information, see [Dashboard](dashboard.md).

### Cloud API

You can perform administrative tasks such as creating, restarting, listing, and
terminating your instances through the Lambda Cloud API. For more details, see
[Cloud API](../cloud-api.md).

## Preinstalled software

Each ODC instance runs Ubuntu 22.04 LTS. Lambda also preinstalls the Lambda
Stack, a standard set of AI/ML-related drivers, tools, and frameworks, on each
instance:

* _NVIDIA tools, libraries, and drivers_: CUDA, cuDNN, NCCL, NVIDIA container
  toolkit, NVIDIA driver
* _Deep learning frameworks and libraries_: TensorFlow, torchvision, Keras,
  PyTorch®, JAX, Triton
* _Dev tools_: Git, Vim, Emacs, Valgrind, tmux, screen, htop, build-essential

For more information on Lambda Stack, see
[Lambda Stack](https://lambdalabs.com/lambda-stack-deep-learning-software).

In addition, each ODC instance provides a JupyterLab installation for creating
and managing Jupyter notebooks. You can access your instance's JupyterLab by
visiting the [Instances page](https://cloud.lambdalabs.com/instances) in the
Lambda Cloud dashboard and clicking **Cloud IDE** in your instance's row.

## Next steps

* [Visit the Lambda Cloud portal](https://cloud.lambdalabs.com)
* [Explore the Lambda Cloud API](../cloud-api.md)
