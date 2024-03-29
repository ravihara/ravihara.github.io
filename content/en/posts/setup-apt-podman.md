+++
title = "Podman for Debian based systems"
description = "Script to setup podman repository for Debian based systems"
date = 2022-08-31T08:30:24+05:30
featured = true
draft = false
comment = true
toc = true
reward = true
diagram = true
categories = [
  "Linux",
  "SysAdmin"
]
tags = [
  "linux",
  "debian",
  "ubuntu",
  "podman"
]
series = [
  "Reference"
]
+++

Here is a link to the [bash-script](https://gist.github.com/ravihara/1a612a6f054e2878de8a9905824891da) to configure apt
repository for [Podman](https://podman.io/). The script is available as a GitHub Gist. Hence, you are requested to open
the link to the bash-script and download the script (or, copy-paste the raw content into a file using a text-editor)
to use it directly from your Debian-based system such as _Debian_, _Ubuntu_ or _LinuxMint_ (including _LMDE_).

The script will setup the proper apt repository configuration file under `/etc/apt/sources.list.d`, installs `podman` and
`buildah`. It will also add the local `libpod.conf` file for the administrator user (_i.e., non-root user with sudo previleges_),
running the script.

## Prerequisites

The script should be run using an administrator account (i.e., with `sudo` access).
