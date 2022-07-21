---
title: "Upgrade software in Ubuntu"
type: docs
---

Ubuntu regularly checks for new versions of installed software. If a new
version of installed software is found, Ubuntu automatically downloads and
installs it.

To manually check for and install new versions of software in Ubuntu, run the
following command in a terminal:
`sudo apt-get update && sudo apt-get upgrade`.

{{% alert title="Note" color="info" %}}
Software that was installed from outside of Ubuntu will not be upgraded
automatically or by using the above command. Examples of software installed
from outside of Ubuntu include:

- Software downloaded from a Git repository
- Software installed from a tarball
- Software built from source code
{{% /alert %}}
