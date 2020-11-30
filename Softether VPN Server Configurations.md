Softether VPN Client Configurations
====================================
###### OS Version : Linux ubuntu 5.4.0-52-generic #57-Ubuntu SMP Thu Oct 15 10:57:00 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux

1. sudo apt update
2. wget https://www.softether-download.com/files/softether/v4.34-9745-rtm-2020.04.05-tree/Linux/SoftEther_VPN_Server/64bit_-_Intel_x64_or_AMD64/softether-vpnserver-v4.34-9745-rtm-2020.04.05-linux-x64-64bit.tar.gz
3. sudo tar xvzf softether-vpnserver ...
4. cd vpnserver
5. make > 1 / 1 / 1
6. sudo vi /lib/systemd/system/vpnserver.service
<pre>
<code>
#!/bin/sh​
[Unit]​
Description=SoftEther VPN Server​
After=network.target​
[Service]​
Type=forking​
ExecStart=/home/server/vpnserver/vpnserver start​
ExecStop=/home/server/vpnserver/vpnserver stop​
[Install]​
WantedBy=multi-user.target
</code>
</pre>
7. sudo systemctl enable vpnserver
8. sudo systemctl start vpnserver
9. ./vpncmd > 1 / enter / enter
10. HubCreate server > enter pass
11. Hub server
12. SecureNatDisable
13. NatDisable
14. DhcpDisable
15. UserCreate client
16. UserPasswordSet client > enter pass
17. BridgeCreate server /DEVICE:soft /TAP:yes
18. sudo apt install dnsamsq
19. sudo nano /etc/dnsmasq.conf
<pre>
<code>
interface=tap_soft
dhcp-range=10.77.77.77,10.77.77.177,1m
dhcp-option=option:router,10.77.77.1
dhcp-leasefile=/var/lib/dnsmasq/dnsmasq.leases
</code>
</pre>
20. sudo systemctl enable dnsmasq
21. sudo systemctl start dnsmasq

22. sudo nano /etc/init.d/vpnserver
<pre>
<code>
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
23. sudo sysctl -w net.ipv4.ip_forward=1
24. sudo reboot