Softether VPN Client Configurations
====================================
#### OS Version : Linux raspberrypi 5.4.51-v7l+ #1333 SMP Mon Aug 10 16:51:40 BST 2020 armv7l GNU/Linux
##### DHCP Server Setting : 10.0.0.0/8, gw : 10.77.77.1, Lease-Range : 10.77.77.77 ~ 10.77.77.177

1. sudo apt update
### SoftEther VPN Downloads
<https://www.softether-download.com/en.aspx?product=softether>
2. wget https://www.softether-download.com/files/softether/v4.34-9745-rtm-2020.04.05-tree/Linux/SoftEther_VPN_Client/32bit_-_ARM_EABI/softether-vpnclient-v4.34-9745-rtm-2020.04.05-linux-arm_eabi-32bit.tar.gz
3. tar xvzf softehter-vpnclient ...
4. cd vpnclient
5. make > 1 / 1 / 1
6. sudo vi /lib/systemd/system/vpnclient.service

<pre>
<code>
#!/bin/sh
[Unit]
Description=SoftEther VPN Client
After=network.target
[Service]
Type=forking
ExecStart=sudo /home/pi/vpnclient/vpnclient start
ExecStop=sudo /home/pi/vpnclient/vpnclient stop
[Install]
WantedBy=multi-user.target
</code>
</pre>

7. sudo systemctl enable vpnclient
8. sudo systemctl start vpnclient
9. ./vpncmd > 2 / enter
10. NicCreate soft
11. AccountCreate client
12. AccountPasswordSet client
13. AccountConnect client
14. AccountStatusGet client
15. AccountStartupSet client

# [Case 1: Used allow-hotplug]
1. sudo nano /etc/network/interfaces
<pre>
<code>
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp

allow-hotplug vpn_soft
iface vpn_soft inet dhcp
# post-up sudo route add default gw 10.77.77.1 dev vpn_soft (used for need g/w)
</code>
</pre>
# [Case 2: Used dhclient command]

1. sudo touch vpnstart
2. sudo chmod +x vpnstart
3. sudo nano vpnstart
<pre>
<code>
#!/bin/sh
sudo /home/pi/vpnclient/vpnclient start
sudo dhclient vpn_soft
</code>
</pre>
4. sudo vi /lib/systemd/system/vpnclient.service
<pre>
<code>
#!/bin/sh
[Unit]
Description=SoftEther VPN Client
After=network.target
[Service]
Type=forking
ExecStart=sudo /home/pi/vpnclient/vpnstart
ExecStop=sudo /home/pi/vpnclient/vpnclient stop
[Install]
WantedBy=multi-user.target
</code>
</pre>
sudo iptables -t nat -A PREROUTING -i {interface card reader} -j DNAT --to {interface to home network ip}