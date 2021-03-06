# 스크립트는 root계정에서 실행해야 합니다.

<pre>
<code>
#!/bin/sh
(echo y) | yum update
cd /usr/local
wget 'https://www.softether-download.com/files/softether/v4.34-9745-rtm-2020.04.05-tree/Linux/SoftEther_VPN_Server/64bit_-_Intel_x64_or_AMD64/softether-vpnserver-v4.34-9745-rtm-2020.04.05-linux-x64-64bit.tar.gz'
tar xvzf softether-vpnserver-v4.34-9745-rtm-2020.04.05-linux-x64-64bit.tar.gz &&
rm softether-vpnserver-v4.34-9745-rtm-2020.04.05-linux-x64-64bit.tar.gz
cd vpnserver
(echo 1; echo 1; echo 1) | make
./vpnserver start
yum install dnsmasq
systemctl enable dnsmasq
systemctl start dnsmasq
(echo 1; echo ;echo ;echo HubCreate server /password:server;echo Hub server;echo SecurenatDisable; echo Natdisable; echo dhcpdisable;echo usercreate client /group: /realname:client /note: ;echo UserPasswordSet client /PASSWORD:client;echo BridgeCreate server /DEVICE:soft /TAP:yes) | /usr/local/vpnserver/vpncmd
echo '
interface=tap_soft
dhcp-range=172.25.1.2,172.25.1.30,12h
dhcp-option=option:router,172.25.1.1
dhcp-leasefile=/var/lib/dnsmasq/dnsmasq.leases
' >> /etc/dnsmasq.conf
echo '#!/bin/sh
# chkconfig: 2345 99 01
# Description: Enable Softether by daemon.
DAEMON=/usr/local/vpnserver/vpnserver
LOCK=/var/lock/subsys/vpnserver
TAP_ADDR=172.25.1.1

test -x $DAEMON || exit 0
case "$1" in
start)
$DAEMON start
touch $LOCK
sleep 1
/sbin/ifconfig tap_soft $TAP_ADDR/27
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
/sbin/ifconfig tap_soft $TAP_ADDR/27
;;
*)
echo "Usage: $0 {start|stop|restart}"
exit 1
esac
exit 0' > /etc/init.d/vpnserver
chmod +x /etc/init.d/vpnserver
chmod 755 /etc/init.d/vpnserver
chkconfig --add vpnserver
systemctl daemon-reload
systemctl start vpnserver
sysctl -w net.ipv4.ip_forward=1
firewall-cmd --zone=public --permanent --add-port=443/tcp
firewall-cmd --zone=public --permanent --add-port=67/udp
firewall-cmd --permanent --direct --add-rule ipv4 nat POSTROUTING 0 -j MASQUERADE
firewall-cmd --permanent --zone=trusted --add-interface=tap_soft
firewall-cmd --reload
</code>
</pre>