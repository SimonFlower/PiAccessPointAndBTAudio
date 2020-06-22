Setting up Wireless Access Point as IP router
=============================================

Instructions come from this website:
https://github.com/billz/raspap-webgui

Install Raspbian Buster Lite:
Raspbian available here: https://www.raspberrypi.org/downloads/raspbian/
I used "rufus" to write the image to memory card.

Change password for "pi" (from the default password of "raspberry").

Set up sftp by running:
```
raspi-config
  5 Interfacing Options
  P2 SSH
```

Turn off predictable network names by running:
```
raspi-config
  2 Network Options
  N3 Network interface names
  <No>
```

Set a static IP address (if the DHCP server doesn't allow you
to fix IP addresses), by adding this section to /etc/dhcpcd.conf:
```
# static IP address for wired connection
# to go back to DHCP simply remove this section
interface eth0
static ip_address=192.168.1.202/24
static routers=192.168.1.254
static domain_name_servers=192.168.1.254
```

Update OS:
```
sudo apt-get update
sudo apt-get dist-upgrade
sudo reboot
```

Set wireless to UK:
```
sudo raspi-config
  Option 2 (Network Options)
  Option N2 (Wi-Fi)
  Select regulatory area ("GB") then exit
```
Check configuration with:
```
sudo iw reg get
```

Install RaspAP:
```
curl -sL https://install.raspap.com | bash
```
Take default options

This marks the end of the instructions from the website.
The default install does not work for UK wireless LAN.

Edit /etc/hostapd/hostapd.conf and make the following changes:

```
channel=1       ==> channel=3
country_code=   ==> country_code=GB
#ieee80211n=1   ==> ieee80211n=1
#wmm_enabled=1  ==> wmm_enabled=1
                ==> ignore_broadcast_ssid=0
                ==> ieee80211d=1
                ==> macaddr_acl=0
```

Apply the new configuration:
  systemctl hostapd restart

When it comes time to change the WiFi SSID or password these
are in the same file - /etc/hostapd/hostapd.conf

Change default password for RaspAP by browsing to the IP
address of the Raspberry PI, e.g.
```
http://192.168.1.201/
```
Then go to the "Configure Auth" tab


Setting up Bluetooth audio
==========================

Instructions based on these web pages:
https://www.raspberrypi.org/forums/viewtopic.php?f=38&t=247892
https://gist.github.com/mill1000/74c7473ee3b4a5b13f6325e9994ff84c

Install bluealsa and python-dbus:
```
sudo apt-get install bluealsa python-dbus
```

Configure the bluealsa service. Edit the ExecStart line for the service as follows:
```
sudo nano /lib/systemd/system/bluealsa.service
  ExecStart=/usr/bin/bluealsa --profile=a2dp-sink
```

Add a service for bluealsa-aplay by copying aplay.service
from this project to /etc/systemd/system/aplay.service then
enable the service:
```
sudo systemctl enable aplay
```

Set the DiscoverableTimeout in /etc/bluetooth/main.conf to 0
```
DiscoverableTimeout = 0
```

Use the bluetooth control utility to turn on discovery:
```
sudo bluetoothctl
  power on
  discoverable on
  exit
```

Copy the included file a2dp-agent to /usr/local/bin 
and make the file executable:
```
sudo chmod +x /usr/local/bin/a2dp-agent
```

Before continuing, verify that the agent is functional. 
The Raspberry Pi should be discoverable, pairable and 
recognized as an audio device. Note: At this point the 
device will not output any audio.  This step is only to 
verify the Bluetooth is discoverable and bindable.
Manually run the agent by executing:
```
sudo /usr/local/bin/a2dp-agent
```
Attempt to pair and connect with the Raspberry Pi using your phone or computer.
The agent should output the accepted and rejected Bluetooth UUIDs

```
A2DP Agent Registered
AuthorizeService (/org/bluez/hci0/dev_94_01_C2_47_01_AA, 0000111E-0000-1000-8000-00805F9B34FB)
Rejecting non-A2DP Service
AuthorizeService (/org/bluez/hci0/dev_94_01_C2_47_01_AA, 0000110d-0000-1000-8000-00805f9b34fb)
Authorized A2DP Service
AuthorizeService (/org/bluez/hci0/dev_94_01_C2_47_01_AA, 0000111E-0000-1000-8000-00805F9B34FB)
Rejecting non-A2DP Service
```

If this doesn't work, try rebooting.

To make the A2DP Bluetooth Agent run on boot copy the included 
file bt-agent-a2dp.service to /etc/systemd/system. Now run the 
following command to enable the A2DP Agent service
```
sudo systemctl enable bt-agent-a2dp.service
```
Bluetooth devices should now be able to discover, pair and connect 
to the Raspberry Pi without any user intervention.

Now that Bluetooth devices can pair and connect with the Raspberry Pi 
we can test the audio playback. The tool bluealsa-aplay is used to 
forward audio from the Bluetooth device to the ALSA output device (sound card).
Execute the following command to accept A2DP audio from any connected Bluetooth device.
```
bluealsa-aplay -vv 00:00:00:00:00:00
```
Play a song on the Bluetooth device and the Raspberry Pi should output audio on 
either the headphone jack or the HDMI port. See this guide for configuring the 
audio output device of the Raspberry Pi.

To make the audio playback run on boot copy the included file a2dp-playback.service 
to /etc/systemd/system (note that this file has been edited from the original
provided on the source project - it contains an ExecStartPost line to set
the output volume to 100%). Now run the following command to enable A2DP Playback service
```
sudo systemctl enable a2dp-playback.service
```

Change the Blutooth discovery name:
```
sudo nano /etc/machine-info   
  PRETTY_HOSTNAME=ShedAP
 ```

Reboot

