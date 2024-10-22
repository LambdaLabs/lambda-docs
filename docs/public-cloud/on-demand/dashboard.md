---
description: Learn how to use the Lambda Public Cloud dashboard.
tags:
  - 1-click clusters
  - on-demand cloud
---

# Using the Lambda Public Cloud dashboard

The [dashboard
:octicons-link-external-16:](https://cloud.lambdalabs.com/instances){target="_blank"}
makes it easy to get started using Lambda GPU Cloud.

From the dashboard, you can:

* [Launch, restart, and terminate
  instances](#launch-restart-and-terminate-instances)
* [Create and manage persistent storage file
  systems](#create-and-manage-persistent-storage-file-systems)
* [Add, generate, and delete SSH keys](#add-generate-and-delete-ssh-keys)
* [Generate and delete API keys](#generate-and-delete-api-keys)
* [Use the Demos feature](#use-the-demos-feature)
* [View usage](#view-usage)
* [Manage a Team](#manage-a-team)
* [Modify account settings](#modify-account-settings)

## Launch, restart, and terminate instances

### Launch instances

To launch an instance:

1.  Click [**Instances**
    :octicons-link-external-16:](https://cloud.lambdalabs.com/instances){target="_blank"}
    in the left sidebar of the dashboard.

      Then, click **Launch instance** at the top-right of the dashboard.

2.  Click the instance type that you want to launch.

3.  Click the region in which you want to launch the instance.

4.  Click the [persistent storage file
    system](#create-and-manage-persistent-storage-file-systems) that you want to
    attach to your instance.

      If you don't want to or can't attach a persistent storage file system to
      your instance, click **Don’t attach a filesystem**.

5.  Select the [SSH key](#add-generate-and-delete-ssh-keys) that you want to use
    for your instance. Then, click **Launch instance**.

    !!! tip

        You can [add additional SSH
        keys](getting-started.md#is-it-possible-to-use-more-than-one-ssh-key) to your instance once
        your instance has launched.

6.  Review the license agreements and terms of service. If you agree to them,
    click **I agree to the above** to launch your instance.

In the dashboard, you should now see your instance listed. Once your instance
has finished booting, you’ll be provided with the details needed to begin using
your instance.

!!! tip

    You can also [launch instances using the Cloud
    API](../cloud-api.md#launching-instances).

    You can also use the Cloud API to [get details of a running
    instance](../cloud-api.md#getting-details-of-a-specific-instance).

### Restart instances

Restart instances by clicking the checkboxes next to the instances you want to
restart. Then, click **Restart** at the top-right of the dashboard.

### Terminate instances

Terminate instances by clicking the checkboxes next to the instances you want to
terminate. Then, click **Terminate** at the top-right of the dashboard.

When prompted to do so, type in **erase data on instance**, then click
**Terminate instances**.

!!! tip

    You can also [terminate instances using the Cloud API](#).

### Create and manage persistent storage file systems

#### Create a persistent storage file system

To create a persistent storage file system:

1.  Click [**Storage**
    :octicons-link-external-16:](https://cloud.lambdalabs.com/file-systems){target="_blank"}
    in the left sidebar of the dashboard.

    Then, click **Create filesystem** at the top-right of the dashboard.

2.  Enter a name and select a region for your file system. Then click **Create
    filesystem**.

You should now see your persistent storage file system listed in the dashboard.

### Add, generate, and delete SSH keys

#### Add or generate an SSH key

##### Add an existing SSH key

1.  Click [**SSH keys**
    :octicons-link-external-16:](https://cloud.lambdalabs.com/ssh-keys){target="_blank"}
    in the left sidebar of the dashboard.

    Then, click **Add SSH key** at the top-right of the dashboard.

2. In the text input box, paste your public SSH key. Enter a name for your key,
   then click **Add SSH key**.

##### Generate a new SSH key

Instead of pasting your public SSH key as instructed, above, click **Generate a
new SSH key**. Type in a name for your key, then click **Create**.

The private key for your new SSH key will automatically download.

!!! tip

    You can also [use the Cloud API to add and generate SSH
    keys](../cloud-api.md#add-an-existing-ssh-key-to-your-account).

#### Delete SSH keys

Delete SSH keys by clicking **Delete** at the far-right of the SSH key you want
to delete.

### Generate and delete API keys

#### Generate API keys

Generate API keys by clicking [**API keys**
:octicons-link-external-16:](https://cloud.lambdalabs.com/api-keys){target="_blank"}
in the left sidebar of the dashboard.

Then, click **Generate API Key** at the top-right of the dashboard.

#### Delete API keys

Delete API keys by clicking **Delete** at the far-right of the API key you want
to delete.

### Use the Demos feature

Use the Demos feature by clicking [**Demos**
:octicons-link-external-16:](https://cloud.lambdalabs.com/edit-demos){target="_blank"}
in the left sidebar of the dashboard.

### View usage

View usage information by clicking [**Usage**
:octicons-link-external-16:](https://cloud.lambdalabs.com/usage){target="_blank"}
in the left sidebar of the dashboard.

### Manage a Team

Click [**Team**
:octicons-link-external-16:](https://cloud.lambdalabs.com/team){target="_blank"}
at the bottom of the left sidebar to access the Team feature.

Learn how to manage a Team by reading our FAQ on [getting started with the Team
feature](#).

### Modify account settings

Click [**Settings**
:octicons-link-external-16:](https://cloud.lambdalabs.com/settings){target="_blank"}
at the bottom of the left sidebar to modify your account settings, including
your password and payment method.
