---
title: "How do I use persistent storage to save datasets and system state?"
type: docs
aliases:
- guides/use-persistent-storage
---

You can use the
[Lambda Cloud Storage](https://lambdalabs.com/blog/persistent-storage-beta/)
feature to save:

- Large datasets that you don't want to re-upload every time you start an
  instance
- The [state of your system]({{< relref "#preserving-the-state-of-your-system" >}}),
  including software packages and configurations

{{% alert title="Note" color="info" %}}
You can have up to 24 persistent storage file systems.
{{% /alert %}}

## Preserving the state of your system

For saving the state of your system, including:

- Packages installed system-wide using `apt-get`
- Python packages installed using `pip`
- conda environments

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
[create a Python virtual environment]({{< relref "../linux/create-python-virtual-environment" >}}).

You can also create a
[conda environment]({{< relref "../linux/create-conda-virtual-environment" >}}).

{{% alert title="Tip" color="success" %}}
For the highest performance when training, we recommend copying your dataset,
containers, and virtual environments from persistent storage to your home
directory. This can take some time but greatly increases the speed of
training.
{{% /alert %}}
