Setting up Wireless Access Point as IP router
=============================================

Instructions come from this website:
https://github.com/billz/raspap-webgui

Install Raspbian Buster Lite:
Raspbian available here: https://www.raspberrypi.org/downloads/raspbian/
I used "rufus" to write the image to memory card.

Change password for "pi" (from the default password of "raspberry").

Update OS:
- sudo apt-get update
- sudo apt-get dist-upgrade
- sudo reboot

Set wireless to UK:
sudo raspi-config
  - Option 2 (Network Options)
  - Option N2 (Wi-Fi)
  - Select regulatory area ("GB") then exit
Check configuration with:
  - sudo iw reg get

Install RaspAP:
- curl -sL https://install.raspap.com | bash
  - Take default options

This marks the end of the instructions from the website.
The default install does not work for UK wireless LAN.

Edit /etc/hostapd/hostapd.conf and make the following changes:

channel=1		==> channel=3
country_code=   ==> country_code=GB
#ieee80211n=1   ==> ieee80211n=1
#wmm_enabled=1  ==> wmm_enabled=1
#ht_capab=...   ==> ht_capab=...
                ==> ignore_broadcast_ssid=0
                ==> ieee80211d=1
				==> macaddr_acl=0
				
Apply the new configuration:
  systemctl hostapd restart



Setting up Bluetooth audio
==========================

Instructions based on this web page:
https://www.raspberrypi.org/forums/viewtopic.php?f=38&t=247892

Start the bluealsa service:
- sudo nano /lib/systemd/system/bluealsa.service # Edit the ExecStart line as follows:
  - ExecStart=/usr/bin/bluealsa --profile=a2dp-sink

Add a service for bluealsa-aplay:
- sudo nano /etc/systemd/system/aplay.service # Fill the file as follows:
  - [Unit]
  - Description=BlueALSA aplay service
  - After=bluetooth.service
  - Requires=bluetooth.service
  -
  - [Service]
  - ExecStart=/usr/bin/bluealsa-aplay 00:00:00:00:00:00
  -
  - [Install]
  - WantedBy=multi-user.target
- sudo systemctl enable aplay

Reboot
