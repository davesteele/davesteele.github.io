---
layout: post
title:  "The Best VPN Service is a VPS"
date:   2014-10-30 22:17:00
categories: administration
---

I felt the need for a [VPN][] capability for use on untrusted mobile Wifi
nodes.

[VPN]: https://en.wikipedia.org/wiki/Virtual_private_network

Commercial VPN services are plentiful, and [cheap enough][], but I believe
I've found a better option for my purposes.

[cheap enough]: http://lifehacker.com/5935863/five-best-vpn-service-providers

At the time of this writing, there are [several][] [options][] for personal
[VPS][] instances priced in the $5/TB/month range. That's more than enough
bandwidth for the purpose, and cheaper than the electricity bill
to run a personal server. Once this is set up, you have a super-cheap VPN
with no advertising or connection restrictions, with logging under your
control, plus a server available for other purposes, as well.

[several]: https://www.digitalocean.com/pricing/
[options]: https://www.linode.com/pricing
[VPS]: http://en.wikipedia.org/wiki/Virtual_private_server

But what about the bother of setting up a VPN server? Well, here's a [script][]
that will automatically install and configure a PPTP service. 
It is possible to sign up with a VPS vendor, spin up a server, and get a
working VPN in about 10 minutes.

[script]: https://raw.githubusercontent.com/davesteele/server-setup-scripts/master/pptpd/setup-pptp.sh

    #!/bin/sh
    # setup-pptpd.sh
    #
    # This will request a CHAP password.
    # The iptables-persistent install may query - say 'y'
    #
    # Once complete, there are opportunities to tighten things up in
    # /etc/ppp/chap-secrets.
    #
    
    apt-get -y update
    apt-get -y install pptpd
    apt-get -y install iptables-persistent
    apt-get -y install vim
    
    #set username and password
    if grep -q CONFIGURED /etc/ppp/chap-secrets ;
    then
      echo "CHAP is configured already";
    else
      echo -n "Enter CHAP password: ";
      read pw;
      echo "# CONFIGURED" >>/etc/ppp/chap-secrets;
      # Note that this set a password valid from and to all hosts
      echo "* * $pw *" >> /etc/ppp/chap-secrets;
    fi
    
    #set the pptpd address
    if grep -q 10.0.0.1 /etc/pptpd.conf ;
    then
      echo "pptpd is configured already";
    else
      echo "localip 10.0.0.1" >> /etc/pptpd.conf
      echo "localip 10.0.0.100-200" >> /etc/pptpd.conf
    fi
    
    #set the dns address
    if grep -q 8.8.8.8 /etc/ppp/pptpd-options ;
    then
      echo "dns is configured already";
    else
      echo "ms-dns 8.8.8.8" >> /etc/ppp/pptpd-options
      echo "ms-dns 8.8.4.4" >> /etc/ppp/pptpd-options
    fi
    
    echo net.ipv4.ip_forward=1 >/etc/sysctl.d/ip_forward.conf
    
    /sbin/iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
    iptables-save >/etc/iptables/rules.v4
    
    sysctl -p /etc/sysctl.d/ip_forward.conf
    /etc/init.d/pptpd restart

The script was developed for Debian 'wheezy', but should work on any
Debian-derivative. Even if it doesn't run successfully to completion on
your distribution, it can serve as a highly-structure howto for manual
installation.

The most up-to-date version of this script is available [on github][].

[on github]: https://github.com/davesteele/server-setup-scripts/blob/master/pptpd/setup-pptp.sh

On the client side, set up a PPTP connection to the IP address of the server,
using the CHAP password given to the script. The user name is not important.
