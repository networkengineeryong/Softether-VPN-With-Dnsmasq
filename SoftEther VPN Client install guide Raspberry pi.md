# VPN Client on Raspberry pi

# 모든 커맨드는 root 권한으로 입력합니다..

1. apt update
<pre>
<code>
apt update
</code>
</pre>
OS Version : Linux raspberrypi 5.4.51-v7l+ #1333 SMP Mon Aug 10 16:51:40 BST 2020 armv7l GNU/Linux

설치 할 기기의 CPU 아키텍처에 맞는 버전의 다운로드 링크를 복사, 테스트 환경: ARM_EABI x32

<https://www.softether-download.com/en.aspx?product=softether>

2. Client 다운로드 후 압축 해제

    __라즈베리 파이 VPN 설치 경로: /usr/local/__
<pre>
<code>
wget https://www.softether-download.com/files/softether/v4.34-9745-rtm-2020.04.05-tree/Linux/SoftEther_VPN_Client/32bit_-_ARM_EABI/softether-vpnclient-v4.34-9745-rtm-2020.04.05-linux-arm_eabi-32bit.tar.gz
tar xvzf softether-vpnclient-v4.34-9745-rtm-2020.04.05-linux-arm_eabi-32bit.tar.gz
rm softether-vpnclient-v4.34-9745-rtm-2020.04.05-linux-arm_eabi-32bit.tar.gz

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

    아파트 모델의 경우 방재실에 설치되는 서버의 ip를 입력한다.

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
./vpnclient start
./vpnclient stop
sed -i 's/'"$bool AllowRemoteConfig false"'/'"$bool AllowRemoteConfig true"'/' vpn_client.config
./vpnclient start
cat /etc/hosts | grep vpnserver > get-dns-ip-addr
sed -i 's/'vpnserver'/''/' get-dns-ip-addr
(echo 2;echo ;echo ;echo niccreate soft; echo AccountCreate client /server:"$(echo "$(cat /usr/local/vpnclient/get-dns-ip-addr | sed '/^$/d;s/[[:blank:]]//g'):443")" /hub:server /username:client /nicname:soft;echo AccountPasswordSet client /password:client /type:standard;echo AccountConnect client;echo AccountStartupSet client) | /usr/local/vpnclient/vpncmd
rm  get-dns-ip-addr

</code>
</pre>

# 일반 보관함일 경우 (Default gateway delete)
5. vpn 클라이언트 서비스에 등록, 시작 프로그램 등록
<pre>
<code>  
echo '#!/bin/sh
[Unit]
Description=SoftEther VPN Client
After=network.target
[Service]
Type=forking
ExecStart=/usr/local/vpnclient/vpnclient start
ExecStartPost=/bin/sleep 2
ExecStartPost=/sbin/dhclient vpn_soft
ExecStartPost=/bin/sleep 2
ExecStartPost=/sbin/route del default dev vpn_soft
ExecStop=/usr/local/vpnclient/vpnclient stop
[Install]
WantedBy=multi-user.target' > /lib/systemd/system/vpnclient.service
systemctl daemon-reload
systemctl enable vpnclient
systemctl start vpnclient
</code>
</pre>
__일반 보관함은 여기까지만 설정하면 된다__

# 아파트 모델일 경우
5. vpn 클라이언트 서비스에 등록, 시작 프로그램 등록
<pre>
<code>  
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
[Install]
WantedBy=multi-user.target' > /lib/systemd/system/vpnclient.service
systemctl daemon-reload
systemctl enable vpnclient
systemctl start vpnclient

</code>
</pre>

##   카드 리더기 인터페이스 정보 조회 방법
라즈베리 파이에 유선 랜카드를 장착해 카드 리더기를 연결하게 되면

네트워크 인터페이스가 하나 추가 되는데, 이 인터페이스가 뭔지 모르겠다면 연결하기 전, 후 ifconfig커맨드로 정보를 비교한다.

ip 할당 전 이기 때문에 MAC주소 정보만 출력되어 있다.
<pre>
<code>
[root@localhost]# ifconfig
eth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        ether ff:ff:ff:ff:ff:ff  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 28  bytes 3343 (3.2 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
</code>
</pre>

__카드리더기가 eth1으로 연결되어 있다 가정하고 작성된 코드 입니다__

1. eth1 static ip 설정
<pre>
<code>
echo 'interface 'eth1'
static ip_address=192.168.224.1/30' >> /etc/dhcpcd.conf
</code>
</pre>
2. Dnsmasq 설치
<pre>
<code>
apt install dnsmasq
systemctl enable dnsmasq
systemctl start dnsmasq
</code>
</pre>
3. Dnsmasq 설정
<pre>
<code>
echo '
interface='eth1'
dhcp-range=192.168.224.2,192.168.224.2,12h
dhcp-option=option:router,192.168.224.1
dhcp-leasefile=/var/lib/dnsmasq/dnsmasq.leases
' >> /etc/dnsmasq.conf
</code>
</pre>
4. ipv4 포워드 설정
<pre>
<code>
sysctl -w net.ipv4.ip_forward=1
</code>
</pre>
5. iptables 설정 (MASQUERADE 설정 추가)
<pre>
<code>
iptables -t nat -A POSTROUTING -j MASQUERADE 
iptables-save > /etc/iptables.ipv4.nat
</code>
</pre>
6. iptables 설정 부팅시 불러오게 설정

    /etc/rc.local 에 다음 내용 추가
<pre>
<code>
iptables-restore < /etc/iptables.ipv4.nat
</code>
</pre>