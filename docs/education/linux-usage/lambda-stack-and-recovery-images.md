# Lambda Stack and recovery images

## Removing and reinstalling Lambda Stack

To remove and reinstall [Lambda Stack](https://lambdalabs.com/lambda-stack-deep-learning-software):

Uninstall (purge) the existing Lambda Stack by running:

```bash
sudo rm -f /etc/apt/sources.list.d/{graphics,nvidia,cuda}* && \
dpkg -l | \
awk '/cuda|lib(accinj64|cu(blas|dart|dnn|fft|inj|pti|rand|solver|sparse)|magma|nccl|npp|nv[^p])|nv(idia|ml)|tensor(flow|board)|torch/ { print $2 }' | \
sudo xargs -or apt -y remove --purge
```

Then, install the latest Lambda Stack by running:

```bash
wget -nv -O- https://lambdalabs.com/install-lambda-stack.sh | sh -
```

## Recovery images

!!! note

    See the [Install Ubuntu desktop tutorial](https://ubuntu.com/tutorials/install-ubuntu-desktop), specifically steps 3 and 4, to learn how to create and boot a USB stick (flash drive) using the below recovery images.


### Workstations

Recovery ISO images for [Vector](https://lambdalabs.com/gpu-workstations/vector)
and [Vector One](https://lambdalabs.com/gpu-workstations/vector-one) can be
downloaded using the following links:

* [Lambda Recovery (Focal)](https://files.lambdalabs.com/recovery/lambda-recovery-focal-20230704.iso) (based on Ubuntu 20.04 LTS _focal_)
* [Lambda Recovery (Jammy)](https://files.lambdalabs.com/recovery/lambda-recovery-jammy-20230704.iso) (based on Ubuntu 22.04 LTS _jammy_)

### Tensorbook

The recovery ISO image for
[Tensorbook](https://lambdalabs.com/deep-learning/laptops/tensorbook) can be
downloaded using the following link:

* [Lambda Recovery for Tensorbook (Jammy)](https://files.lambdalabs.com/recovery/tensorbook-jammy-20230704.iso) (based on Ubuntu 22.04 LTS _jammy_)

!!! note

    This recovery image is for the _Razer x Lambda Tensorbook_ only and won't work on older Tensorbook models.


!!! note
    The recovery images contain software distributed under various licenses, including the [Software License Agreement (SLA) for NVIDIA cuDNN](https://docs.nvidia.com/deeplearning/cudnn/sla/index.html). The licenses can be viewed in the recovery images at `/usr/share/doc/*/copyright`. By using the software contained in the recovery images, you agree to these licenses.


### Servers

Recovery images aren't available for servers.

To reinstall Ubuntu and Lambda Stack on your Lambda server, [download the Ubuntu
22.04 Server install image](https://releases.ubuntu.com/22.04/), then follow
[Ubuntu's Server installation
instructions](https://ubuntu.com/server/docs/installation).

Install the latest Lambda Stack by logging into Ubuntu and running:

```bash
wget -nv -O- https://lambdalabs.com/install-lambda-stack.sh | I_AGREE_TO_THE_CUDNN_LICENSE=1 sh -
```
