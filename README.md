### Install and update Raspbian "lite" ###

Raspbian available here: https://www.raspberrypi.org/downloads/raspbian/
I used "rufus" to write the image to memory card

Change password for default user ("pi") from "raspberry"

### Wireless access point ###

Instructions based on this web page:
https://www.raspberrypi.org/documentation/configuration/wireless/access-point.md#internet-sharing
But using interface setup from this page:
https://raspberrypi.stackexchange.com/questions/47010/losing-access-to-raspberry-pi-after-bridge-br0-comes-up

Update Rapsbian:
- sudo apt-get update
- sudo apt-get upgrade

Install software for wireless access point and bridging:
- sudo apt install hostapd bridge-utils
- sudo systemctl stop hostapd

Bridging creates a higher-level construct over the two ports being bridged. It is the bridge that is the network device, so we need to stop the eth0 and wlan0 ports being allocated IP addresses by the DHCP client on the Raspberry Pi.
- sudo nano /etc/dhcpcd.conf

Add "denyinterfaces wlan0" and "denyinterfaces eth0" to the end of the file (but above any other added interface lines) and save the file.

Add a new bridge, which in this case is called br0.
- sudo brctl addbr br0

Connect the network ports. In this case, connect eth0 to the bridge br0.
- sudo brctl addif br0 eth0

Build the interface definition:
- sudo nano /etc/network/interfaces
  - auto eth0    
  - auto wlan0
  - 
  - auto br0
  - iface br0 inet static
  -      address 192.168.1.29
  -      netmask 255.255.255.0
  -      gateway 192.168.1.30
  - 
  - bridge_ports eth0 wlan0 # build bridge
  - bridge_fd 0             # no forwarding delay
  - bridge_stp off          # disable Spanning Tree Protocol

Configure the access point host software (hostapd). You need to edit the hostapd configuration file, located at /etc/hostapd/hostapd.conf, to add the various parameters for your wireless network. After initial install, this will be a new/empty file.
- sudo nano /etc/hostapd/hostapd.conf
  - interface=wlan0
  - bridge=br0
  - ssid=NETWORK
  - hw_mode=g
  - channel=7
  - wmm_enabled=0
  - macaddr_acl=0
  - auth_algs=1
  - ignore_broadcast_ssid=0
  - wpa=2
  - wpa_passphrase=PASSWORD
  - wpa_key_mgmt=WPA-PSK
  - wpa_pairwise=TKIP
  - rsn_pairwise=CCMP

We now need to tell the system where to find this configuration file.
- sudo nano /etc/default/hostapd
Find the line with #DAEMON_CONF, and replace it with this:
  - DAEMON_CONF="/etc/hostapd/hostapd.conf"

Now enable and start hostapd:
- sudo systemctl unmask hostapd
- sudo systemctl enable hostapd
- sudo systemctl start hostapd

Reboot

### Bluetooth audio ###

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
