---
layout: post
title:  "A Raspberry Pi Cloud Print Proxy"
date:   2016-4-23 12:34:00
categories: raspberrypi
---

## Making a Raspberry Pi Google Cloud Print Proxy

Google [Cloud Print](https://www.google.com/cloudprint/learn/) has become
a requirement for me - no more dealing
with drivers, and I can print from anywhere using my phone. But, not all
printers support the service (or support it adequately - I'm looking at you,
Brother).

Here's how to turn a headless Raspberry Pi into a Google Cloud Print proxy, making
your local printer(s) visible to Cloud Print. This is made possible using
[armoo](https://github.com/armooo)'s cloudprint proxy software.

Before you start, you'll need a Raspberry Pi 2 or newer with:

* Raspbian Jessie (Stretch is even better - see Notes)
* A root partition that is bigger than the default 4G (16GB recommended)
* Internet connectivity
* Access to a printer via the network or USB

You'll also need another computer on the same network with an ssh client
(like [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/)) and
a web browser logged in to your Google account. What you don't need is a
desktop environment on the Pi, or a connected
keyboard/monitor/mouse. All interaction can be achieved via ssh and
another browser connected to the network.

### Step 1 - Install

First, establish an ssh session with the Pi, and get the cloudprint packages:

    wget http://davesteele.github.io/cloudprint-service/deb/cloudprint-service_0.13-1+deb8u1+ppa1_all.deb
    wget http://davesteele.github.io/cloudprint-service/deb/cloudprint_0.13-1+deb8u1+ppa1_all.deb

You may want to check http://davesteele.github.io/cloudprint-service/ to see
if there are newer versions of the packages.

Next, install the packages:

    sudo dpkg -i cloudprint*.deb
    sudo apt-get update
    sudo apt-get -f install

The first command will likely complain that some dependencies are not
installed. The last command will install those dependencies (in my case,
there were over 40).

Now that [CUPS](https://www.cups.org/) is installed, make it's web interface
available on the network, accessible to the user _pi_:

    sudo sed -i 's/Listen localhost:631/Listen \*:631/' /etc/cups/cupsd.conf
    sudo sed -r -i 's/(Order allow\,deny)/\1\n  Allow all/' /etc/cups/cupsd.conf
    sudo usermod -a -G lpadmin pi
    sudo systemctl restart cups

At this point you should have the _cups_ and _cloudprintd_ services running.

### Step 2 - Configure

Use the CUPS web interface at _http&#58;//&lt;piaddr&gt;:631/_ to add a
printer to your setup. Your browser may complain about unsafe connections
for _https_ links. Allow these connections. When it asks, use the _pi_
user credentials.

From the command line on the Pi, restart the proxy (to register the new
printer) and establish Google Cloud Print authentication
with:

    sudo systemctl restart cloudprintd
    sudo cps-auth <cloudprint account>

The output of the _cps_auth_ command will include a URL. Copy this URL to your browser,
and use it to establish authentication. 

Now restart the cloudprint service to use this account.

    sudo systemctl restart cloudprintd

### Step 3 - Print

Your printer should be visible at https://www.google.com/cloudprint/#printers. Enjoy!

#### Notes

* I say "Raspberry Pi 2 or newer" because I've had problems running this with only 
512 MB of RAM.
* Don't like insecure _https_? Then skip the _/etc/cups/cupsd.conf_ edits and
manage printers from the Pi desktop.
* If you are running a newer version of Raspbian than Jessie, package
installation is just:

    sudo apt-get install cloudprint-service

