---
title: "How do I create a conda virtual environment?"
type: docs
tags:
- CUDA
- PyTorch
---

To create a conda virtual environment:

1. Download the latest version of
   [Miniconda3](https://docs.conda.io/en/latest/miniconda.html) by running:

   ```bash
   curl -O https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
   ```

   Then, install Miniconda3 by running the command:

   ```bash
   sh Miniconda3-latest-Linux-x86_64.sh
   ```

   Follow the installer prompts. Install Miniconda3 in the default location.
   Allow the installer to initialize Miniconda3.

1. If you want to create a conda virtual environment immediately after
   installing Miniconda3, you need to load the changes made to your `.bashrc`.

   You can either:

   - Exit and reopen your shell (terminal).
   - Run `source ~/.bashrc`.

   {{% alert title="Tip" color="success" %}}
   For compatibility with the
   [Python venv module]({{< relref "create-python-virtual-environment" >}}),
   it's recommended that you disable automatic activation of the conda base
   environment by running:

   ```bash
   conda config --set auto_activate_base false
   ```
   {{% /alert %}}

1. Create a conda virtual environment using Miniconda3 by running:

   ```bash
   conda create OPTIONS -n NAME PACKAGES
   ```

   Replace **NAME** with the name you want to give your virtual environment.

   Replace **PACKAGES** with the list of packages you want to install in your
   virtual environment.

   {{% alert title="Note" color="info" %}}
   conda doesn't use packages installed locally, including packages part of
   [Lambda Stack](https://lambdalabs.com/lambda-stack-deep-learning-software).
   {{% /alert %}}

   (Optional) Replace **OPTIONS** with options for the `conda create` command.
   [See the `conda create` documentation](https://docs.conda.io/projects/conda/en/latest/commands/create.html)
   to learn more about available options.

   For example, to create a conda virtual environment for PyTorch® with CUDA
   11.8, run the below command and follow the prompts:

   ```bash
   conda create -c pytorch -c nvidia -n pytorch+cuda_11-8 pytorch torchvision torchaudio pytorch-cuda=11.8
   ```
1. Activate the conda virtual environment by running:

   ```bash
   conda activate NAME
   ```

   Replace **NAME** with the name of the virtual environment created in the
   previous step.

   For instance, to activate the example PyTorch with CUDA 11.8 virtual
   environment mentioned in the previous step, run:

   ```bash
   conda activate pytorch+cuda_11-8
   ```

   Once activated, you can test the example virtual environment is working by
   running:

   ```bash
   python -c 'import torch ; print("\nIs available: ", torch.cuda.is_available()) ; print("Pytorch CUDA Compiled version: ", torch._C._cuda_getCompiledVersion()) ; print("Pytorch version: ", torch.__version__) ; print("pytorch file: ", torch.__file__) ; num_of_gpus = torch.cuda.device_count(); print("Number of GPUs: ",num_of_gpus)'
   ```

   You should see output similar to:

   ```
   Is available:  True
   Pytorch CUDA Compiled version:  11080
   Pytorch version:  2.0.1
   pytorch file:  /home/ubuntu/miniconda3/envs/pytorch+cuda_11-8/lib/python3.11/site-packages/torch/__init__.py
   Number of GPUs:  1
   ```

{{% alert title="Tip" color="success" %}}
To deactivate the conda environment, run `conda deactivate`.
{{% /alert %}}

{{% alert title="Note" color="info" %}}
Locally installed packages can conflict with packages installed in virtual
environments. For this reason, it's recommended to uninstall locally installed
packages by running:

To uninstall packages installed locally for your user only, run:

```bash
pip uninstall -y $(pip -v list | grep ${HOME}/.local | awk '{printf "%s ", $1}')
```

**Don't run the above uninstall command on Lambda GPU Cloud on-demand
instances!** The command will break programs including pip and JupyterLab.

To uninstall packages installed locally, system-wide (for all users), run:

```bash
sudo pip uninstall -y $(pip -v list | grep /usr/local | awk '{printf "%s ", $1}')
```
{{% /alert %}}

{{% alert title="Tip" color="success" %}}
[See the Conda documentation](https://docs.conda.io/projects/conda/en/stable/commands.html)
to learn more about how to manage conda virtual environments.
{{% /alert %}}
