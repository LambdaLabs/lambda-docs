---
description: Lambda's security posture for 1-Click Clusters
tags:
  - 1-click clusters
  - security
---

# Security posture

This document describes the physical and logical security properties of the Lambda
1-Click Clusters™ (1CC) product, including the default software configuration on
cluster nodes. The following diagram illustrates the 1CC network architecture:

![1cc-network-architecture](../../assets/images/1cc-network-architecture.png)

## Compute (GPU) nodes

1CC compute nodes run on single-tenant hardware with tenant isolation enforced using
logical network segmentation. Underlying hardware resources, including GPUs, local
storage, memory, and network interfaces, aren't shared with or accessible by any
other customers.

Compute nodes live on a dedicated network segment with no inbound connectivity from
the firewall. Compute nodes can be reached either by using a management node as a jump
box or via a public reverse tunnel to a JupyterLab service running on each compute node.
Each JupyterLab instance is configured with a unique, random authentication token shared
via the Lambda Public Cloud dashboard. For more information, see the [JupyterLab security
documentation :octicons-link-external-16:](https://jupyter-server.readthedocs.io/en/latest/operators/security.html){target="_blank"}.

Customers have full control over the configuration of their compute nodes
and can reconfigure them at will.

## Management (Head) nodes

1CC management nodes run on multi-tenant hardware with tenant isolation enforced using
hardware virtualization. Underlying resources, including local storage, memory, and network
interfaces, are shared with other customers.

By default, management nodes are directly accessible over the internet via SSH and via a public
reverse tunnel to a JupyterLab service running on each management node. Each JupyterLab instance
is configured with a unique, random authentication token shared via the Lambda Public Cloud dashboard.
For more information, see
[JupyterLab's security documentation :octicons-link-external-16:](https://jupyter-server.readthedocs.io/en/latest/operators/security.html){target="_blank"}.

Customers can configure their own inbound firewall rules to expand or reduce the exposure of their
management nodes. Customers have full control over the configuration of their management nodes and
can reconfigure them at will.

## Ethernet connect

1CC compute and management nodes share a logically isolated Ethernet switching fabric. Logical
isolation ensures that customers have no interaction with each other.

## InfiniBand interconnect

1CC compute nodes share a specially isolated InfiniBand fabric that ensures that customer traffic
only ever transits physical IB links dedicated to that customer, ensuring complete isolation of
customer InfiniBand traffic.

## Persistent file storage

All 1CC compute and management nodes have pre-configured access to a customer-specific portion of a
multi-tenant persistent file storage system. The storage system is on an isolated network accessible
only to management and compute nodes. All data on the storage system is encrypted at rest using industry
standard algorithms and parameters.

## Lambda employee access

Logical and physical access to 1CC infrastructure, such as network and storage solutions, is limited
to Lambda employees with a specific need for access. Underlying 1CC infrastructure is monitored for
security, utilization, performance, and reliability. Lambda employees do not access customer environments
without customers' express authorization. Customers are responsible for all security instrumentation and
monitoring of their management and compute nodes.

## Physical security

1CC infrastructure is located in secure data centers with the following access controls:

*   Qualified in-house security personnel on site 24x7x365
*   CCTV surveillance, with a minimum 90 days of retention
*   Multiple security checkpoints:
    *   Controlled fenced access into data center property
    *   Lobby mantraps in data center hallway
    *   2FA Access Control (Biometric and RFID badge) into data hall
    *   2FA Access Control (Biometric and RFID badge) into secured cage

Authorized Lambda employees may access data centers for maintenance, upgrades, or other hardware
infrastructure work.
