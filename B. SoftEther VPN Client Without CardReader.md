## __VPN Client 설치 전 숙지 해야할 내용__
- - -

* __모든 커맨드는 root 계정에서 입력합니다__

* __설치 경로: /usr/local__

* 현장 설치 후 vpn_soft 인터페이스에 아이피 할당이 되었는지 확인해주세요

* vpn_soft 인터페이스로 디폴트 게이트웨이가 생성되지 않았는지 확인해주세요

<pre>
<code>[root@raspberrypi]# route
default         router.asus.com 0.0.0.0         UG    202    0        0 eth0
172.25.1.0      0.0.0.0         255.255.255.240 U     0      0        0 vpn_soft
192.168.0.0     0.0.0.0         255.255.255.0   U     202    0        0 eth0</code></pre>

* 위와 같이 default 설정에 eth0만 올라와 있어야 합니다

# __목차__
[1. 사전 설정](#사전-설정)

[2. SoftEther VPN Client 설치](#softether-vpn-client-설치)

[3. /etc/hosts 파일에 vpnserver ip 지정](#hosts-파일에-vpnserver-ip-지정) 

[4. SoftEther VPN Client 기본 설정](#softether-vpn-client-기본-설정)

[5. DHCP timeout 설정](#dhcp-timeout-설정)

[6. 서비스 등록](#서비스-등록)
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
<code>cd vpnclient

make</code>
</pre>

## __hosts 파일에 vpnserver ip 지정__

* VPN 서버와 연결할 때 사용하는 __IP__ 를 지정하는 부분입니다
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

VPN Client> AccountCreate client /server:vpnserver:443 /hub:VPN /username:client /nicname:soft
# 연결 계정 설정 {계정명, 서버IP, 허브명, 유저명, 가상 랜카드 명}

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

    __/lib/systemd/system/vpnclient.service 파일 생성 후 다음 내용 추가__
<pre>
<code>#!/bin/sh
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
WantedBy=multi-user.target</code>
</pre>

2. daemon reload, VPN Client 시작프로그램 등록, 시작
<pre>
<code>systemctl daemon-reload
systemctl enable vpnclient
systemctl start vpnclient</code>
</pre>
