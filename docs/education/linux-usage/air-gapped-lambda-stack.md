---
description: Learn how to install Lambda Stack on air-gapped servers and workstations.
tags:
  - api
  - llama
  - llm
---

# Installing Lambda Stack on air-gapped servers and workstations

## Introduction

[Lambda Stack](https://lambdalabs.com/lambda-stack-deep-learning-software){ .external target="_blank" }
can be installed on air-gapped servers and workstations (systems), that is,
servers and workstations that are physically disconnected from a network.
Air-gapped systems are commonly used in secure government facilities, national
laboratories, and financial institutions.

This guide shows you how to:

1. [Download the complete Lambda Stack repository](#download-the-lambda-stack-repository).
1. [Copy the Lambda Stack repository to a USB stick](#).
1. [Copy the Lambda Stack repository to an air-gapped system](#copy-the-lambda-stack-repository-to-the-air-gapped-system).
1. [Configure the air-gapped system to use the Lambda Stack repository](#configure-the-air-gapped-system-to-use-the-lambda-stack-repository).
1. Install Lambda Stack on the air-gapped system.

For this guide, you need:

- An
  [Ubuntu Desktop](https://ubuntu.com/download/desktop){ .external target="_blank"}
  system connected to the internet (referred to in this tutorial as the
  "connected system") that you can use to:
    - Download the Lambda Stack repository.
    - Copy the repository to a USB stick.
- A USB stick (USB drive or thumb drive) with a capacity of at least 8 GiB. The
  USB stick should be formatted with an `ext4` filesystem.

This guide assumes you already have Ubuntu installed on your air-gapped system
and only seek to install Lambda Stack. This guide also assumes that you know how
to mount and unmount filesystems.

## Download the Lambda Stack repository

1. On the connected system, install `apt-mirror` by running the following
   command in a terminal:

    ```bash
    sudo apt update && sudo apt -y install apt-mirror
    ```

1. Run the following command to configure `apt-mirror` to download the Lambda
   Stack repository:

    ```bash
    cat <<EOF | sudo tee /etc/apt/mirror.list
    set nthreads 20
    deb http://archive.lambdalabs.com/ubuntu jammy main
    clean http://archive.lambdalabs.com/ubuntu
    EOF
    ```

1. Download the Lambda Stack repository by running:

    ```bash
    sudo apt-mirror
    ```

    The repository is downloaded once you see:

    ```{ .text .no-copy}
    Post Mirror script has completed. See above output for any possible errors.
    ```

1. Download the Lambda Stack repository configuration package, and the Lambda
   Stack installation script, to your home directory by running:

    ```bash
    wget -P ~/ https://lambdalabs.com/static/misc/lambda-stack-repo.deb &&
    wget -P ~/ https://lambdalabs.com/install-lambda-stack.sh
    ```

## Copy the Lambda Stack repository to a USB stick

1. Plug the USB stick into the connected system. Mount the filesystem if it's
   not automatically mounted and use `cd` change into the filesystem mount
   point.

1. Copy the Lambda Stack repository to the USB stick by running:

    ```bash
    sudo rsync -av --progress /var/spool/apt-mirror/mirror/archive.lambdalabs.com .
    ```

1. Copy the Lambda Stack repository configuration package, and the Lambda
   Stack installation script, to the USB stick by running:

    ```bash
    sudo cp ~/lambda-stack-repo.deb ~/install-lambda-stack.sh .
    ```

1. Unmount the filesystem, then unplug the USB stick.

## Copy the Lambda Stack repository and other files to the air-gapped system

1. Plug the USB stick into the air-gapped system. Mount the filesystem if it's
   not automatically mounted and use `cd` to change into the filesystem mount
   point.

1. Copy the Lambda Stack repository from the USB stick to the air-gapped system
   by running:

    ```bash
    sudo rsync -av --progress archive.lambdalabs.com /var/cache/apt/archives
    ```

1. Copy the Lambda Stack repository configuration package, and the Lambda Stack
   installation script, from the USB stick to the air-gapped system by running:

    ```bash
    cp lambda-stack-repo.deb install-lambda-stack.sh ~/
    ```

1. Unmount the filesystem, then unplug the USB stick.

## Configure the air-gapped system to use the local Lambda Stack repository

1. Modify the `install-lambda-stack.sh` (the Lambda Stack installation script)
   in your home directory by removing the lines:

    ```{ .text .no-copy }
    # Install Lambda repository
    LAMBDA_REPO=$(mktemp)
    stderr "Installing Lambda Stack Repository."
    wget --quiet -O"${LAMBDA_REPO}" "${LAMBDA_REPO_URL}"
    sudo dpkg -i "${LAMBDA_REPO}"
    rm -f "${LAMBDA_REPO}"
    ```

1. Install the Lambda Stack repository configuration package by running:

    ```bash
    sudo apt -y install ~/lambda-stack-repo.deb
    ```

1. Run the following command to configure the air-gapped system to use the local
   Lambda Stack repository:

    ```bash
    cat <<EOF | sudo tee /etc/apt/sources.list.d/lambda-repository.list &&
    deb file:///var/cache/apt/archives/archive.lambdalabs.com/ubuntu jammy main
    # deb-src file:///var/cache/apt/archives/archive.lambdalabs.com/ubuntu jammy main
    EOF
    cat <<EOF | sudo tee /etc/apt/preferences.d/lambda-repository
    Package: *
    Pin: release o=lambdalabs.com
    Pin-Priority: 1001
    EOF
    ```

## Install Lambda Stack on the air-gapped system

- Run:

    ```bash
    bash ~/install-lambda-stack.sh
    ```

When the installation completes, restart your system.
