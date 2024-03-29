---
title: "How are on-demand instances invoiced?"
type: docs
tags:
- billing
---

Billing for [on-demand instances](https://lambdalabs.com/service/gpu-cloud)
is in one-minute increments. Billing starts when an instance is launched and
the [dashboard]({{< relref "cloud-dashboard" >}}) shows the instance's status
is **Running**.

{{% alert title="Tip" color="success" %}}
The
[Cloud API's `/instances` endpoint]({{< relref "list-running-instances" >}})
will also show the instance's status is `active`.
{{% /alert %}}

Billing stops when an instance is
[terminated from the dashboard]({{< relref "cloud-dashboard#terminate-instances" >}})
or using the
[Cloud API's `/terminate` endpoint]({{< relref "terminate-instance-api" >}}).

{{% alert title="Note" color="info" %}}
You're not billed for the time an instance's status in the dashboard is
**Booting**. Similarly, you're not billed for the time an instance's status in
the dashboard is **Terminating**.
{{% /alert %}}

{{% alert title="Warning" color="warning" %}}
Be sure to terminate any instances that you're not using!

**You will be billed for all minutes that an instance is running, even if the
instance isn't actively being used.**
{{% /alert %}}

The GPU Cloud dashboard allows you to
[view your resource usage](https://cloud.lambdalabs.com/usage).

Invoices are sent weekly for the previous week's usage.

{{% alert title="Note" color="info" %}}
On-demand instances require us to maintain excess capacity at all times so we
can meet the changing workloads of our customers. For this reason, on-demand
instances are priced higher than reserved instances.

Conversely, we offer
[reserved GPU Cloud instances](https://lambdalabs.com/service/gpu-cloud/reserved)
at a significant savings over on-demand instances, since they allow us to more
accurately determine our capacity needs ahead of time.
{{% /alert %}}
