+++
title = "Earthly CI-CD with Podman"
description = "Setup Earthly CI-CD to use Podman instead of Docker"
date = 2022-09-06T09:30:24+05:30
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
  "podman",
  "earthly",
  "cicd"
]
series = [
  "Reference"
]
+++

The [Earthly](https://earthly.dev/) is an effortless CI/CD framework that combines the power of Docker, Makefile and
Bash and provides an easy to use build system that runs locally and on cloud based systems such as Github-Actions,
Circle-CI and so on.

`Earthly` will look for docker eventhough it doesn't depend on it explicitly. For those who prefer `podman` as an alternate
to using `docker` the following guide will help in setting up Earthly to use podman instead of docker.

## Setup Podman on Ubuntu

For details on installing Podman on Ubuntu, please refer to [__this__](https://www.ravihara.in/en/posts/setup-apt-podman/)
blog post. Once you setup the `podman` on your system, please move to the next sections.

## Enable unified cgroup hierarchy

The sysfs on your system should have "/sys/fs/cgroup/user.slice". If this does not exists, you need to enable it in grub
configuration. If the folder already exists, you can skip this step.

* Open the file `/etc/default/grub` as administrator (i.e., user with sudo access) in a text editor (Ex., vim, nano).
* Add the entry `systemd.unified_cgroup_hierarchy=1` to GRUB_CMDLINE_LINUX_DEFAULT (i.e., append it to the end of the
  space separated value string if, it doesn't already exists).
* Save the file and close the editor.
* Reboot the machine.

## Setup systemd for cgroup permissions

The non-root users of the system need cgroup delegation permissions for cpu, memory, io and pids. This needs to be
configured in systemd as follows. All the actions need to be performed as administrator (i.e., root user or,
user with sudo access).

{{< note title="NOTE" >}}
On Ubuntu 22.04, it is sufficient to add the file `/etc/systemd/system/user@.service.d/delegate.conf` as described below.
Older versions (or other Debian variants) might require all the steps.
{{< /note >}}

* Create a file `/etc/systemd/system/user-0.slice` with the following content. This step is mainly needed for systems
  with [systemd version 239](https://github.com/systemd/systemd/issues/9578) or below. You may skip this step if the
  version of systemd in your system is 240 or above or, if you are using Ubuntu 22.04.

  ````ini
  [Unit]
  Before=systemd-logind.service
  [Slice]
  Slice=user.slice
  [Install]
  WantedBy=multi-user.target
  ````

* Create the user-service folders as below.
  
  ````bash
  mkdir -p /etc/systemd/system/user@.service.d
  mkdir -p /etc/systemd/system/user-.slice.d # Not needed for Ubuntu 22.04
  ````

* Create the file `/etc/systemd/system/user@.service.d/delegate.conf` with the following content.
  
  ````ini
  [Service]
  Delegate=cpu cpuset io memory pids
  ````

* Create the file `/etc/systemd/system/user-.slice.d/override.conf` with the following content. This step is not
  needed if you are using Ubuntu 22.04.
  
  ````ini
  [Slice]
  Slice=user.slice

  CPUAccounting=yes
  MemoryAccounting=yes
  IOAccounting=yes
  TasksAccounting=yes
  ````

* Reload systemctl daemon and enable the added `user-0.slice` service.
  
  ````bash
  systemctl daemon-reload
  systemctl enable user-0.slice # Not needed for Ubuntu 22.04
  ````

* Finally, reboot the machine for the permissions to show up per user.

* With your user account (not root), run the following command to verify that you have the required
  cgroup delegation permissions.

  ````bash
  cat "/sys/fs/cgroup/user.slice/user-$(id -u).slice/user@$(id -u).service/cgroup.controllers"
  ````

  It should show up the items - _cpu cpuset io memory pids_, as set in the delegate.conf file above. You are
  now ready to install earthly and use it with podman.

## Setup Earthly on Ubuntu

For details on installing Podman on Ubuntu, please refer to [__this__](https://www.ravihara.in/en/posts/setup-apt-earthly/)
blog post.

## Testing Earthly with Podman

If you have followed the above steps and have installed and bootstraped earthly using podman, create a file named `Earthfile`
locally with the following content.

````docker
VERSION 0.6
FROM python:3

build:
  RUN mkdir -p /src && echo "print('Hello World')" >> /src/hello.py
  SAVE ARTIFACT src /src

docker:
  COPY +build/src src
  ENTRYPOINT ["python3", "./src/hello.py"]
  SAVE IMAGE python-example:latest
````

From the same folder where the file is created, run `earthly +docker`. From the output, you should see that `earthly` is
using `podman` internally to run the build.

{{< note title="References" >}}
Special thanks to the people who have already given the information in various blogs and forums.

* [Earthly Blog](https://earthly.dev/blog/earthly-podman/)
* [Stack Exchange](https://unix.stackexchange.com/a/625079/540328)
{{< /note >}}
