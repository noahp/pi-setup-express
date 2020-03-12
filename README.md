# 🥧 Pi setup express mode

Likely out of date any minute. But this worked as of 2020-03-12 (on pi w).

> Note- On a pi 3, there's some issue around wifi not coming up due to rfkill,
apparently due to something `raspi-config` sets regarding regulatory reqs for
country code in wpa_supplicant. TODO test/research more for a solution. I also
didn't have sshd start up automatically, more research. pi w worked great.

See this nice Adafruit recipe too:
https://learn.adafruit.com/pi-hole-ad-blocker-with-pi-zero-w/prepare-the-pi

Note that using usb gadget (ssh over usb) is only available only on these pi
devices:

> **Raspberry Pi 0, 0W, A, A+, and 4**

The **Pi B, B+, 2, 3, 3A+, and 3B+** don't have the OTG pin 😕

## Recipe

1. grab a raspbian image. I like the buster lite one:
   https://www.raspberrypi.org/downloads/raspbian/
2. SD card reader on your PC. copy the .img file into the sd card, eg:

   ```bash
   # copy the image onto the sd card. watch the 'of=' arg!
   sudo dd bs=4M \
     if=~/Downloads/2020-02-13-raspbian-buster-lite.img \
     of=/dev/sda conv=fsync status=progress

   # make sure everything's flushed out
   sync
   ```

3. unmount, unplug and replug the SD card. you should see the PI file system
   now. set up usb gadget so we can ssh over the usb connection:

   ```bash
   # first cd into the boot dir on the SD card
   cd /media/$USER/boot

   # enable ssh- just need an empty file 'ssh' in boot
   touch ssh

   # enable alternate USB driver for OTG, so we can gadget
   echo "dtoverlay=dwc2" >> config.txt

   # load the dwc2 + gadget-ethernet drivers on boot by inserting these lines
   # into cmdline.txt, right after 'rootwait'
   sed --in-place 's/rootwait/rootwait modules-load=dwc2,g_ether/' cmdline.txt
   ```

4. optionally add wifi creds:

 ```bash
echo 'ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=US

network={
    ssid="YOURSSID"
    psk="YOURPASSWORD"
    scan_ssid=1
}' > wpa_supllicant.conf
```

5. unmount, insert the card into the pi, and you should be able to ssh in!

   ```bash
   # requires mDNS on the host
   ssh pi@raspberrypi.local
   ```
