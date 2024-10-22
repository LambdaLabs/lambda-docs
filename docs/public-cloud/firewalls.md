---
description: Use the Firewall feature to help keep your instances and management nodes secure.
tags:
  - 1-click clusters
  - on-demand cloud
---

# Firewalls

You can restrict incoming traffic to your instances by creating firewall rules
on the [Firewall page](https://cloud.lambdalabs.com/firewall) in the Lambda
Cloud dashboard. Currently, the rules you create apply to all instances
associated with your account except those in the Texas, USA (us-south-1)
regions.

By default, Lambda allows only incoming ICMP traffic and traffic on port 22
(SSH).

## Managing your firewall rules

### Creating a firewall rule

You can create up to 20 firewall rules to restrict incoming traffic to your
instances. If you create more than 20 rules, new instances you create might not
launch. Also, it’s possible that not all of your rules will be active, which
might leave your instances insecure.

!!! warning

    Each port you open increases the attack surface of your instances. Make
    sure to vet any services you run on your instances, and be judicious about
    exposing new ports. If possible, restrict your incoming traffic to known
    sources.

To create a new firewall rule:

1. Navigate to the [Firewall page](https://cloud.lambdalabs.com/firewall) in
   the Lambda Cloud dashboard.
1. In the **Inbound Rules** section, click **Edit** to begin creating a rule.
1. In the dropdown menu under **Type**, select the type of rule you want to
   create:

      * Select **Custom TCP** to manually configure a rule to allow incoming TCP
        traffic.
      * Select **Custom UDP** to manually configure a rule to allow incoming UDP
        traffic.
      * Select **HTTPS** to automatically configure a rule to allow incoming
        HTTPS traffic.
      * Select **SSH** to automatically configure a rule to allow incoming SSH
        traffic.
      * Select **All TCP** to automatically configure a rule to allow all
        incoming TCP traffic.
      * Select **All UDP** to automatically configure a rule to allow all
        incoming UDP traffic.

1.  In the **Source** field, enter an IP address or range to restrict incoming
   traffic to only that set of sources. To allow incoming traffic from any source,
   enter `0.0.0.0/0`.

      * Click the 🔎 to automatically enter your current IP address.
      * Enter a single IP address—for example, `203.0.113.1`.
      * Enter an IP address range in CIDR notation—for example, `203.0.113.0/24`.

1. If you chose **Custom TCP** or **Custom UDP** as your rule type, enter a
   port range in the **Port range** field. You can enter either a single port
   (for example,`8080`) or a range of ports (for example, `8080-8081`).
1. Optionally, add a description for the rule in the **Description** field.
1. Click **Update** to apply your changes.

### Updating a firewall rule

To update a firewall rule:

1. Navigate to the [Firewall page](https://cloud.lambdalabs.com/firewall) in
   the Lambda Cloud dashboard.
2. In the **Inbound Rules** section, click **Edit**.
3. Find the rule you want to update and modify it as needed.
4. Click **Update** to apply your changes.

### Deleting a firewall rule

To delete a firewall rule:

1. Navigate to the [Firewall page](https://cloud.lambdalabs.com/firewall) in
   the Lambda Cloud dashboard.
2. In the **Inbound Rules** section, click **Edit**.
3. Click the **x** next to any rule you want to delete.
4. Click **Update** to apply your changes.

!!! warning

    If you delete the rule that allows incoming traffic to port TCP/22, you
    won’t be able to access your instances using SSH.

## Allowing or restricting ICMP traffic

To allow or restrict ICMP traffic, visit the
[Firewall page](https://cloud.lambdalabs.com/firewall) in the Lambda Cloud
dashboard and toggle **Allow ICMP traffic (ping)** in the General Settings
section.

!!! note

    For network diagnostic tools such as `ping` and `mtr` to be able to
    reach your instances, you need to allow incoming ICMP traffic.
