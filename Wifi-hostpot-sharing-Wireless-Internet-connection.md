### 0. Issue
I can't set up a hotspot and connect to the Internet at the same time, reasons:
- Use the same antenna
- Wireless is half-duplex protocol, expect for some new products or services like Wifi 6
  - The radio can only transmit or receive at one time, it can't do both at the same time
  - If any software is developed to allow both Station and AP mode, it first needs to communicate like a Station, and then pass the data along as the AP
  - The latency is undesirable
  
However, if you have a second Wifi Adapter (like Wifi USB), you can do it without problem.

### 1. Steps
- Install requirements
```
sudo apt-get install iw iwconfig hostapd iptables udhcpd wpa_supplicant macchanger
```
- Shut down network manager
```
sudo service network-manager stop
```
- Create 2 virtual interface for your existing wireless interface
```
sudo iw phy phy0 interface add new0 type station
sudo iw phy phy0 interface add new1 type __ap
```
- Change MAC address for those virtual interface
```
sudo ifconfig new0 hw ether 00:11:22:33:44:55
sudo ifconfig new1 hw ether 00:11:22:33:44:66

```
or
```
ifconfig new0 down
macchanger --mac 00:11:22:33:44:55 new0
ifconfig new1 down
macchanger --mac 00:11:22:33:44:66 new1
ifconfig new0 up
ifconfig new1 up
```
- Check if the virtual interface new0, new1 and its mac has been created by
```
ifconfig -a
```
- Set up AP interface in /etc/hostapd.conf with those configurations
```
interface=new1
driver=nl80211
ssid=my_wifi_hotspot      #Change the ssid name as you wish
channel=11                #I sugest you to use the same channel as your wireless network
hw_mode=g
wme_enabled=1
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=3
wpa_passphrase=1234567890 #Change the passphrase as you wish
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```
- Start wireless hotspot
```
sudo hostapd /etc/hostapd
```
- Set a static ip address to interface new1, this IP will be your router (gateway) address
```
sudo ifconfig new1 192.168.0.101 up
```
- Set up dhcp server in /etc/udhcpd.conf with those configurations
```
start 192.168.0.102         #These IPs must to be in the same subset as your current default route
end 192.168.0.117 
interface new1 

opt dns 192.168.0.1         #Your current default route (gateway)
option subnet 255.255.255.0
opt router 192.168.0.101    #This IP must to be in the same subset as your current default route
option  domain  localhost
```
- In /etc/default/udhcpd, comment the line that says ```DHCPD_ENABLED="no"```.
- Open port 67 for receiving DHCP Client request packets, DHCP server will then send DHCP Offer packets to port 68 of client
```
sudo ufw allow 67
sudo ufw status verbose
```
- Start dhcp server and check dhcp status ```Active: active (running)```
```
sudo service udhcpd start
systemctl status udhcpd.service
```
- Check if other devices can connect to AP and request IP address
- Remaining steps haven't been tested yet, I will come back later for config Station connection.
Reference
- [Ask Ubuntu - How to create a wifi hotspot sharing wireless-internet connection, single adapter](https://askubuntu.com/questions/318973/how-do-i-create-a-wifi-hotspot-sharing-wireless-internet-connection-single-adap)
- [Wordpress - Connectify for linux with single wireless interface](https://linuxalfi.wordpress.com/2011/11/08/connectify-for-linux-with-single-wireless-interface/)
