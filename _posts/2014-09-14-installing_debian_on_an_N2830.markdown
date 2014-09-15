---
layout: post
title:  "Installing Debian on a Celeron N2830"
date:   2014-09-14 21:29:00
categories: debian bugs
---
These are Debian installation notes for:

<p>
<table>
<tr><th>Regulatory Type</th><td>P28F005</td></tr>
<tr><th>Marketing Name</th><td>Dell Inspiron 15-3531</td></tr>
<tr><th>Processor</th><td>Celeron N2830 2.16 NHz</td></tr>
</table>
</p>

I don't normally do laptops. I lean more towards the camp of having a computer available wherever you need one - especially now that those possible locations include your pocket, I don't run across many situations that scream out for a laptop.

Twice now I have run into special occasions that forced exceptions to my rule. Both times I  bought the cheapest laptop I could find, swapped out the drive for an SSD Linux install, and unloaded the machine after the conference. The first time was in 2011. I recall the installation on the Toshiba Celeron 925 went smoothly. I expected last months repeat on a Dell Celeron N2830 to go as well.

Well, it didn't. I list the particular issues here, with my work-arounds, in case it might help someone else.

BTW, the computer works fine once the installation problems are worked out. I do not see the performance problems reported for the stock Windows install - it is quite responsive with the SSD and XFCE. Battery life is a solid 5 hours on a charge.



TPM
---

This is my first time encountering this. I disabled Secure Boot in the BIOS Advanced section of Setup, and enabled the Legacy ROM, for the quickest way past the problem.

USB Compatibility
-----------------

This computer doesn't ship with support for removable media. The installation required USB support. USB installation media boots fine, but the current stable (wheezy) installer couldn't see it - apparently related to the new Celeron. The testing installation (jessie) saw the installation media, and could procede, but...

Jessie installer WPA support
----------------------------

... I was unable to get the testing/jessie installer to authenticate against my well-established-and-fully compliant 'N' router. I was only able to connect to remote repositories by disabling all authentication on the router. After that, the process went fine, installing the currently standard XFCE desktop...


GNOME 3
-------

... but I prefer GNOME. Upgrading/transitioning to GNOME broke the install. I reinstalled to XFCE and let it alone after that.


"mmc0: got irq while runtime suspended"
---------------------------------

/var/log/messages was getting hit with the above log message a half dozen times per second. I didn't even realize that the system had a card slot until after I disabled the driver, to stop the log spam...

    # cat /etc/modprobe.d/blacklist 
    # eliminate the mmc0: got irq while runtime suspended
    # multiple times per second.
    sdhci_acpi
    sdhci_pci
    sdhci
    # 

XFCE Wallpaper
--------------

So now the system works OK, but there is a problem with the wallpaper. It is reset to a standard mouse mascot background on every login, which only covers part of the screen. To fix, I toggle a wallpaper every time.


Honestly, this is the kind of experience I expected 10 years ago, when we were only a few years into the "Year of the Linux Desktop". Nowadays I expect better.

