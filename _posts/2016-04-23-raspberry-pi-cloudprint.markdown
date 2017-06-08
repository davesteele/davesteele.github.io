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

Here's a simple procedure to turn a headless Raspberry Pi into a Google Cloud Print proxy, making
your local printer(s) visible to Cloud Print. This is made possible using
[armoo](https://github.com/armooo)'s cloudprint proxy software.

Before you start, you'll need a Raspberry Pi 2 (Edit: maybe not - see Notes) or
newer with:

* Raspbian Jessie (Stretch is even better - see Notes)
* A root partition that is bigger than the default 4G (16GB recommended)
* Internet connectivity (possibly via [Comitup](https://davesteele.github.io/comitup/))
* Access to a printer via the network or USB

For the installation, you'll also want another computer on the same network with an ssh client
(like [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/)) and
a web browser logged in to your Google account. What you don't need is a
desktop environment on the Pi, or a connected
keyboard/monitor/mouse. All interaction can be achieved via ssh and
another browser connected to the network.

### Step 1 - Install

Establish an ssh session with the Pi, and run the following commands
to install the required software from the [cloudprint repository].

[cloudprint repository]: http://davesteele.github.io/cloudprint-service/

Add the cloudprint repo to your apt environment.

    echo "deb http://davesteele.github.io/cloudprint-service/repo cloudprint-jessie main" | sudo tee /etc/apt/sources.list.d/cloudprint.list
    wget -q -O - https://davesteele.github.io/key-366150CE.pub.txt | sudo apt-key add -
    sudo apt-get update

Install the software. This will likely involve more than 40 packages.

    sudo apt-get -y upgrade
    sudo apt-get -y install cloudprint-service

Make the [CUPS] web page externally accessible.

[CUPS]: https://www.cups.org/

    sudo cupsctl --remote-admin
    sudo usermod -a -G lpadmin pi
    sudo systemctl restart cups
    

At this point you should have the _cups_ service running.

### Step 2 - Configure

Use the CUPS web interface at _http&#58;//&lt;piaddr&gt;:631/_ to add a
printer to your setup. Your browser may complain about unsafe connections
for _https_ links. Allow these connections. When it asks, use the _pi_
user credentials.

From the command line on the Pi,
establish Google Cloud Print authentication
with:

    sudo cps-auth

The output of the _cps_auth_ command will include a URL. Copy this URL to your browser,
and use it to establish authentication. 

Now restart the cloudprint service to use this account.

    sudo systemctl restart cloudprintd

Consider disabling CUPS remote administration, to improve security. Also, make sure you
have changed the default SSH credentials.

    sudo cupsctl --no-remote-admin
    sudo systemctl restart cups
    passwd


### Step 3 - Print

Your printer should be visible at [https://www.google.com/cloudprint/#printers](https://www.google.com/cloudprint/#printers). Enjoy!

#### Notes

* I say "Raspberry Pi 2 or newer" because I've had problems running this with only 
512 MB of RAM. Many others have reported success with smaller Pis, such as the Zero.
* Don't like insecure _https_? Then skip the _/etc/cups/cupsd.conf_ edits and
manage printers from the Pi desktop.
* If you are running a newer version of Raspbian than Jessie, cloudprint-service
is in the official repository. You don't need to modify the apt environment.

