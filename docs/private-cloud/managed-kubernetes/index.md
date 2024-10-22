---
tags:
- private cloud
- managed kubernetes
---

# Overview

During the Private Cloud reservation process, you can choose to configure your
cluster as a Managed Kubernetes cluster. In this configuration, Lambda manages
your cluster's underlying environment, and you interact with the cluster through
a browser-based Kubernetes administration UI and the Kubernetes API.

This document outlines the standard configuration for a Managed Kubernetes
cluster in Lambda Private Cloud.

## Hardware

Lambda Private Cloud provides single-tenant clusters that are isolated from
other clusters. The hardware details for your specific cluster depend on what
you chose when reserving your cluster. Each cluster includes at least three
control (CPU) nodes for cluster administration and job scheduling.

## Software

Your Managed Kubernetes deployment is configured to use Rancher with Rancher
Kubernetes Engine 2 (RKE2).

-  [Rancher](https://ranchermanager.docs.rancher.com/){ .external target="_blank" }
    provides a web UI for monitoring and managing aspects of your Kubernetes
    cluster. Rancher also provides your cluster's Kubenetes API server.
-  [RKE2](https://docs.rke2.io/){ .external target="_blank" } is a fully conformant
    Kubernetes distribution focused on security and compliance.

## Cluster management

### Rancher dashboard

The Rancher dashboard serves as the main UI for your Managed Kubernetes cluster.
After you set up your SSL VPN connection, you can access your dashboard at
[https://10.141.3.1](https://10.141.3.1){ .external target="_blank" }. The
login details for your dashboard can be found in your 1Password vault.

For details on setting up your SSL VPN connection, see
[Getting started &gt; Establishing a secure connection to your cluster](getting-started.md#establishing-a-secure-connection).

### Kubernetes API

The Kubernetes API is available at `https://10.141.0.250:6443` through your SSL
VPN connection. You can obtain your `kubeconfig` file from the Rancher
dashboard. For details, see
[Getting started &gt; Accessing the Kubernetes API](getting-started.md#accessing-the-kubernetes-api).

## Storage

In the default cluster configuration, your cluster comes with three types of
storage, each with its own performance and access characteristics:

-  Common (Longhorn)
-  Shared workload (Intelliflash)
-  Scratch/HPC (directly attached local storage)

Each type is mapped to a corresponding storage class in Kubernetes:

-  `longhorn`
-  `intelliflash`
-  `local-path`

!!! note

    Each cluster also includes a `local-storage` storage class that Lambda
    uses to route monitoring metric replication. You can safely ignore this
    class&mdash;`local-path` is the dynamic provisioner routed to the fast
    NVMe arrays.

### Common

Longhorn is configured by default to pool the local drives on your control
nodes, and is best used for everyday storage needs such as shared home
directories, configuration files, code checkouts, metrics output, and so on. It
provides fault tolerance through replication and is available as single or
shared PVCs.

!!! note

    If desired, you can extend this common pool to your compute nodes.
    However, the other storage tiers provide better performance for compute
    workloads.

### Shared workload

Intelliflash is a dedicated NAS device with its own high availability (HA),
replication, and deduplication system. It is capable of much higher performance
and is uplinked into the inband network via 4x100Gb links. It is also available
as single or shared PVCs.

We recommend the Intelliflash as your primary workload storage if you require
shared PVCs.

### Scratch/HPC

The direct-attached local storage on your compute nodes provides the highest
performance of the storage tiers available in your cluster. However, this
storage is limited to single attachment PVCs only&mdash;that is, each pod will
provision their own volume out of the local drives on that compute node and
attach to only that volume.

This direct-attached storage can be used for high-speed scratch space during
jobs. Alternatively, if you construct your jobs to do their own data sharding,
you can maximize performance by duplicating your whole dataset to each pod's
local volume.

## Networking

All cluster networking is directly attached using native routing, with no
overlays or tunnels.

### Firewall defaults

Your cluster uses FortiGate, Fortinet's Next-Generation Firewall (NGFW), to
provide network protection and remote access services. All traffic into and out
of the cluster is mediated by the FortiGate firewall appliance. By default, your
firewall is deployed in high availability (HA) mode with the following rules:

| Direction      | Default behavior                                           |
| -------------- | ---------------------------------------------------------- |
| Ingress        | Block, unless it's an SSL VPN connection                   |
| Egress         | Allow all traffic going out to Internet                    |

### Inband

Your Inband network is on `10.141.0.0/22`, as follows:

-  `10.141.0.1`: Firewall gateway
-  `10.141.0.250`: Kubernetes API virtual IP
-  `10.141.0.251-253`: Control nodes
-  `10.141.1.1-8`: Compute nodes

### Infiniband

The Infiniband network is meshed between the compute nodes on dedicated
switches.

### Kubernetes networks

The container network is on `10.42.0.0/16`, and the service network is on
`10.43.0.0/16`.

### Load balancer services

The Managed Kubernetes system reserves part of your inband network
for use as an IPAM-managed pool of LoadBalancer service IPs. The first three
of these IPs are allocated to the following services:

-  `10.141.3.0`: Ingress controller
-  `https://10.141.3.1:443`: Rancher dashboard
-  `http://10.141.3.2:80`: Hubble UI

When you apply a new LoadBalancer service, the system automatically
allocates one of the remaining IPs (`10.141.3.4-254`) to the service.

## Next steps

For details on accessing your cluster, as well as a walkthrough of the
Rancher dashboard, see the [Managed Kubernetes getting started](getting-started.md)
for Private Cloud.
