+++
title = "Mount Google-Drive in Ubuntu"
description = "Mounting Google drive accounts in local folders using google-drive-ocamlfuse tool"
date = 2021-11-14T13:55:24+05:30
featured = true
draft = false
comment = true
toc = true
reward = true
diagram = true
categories = [
  "Ubuntu",
  "System"
]
tags = [
  "gdrive",
  "google-drive",
  "linux",
  "ubuntu"
]
series = [
  "Manual"
]
+++

In this post, I'll be describing how to setup automatic mounting of one or more Google Drives locally, on your Ubuntu system.
Following steps should generally work for the supported Ubuntu versions of the [google-drive-ocamlfuse](https://github.com/astrada/google-drive-ocamlfuse)
tool. Let's just right into the setup steps.

## Assumptions

Here are the list of assumptions made for descriptive purposes. Please replace them with appropriate values based on
your system and environment.

* **Normal user**: ubuntu
* **Base path for GDrive mount folders**: $HOME/Store (Ex., /home/ubuntu/Store)
* **Gmail address corresponding to the drive**: sample01@gmail.com
* **Mount folder for GDrive under base path**: gdrive01

## Install Google Drive tool for Ubuntu

Thanks to [Alessandro Strada](https://github.com/astrada) for this nice tool called
[google-drive-ocamlfuse](https://github.com/astrada/google-drive-ocamlfuse), written in OCAML language. We will be using
this tool to mount one or more Google Drives under appropriate folder(s) locally. Pre-built
binaries for this tool are available from [Ubuntu PPA](https://launchpad.net/~alessandro-strada/+archive/ubuntu/ppa).
It can be installed as follows. For more detailed installation steps and other ways to install and setup your system,
please refer to the tool's link as mentioned earlier.

Install `google-drive-ocamlfuse` tool by running the following commands in a terminal, with superuser privileges (i.e., sudo).

{{< highlight bash >}}
sudo add-apt-repository ppa:alessandro-strada/ppa
sudo apt-get update
sudo apt-get install google-drive-ocamlfuse
sudo dpkg --configure -a ## Optional, mainly to complete any pending package configurations
{{< /highlight >}}

## Setup BASH script to mount Google Drive(s)

1. Open a text editor as normal user and create a new BASH script by name `mount-google-drives` with the following content.
   Edit the values in the `configuration block` based on your need.
    {{< highlight bash >}}
    #!/bin/bash
    # Script to mount google-drive accounts locally

    ############ Configuration block - START ############
    ## Either 'export' GDRIVE_BASE folder in ~/.bashrc or,
    ## completely hardcode the absolute path here.
    ##
    ## This indicates the base folder under which, various
    ## google-drive account folders are mounted.
    GDRIVE_BASE=${GDRIVE_BASE:-$HOME/Store}

    ## Define mapping between a google-account label and
    ## corresponding destination folder to mount it. The
    ## mount-dir (i.e., destination folder) will be a folder
    ## within the GDRIVE_BASE.
    ##
    ## The label need not match exactly with your gmail username
    ## part. It's just an identifier to help you in identifying
    ## the mounted google-drive.
    ##
    ## Multiple label-mount-dir can be provided in the array
    ## below, by adding them in their own line.
    GDRIVE_MAPPING=(
      #"label|mount-dir"
      "sample01|gdrive01"
    )
    ############ Configuration block - END ############

    ############ No need to touch anything below this ############
    for drv in $GDRIVE_MAPPING; do
      label=$(echo $drv | awk -F '|' '{print $1}')
      mount_dir=$(echo $drv | awk -F '|' '{print $2}')

      [[ -z "$label" ]] || [[ -z "$mount_dir" ]] && echo -e "GDrive label or mount-dir not defined!" && exit 1

      mount_point="$GDRIVE_BASE/$mount_dir"

      CHECK_DRIVE=$(mount | grep -E "$mount_point" | grep -v "grep")
      [[ -z "${CHECK_DRIVE}" ]] && mkdir -p $mount_point && google-drive-ocamlfuse -label $label $mount_point && \
      sync || echo -e "GDrive for ${label} already mounted"
    done

    exit 0
    {{< /highlight >}}
2. Save the file and close the editor.
3. Move the file into `$HOME/.local/bin` folder. If the folder doesn't exist, create it by running `mkdir -p $HOME/.local/bin`.
4. Set the executable bit for the script by running `chmod +x $HOME/.local/bin/mount-google-drives`.
5. Ensure that `$HOME/.local/bin` is set as part of the PATH environment variable by running `echo $PATH`.
6. If `$HOME/.local/bin` is not set as part of the PATH environment, you can add the following line to $HOME/.bashrc file

    ```bash
    PATH="$HOME/.local/bin:$PATH"
    ```

7. Close the terminal.

The script should be good enough for manual mounting of Google drives at any point. This script also ensures that, the
Google drives are mounted only once, even if it is run multiple times.

In case, you would like the Google drives to get automated on every login, you just need to do the following.

1. Open a text editor and create the file `mount-google-drives.desktop` under `$HOME/.config/autostart` folder.
   Create the folder, if it doesn't already exist.

    ```ini
    [Desktop Entry]
    Type=Application
    Exec=/home/ubuntu/.local/bin/mount-google-drives
    Hidden=false
    NoDisplay=false
    X-GNOME-Autostart-enabled=true
    Name[en_US]=Google Drives
    Name=Google Drives
    Comment[en_US]=Mount Google Drive locally
    Comment=Mount Google Drive locally
    ```

    {{< tip title="TIP" >}}
    Update the absolute path to the `mount-google-drives` script for `Exec` entry based on your username.
    {{< /tip >}}
2. Save the file and close the editor.

Now, logout of your desktop session and login again. If all the steps are followed correctly, you should see your
mounted Google drive(s) locally, using File Explorer or by running `df -TH` command in a terminal.

Hope it helped you!
