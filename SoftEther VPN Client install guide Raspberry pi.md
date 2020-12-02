# VPN Client on Raspberry pi

## 모든 커맨드는 일반 유저 권한으로 입력합니다.

1. apt update
<pre>
<code>
sudo apt update
</code>
</pre>
OS Version : Linux raspberrypi 5.4.51-v7l+ #1333 SMP Mon Aug 10 16:51:40 BST 2020 armv7l GNU/Linux

자신의 CPU 아키텍처에 맞는 버전의 다운로드 링크를 복사, 테스트 환경: ARM_EABI x32

<https://www.softether-download.com/en.aspx?product=softether>

2. Client 다운로드 후 압축 해제
<pre>
<code>
wget https://www.softether-download.com/files/softether/v4.34-9745-rtm-2020.04.05-tree/Linux/SoftEther_VPN_Client/32bit_-_ARM_EABI/softether-vpnclient-v4.34-9745-rtm-2020.04.05-linux-arm_eabi-32bit.tar.gz
tar xvzf softether-vpnclient-v4.34-9745-rtm-2020.04.05-linux-arm_eabi-32bit.tar.gz
sudo rm -r softether-vpnclient-v4.34-9745-rtm-2020.04.05-linux-arm_eabi-32bit.tar.gz
</code>
</pre>
3. vpnclient 실행, make (라이선스 동의)
<pre>
<code>
cd vpnclient
(echo 1; echo 1; echo 1) | make
</code>
</pre>
4. vpn 클라이언트 계정 생성, 서버 연결하기

    /etc/hosts 파일에 vpnserver로 서버 ip를 지정해 놓으면 서버에 연결하는 커맨드다.

    만약 hosts에 vpnserver가 없어 계정 생성이 안된다면 수동으로 계정을 생성해야 한다. (링크 참조)

    <https://github.com/networknegineeryong/Softether-VPN-With-Dnsmasq/blob/main/SoftEther%20VPN%20Client%20config%20guide.md>

<pre>
<code>
/etc/hosts
127.0.0.1       localhost
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters

127.0.1.1               raspberrypi
192.168.0.6     vpnserver
</code>
</pre>
<pre>
<code>
cat /etc/hosts | grep vpnserver > ~/test
sed -i 's/'vpnserver'/''/' ~/test
(echo 2;echo ;echo ;echo AccountCreate client /server:"$(echo "$(cat ~/test | sed '/^$/d;s/[[:blank:]]//g'):443")" /hub:server /username:client /nicname:soft;echo AccountPasswordSet client /password:client /type:standard;echo AccountConnect client;echo AccountStartupSet client) | /home/$USER/vpnclient/vpncmd
sudo rm -r ~/test
</code>
</pre>

5. vpn 클라이언트 서비스에 등록, 시작 프로그램 등록
<pre>
<code>  
user=$USER
echo '#!/bin/sh
[Unit]
Description=SoftEther VPN Client
After=network.target
[Service]
Type=forking
ExecStart=/home/username/vpnclient/vpnclient start
ExecStartPost=/bin/sleep 2
ExecStartPost=/sbin/dhclient vpn_soft
ExecStop=/home/username/vpnclient/vpnclient stop
[Install]
WantedBy=multi-user.target' > ~/vpnservice
sudo sed -i 's/username/'$user'/' ~/vpnservice
sudo mv ~/vpnservice /lib/systemd/system/vpnclient.service
sudo systemctl daemon-reload
sudo systemctl enable vpnclient
sudo systemctl start vpnclient
</code>
</pre>

# 라즈베리 파이에 연결된 기기의 트래픽을 내보내야 할 때

인터넷은 eth0, 기기는 eth1에 연결되어 있다고 가정한다

eth0 ip = 192.168.0.178

1. eth1 static ip 설정
<pre>
<code>
sudo sh -c "echo 'interface eth1
static ip_address=192.168.224.1/30' >> /etc/dhcpcd.conf"
</code>
</pre>
2. Dnsmasq 설치
<pre>
<code>
sudo apt install dnsmasq
sudo systemctl enable dnsmasq
sudo systemctl start dnsmasq
</code>
</pre>
3. Dnsmasq 설정
<pre>
<code>
sudo sh -c "cat /dev/null > /etc/dnsmasq.conf"
echo 'interface=eth1
dhcp-range=192.168.224.2,192.168.224.2,12h
dhcp-option=option:router,192.168.224.1/30
dhcp-leasefile=/var/lib/dnsmasq/dnsmasq.leases' > ~/dnsmasq.conf
sudo sh -c "cat ~/dnsmasq.conf > /etc/dnsmasq.conf"
sudo rm ~/dnsmasq.conf
</code>
</pre>
4. ipv4 포워드 설정
<pre>
<code>
sudo sysctl -w net.ipv4.ip_forward=1
</code>
</pre>
5. iptables 설정
<pre>
<code>
sudo iptables -t nat -A PREROUTING -j DNAT --to 192.168.0.178
sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"

sudo nano /etc/rc.local 에 다음 내용 추가
iptables-restore < /etc/iptables.ipv4.nat
</code>
</pre>