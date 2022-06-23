---
title: "Cheat sheet for using Ubuntu and Lambda Stack"
linkTitle: "Cheat sheet for using Ubuntu and Lambda Stack"
type: docs
---

Here are some useful commands to know for using Ubuntu and Lambda Stack:

## System monitoring

| Command           | Description                                                                            |
| -------           | -----------                                                                            |
| `top`             | Displays processes and other system information                                        |
| `htop`            | Displays processes and other system information, and allows interaction with processes |
| `nvtop`           | Displays status of NVIDIA GPUs                                                         |
| `nvidia-smi-pmon` | ???                                                                                    |
| `ps aux`          | Outputs list of every process on the system                                            |
| `free`            | Outputs amount of free memory and used memory                                          |

## Navigating and finding files

| Command                     | Description                                                                             |
| -------                     | -----------                                                                             |
| `pwd`                       | Outputs name of directory you're currently working in                                   |
| `ls`                        | Lists the files and directories in your current working directory                       |
| `cd NEW_DIRECTORY`          | Changes working directory to *`NEW_DIRECTORY`*                                          |
| `find . -name 'FILE_NAME*'` | Recursively finds files in the current directory that start with the name *`FILE_NAME`* |

## Disk and file systems

| Command              | Description                                                    |
| -------              | -----------                                                    |
| `df -h`              | Outputs used and available space on each file system           |
| `df -ih`             | Outputs inode usage and inode availability on each file system |
| `du -s * \| sort -n` | Outputs list of current directory's contents, sorted by size   |
| `du -sh FILE_NAME`   | Outputs size of *`FILE_NAME`*                                  |

## Networking

| Command                 | Description                                                         |
| -------                 | -----------                                                         |
| `ip address show`       | Outputs IP addresses detailed information about network interfaces  |
| `ip -br address show`   | Outputs IP addresses and brief information about network interfaces |
| `ip link`               | Outputs detailed information about network interfaces               |
| `ip -br link`           | Outputs brief information about network interfaces                  |
| `ip route`              | Outputs network routes                                              |
| `ping -c 3 IP_ADDRESS`  | Pings *`IP_ADDRESS`* three times                                    |
| `traceroute IP_ADDRESS` | Outputs route, ping time, and latency to reach *`IP_ADDRESS`*       |
| `/etc/netplan`          | Location of the network interface configurations.                   |

## Managing users

| Command                       | Description                                |
| -------                       | -----------                                |
| `group USERNAME`              | Outputs the groups that *`USERNAME`* is in |
| `sudo adduser USERNAME`       | Adds a new user to the system              |
| `sudo deluser USERNAME`       | Deletes an existing user from the system   |
| `sudo adduser USERNAME GROUP` | Adds *`USERNAME`* to *`GROUP`*             |
| `sudo deluser USERNAME GROUP` | Removes *`USERNAME`* from *`GROUP`*        |

## Firewall

| Command                  | Description                                 |
| -------                  | -----------                                 |
| `sudo iptables -L`       | Outputs current firewall rules              |
| `sudo ufw status`        | Displays the status of the firewall         |
| `sudo ufw allow SERVICE` | Allows *`SERVICE`* traffic through firewall |

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
