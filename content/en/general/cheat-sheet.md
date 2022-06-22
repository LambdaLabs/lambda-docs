---
title: "Cheat sheet for using Ubuntu and Lambda Stack"
linkTitle: "Cheat sheet for using Ubuntu and Lambda Stack"
type: docs
---

Here are some useful commands to know for using Ubuntu and Lambda Stack:

## System monitoring

| Command         | Description |
| -------         | ----------- |
| top             | XXX         |
| htop            | XXX         |
| nvtop           | XXX         |
| nvidia-smi-pmon | XXX         |
| ps aux          | XXX         |
| free            | XXX         |

## Navigating and finding files

| Command                  | Description                                   |
| -------                  | -----------                                   |
| pwd                      | present working directory                     |
| ls                       | see files in current directory                |
| cd                       | change to a new directory                     |
| find . -name 'example\*' | find files that start with the name 'example' |

## Disk and file systems

| Command               | Description                                                 |
| -------               | -----------                                                 |
| df -h                 | Show much space is in all file systems                      |
| df -ih                | Show how many inodes (number of files) on each file system  |
| du -s * \| sort -n    | Show the largest files/directories in the current directory |
| du -sh example.tar.gz | Show how large a file 'example.tar.gz' is                   |

## Networking

| Command             | Description                                            |
| -------             | -----------                                            |
| ip address show     | Long list of information about interfaces              |
| ip -br address show | more brief version of the commands                     |
| ip link             | show links                                             |
| ip -br link         | brief view of the links                                |
| ip route            | show routes on your system                             |
| ping -c 3 10.0.0.1  | Ping the IP address 10.0.0.1 three times               |
| traceroute 10.0.0.1 | Check the route and performance to IP address 10.0.0.1 |
| /etc/netplan        | Location of the network interface configurations.      |

## Managing users

| Command                         | Description                                             |
| -------                         | -----------                                             |
| group <username>                | Check if there is a ‘user’ and which groups they are in |
| sudo adduser <username>         | Adding a new user                                       |
| sudo deluser <username>         | Deleting a existing username                            |
| sudo adduser <username> <group> | Add a ‘user’ to a ‘group’ both that existing            |
| sudo deluser <username> <group> | Delete/remove a ‘user’ from a ‘group’                   |

## Firewall

| Command            | Description                |
| -------            | -----------                |
| sudo iptables -L   | List iptables rules        |
| sudo ufw status    | Show the status of the ufw |
| sudo ufw allow ssh | XXX                        |

## Linux and Lambda Stack upgrades and packaging

| Command                     | Description                                           |
| -------                     | -----------                                           |
| sudo apt-get update         | Update the list of packages from repository (sync up) |
| apt apt list --upgradeable  | list upgradable packages (after the update)           |
| sudo apt-get upgrade        | Upgrade packages                                      |
| sudo apt-get dist-upgrade   | more aggressive upgrade                               |
| dpkg -L <installed package> | List the contents of a given packages                 |
| dpkg -S <full path to file> | show the package a file came from                     |
| dpkg --list                 | Show the list of packages                             |
| apt list --installed        | XXX                                                   |

## Linux/Ubuntu security

| Command      | Description     |
| -------      | -----------     |
| iptables -L  | XXX             |
| /etc/sudoers | XXX             |
| visudo       | to edit sudoers |

## Linux/Ubuntu NVIDIA GPU cheat sheets

| Command                                    | Description                                                                  |
| -------                                    | -----------                                                                  |
| nvtop                                      | watch the GPUs utilization and memory utilization                            |
| nvidia-smi                                 | see the driver version (supported CUDA and usage)  note the persistence mode |
| nvidia-smi -q                              | gives more detailed information for each GPU                                 |
| nvidia-smi topo -m                         | To see the NVLink connection topology                                        |
| nvidia-smi nvlink -s                       | XXX                                                                          |
| grep "kernel: NVRM: Xid" /var/log/kern.log | XXX                                                                          |

## Containers and Virtual Environments:

### Docker

Install docker (with Lambda Stack installed):
sudo apt-get install -y docker.io nvidia-container-toolkit
sudo systemctl daemon-reload
sudo systemctl restart docker

### Finding many docker images for Deep Learning

https://catalog.ngc.nvidia.com/

### Pull a docker image

```
sudo docker pull <image name>
sudo docker pull nvcr.io/nvidia/tensorflow:22.05-tf1-py3
sudo docker pull nvcr.io/nvidia/pytorch:22.05-py3
```

### Run Docker (it will pull the image if not found)

sudo docker run --gpus all -it --rm nvcr.io/nvidia/pytorch:22.05-py3

### List running docker containers

docker ps

### List docker images
docker images

### Mount a directory in a docker image on start up

sudo docker run --gpus all -it --rm -v `pwd`/data:/data/ nvcr.io/nvidia/pytorch:22.05-py3

You can add a command to the end of the line: ls, python code
Mounts the ‘data’ directory from the current directory into the container as /data.

### Copy a file from the host to the container

docker cp input.txt container_id:/input.txt

### Copy a file from the container to the local file system

docker cp container_id:/output.txt output.txt

### Copy a group of files in the ‘data’ directory to the container

docker cp data/. container_id:/target

### Copy a group of files in the container ‘output’ directory to local host

docker cp container_id:/output/. target


- Python venv
- viritualenv
- Anaconda
- Kubernetes



For servers with IPMI:

```
sudo apt-get install ipmitool
sudo ipmitool user list 1
        Confirm that ADMIN is 2
sudo ipmitool user set password 2
        Then, enter the new password twice.
```
