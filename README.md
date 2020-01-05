### Install and update Raspbian "lite" ###

Raspbian available here: https://www.raspberrypi.org/downloads/raspbian/
I used "rufus" to write the image to memory card

Change password for default user ("pi") from "raspberry"

### Wireless access point ###

Instructions based on this web page:
https://www.raspberrypi.org/documentation/configuration/wireless/access-point.md#internet-sharing

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

Now the interfaces file needs to be edited to adjust the various devices to work with bridging. To make this work with the newer systemd configuration options, you'll need to create a set of network configuration files. If you want to create a Linux bridge (br0) and add a physical interface (eth0) to the bridge, create the following configuration.
- sudo nano /etc/systemd/network/bridge-br0.netdev
  - [NetDev]
  - Name=br0
  - Kind=bridge

Then configure the bridge interface br0 and the slave interface eth0 using .network files as follows:
- sudo nano /etc/systemd/network/bridge-br0-slave.network
  - [Match]
  - Name=eth0
  - 
  - [Network]
  - Bridge=br0
  - sudo nano /etc/systemd/network/bridge-br0.network
  - 
  - [Match]
  - Name=br0
  - 
  - [Network]
  - Address=192.168.1.202/24
  - Gateway=192.168.1.254
  - DNS=192.168.1.254

Restart systemd-networkd:
- sudo systemctl restart systemd-networkd

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







Instructions based on this web page:
https://thepi.io/how-to-use-your-raspberry-pi-as-a-wireless-access-point/

Update Rapsbian:
- sudo apt-get update
- sudo apt-get upgrade

Install and configure hostapd - access point software:
- sudo apt-get install hostapd
- sudo systemctl stop hostapd
- sudo nano /etc/dhcpcd.conf # add lines at end:
  - interface wlan0
  - static ip_address=192.168.0.10/24
  - denyinterfaces eth0
  - denyinterfaces wlan0
- sudo nano /etc/hostapd/hostapd.conf # new file:
  - interface=wlan0
  - bridge=br0
  - hw_mode=g
  - channel=7
  - wmm_enabled=0
  - macaddr_acl=0
  - auth_algs=1
  - ignore_broadcast_ssid=0
  - wpa=2
  - wpa_key_mgmt=WPA-PSK
  - wpa_pairwise=TKIP
  - rsn_pairwise=CCMP
  - ssid=NETWORK
  - wpa_passphrase=PASSWORD
- sudo nano /etc/default/hostapd # find the line starting DAEMON_CONF and replace with:
  - DAEMON_CONF="/etc/hostapd/hostapd.conf"
- sudo systemctl unmask hostapd
- sudo systemctl enable hostapd

Install and configure dnsmasq for DHCP
- sudo apt-get install dnsmasq
- sudo systemctl stop dnsmasq
- sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig
- sudo nano /etc/dnsmasq.conf
  - interface=wlan0
  -   dhcp-range=192.168.1.220,192.168.1.239,255.255.255.0,24h
  
Set up packet forwarding:
- sudo nano /etc/sysctl.conf # uncomment the line:
  - net.ipv4.ip_forward=1

Add IP masquerading for outbound traffic on eth0:
- sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
- sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"
- iptables-restore < /etc/iptables.ipv4.nat

Enable internet connection:
- sudo apt-get install bridge-utils
- sudo brctl addbr br0
- sudo brctl addif br0 eth0
- sudo nano /etc/network/interfaces # add the following at the end of the file:
  - auto br0
  - iface br0 inet manual
  - bridge_ports eth0 wlan0

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
