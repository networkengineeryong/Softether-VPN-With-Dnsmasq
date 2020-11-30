Softether VPN Client Connect to Server
====================================
### OS Version : Linux raspberrypi 5.4.51-v7l+ #1333 SMP Mon Aug 10 16:51:40 BST 2020 armv7l GNU/Linux
***
### DHCP Server Setting : 10.0.0.0/8, gw : 10.77.77.1, Lease-Range : 10.77.77.77 ~ 10.77.77.177
***
### SoftEther VPN Download
<https://www.softether-download.com/en.aspx?product=softether>
***
<pre>
<code>
1. sudo apt update
</code>
</pre>
##### it updates the package lists for upgrades for packages that need upgrading, as well as new packages that have just come to the repositories.
<pre>
<code>
2. wget https://www.softether-download.com/files/softether/v4.34-9745-rtm-2020.04.05-tree/Linux/SoftEther_VPN_Client/32bit_-_ARM_EABI/softether-vpnclient-v4.34-9745-rtm-2020.04.05-linux-arm_eabi-32bit.tar.gz
3. tar xvzf softehter-vpnclient ...
</code>
</pre>
#### Download VPN Client, Unzip it.
<pre>
<code>
4. cd vpnclient
5. make

Do you want to read the License Agreement for this software ?

 1. Yes
 2. No

Please choose one of above number: 1

Did you read and understand the License Agreement ?
(If you couldn't read above text, Please read 'ReadMeFirst_License.txt'
 file with any text editor.)

 1. Yes
 2. No

Please choose one of above number: 1

Did you agree the License Agreement ?

1. Agree
2. Do Not Agree

Please choose one of above number: 1
</code>
</pre>
#### License Agreement, Answer 1(Yes) All The Questions
<pre>
<code>
6. sudo vi /lib/systemd/system/vpnclient.service
</code>
</pre>
#### Create Vpnclient Service, Use to Startup Configuration
<pre>
<code>
#!/bin/sh
[Unit]
Description=SoftEther VPN Client
After=network.target
[Service]
Type=forking
ExecStart=sudo /home/pi/vpnclient/vpnclient start # enter your vpnclient directory
ExecStop=sudo /home/pi/vpnclient/vpnclient stop   #              ""
[Install]
WantedBy=multi-user.target
</code>
</pre>
<pre>
<code>
7. sudo systemctl enable vpnclient
8. sudo systemctl start vpnclient
</code>
</pre>
#### Vpnclient Startup Set, Start Vpnclient

### VPN Client Management Commands
<https://www.softether.org/4-docs/1-manual/6._Command_Line_Management_Utility_Manual/6.5_VPN_Client_Management_Command_Reference>
<pre>
<code>
9. ./vpncmd
By using vpncmd program, the following can be achieved. 

1. Management of VPN Server or VPN Bridge 
2. Management of VPN Client
3. Use of VPN Tools (certificate creation and Network Traffic Speed Test Tool)

Select 1, 2 or 3: 2

Specify the host name or IP address of the computer that the destination VPN Client is operating on. 
If nothing is input and Enter is pressed, connection will be made to localhost (this computer).
Hostname of IP Address of Destination: 

Connected to VPN Client "localhost".
</code>
</pre>
#### Select 2, VPN Client, Skip Destination Address
<pre>
<code>
10. NicCreate soft
11. AccountCreate client
12. AccountPasswordSet client
13. AccountConnect client
14. AccountStatusGet client
15. AccountStartupSet client
</code>
</pre>

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
#  post-up sudo route add default gw 10.77.77.1 dev vpn_soft # used for need g/w
</code>
</pre>
# [Case 2: Used dhclient command]
#### Make Script, Use to When VPN Started
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
ExecStart=sudo /home/pi/vpnclient/vpnstart       # Enter Your Script File's Directory
ExecStop=sudo /home/pi/vpnclient/vpnclient stop
[Install]
WantedBy=multi-user.target
</code>
</pre>