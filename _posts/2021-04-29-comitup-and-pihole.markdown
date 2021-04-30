---
layout: post
title:  "Install Pi-hole onto a Headless Raspberry Pi using Comitup"
date:   2021-04-29 18:47:00
categories: raspberrypi
excerpt_separator: <!--more-->
---

[Pi-hole](https://en.wikipedia.org/wiki/Pi-hole) is a popular, useful Raspberry
Pi project for removing ads from your browsing, site-wide.
[Comitup](https://github.com/davesteele/comitup) lets you configure and
WiFi-connect a Pi with no keyboard, mouse, monitor, or SD card pre-fiddling
required. Together, they provide an effective means to manage your
ad-visibility experience at home.

Comitup and Pi-hole both manage DNS, DHCP, and HTTP, which can cause some
conflicts. We'll work around this by letting Comitup start and stop Pi-hole
when it is safe to do so. 


<!--more-->

Start by [loading the
Pi](https://github.com/davesteele/comitup/wiki/Tutorial#download-the-comitup-image)
with the [Comitup Image](https://davesteele.github.io/comitup/), which is just
Raspberry Pi OS with the comitup package installed. Alternately, the comitup
package can be added to an existing Raspberry Pi OS installation, but note that
this is not a headless operation.

[Connect the
device](https://github.com/davesteele/comitup/wiki/Tutorial#connect-to-the-comitup-hotspot)
to your upstream Access Point.

[Install Pi-hole](https://github.com/pi-hole/pi-hole/#one-step-automated-install).

Start the Comitup-specific configuration by disabling Pi-hole autostart:

    sudo systemctl stop pihole-FTL
    sudo systemctl disable pihole-FTL

Add a script for Comitup to manage Pi-hole, using the default
[_external_callback_](https://github.com/davesteele/comitup/blob/fee746a6830be5e47ab82d4b9221c712966a2c4e/conf/comitup.conf#L53)
configuration.

    $ cat /usr/local/bin/comitup-callback 
    #!/usr/bin/bash
    
    if [ $1 == "CONNECTED" ] ; then
      systemctl start pihole-FTL
    else
      if [ $1 == "HOTSPOT" ] ; then
        systemctl stop pihole-FTL
      fi
    fi

    $ sudo chmod 755 /usr/local/bin/comitup-callback
    $ sudo chown root.root /usr/local/bin/comitup-callback

Now Pi-hole will only run when the device is connected to the upstream WiFi.

The Pi-hole installation instructions talk about the importance that the IP
address of the device be persistent. The preferred mechanism is to configure
your router to return a consistent IP configuration for the device. In
practice, the upstream router will continue to provide the same pool ip address
if the device is generally online.  But, if those options are not practical,
you can set the current address as static using the NetworkManager _nmcli_
command.

For instance, if _ip addr_ and _ip route_ return:


    # ip addr
    ...
    3: wlan0: <BROADCAST,MULTICAST,UP,LOWER-UP> mtu 1500 qdisc pfifo-fast state UP group default qlen 1000
        link/ether dc:a6:32:02:a5:9b brd ff:ff:ff:ff:ff:ff
        inet 192.168.200.229/24 brd 192.168.200.255 scope global dynamic noprefixroute wlan0
    ...


    # ip route
    default via 192.168.200.1 dev wlan0 proto dhcp src 192.168.200.229 metric 303 
    172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown 
    192.168.200.0/24 dev wlan0 proto dhcp scope link src 192.168.200.229 metric 303 


You can set a static configuration for the interface using nmcli. First, determine the connection name:

    $ nmcli con show

    NAME              UUID                                  TYPE      DEVICE 
    ISPRouter         5656e708-8bdc-43ad-bbdd-90b060510f83  wifi      wlan0  
    comitup-707-0000  df1367d2-856d-4124-9295-ab048e688110  wifi      --     
    dhcp              7c7e0483-1532-4a7e-87aa-8247db11b31f  ethernet  --     
    static            6f7f1894-d1ad-42b1-bc28-f46d6fe2e8e5  ethernet  --     

The connection currently controlling _wlan0_ is _ISPRouter_.

Set the address and netmask per the information returned by _ip addr_:

    sudo nmcli con mod ISPRrouter ipv4.addresses 192.168.200.229/24

Set the gateway per _ip route_:

    sudo nmcli con mod ISPRouter ipv4.gateway 192.168.200.1

Set up DNS

    sudo nmcli con mod ISPRouter ipv4.dns "8.8.8.8"

Set the interface to "static":

    sudo nmcli con mod ISPRouter ipv4.method manual

A [similar
process](https://unix.stackexchange.com/questions/575467/create-static-ipv6-address-without-noprefixroute-using-networkmanager)
can be used to set a static IPv6 address.

Reboot, and verify that the device connects to the router, and that Pi-hole is
running.
