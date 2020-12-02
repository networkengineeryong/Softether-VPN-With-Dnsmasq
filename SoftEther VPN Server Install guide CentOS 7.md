# VPN SERVER on CentOS 7
====================================
1. yum update
<pre>
<code>
sudo yum update
</code>
</pre>
2. 자신의 CPU 아키텍처와 맞는 버전의 다운로드 링크 복사. 테스트 환경 :Linux x64 (CentOS 7)

    wget으로 파일을 다운로드 받고, 압축을 푼 뒤 vpnserver를 실행한다.

    <https://www.softether-download.com/en.aspx?product=softether>

<pre>
<code>
wget https://www.softether-download.com/files/softether/v4.34-9745-rtm-2020.04.05-tree/Linux/SoftEther_VPN_Server/64bit_-_Intel_x64_or_AMD64/softether-vpnserver-v4.34-9745-rtm-2020.04.05-linux-x64-64bit.tar.gz
sudo tar xvzf softether-vpnserver-v4.34-9745-rtm-2020.04.05-linux-x64-64bit.tar.gz
sudo rm -r softether-vpnserver-v4.34-9745-rtm-2020.04.05-linux-x64-64bit.tar.gz
cd vpnserver
(echo 1; echo 1; echo 1) | make
./vpnserver start
</code>
</pre>
3. DNSMASQ 설치, 시작 프로그램 등록
<pre>
<code>
sudo yum install dnsmasq
sudo systemctl enable dnsmasq
sudo systemctl start dnsmasq
</code>
</pre>
4. VPN Server 기본 설정
    명령어를 입력하면 Hub: server, pw: server / user client, pw:client / SecureNAT, NAT, DHCP가 비활성화 되고, TAP Device가 생성된다.
<pre>
<code>
(echo 1; echo ;echo ;echo HubCreate server /password:server;echo Hub server;echo SecurenatDisable; echo Natdisable; echo dhcpdisable; usercreate client /group: /realname:client /note: ;echo UserPasswordSet client /PASSWORD:client;echo BridgeCreate server /DEVICE:soft /TAP:yes) | /home/$USER/vpnserver/vpncmd
</code>
</pre>

### 자세한 설정 방법
<https://github.com/networknegineeryong/Softether-VPN-With-Dnsmasq/blob/main/SoftEther%20VPN%20Server%20config%20guide.md#%EC%84%9C%EB%B2%84-%EA%B4%80%EB%A6%AC-%EB%AA%85%EB%A0%B9%EC%96%B4>

5. dnsmasq 설정
<pre>
<code>
sudo sh -c "cat /dev/null > /etc/dnsmasq.conf"
echo 'interface=tap_soft
dhcp-range=172.16.0.2,172.16.3.254,12h
dhcp-option=option:router,172.24.0.1/22
dhcp-leasefile=/var/lib/dnsmasq/dnsmasq.leases' > ~/dnsmasq.conf
sudo sh -c "cat ~/dnsmasq.conf > /etc/dnsmasq.conf"
sudo rm ~/dnsmasq.conf
</code>
</pre>
6. VPN SERVER init.d 스크립트 등록
<pre>
<code>
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
</code>
</pre>
7. daemon-reload, start
<pre>
<code>
sudo systemctl daemon-reload
sudo systemctl start vpnserver.service
</code>
</pre>
8. ipv4 포워딩 설정
<pre>
<code>
sudo sysctl -w net.ipv4.ip_forward=1
</code>
</pre>
9. VPN port 443/TCP, DHCP port 67/UDP 개방
<pre>
<code>
firewall-cmd --zone=public --permanent --add-port=443/tcp
firewall-cmd --zone=public --permanent --add-port=67/udp
firewall-cmd --reload
</code>
</pre>