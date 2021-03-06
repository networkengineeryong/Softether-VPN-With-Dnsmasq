# 스크립트 실행 전 /etc/hosts에 vpnserver로 ip를 등록해야 합니다.
/etc/hosts
ex) 192.168.0.1     vpnserver

# /etc/hosts 에 vpnserver가 등록되어 있지 않거나, 잘못된 주소 일 경우 스크립트가 정상적으로 실행되지 않습니다!

# 카드 리더기가 eth1에 연결되어 있어야 정상적으로 동작하는 스크립트 입니다.

# 스크립트는 root계정에서 실행해야 합니다.

## 일반 보관함
<pre>
<code>
#!/bin/sh
apt-get -y update
cd /usr/local
wget https://www.softether-download.com/files/softether/v4.34-9745-rtm-2020.04.05-tree/Linux/SoftEther_VPN_Client/32bit_-_ARM_EABI/softether-vpnclient-v4.34-9745-rtm-2020.04.05-linux-arm_eabi-32bit.tar.gz
tar xvzf softether-vpnclient-v4.34-9745-rtm-2020.04.05-linux-arm_eabi-32bit.tar.gz
rm -f softether-vpnclient-v4.34-9745-rtm-2020.04.05-linux-arm_eabi-32bit.tar.gz
cd vpnclient
(echo 1; echo 1; echo 1) | make
./vpnclient start
./vpnclient stop
sed -i 's/'"$bool AllowRemoteConfig false"'/'"$bool AllowRemoteConfig true"'/' vpn_client.config
./vpnclient start
cat /etc/hosts | grep vpnserver >get-dns-ip-addr
sed -i 's/'vpnserver'/''/' get-dns-ip-addr
(echo 2;echo ;echo ;echo niccreate soft;echo AccountCreate client /server:"$(echo "$(cat get-dns-ip-addr | sed '/^$/d;s/[[:blank:]]//g'):443")" /hub:server /username:client /nicname:soft;echo AccountPasswordSet client /password:client /type:standard;echo AccountConnect client;echo AccountStartupSet client) | /usr/local/vpnclient/vpncmd
echo '#!/bin/sh
[Unit]
Description=SoftEther VPN Client
After=network.target
[Service]
Type=forking
ExecStart=/usr/local/vpnclient/vpnclient start
ExecStartPost=/bin/sleep 2
ExecStartPost=/sbin/dhclient vpn_soft
ExecStop=/usr/local/vpnclient/vpnclient stop
TimeoutSec=1728000
[Install]
WantedBy=multi-user.target' > /lib/systemd/system/vpnclient.service
systemctl daemon-reload
systemctl enable vpnclient
systemctl start vpnclient

</code>
</pre>

## 아파트 보관함 (카드 단말기 사용 시)
<pre>
<code>
#!/bin/sh
apt-get -y update
cd /usr/local
wget https://www.softether-download.com/files/softether/v4.34-9745-rtm-2020.04.05-tree/Linux/SoftEther_VPN_Client/32bit_-_ARM_EABI/softether-vpnclient-v4.34-9745-rtm-2020.04.05-linux-arm_eabi-32bit.tar.gz
tar xvzf softether-vpnclient-v4.34-9745-rtm-2020.04.05-linux-arm_eabi-32bit.tar.gz
rm -f softether-vpnclient-v4.34-9745-rtm-2020.04.05-linux-arm_eabi-32bit.tar.gz
cd vpnclient
(echo 1; echo 1; echo 1) | make
./vpnclient start
./vpnclient stop
sed -i 's/'"$bool AllowRemoteConfig false"'/'"$bool AllowRemoteConfig true"'/' vpn_client.config
./vpnclient start
cat /etc/hosts | grep vpnserver >get-dns-ip-addr
sed -i 's/'vpnserver'/''/' get-dns-ip-addr
(echo 2;echo ;echo ;echo niccreate soft;echo AccountCreate client /server:vpnserver:443 /hub:server /username:client /nicname:soft;echo AccountPasswordSet client /password:client /type:standard;echo AccountConnect client;echo AccountStartupSet client) | /usr/local/vpnclient/vpncmd
echo '#!/bin/sh
[Unit]
Description=SoftEther VPN Client
After=network.target
[Service]
Type=forking
ExecStart=/usr/local/vpnclient/vpnclient start
ExecStartPost=/bin/sleep 2
ExecStartPost=/sbin/dhclient vpn_soft
ExecStop=/usr/local/vpnclient/vpnclient stop
TimeoutSec=1728000
[Install]
WantedBy=multi-user.target' > /lib/systemd/system/vpnclient.service
systemctl enable vpnclient
apt-get -y install dnsmasq
systemctl enable dnsmasq
systemctl start dnsmasq
cat /dev/null > /etc/dnsmasq.conf
echo 'interface=eth1
dhcp-range=172.26.1.2,172.26.1.2,12h
dhcp-option=option:router,172.26.1.1
dhcp-leasefile=/var/lib/dnsmasq/dnsmasq.leases' > /etc/dnsmasq.conf
echo 'interface eth1
static ip_address=172.26.1.1/30' >> /etc/dhcpcd.conf
sed -i 's/'"#timeout 60;"'/'"timeout 1728000;"'/' /etc/dhcp/dhclient.conf
sysctl -w net.ipv4.ip_forward=1
echo '
iptables -t nat -A POSTROUTING -o vpn_soft -j MASQUERADE' >> /etc/iptables-hs
systemctl daemon-reload
</code>
</pre>