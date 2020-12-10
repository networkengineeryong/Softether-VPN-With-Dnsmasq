## __VPN Server 설치 전 숙지 해야할 내용__
- - -

* __모든 커맨드는 root 계정에서 입력합니다__

* __SoftEther VPN Server 설치 경로: /usr/local__

*  __Default 게이트웨이가 VPN으로__ 설정되어야 카드 리더기 트래픽이 인터넷으로 나갈 수 있습니다

    * 해당 설정은 __DNSMASQ__ 설정 부분에 있으니 참고해 주세요


# __목차__

[1. 사전 설정](#사전-설정)

[2. SoftEther VPN Server 설치](#SoftEther-VPN-Server-설치)

[3. DNSMASQ 설정](#DNSMASQ-설정)

[4. 서비스 파일 생성](#서비스-파일-생성)

[5. 포워딩 설정](#포워딩-설정)

[6. 방화벽 설정](#방화벽-설정)
- - -
&nbsp;
## __사전 설정__

* yum update
<pre>
<code>yum -y update</code>
</pre>
&nbsp;
## __SoftEther VPN Server 설치__
&nbsp;&nbsp;&nbsp;[다운로드 링크](https://www.softether-download.com/en.aspx?product=softether)

    위 링크에서 자신의 CPU 아키텍처와 맞는 버전의 다운로드 링크를 복사합니다

    테스트 환경 :Linux x64 (CentOS 7)

- - -

1. 다운로드 후 압축 해제

    __VPN 서버 설치 경로: /usr/local__

    압축 해제 시 vpnserver 파일이 생성됩니다

<pre>
<code>cd /usr/local

wget https://www.softether-download.com/files/softether/v4.34-9745-rtm-2020.04.05-tree/Linux/SoftEther_VPN_Server/64bit_-_Intel_x64_or_AMD64/softether-vpnserver-v4.34-9745-rtm-2020.04.05-linux-x64-64bit.tar.gz

tar xvzf softether-vpnserver-v4.34-9745-rtm-2020.04.05-linux-x64-64bit.tar.gz

rm -f softether-vpnserver-v4.34-9745-rtm-2020.04.05-linux-x64-64bit.tar.gz</code>
</pre>

2. make (라이선스 동의, 파일 생성하는 과정)
    * make 명령 입력 시 라이센스 동의를 묻는데 모두 1로 응답하면 됩니다
<pre>
<code>cd vpnserver

make</code>
</pre>

3. VPN Server 기본 설정

    __DHCP, SecureNAT, NAT__ 기능을 비활성화 하는 이유는 해당기능을 __DNSMASQ__ 로 대체해서 사용할 것이기 때문입니다

    TAP Device 명: __soft__ (실제 인터페이스는 __tap_soft__ 로 표시된다)
<pre>
<code>[root@localhost vpnserver]# ./vpnserver start

[root@localhost vpnserver]# ./vpncmd

1. 1 입력

2. 아무것도 입력하지 않고 엔터

3. 아무것도 입력하지 않고 엔터

VPN Server> HubCreate server /password:server
# server이름의 허브 생성, 비밀번호 server

VPN Server/server> Hub server 
# 허브 server 설정으로 진입

VPN Server/server> SecurenatDisable
# Securenat 비활성화

VPN Server/server> NatDisable 
# NAT 비활성화

VPN Server/server> DhcpDisable 
# DHCP 비활성화

VPN Server/server> UserCreate client /group: /realname:client /note: 
# client 이름의 유저 생성

VPN Server/server> UserPasswordSet /PASSWORD:client  
# 유저 비밀번호 설정

VPN Server/server> BridgeCreate server /DEVICE:soft /TAP:yes 
# TAP Device 생성
</code>
</pre>

## __DNSMASQ__ 설정
   * __카드 리더기가 있는 아파트의 경우 한 VPN 네트워크에 많은 호스트가 필요없다__
   * IP 대역대를 __172.25.1.0/27__ 로 설정해 __30__ 개의 호스트를 갖게끔 설정한다

   * VPN 클라이언트에게 게이트웨이 정보를 넘겨준다
   * __/etc/dnsmasq.conf 파일 하단에 다음 내용 추가__
<pre>
<code>interface=tap_soft
dhcp-range=tap_soft,172.25.1.2,172.25.1.30,12h
dhcp-option=tap_soft,3,172.25.1.1</code>
</pre>

## __서비스 파일 생성__

1. __/etc/init.d__ 에 __vpnserver__ 파일 생성
<pre>
<code>[root@localhost init.d]# vi /etc/init.d/vpnserver</code>
</pre>

2. __vpnserver__ 파일에 다음 내용 붙여넣기
<pre>
<code>#!/bin/sh
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
exit 0</code>
</pre>
* __DAEMON=__ VPN Server 설치 경로 입력란

* __TAP_ADDR=__ TAP Device의 IP 입력란

&nbsp;

3. 권한설정, chkconfig
<pre>
<code>chmod +x /etc/init.d/vpnserver
chmod 755 /etc/init.d/vpnserver
chkconfig --add vpnserver</code>
</pre>

4. daemon-reload, start
<pre>
<code>systemctl daemon-reload
systemctl start vpnserver</code>
</pre>

## __포워딩 설정__

* ipv4 포워딩 설정
<pre>
<code>sysctl -w net.ipv4.ip_forward=1</code>
</pre>

## __방화벽 설정__

* 포트 오픈, MASQUERADE 설정

    * SoftEther VPN 사용 포트: 443/TCP

    * DNSMASQ 사용 포트: 67/UDP
<pre>
<code>firewall-cmd --zone=public --permanent --add-port=443/tcp
firewall-cmd --zone=public --permanent --add-port=67/udp
firewall-cmd --permanent --direct --add-rule ipv4 nat POSTROUTING 0 -j MASQUERADE
firewall-cmd --zone=trusted --permanent --add-interface=tap_soft
firewall-cmd --reload</code>
</pre>
