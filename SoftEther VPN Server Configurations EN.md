Softether VPN Client Configurations
====================================
### OS Version : Linux ubuntu 5.4.0-52-generic #57-Ubuntu SMP Thu Oct 15 10:57:00 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
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
2. wget https://www.softether-download.com/files/softether/v4.34-9745-rtm-2020.04.05-tree/Linux/SoftEther_VPN_Server/64bit_-_Intel_x64_or_AMD64/softether-vpnserver-v4.34-9745-rtm-2020.04.05-linux-x64-64bit.tar.gz
3. sudo tar xvzf softether-vpnserver ...
</code>
</pre>
#### Download VPN Client, Unzip it.
<pre>
<code>
4. cd vpnserver
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
<pre>
<code>
6. sudo vi /lib/systemd/system/vpnclient.service
</code>
</pre>
#### Create Vpnserver Service, Use to Startup Configuration
<pre>
<code>
#!/bin/sh​
[Unit]​
Description=SoftEther VPN Server​
After=network.target​
[Service]​
Type=forking​
ExecStart=/home/server/vpnserver/vpnserver start​ # enter your vpnserver directory
ExecStop=/home/server/vpnserver/vpnserver stop​   #              ""
[Install]​
WantedBy=multi-user.target
</code>
</pre>
<pre>
<code>
7. sudo systemctl enable vpnserver
8. sudo systemctl start vpnserver
</code>
</pre>
#### Vpnserver Startup Set, Start Vpnclient

### VPN Server Management Commands
<https://www.softether.org/4-docs/1-manual/6._Command_Line_Management_Utility_Manual/6.3_VPN_Server_%2F%2F_VPN_Bridge_Management_Command_Reference_(For_Entire_Server)>
<pre>
<code>
9. ./vpncmd > 1 / enter / enter

By using vpncmd program, the following can be achieved. 

1. Management of VPN Server or VPN Bridge 
2. Management of VPN Client
3. Use of VPN Tools (certificate creation and Network Traffic Speed Test Tool)

Select 1, 2 or 3: 1

Specify the host name or IP address of the computer that the destination VPN Server or VPN Bridge is operating on. 
By specifying according to the format 'host name:port number', you can also specify the port number. 
(When the port number is unspecified, 443 is used.)
If nothing is input and the Enter key is pressed, the connection will be made to the port number 8888 of localhost (this computer).
Hostname of IP Address of Destination: 

If connecting to the server by Virtual Hub Admin Mode, please input the Virtual Hub name. 
If connecting by server admin mode, please press Enter without inputting anything.
Specify Virtual Hub Name: 
</code>
</pre>
#### Select 1, VPN Server, Skip Destination Address, Hub name
<pre>
<code>
10. HubCreate server > enter pass
11. Hub server
12. SecureNatDisable
13. NatDisable
14. DhcpDisable
15. UserCreate client
16. UserPasswordSet client > enter pass
17. BridgeCreate server /DEVICE:soft /TAP:yes
</code>
</pre>
<pre>
<code>
18. sudo apt install dnsamsq
19. sudo nano /etc/dnsmasq.conf

interface=tap_soft
dhcp-range=10.77.77.77,10.77.77.177,1m
dhcp-option=option:router,10.77.77.1
dhcp-leasefile=/var/lib/dnsmasq/dnsmasq.leases
</code>
</pre>
#### dnsmasq Setting, netmask : 10.0.0.0/8, g/w : 10.77.77.1, lease-range : 10.77.77.77 - 10.77.77.177 lease for 1 min
<pre>
<code>
20. sudo systemctl enable dnsmasq
21. sudo systemctl start dnsmasq
</code>
</pre>
#### dnsmasq Startup Set, Start dnsmasq
<pre>
<code>
22. sudo nano /etc/init.d/vpnserver

#!/bin/sh
### BEGIN INIT INFO
# Provides: vpnserver
# Required-Start: $remote_fs $syslog
# Required-Stop: $remote_fs $syslog
# Default-Start: 2 3 4 5
# Default-Stop: 0 1 6
# Short-Description: Start daemon at boot time
# Description: Enable Softether by daemon.
### END INIT INFO
DAEMON=/home/server/vpnserver/vpnserver # need to change
LOCK=/var/lock/subsys/vpnserver
TAP_ADDR=10.77.77.1 # need to change

test -x $DAEMON || exit 0
case "$1" in
start)
$DAEMON start
touch $LOCK
sleep 1
/sbin/ifconfig tap_wlan $TAP_ADDR/8 # need to change
iptables -t nat -A POSTROUTING -j MASQUERADE
;;
stop)
$DAEMON stop
rm $LOCK
;;
restart)
$DAEMON stop
sleep 3
$DAEMON start
sleep 1
/sbin/ifconfig tap_wlan $TAP_ADDR/8 # need to change
;;
*)
echo "Usage: $0 {start|stop|restart}"
exit 1
esac
exit 0
</code>
</pre>
#### Script When Started Vpnserver
<pre>
<code>
23. sudo sysctl -w net.ipv4.ip_forward=1
24. sudo reboot
</code>
</pre>
#### ip_forward Enable And Reboot
