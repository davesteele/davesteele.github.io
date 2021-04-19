---
layout: post
title:  "Make a Keyboard Jiggler using a Raspberry Pi Zero W"
date:   2021-04-18 12:34:00
categories: raspberrypi
excerpt_separator: <!--more-->
---

## Make a Keyboard Jiggler using a Raspberry Pi Zero W

![Pi Keyboard Jiggler](/images/2021-04-18-keyboard-jiggler/keyboard-jiggler-jiggler.jpg)

The problem - your computer automatically sleeps or locks up when unused, there are times you don't want it to do that, and it is either inconvenient or impossible to disable.

There is a generally-available solution for this - the [Mouse Jiggler](https://en.wikipedia.org/wiki/Mouse_Jiggler). This USB device emulates a  mouse, and strives to generates just enough simulated mouse movement to convince your computer that there is an active human present.

This project turns a small Raspberry Pi into a USB jiggler device for your main computer. It emulates a keyboard instead of a mouse, which can implement a more robust strategy for staving off screen savers. It is reliable, flexible, and configurable.

This is a pretty easy project. This can be done in well under an hour, with most of that time dedicated to booting and connecting to the Pi.

<!--more-->

There is room for improvement for the typical commercial jiggler. In some cases, the ability of the device to work properly can depend on what window is selected, and where the cursor is. One jiggler I own won't register properly if plugged into a USB hub. Also, store-bought jigglers can be easy to identify on the USB bus - a problem for some.

Again, this project turns a Raspberry Pi into a simlated USB keyboard. When attached to a host computer, it boots off of host power, and registers as a second keyboard on the host. Then, about once a minute, it quickly toggles Caps Lock on the host, proving beyond a shadow of a doubt that a human is actively working on the computer.

![Blinkie](/images/2021-04-18-keyboard-jiggler/keyboard-jiggler-capsblink.gif)

The build-your-own option is pretty cheap - cheaper than many commercial choices. And it's satisfying to see your handiwork do something useful in just a short time.

### How?

#### Step 0 - Acquire the Pi

The Raspberry Pi Zero W is the preferred platform for this project. Most other modern Pis
do not have the required [OTG USB
port](https://en.wikipedia.org/wiki/USB_On-The-Go). The
Raspberry Pi 4 USB-C port is said to be compatible, if your computer's USB port can
provide enough Pi power for the 4.

The base Pi Zero will do the job, but I find the built in wireless on the "W" extremely useful.

#### Step 1 - Access the Pi

For this next part, you need to be logged into the Pi, and it needs Internet
access. I recommend using the [Comitup](https://davesteele.github.io/comitup/)
Lite disk image to bring the headless Pi
[online](https://github.com/davesteele/comitup/wiki/Tutorial).

#### Step 2 - Install the USB client keyboard software

On the Pi, install git:

    sudo apt-get update
    sudo apt-get install git

Clone the
[piezero-usb-hid-keyboard](https://github.com/raspberrypisig/pizero-usb-hid-keyboard)
repository to the Pi user home directory:

    cd ~
    git clone https://github.com/raspberrypisig/pizero-usb-hid-keyboard.git

Install the repository software to the Pi:

    cd pizero-usb-hid-keyboard
    sudo ./setup-hid-modules.sh
    sudo ./enableHIDRCLocal.sh

Reboot the Pi.

If it is not already, plug the Pi into a USB port of your host computer, using the OTG USB port. On the Pi Zero, that that is the USB port closest to the middle.

Log on to the Pi again, and test the hid-keyboard install by sending a keystroke to the host:

    sudo /home/pi/pizero-usb-hid-keyboard/sendkeys.sh h

You should see the results of pressing a keyboard "h" on the host.


#### Step 3 - Add the Keyboard Jiggler code


Create the file _/home/pi/tickle.py_, and populate it according to [this
gist](https://gist.github.com/davesteele/276a17315cff728bb5932c59329da850#file-tickle-py).
This code is Python, so spaces are significant.

Make the file executable:

    chmod 755 /home/pi/tickle.py

Edit the file [_/etc/rc.local_](https://gist.github.com/davesteele/276a17315cff728bb5932c59329da850#file-rc-local), and add a call to run _tickle.py_ in the background before the exit call:

    /home/pi/tickle.py &

Cycle power on the Pi, and you should see the Caps Lock light on your keyboard flash about once a minute.

#### Step 4 - Reconfigure (optional)

The configuration for the USB device created by the Pi is stored in the executable file _/home/pi/pizero-usb-hid-keyboard/rpi-hid.sh_. Most of the default parameters, which are published by the USB subsystem, are pretty generic (Manufacturer = "raspberrypi", Product = "Generic USB Keyboard". If you like, you can change these parameters to make the device look more like a real keyboard. For instance, to make it look more like my host keyboard, change the following:

  * idVendor: 0x046d
  * idProduct: 0xc313
  * serialnumber: 0
  * manufacturer: Logitech, Inc.
  * product: Internet 350 Keyboard

Reboot after making the changes.

#### Step 5 - Cleanup (optional)

The Comitup WiFi access software is still running at this point. If you wish to lock the Pi down, and make it just a jiggler, disable the comitup service and reboot:

    sudo systemctl disable comitup
    sudo reboot


And that's it. 


