---
title: "Home WiFi using CDMA modem"
date: 2021-11-08T20:51:10+05:30
draft: false
categories:
  - linux
  - networking
authors:
  - Ravi Hara
---

In case CDMA technology is still offered in your area, this post might be useful to you in setting up a simple Home WiFi
using a CDMA modem, WiFi router and a Linux machine. This could be a good excercise to revive your old Linux box and that
old WiFi router which had been laying useless :-).

## What you need

* **Linux machine** - *An old Debian / Ubuntu Linux pc will do too*.
* **WiFi router** - *An old 2.4GHz router from DLink would work*.
* **CDMA-1x USB modem** - *Typically available as CDMA USB-Dongle*.
* **Ethernet cable** to connect the Router to the Linux machine.

## Hardware setup

* Configure the WiFi router to act as an access point using the Router's web interface.
* Connect the Ethernet cable to one of the 4 ports of the Router and the other end to the Linux machine's Ethernet port.
* Connect the CDMA USB modem to one of the USB port of the Linux machine.

## Software setup

* Install the following packages using the package manager.

  ```bash
  apt-get install -y pppd wvdial iptables dnsmasq
  ```

* Ensure that the Linux device driver modules `usbserial` and `cdc_acm` are loaded. You may use the command `insmod`
  in case they are not already loaded.

## Setting up the WiFi

This process involves the following main steps.

1. Configuring kernel modules for auto-loading at bootup
2. Configuring and running “wvdial” for dialup (ppp) connection
3. Setting up “iptables” for NAT forwarding
4. Setting up “dnsmasq” for resolving DNS and as DHCP server

### Configure Kernel modules

1. Open a terminal and the command `sudo bash` in order to enter BASH as super-user.
2. Edit the file `/etc/modules` and append the following lines if they do not already exist.
   Check and use the appropriate values for `vendor` and `product` ids for the USB modem, by
   referring to the file `/proc/bus/usb/devices`.<br/><br/>

    ```bash
   usbserial vendor=0x19d2 product=0xfffd
   cdc_acm
   ```

3. Save the file and exit the terminal.
4. Reboot the machine for automatic driver loading to happen.

### Configure Dial-Up connection

1. Open a terminal and the command `sudo bash` in order to enter BASH as super-user.
2. Take a backup of the file `/etc/wvdial.conf` and then replace it's content with the following configuration.<br/><br/>

    ```ini
    [Dialer Defaults]
    Init1 = ATZ
    Init2 = AT+CRM=1
    Modem Type = Analog Modem
    SetVolume = 0
    Baud = 115200
    New PPPD = yes
    Modem = /dev/ttyUSB0
    Carrier Check = no
    Stupid Mode = 1
    ISDN = 0
    Phone = <dial-number>
    Password = <dial-password>
    Username = <dial-username>
    ```

    <br/>Use appropriate values for `dial-number`, `dial-password` and `dial-username`. For Reliance CDMA modem,
    dial-number will be #777 and, dial-password and dial-username will be the phone number itself.

3. Save and close the file.
4. Take a backup of `/etc/resolv.conf` then, clear all the content of `/etc/resolv.conf` and save it.
5. Run `wvdial` to initiate the PPP connection.

### NAT forwarding with iptables

In this step, we will be configuring the NAT table to masquerade ppp0 network interface
and configure forwarding rule for the ethernet interface.

1. Open a terminal and the command `sudo bash` in order to enter BASH as super-user.
2. Create or open the file `/usr/local/sbin/ppp_eth_route` and enter the following content.
    If the file altready exists, first take a backup and then replace its content with the following.

    ```bash
    #!/bin/bash

    iptables —flush
    iptables —table nat —flush

    iptables —delete-chain
    iptables —table nat —delete-chain

    # Set up IP FORWARDing and Masquerading
    iptables —table nat —append POSTROUTING —out-interface ppp0 -j MASQUERADE
    iptables —append FORWARD —in-interface eth0 -j ACCEPT

    echo 1 > /proc/sys/net/ipv4/ip_forward
    ```

3. Save the file and change its permission to 0755 using the command;

    ```bash
    chown 755 /usr/local/sbin/ppp_eth_route
    ```

4. Run the `ppp_eth_route` script created above.

### Setting up local DNS and DHCP server

We will be using the `dnsmasq` tool to setup a DHCP server and configure the DNS host resolution.

1. Open a terminal and the command `sudo bash` in order to enter BASH as super-user.
2. Edit the file `/etc/network/interfaces` and replace it's content with the following.
    It is recommended to take a backup of the file before changing it's content.

  ```bash
  auth eth0
  iface eth0 inet static
  address 10.10.1.1
  netmask 255.255.255.0
  ```

  Save and close the file.
3. Edit the file `/etc/dnsmasq.conf` as follows. Then, save and close the file.

   1. Uncomment the line containing `interface=eth0`
   2. Uncomment the line that starts with `dhcp-range` to enable integrated DHCP server. The line
      must be edited as `dhcp-range=10.10.1.10,10.10.1.200,12h`

  Here, the `dnsmasq` tool is allowed to provide IPs in the range 10.10.1.10 to 10.10.1.200 with a lease time of 12 hours.
4. Restart the system services - `network-manager` and `dnsmasq`.

NOTE: The above steps are one time activity. For subsequent use, you need to run the following steps
as superuser (i.e., sudo). You need to ensure the USB modem and the WiFi router are properly connected to
the Linux machine before proceeding.

1. Restart the system `networking / network-manager` service.
2. Run `wvdial &`. Wait till it fetches IP address and DNS entries. Otherwise, there is no use
   in proceeding further since, there is likely a PPP connection issue.
3. Run `/usr/local/sbin/ppp_eth_route`.
4. Restart the system `dnsmasq` service.

Of course, you can put the above commands in a shell script and run them as super-user. That's it!
