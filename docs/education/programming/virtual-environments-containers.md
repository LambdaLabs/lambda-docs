---
description: Learn how to create a Python or conda virtual development environment
tags:
  - Virtualization
---
# Virtual environments and Docker containers

## What are virtual environments?

Virtual environments allow you to create and maintain development environments
that are isolated from each other. Lambda recommends using either:

- [Python venv](#creating-a-python-virtual-environment)
- [conda](#creating-a-conda-virtual-environment)

### Creating a Python virtual environment

1. Create a Python virtual environment using the `venv` module by running:

    ```bash
    python -m venv --system-site-packages NAME
    ```

    Replace `NAME` with the name you want to give to your virtual environment.

    !!! note

        The command, above, creates a virtual environment that has access to
        Lambda Stack packages and packages installed from Ubuntu repositories.

        To create a virtual environment that _doesn't_ have access to Lambda
        Stack and Ubuntu packages, omit the `--system-site-packages` option.

1. Activate the virtual environment by running:

    ```bash
    source NAME/bin/activate
    ```

    Replace `NAME` with the name you gave your virtual environment in the previous step.

Python packages you install in your virtual environment are isolated from the
base environment and other virtual environments.

!!! note

    Locally installed packages can conflict with packages installed in virtual
    environments. For this reason, it's recommended to uninstall locally
    installed packages by running:

    To uninstall packages installed locally for your user only, run:

    ```bash
    pip uninstall -y $(pip -v list | grep ${HOME}/.local | awk '{printf "%s ", $1}')
    ```

    To uninstall packages installed locally, system-wide (for all users), run:

    ```bash
    sudo pip uninstall -y $(pip -v list | grep /usr/local | awk '{printf "%s ", $1}')
    ```

!!! warning

    **Don't run the above uninstall commands on Lambda GPU Cloud on-demand instances!**

    The above uninstall commands remove all locally installed packages and, on
    on-demand instances, break programs including pip and JupyterLab.

!!! tip

    See the
    [Python venv module documentation](https://docs.python.org/3/library/venv.html){ .external target="_blank" }
    to learn more about Python virtual environments.

### Creating a conda virtual environment

To create a conda virtual environment:

1. Download the latest version of
   [Miniconda3](https://docs.conda.io/en/latest/miniconda.html){ .external target="_blank" }
   by running:

    ```bash
    curl -O https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
    ```

    Then, install Miniconda3 by running the command:

    ```bash
    sh Miniconda3-latest-Linux-x86_64.sh
    ```

    Follow the installer prompts. Install Miniconda3 in the default location. Allow the installer to initialize Miniconda3.

1. If you want to create a conda virtual environment immediately after
   installing Miniconda3, you need to load the changes made to your `.bashrc`.

    You can either:

    - Exit and reopen your shell (terminal).
    - Run `source ~/.bashrc`.

    !!! tip

        For compatibility with the
        [Python venv module](#creating-a-python-virtual-environment),
        it's recommended that you disable automatic activation of the conda base
        environment by running:

        ```bash
        conda config --set auto_activate_base false
        ```

1. Create a conda virtual environment using Miniconda3 by running:

    ```bash
    conda create OPTIONS -n NAME PACKAGES
    ```

    Replace `NAME` with the name you want to give your virtual environment.

    Replace `PACKAGES` with the list of packages you want to install in your virtual
    environment.

    (Optional) Replace `OPTIONS` with options for the `conda create` command.
    [See the `conda create` documentation](https://docs.conda.io/projects/conda/en/latest/commands/create.html){ .external target="_blank" }
    to learn more about available options.

    For example, to create a conda virtual environment for PyTorch® with CUDA 11.8, run the below command and follow the prompts:

    ```bash
    conda create -c pytorch -c nvidia -n pytorch+cuda_11-8 pytorch torchvision torchaudio pytorch-cuda=11.8
    ```

1. Activate the conda virtual environment by running:

    ```bash
    conda activate NAME
    ```

    Replace `NAME` with the name of the virtual environment created in the
    previous step.

    For instance, to activate the example PyTorch with CUDA 11.8 virtual
    environment mentioned in the previous step, run:

    ```bash
    conda activate pytorch+cuda_11-8
    ```

    Once activated, you can test the example virtual environment is working by running:

    ```bash
    python -c 'import torch ; print("\nIs available: ", torch.cuda.is_available()) ; print("Pytorch CUDA Compiled version: ", torch._C._cuda_getCompiledVersion()) ; print("Pytorch version: ", torch.__version__) ; print("pytorch file: ", torch.__file__) ; num_of_gpus = torch.cuda.device_count(); print("Number of GPUs: ",num_of_gpus)'
    ```

    You should see output similar to:

    ```{ .text .no-copy }
    Is available:  True
    Pytorch CUDA Compiled version:  11080
    Pytorch version:  2.0.1
    pytorch file:  /home/ubuntu/miniconda3/envs/pytorch+cuda_11-8/lib/python3.11/site-packages/torch/__init__.py
    Number of GPUs:  1
    ```

!!! note

    Locally installed packages can conflict with packages installed in virtual
    environments. For this reason, it's recommended to uninstall locally
    installed packages by running:

    To uninstall packages installed locally for your user only, run:

    ```bash
    pip uninstall -y $(pip -v list | grep ${HOME}/.local | awk '{printf "%s ", $1}')
    ```

    To uninstall packages installed locally, system-wide (for all users), run:

    ```bash
    sudo pip uninstall -y $(pip -v list | grep /usr/local | awk '{printf "%s ", $1}')
    ```

!!! warning

    **Don’t run the above uninstall commands on Lambda GPU Cloud on-demand instances!**

    The above uninstall commands remove all locally installed packages and, on
    on-demand instances, break programs including pip and JupyterLab.

[See the Conda documentation](https://docs.conda.io/projects/conda/en/stable/commands.html){ .external target="_blank" }
to learn more about how to manage conda virtual environments.

## Installing Docker and creating a container

!!! note

    Docker and NVIDIA Container Toolkit are preinstalled on Cloud on-demand
    instances.

    **If you're using an on-demand instance, skip step 1, below.**

To create and run a Docker container:

1. Install Docker and
   [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/overview.html){ .external target="_blank" }
   by running:

    ```bash
    sudo apt -y update && sudo apt -y install docker.io nvidia-container-toolkit && \
    sudo systemctl daemon-reload && \
    sudo systemctl restart docker
    ```

1. Add your user to the `docker` group by running:

    ```bash
    sudo adduser "$(id -un)" docker
    ```

    Then, exit and reopen a shell (terminal) so that your user can create and
    run Docker containers.

1. Locate the Docker image for the container you want to create. For example,
   the
   [NVIDIA NGC Catalog](https://catalog.ngc.nvidia.com/){ .external
   target="_blank" }
   has
   [images for creating TensorFlow NGC containers](https://catalog.ngc.nvidia.com/orgs/nvidia/containers/tensorflow/tags){ .external target="_blank" }.

1. Create a container from the Docker image, and run a command in the container, by running:

    ```bash
    docker run --gpus all -it IMAGE COMMAND
    ```

    Replace `IMAGE` with the URL to the image for the container you want to
    create.

    Replace `COMMAND` with the command you want to run in the container.

    For example, to create a
    [TensorFlow NGC container](https://catalog.ngc.nvidia.com/orgs/nvidia/containers/tensorflow){ .external target="_blank" }
    and run a command to get the container's TensorFlow build information, run:

    ```bash
    docker run --gpus all -it nvcr.io/nvidia/tensorflow:23.05-tf2-py3 python -c "import tensorflow as tf ; sys_details = tf.sysconfig.get_build_info() ; print(sys_details)"
    ```

You should see output similar to the following:

```{ .text .no-copy }
================
== TensorFlow ==
================

NVIDIA Release 23.05-tf2 (build 59341886)
TensorFlow Version 2.12.0

Container image Copyright (c) 2023, NVIDIA CORPORATION & AFFILIATES. All rights reserved.
Copyright 2017-2023 The TensorFlow Authors.  All rights reserved.

Various files include modifications (c) NVIDIA CORPORATION & AFFILIATES.  All rights reserved.

This container image and its contents are governed by the NVIDIA Deep Learning Container License.
By pulling and using the container, you accept the terms and conditions of this license:
https://developer.nvidia.com/ngc/nvidia-deep-learning-container-license

NOTE: CUDA Forward Compatibility mode ENABLED.
  Using CUDA 12.1 driver version 530.30.02 with kernel driver version 525.85.12.
  See https://docs.nvidia.com/deploy/cuda-compatibility/ for details.

NOTE: The SHMEM allocation limit is set to the default of 64MB.  This may be
   insufficient for TensorFlow.  NVIDIA recommends the use of the following flags:
   docker run --gpus all --ipc=host --ulimit memlock=-1 --ulimit stack=67108864 ...

2023-06-08 17:09:50.643793: I tensorflow/core/util/port.cc:110] oneDNN custom operations are on. You may see slightly different numerical results due to floating-point round-off errors from different computation orders. To turn them off, set the environment variable `TF_ENABLE_ONEDNN_OPTS=0`.
2023-06-08 17:09:50.680974: I tensorflow/core/platform/cpu_feature_guard.cc:183] This TensorFlow binary is optimized to use available CPU instructions in performance-critical operations.
To enable the following instructions: SSE3 SSE4.1 SSE4.2 AVX, in other operations, rebuild TensorFlow with the appropriate compiler flags.
OrderedDict([('cpu_compiler', '/opt/rh/devtoolset-9/root/usr/bin/gcc'), ('cuda_compute_capabilities', ['sm_52', 'sm_60', 'sm_61', 'sm_70', 'sm_75', 'sm_80', 'sm_86', 'compute_90']), ('cuda_version', '12.1'), ('cudnn_version', '8'), ('is_cuda_build', True), ('is_rocm_build', False), ('is_tensorrt_build', True)])
```

See the
[Docker documentation](https://docs.docker.com/){ .external target="_blank" } to
learn more about using Docker.

You can also check out the Lambda blog post:
[NVIDIA NGC Tutorial: Run A PyTorch Docker Container Using Nvidia-Container-Toolkit On Ubuntu](https://lambdalabs.com/blog/nvidia-ngc-tutorial-run-pytorch-docker-container-using-nvidia-container-toolkit-on-ubuntu){ .external target="_blank" }.
