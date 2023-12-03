---
title: "How do I use tmux to keep programs running?"
type: docs
tags:
- SSH
- Ubuntu
---

You can use the program `tmux` to keep programs and commands running after you
log out of a server or
[Cloud instance](https://lambdalabs.com/service/gpu-cloud). This is useful,
for example, when
[copying a large amount of data directly between servers or Cloud instances using `rsync`](https://rsync-to-copy-and-synchroniz.lambda-docs.pages.dev/linux/rsync-to-copy-and-synchronize/).

To use `tmux`, first install `tmux` on the server or Cloud instance by
running:

```bash
sudo apt -y update && sudo apt -y install tmux
```

Then, run the command `tmux` to create a `tmux` session. In the `tmux`
session, run commands like you normally would.

When you're ready to log out of the server or Cloud instance, press **Ctrl** +
**b**, then press **d**. You'll "detach" from the `tmux` session and can
safely log out of the server or Cloud instance.

To reattach to the `tmux` session, first log into the server or Cloud
instance. Then, run `tmux attach`.

{{% alert title="Tip" color="success" %}}
`tmux` has many useful features, such as the ability to create multiple panes
to run commands in. See the [`tmux` wiki](https://github.com/tmux/tmux/wiki)
to learn more about the features and customization options available in
`tmux`.
{{% /alert %}}
