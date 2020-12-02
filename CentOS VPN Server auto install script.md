<pre>
<code>
#!/bin/sh
user=$USER
(echo y) | sudo yum update
wget 'https://www.softether-download.com/files/softether/v4.34-9745-rtm-2020.04.05-tree/Linux/SoftEther_VPN_Server/64bit_-_Intel_x64_or_AMD64/softether-vpnserver-v4.34-9745-rtm-2020.04.05-linux-x64-64bit.tar.gz'
tar xvzf softether-vpnserver-v4.34-9745-rtm-2020.04.05-linux-x64-64bit.tar.gz &&
sudo rm -r softether-vpnserver-v4.34-9745-rtm-2020.04.05-linux-x64-64bit.tar.gz
cd vpnserver
(echo 1; echo 1; echo 1) | make
sudo ./vpnserver start
sudo yum install dnsmasq
sudo systemctl enable dnsmasq
sudo systemctl start dnsmasq
(echo 1; echo ;echo ;echo HubCreate server /password:server;echo Hub server;echo SecurenatDisable; echo Natdisable; echo dhcpdisable;echo usercreate client /group: /realname:client /note: ;echo UserPasswordSet client /PASSWORD:client;echo BridgeCreate server /DEVICE:soft /TAP:yes) | /home/$USER/vpnserver/vpncmd
echo 'interface=tap_soft
dhcp-range=172.16.0.2,172.16.3.254,12h
dhcp-option=option:router,172.24.0.1/22
dhcp-leasefile=/var/lib/dnsmasq/dnsmasq.leases' > ~/dnsmasq.conf
sudo mv ~/dnsmasq.conf /etc/dnsmasq.conf
echo '#!/bin/sh
# chkconfig: 2345 99 01
# Description: Enable Softether by daemon.
DAEMON=/home/username/vpnserver/vpnserver
LOCK=/var/lock/subsys/vpnserver
TAP_ADDR=172.16.0.1

test -x $DAEMON || exit 0
case "$1" in
start)
$DAEMON start
touch $LOCK
sleep 1
/sbin/ifconfig tap_soft $TAP_ADDR/22
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
/sbin/ifconfig tap_soft $TAP_ADDR/22
;;
*)
echo "Usage: $0 {start|stop|restart}"
exit 1
esac
exit 0' > ~/initvpnserver
sudo sed -i 's/username/'$user'/' ~/initvpnserver
sudo mv ~/initvpnserver /etc/init.d/vpnserver
sudo chmod +x /etc/init.d/vpnserver
sudo chmod 755 /etc/init.d/vpnserver
sudo chkconfig --add vpnserver
sudo systemctl daemon-reload
sudo systemctl start vpnserver.service
sudo sysctl -w net.ipv4.ip_forward=1
firewall-cmd --zone=public --permanent --add-port=443/tcp
firewall-cmd --zone=public --permanent --add-port=67/udp
firewall-cmd --reload
echo '
======================================================
======================================================
SoftEther VPN Server install Successful, reboot please
======================================================
======================================================
'
</code>
</pre>