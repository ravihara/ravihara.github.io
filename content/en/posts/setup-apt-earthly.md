+++
title = "Earthly for Debian based systems"
description = "Script to setup earthly repository for Debian based systems"
date = 2022-09-06T08:30:24+05:30
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
  "earthly",
  "cicd"
]
series = [
  "Reference"
]
+++

Here is a link to the [bash-script](https://gist.github.com/ravihara/624b64eb63c1664bfb2fd58b3f3fe52c) to configure apt
repository for [Earthly](https://earthly.dev/). The script is available as a GitHub Gist. Hence, you are requested to open
the link to the bash-script and download the script (or, copy-paste the raw content into a file using a text-editor)
to use it directly from your Debian-based system such as _Debian_, _Ubuntu_ or _LinuxMint_ (including _LMDE_).

The script will setup the proper apt repository configuration file under `/etc/apt/sources.list.d` and installs `earthly`.
It will also try to use `podman` to bootstrap `earthly`, if podman is already installed.

## Prerequisites

The script should be run using an administrator account (i.e., with `sudo` access).
