## __VPN Client 설치 전 숙지 해야할 내용__
- - -

* __모든 커맨드는 root 계정에서 입력합니다__

* __설치 경로: /usr/local__

# __목차__
[1. 사전 설정](#사전-설정)

[2. SoftEther VPN Client 설치](#softether-vpn-client-설치)

[3. /etc/hosts 파일에 vpnserver ip 지정](#hosts-파일에-vpnserver-ip-지정)

[4. SoftEther VPN Client 기본 설정](#softether-vpn-client-기본-설정)

[5. DHCP timeout 설정](#dhcp-timeout-설정)

[6. 서비스 등록](#서비스-등록)

[7. 카드리더기에 IP 할당](#카드리더기에-IP-할당)

[8. DNSMASQ 설정](#DNSMASQ-설정)

[9. 시스템 설정](#시스템-설정)

[10. 현장 설치 메뉴얼](#현장-설치-메뉴얼)
- - -
&nbsp;
## __사전 설정__

* apt update
<pre>
<code>apt -y update</code>
</pre>
&nbsp;
## __SoftEther VPN Client 설치__
&nbsp;&nbsp;&nbsp;[다운로드 링크](https://www.softether-download.com/en.aspx?product=softether)

    위 링크에서 설치 할 기기의 CPU 아키텍처에 맞는 버전의 다운로드 링크를 복사합니다

    테스트 환경: ARM_EABI x32 (OS Version : Linux raspberrypi 5.4.51-v7l+ #1333 SMP Mon Aug 10 16:51:40 BST 2020 armv7l GNU/Linux)

- - -

1. 다운로드 후 압축 해제

    VPN 클라이언트 설치 경로: __/usr/local__

    압축 해제 시 __vpnclient__ 파일이 생성됩니다
<pre>
<code>cd /usr/local

wget https://www.softether-download.com/files/softether/v4.34-9745-rtm-2020.04.05-tree/Linux/SoftEther_VPN_Client/32bit_-_ARM_EABI/softether-vpnclient-v4.34-9745-rtm-2020.04.05-linux-arm_eabi-32bit.tar.gz

tar xvzf softether-vpnclient-v4.34-9745-rtm-2020.04.05-linux-arm_eabi-32bit.tar.gz

rm -f softether-vpnclient-v4.34-9745-rtm-2020.04.05-linux-arm_eabi-32bit.tar.gz</code>
</pre>

2. make (라이선스 동의, 파일 생성하는 과정)
    * make 명령 입력 시 라이센스 동의를 묻는데 모두 1로 응답하면 됩니다
<pre>
<code>cd vpnserver

make</code>
</pre>

## __hosts 파일에 vpnserver ip 지정__

* VPN 서버와 연결할 때 사용하는 __IP__ 를 지정하는 부분입니다

* 아파트 모델의 경우 /etc/hosts 파일에 방재실에 설치되는 __서버의 IP__ 를 __vpnserver__ 로 추가합니다

* 예를 들어 방재실 서버 ip가 192.168.0.6일 때 다음과 같이 추가하면 됩니다
<pre>
<code>/etc/hosts
127.0.0.1       localhost
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters

127.0.1.1               raspberrypi
192.168.0.6     vpnserver</code>
</pre>

## __SoftEther VPN Client 기본 설정__
  
1. vpnclient 실행 후 중지 (config 파일 생성)
2. __/usr/local/vpnclient/vpn_client.config__ 파일에서 AllowRemoteConfig false 를 true로 변경 (VPN 원격 관리 허용)
<pre>
<code>./vpnclient start

./vpnclient stop

sed -i 's/'"$bool AllowRemoteConfig false"'/'"$bool AllowRemoteConfig true"'/' vpn_client.config

./vpnclient start</code>
</pre>

2. 클라이언트 설정

<pre>
<code>[root@localhost vpnserver]# ./vpncmd

1. 2 입력

2. 아무것도 입력하지 않고 엔터

3. 아무것도 입력하지 않고 엔터

VPN Client> NicCreate soft
# 가상 랜카드 생성 {실제 생성되는 인터페이스 = vpn_soft}

VPN Client> AccountCreate client /server:vpnserver:443 /hub:server /username:client /nicname:soft
# 연결 계정 설정 {계정명, 서버IP, 허브 명, 유저명, 가상 랜카드 명}

VPN Client> AccountPasswordSet client /password:client /type:standard
# 유저 비밀번호 설정 {서버에 설정된 비밀번호와 같아야 함}

VPN Client> AccountConnect client
# 서버에 연결

VPN Client> AccountStartupSet client
# 부팅 시 자동 연결 설정</code>
</pre>

## __DHCP timeout 설정__

* 네트워크 문제로 VPN Server와 연결이 끊어졌을 때 다시 연결을 시도하는데 기본으로 60초 동안 시도하도록 되어있다

* 설정을 1728000초(20일)로 변경해서 네트워크 문제가 해결된 후에도 VPN Server와 연결을 맺도록 설정 한다

* __/etc/dhcp/dhclient.conf__ 파일에 #timeout 60; 주석해제 후 값 변경
<pre>
<code>sed -i 's/'"#timeout 60;"'/'"timeout 1728000;"'/' /etc/dhcp/dhclient.conf</code>
</pre>

## __서비스 등록__

* SoftEther VPN 클라이언트 서비스에 등록, 시작 프로그램 등록

* VPN이 실행된 후 내부 인터넷과의 통신은 그대로 이더넷을 사용해야 한다.

* 일반적으로 아파트 홈 네트워크는 10.0.0.0 대역대의 아이피를 사용한다

* /sbin/ip route add 10.0.0.0/8 dev eth0 명령을 추가해주면 10으로 시작하는 모든 내부인터넷은 eth0을 사용하게 된다
<pre>
<code>vi /lib/systemd/system/vpnclient.service</code>
</pre>

<pre>
<code>[Unit]
Description=SoftEther VPN Client
After=network.target
[Service]
Type=forking
ExecStart=sudo /usr/local/vpnclient/vpnclient start
ExecStartPost=/bin/sleep 2
ExecStartPost=/sbin/dhclient vpn_soft
ExecStartPost=/sbin/ip route add 10.0.0.0/8 dev eth0
ExecStop=sudo /usr/local/vpnclient/vpnclient stop
KillMode=control-group
Restart=on-failure
TimeoutSec=1728000
[Install]
WantedBy=multi-user.target</code>
</pre>

2. daemon reload, VPN Client 시작프로그램 등록, 시작
<pre>
<code>systemctl daemon-reload
systemctl enable vpnclient
systemctl start vpnclient</code>
</pre>

## __카드리더기에 IP 할당__

* 라즈베리 파이에 유선 랜카드를 장착해 카드 리더기를 연결하게 되면 네트워크 인터페이스가 하나 추가 되는데

* 추가된 인터페이스가  뭔지 모르겠다면 연결하기 전, 후 ifconfig커맨드로 정보를 비교한다

* ip 할당 전 이기 때문에 MAC주소 정보만 출력되어 있다
<pre>
<code>[root@localhost]# ifconfig
eth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        ether ff:ff:ff:ff:ff:ff  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 28  bytes 3343 (3.2 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0</code>
</pre>

## __DNSMASQ 설정__

    카드리더기가 eth1으로 연결되어 있다 가정하고 작성된 코드 입니다

    만약 eth1이 아닌 다른 인터페이스로 연결 되어 있다면 (인터페이스 설정) 이라고 강조되어 있는 부분의 코드를 수정해서 사용해야 합니다

1. eth1 static ip 설정  __(인터페이스 설정)__

    __/etc/dhcpcd.conf__ 파일 하단에 다음 내용 추가
<pre>
<code>interface eth1
static ip_address=172.26.1.1/30</code>
</pre>

2. dnsmasq 설정  __(인터페이스 설정)__

    __/etc/dnsmasq.conf__ 파일 하단에 다음 내용 추가
<pre>
<code>interface=eth1
dhcp-range=eth1,172.26.1.2,172.26.1.2,12h
dhcp-option=eth1,3,172.26.1.1</code>
</pre>

## __시스템 설정__

1. ipv4 포워드 설정
<pre>
<code>sysctl -w net.ipv4.ip_forward=1</code>
</pre>

2. __/etc/iptables-hs__ 파일에 MASQUERADE추가

    현재 사용중인 AP모드에 필요한 규칙이 있는 파일입니다

    파일 하단에 다음 내용을 추가하면 됩니다
    
    __iptables -t nat -A POSTROUTING -o vpn_soft -j MASQUERADE__

&nbsp;

# __현장 설치 메뉴얼__

## /etc/hosts 파일에 vpnserver ip 지정 

    사무실에서 기본 설정을 마친 라즈베리 파이를 현장에 설치 후 (랜선 연결 까지 완료 후)

    __/etc/hosts__ 파일에 __vpnserver__ 의 __IP__ 를 __현장 서버 IP__ 로 변경 혹은 추가해 주면 됩니다 ([참고](#hosts-파일에-vpnserver-ip-지정))

&nbsp;

## VPN 연결 확인
* 인터페이스 vpn_soft에 ip가 할당되어야 한다

    <pre>
    <code>[root@raspberrypi]# ifconfig
    vpn_soft: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.25.1.10  netmask 255.255.255.224  broadcast 172.25.1.15
        inet6 fe80::5c5f:6cff:fe2c:abb9  prefixlen 64  scopeid 0x20<link>
        ether 5e:5f:6c:2c:ab:b9  txqueuelen 1000  (Ethernet)
        RX packets 4058  bytes 288117 (281.3 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 4897  bytes 352284 (344.0 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0</code></pre>
* vpn_soft로 default 게이트웨이가 추가 되어야 한다
    <pre>
    <code>[root@raspberrypi]# route
    Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
    default         172.25.1.1      0.0.0.0         UG    0      0        0 vpn_soft</code></pre>

&nbsp;

## 카드 단말기 아이피 할당 체크
    <pre>
    <code>vi /var/lib/misc/dnsmasq.leases</code></pre>
    위 파일에 __172.26.1.2__ 의 정보가 추가 되어 있는지 확인한다

&nbsp;

## traceroute 명령으로 트래픽 경로 추적

* 10.0.0.1 (홈 네트워크 메인 라우터)까지의 경로를 확인한다, 거치는 경로에 172.25.1.1가 없어야 한다.
<pre>
<code>[root@localhost ~]# traceroute 10.0.0.1
10.x.0.1  3.370 ms  4.031 ms  4.366 ms    # 동 라우터
10.0.0.1    6.407 ms  6.359 ms  6.523 ms  # 메인 라우터
</code></pre>

* 8.8.8.8 (외부 아이피) 까지의 경로를 확인한다

* 확인하는 시점에선 네트워크가 안 들어와 있을테니 경로에 172.25.1.1이 있는지 확인한다.

<pre>
<code>[root@localhost ~]# traceroute 8.8.8.8
172.25.1.1  3.370 ms  4.031 ms  4.366 ms  # VPN 서버</code></pre>

&nbsp;

## 카드 단말기가 __없는__ 아파트 모델일 경우
    
* vpnclient 서비스를 __disable__, __stop__

<pre>
<code>systemctl disable vpnclient
systemctl stop vpnclient</code></pre>

* __/etc/dnsmasq.conf__ 파일에 __eth1__ 관련 설정들 __주석처리__

<pre>
<code>#interface=eth1
#dhcp-range=eth1,172.26.1.2,172.26.1.2,12h
#dhcp-option=eth1,3,172.26.1.1</code></pre>
