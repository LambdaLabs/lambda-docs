# Basic Linux commands and system administration

## Importing SSH keys from GitHub accounts

To import an SSH key from a GitHub account and add it to your server (or Lambda
GPU Cloud on-demand instance):

1.  Using your existing SSH key, SSH into your server.

    Alternatively, if you're using an on-demand instance, open a terminal in
    [Jupyter
    Notebook](https://docs.lambdalabs.com/cloud/open-jupyter-notebook/).

1.  Import the SSH key from the GitHub account by running:

    ```bash
    ssh-import-id gh:USERNAME
    ```

    Replace `USERNAME` with the GitHub account's username.

If the SSH key is successfully imported, `ssh-import-id` will output a message similar to:

```
2023-08-04 15:03:52,622 INFO Authorized key ['256', 'SHA256:C6pl0q4evVYZWcyByVF69D6fdbdKa7F8ei8V2F/bTW0', 'cbrownstein-lambda@github/67649580', '(ED25519)']
2023-08-04 15:03:52,623 INFO [1] SSH keys [Authorized]
```

If the SSH key _isn't_ successfully imported, `ssh-import-id` will output a
message similar to:

```
2023-08-04 15:06:36,425 ERROR Username "fake-cbrownstein-lambda" not found at GitHub API. status_code=404 user=fake-cbrownstein-lambda
```

## Using rsync to copy and synchronize files

`rsync` is a tool that you can use to copy files between your computer and a
remote server.

`rsync` can also be used to copy files directly between remote servers,
bypassing your computer entirely.

!!! tip

    `rsync` is useful for copying files between Cloud persistent storage file
    systems in different regions.

!!! note

    `rsync` copies files using SSH. For this reason, to copy files between your
    computer and a remote server, you need to be able to SSH into the remote
    server.

    To use `rsync` to copy files between remote servers directly, you need to be
    able to SSH into the remote servers using public key authentication with an
    SSH agent.

### Copy files between your computer and a remote server

To copy files from your computer to a remote server using`rsync`, run:

```bash
rsync -av --info=progress2 FILES USERNAME@SERVER-IP:REMOTE-PATH
```

Replace `FILES` with the files you want to copy to the remote server.
Alternatively, you can specify a directory.

Replace `USERNAME` with your username on the remote server.

Replace `SERVER-IP` with the IP address of the remote server.

Replace `REMOTE-PATH` with the directory into which you want to copy files.

In the below example, `rsync` was used to copy the local directory
`rsync_example_dir`, containing a single empty file named `EXAMPLE_FILE`, into
the home directory of the user `ubuntu` on a remote server with the IP address
`146.235.208.193`.

```
$ rsync -a --progress rsync_example_dir ubuntu@146.235.208.193:~
sending incremental file list
rsync_example_dir/
rsync_example_dir/EXAMPLE_FILE
              0 100%    0.00kB/s    0:00:00 (xfr#1, to-chk=0/2)
```

### Copy files directly between remote servers

!!! note

    To copy files directly between remote servers using `rsync`, you must use
    public key (rather than password) authentication for SSH with an SSH agent.

    You can add your private key to the SSH agent by running:

    ```bash
    ssh-add SSH-PRIVATE-KEY
    ```

    Replace `SSH-PRIVATE-KEY` with the path to your SSH private key, for example,
    `~/.ssh/id_ed25519`.

    You can confirm your key was added to the SSH agent by running:

    ```bash
    ssh-add -L
    ```

    Your public key will be listed in the output.

To copy files directly between remote servers using `rsync`, first SSH into the
server you want to copy files _from_ by running:

```bash
ssh -A USERNAME-1@SERVER-IP-1
```

Replace `SERVER-IP-1` with the IP address of the server you want to copy files
from, referred to below as _Server 1_.

Replace `USERNAME-1` with your username on _Server 1_.

!!! tip

    It's recommended to run the `rsync` command, below, in a `tmux` or `screen`
    session. This way, you can log out of _Server 1_ and the `rsync` command
    will continue to run.

Then, on _Server 1_, run:

```bash
rsync -av --info=progress2 FILES USERNAME-2@SERVER-IP-2:REMOTE-PATH
```

Replace `SERVER-IP-2` with the IP address of the server you want to copy files
_to_, referred to below as _Server 2_.

Replace `FILES` with the files (or directory) you want to copy to _Server 2_.

Replace `USERNAME-2` with your username on _Server 2_.

Replace `SERVER-IP-2` with the IP address of _Server 2_.

Replace `REMOTE-PATH` with the directory into which you want to copy files.

## Preventing system from suspending or sleeping

To prevent your system from going to sleep or suspending, run:

```bash
sudo systemctl mask hibernate.target hybrid-sleep.target \
suspend-then-hibernate.target sleep.target suspend.target
```

## Creating additional user accounts in Ubuntu Desktop

By having their own accounts, users can manage their own files, datasets, and
programs, as well as manage their own [Python virtual
w[conda virtual
environments](https://docs.lambdalabs.com/linux/create-conda-virtual-environment/),
and [Docker
containers](https://docs.lambdalabs.com/linux/install-docker-run-container/).

Also, by having additional accounts, you can assign system administrator
privileges to other users.

You can add user accounts from the **Users** panel in **GNOME Settings**:

1. Press the ++super++ key on your keyboard to open the **Activities** overview. Then, type `users`.

    !!! tip

        The Super key on your keyboard is located between the ++ctrl++ and
        ++alt++ keys.


1. Click **Users** to open the **Users** panel in **GNOME Settings**.
1. Click **Unlock** at the top of the panel, then click **Add User**.
1. For **Account Type**, choose either **Standard** or **Administrator**.
   - **Standard** account users can create, modify, and delete only their own
     files, not system files or other users' files. Standard account users also
     can change their own settings only, not system settings or other users’
     settings.
   - **Administrator** account users have the same privileges as standard
     account users. However, administrator account users can also create,
     modify, and delete system files and other users' files. Administrator
     account users can also change their system settings and other users'
     settings.
1. For **Full Name**, enter the user's full name, that is, their "real" name or
   name they use to identify themselves.
1. For **Username**, enter the name the user will use to log into the system.
   This name will also be the name of the user's home directory, for example,
   `/home/username`.
1.  Under **Password**, choose either **Allow user to set a password when they
    next login**, or **Set a password now**. If you choose to set a password
    now, in the **Password** field, enter a custom password, or click the
    Settings button to automatically generate a password.
1. Click **Add** at the top of the dialog to add the user.

## Creating encrypted data drives

!!! warning

    **These instructions erase any existing data on the drive you're
    encrypting!**

    Before proceeding with these instructions, back up all data that you want to keep.

    Make sure you correctly choose the drive you want to encrypt.

To create an encrypted data drive that automatically mounts when you boot your system:

1. Identify the drive you want to encrypt by running:

    ```bash
    lsblk -e 7 -o NAME,VENDOR,MODEL,SIZE,TYPE,MOUNTPOINTS
    ```

    The output will be similar to:

    ```{ .text .no-copy }
    NAME                      VENDOR   MODEL         SIZE TYPE  MOUNTPOINTS
    vda                       0x1af4                  25G disk
    ├─vda1                                             1M part
    ├─vda2                                             2G part /boot
    └─vda3                                            23G part
    └─ubuntu--vg-ubuntu--lv                       11.5G lvm  /
    vdb                       0x1af4                   1G disk
    ```

    The above example output shows 2 drives: `vda` and `vdb`.

    !!! warning

        **Be 100% sure you're identifying the correct drive! Look especially at
        the mountpoints to make sure they're not system mounts such as `/`,
        `/home`, and `/var`.**

        **Any existing data on the drive is unrecoverable once the drive is
        encrypted!**

1. Partition the drive you want to encrypt by running:

    ```bash
    sudo parted -s /dev/DRIVE mklabel gpt mkpart PARTITION-TO-ENCRYPT 0% 100%
    ```

    Replace `DRIVE` with the drive you want to encrypt.

    Replace `PARTITION-TO-ENCRYPT` with the label (name) you want to assign to the
    partition you're creating.

    The above command creates a single partition that uses the entire capacity
    of the drive.

    Obtain the name of the partition by running:

    ```bash
    lsblk /dev/DRIVE
    ```

    Replace `DRIVE` with the drive you're encrypting.

    You'll see output similar to:

    ```
    NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
    vdb    252:16   0    1G  0 disk
    └─vdb1 252:17   0 1022M  0 part
    ```

    In the above example output, the newly created partition is `vdb1`.

1. Install cryptsetup by running:

    ```bash
    sudo apt update && sudo apt -y install cryptsetup
    ```

    Then, encrypt the partition you created in the previous step by running:

    ```bash
    sudo cryptsetup --verbose --verify-passphrase luksFormat /dev/PARTITION
    ```

    Replace `PARTITION` with the name of the partition you created in the
    previous step.

    You'll be prompted with:

    ```{ .text .no-copy }
    WARNING!
    ========
    This will overwrite data on /dev/vdb1 irrevocably.

    Are you sure? (Type 'yes' in capital letters):
    ```

    At the prompt, follow the instruction to confirm that you want to proceed.

    You'll be asked to enter a passphrase, then you'll be asked to verify your passphrase.

    Once encryption of the partition has finished, you'll see `Command successful.`

    !!! warning

        **Make sure not to lose your passphrase! Your passphrase can't be
        recovered if it's lost and, unless you also create a keyfile (optional),
        it's impossible to decrypt your data without your passphrase.**

    !!! tip

        In addition to having a passphrase to decrypt your data, you can create
        a keyfile to automatically decrypt your data when you boot your system.

        To create a keyfile:

        1. Run `sudo dd if=/dev/urandom of=PATH-TO-KEYFILE bs=1024 count=4`.

            Replace `PATH-TO-KEYFILE` with the path to the keyfile you're creating.

            For security, it's recommended to create the keyfile in your `/root`
            directory, for example, `/root/keyfile`. Also, restrict permissions
            to the keyfile by running `sudo chmod 600 /root/keyfile`.

        1. Add the keyfile to the encrypted partition by running:

            ```bash
            sudo cryptsetup luksAddKey /dev/PARTITION PATH-TO-KEYFILE
            ```

            Replace `PARTITION` with the name of the partition you just
            encrypted.

            Replace `PATH-TO-KEYFILE` with the path to the keyfile you just
            created.

            When prompted to do so, enter the passphrase you used to encrypt the
            partition.

1. Unlock the encrypted partition by running:

    ```bash
    sudo cryptsetup open /dev/PARTITION PARTITION-NAME
    ```

    Enter your passphrase when prompted to do so.

    Replace `PARTITION` with the name of the partition you just encrypted.

    Replace `PARTITION-NAME` with a name you want to use for the partition while it's decrypted.

1. Create a file system on the partition by running:

    ```bash
    sudo mkfs.ext4 /dev/mapper/PARTITION-NAME
    ```

    Replace `PARTITION-NAME` with the name you gave the partition in the
    previous step.

1. Obtain the UUID of your encrypted partition by running:

    ```bash
    sudo blkid -c /dev/null | grep /dev/PARTITION | cut -d ' ' -f 2
    ```

    Replace `PARTITION` with the name of your partition (vdb1 in the above
    examples).

    The command output will look similar to:

    ```{ .text .no-copy }
    UUID="908f6b4c-3103-4ad3-96e6-96babe8fc8db"
    ```

    Then, create the file `/etc/crypttab` and add the line:

    ```
    PARTITION-NAME UUID=PARTITION-UUID KEYFILE luks
    ```

    Replace `PARTITION-NAME` with the name you gave the partition in step 4.

    Replace `UUID` with the partition's UUID.

    Replace `KEYFILE` with `none` if you didn't create a keyfile. If you did
    create a keyfile, replace `KEYFILE` with the path to your keyfile.

    The complete line will look similar to:

    ```{ .text .no-copy }
    encrypted-drive UUID=908f6b4c-3103-4ad3-96e6-96babe8fc8db none luks
    ```

1. Create a mount point for your encrypted drive by running:

    ```bash
    sudo mkdir --parents MOUNT-POINT
    ```

    Replace `MOUNT-POINT` with the path you want your encrypted drive to be
    accessible at, for example, `/mnt/encrypted-drive`.

    Add to `/etc/fstab` the line:

    ```
    /dev/mapper/PARTITION-NAME MOUNT-POINT ext4 defaults 0 2
    ```

    Replace `PARTITION-NAME` with the name you gave your partition.

    Replace `MOUNT-POINT` with the mount point you created.

    The complete line will look similar to:

    ```{ .text .no-copy }
    /dev/mapper/encrypted-drive /mnt/data ext4 defaults 0 2
    ```

1. Reboot your system and when prompted to do so, enter the passphrase for your
   encrypted partition. Your encrypted drive will be accessible at the mount
   point you created.

!!! tip

    To create a directory on your encrypted drive that your normal, unprivileged
    (non-root) account can create files and directories in, run:

    ```bash
    sudo mkdir MOUNT-POINT/USER-DIRECTORY && \
    sudo chown $(id -u):$(id -g) MOUNT-POINT-DIRECTORY/USER-DIRECTORY
    ```

    Replace `MOUNT-POINT` with the mount point you created in the previous step.

    Replace `USER-DIRECTORY` with a name for the directory you want to create.

    The complete command will look similar to:

    ```{ .text .no-copy }
    sudo mkdir /mnt/encrypted-drive/ubuntu && \
    sudo chown $(id -u):$(id -g) /mnt/encrypted-drive/ubuntu
    ```
