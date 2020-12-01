Softether VPN 서버 설정
====================================
### OS Version : Linux ubuntu 5.4.0-52-generic #57-Ubuntu SMP Thu Oct 15 10:57:00 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
***
### SoftEther VPN 다운로드 링크 복사
### 자신의 CPU 아키텍처에 맞는 버전의 링크를 복사, 본 글은 Linux x64 버전을 사용
<https://www.softether-download.com/en.aspx?product=softether>
***
<pre>
<code>
1. sudo apt update
</code>
</pre>
##### 새 패키지, 업그레이드가 필요한 패키지의 업그레이드를 위해 패키지 목록을 업데이트
<pre>
<code>
2. wget https://www.softether-download.com/files/softether/v4.34-9745-rtm-2020.04.05-tree/Linux/SoftEther_VPN_Server/64bit_-_Intel_x64_or_AMD64/softether-vpnserver-v4.34-9745-rtm-2020.04.05-linux-x64-64bit.tar.gz
3. sudo tar xvzf softether-vpnserver ...
</code>
</pre>
#### 서버 다운로드, 압축 풀기
<pre>
<code>
4. cd vpnserver
5. make

Do you want to read the License Agreement for this software ?

 1. Yes
 2. No

Please choose one of above number: 1

Did you read and understand the License Agreement ?
(If you couldn't read above text, Please read 'ReadMeFirst_License.txt'
 file with any text editor.)

 1. Yes
 2. No

Please choose one of above number: 1

Did you agree the License Agreement ?

1. Agree
2. Do Not Agree

Please choose one of above number: 1
</code>
</pre>
#### 라이선스 동의, 모든 질문에 1로 답변
### "The preparation of SoftEther VPN Server is completed !" < 파일 생성 성공 문구
#### 해당 문구가 뜨지않고 오류가 발생했을 경우 CPU와 VPN 서버 버전이 맞는지 확인
<pre>
<code>
6. sudo vi /lib/systemd/system/vpnclient.service
</code>
</pre>
#### 서버 서비스 파일 생성, 시작 프로그램 등록에 사용
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
<pre>
<code>
7. sudo systemctl enable vpnserver
8. sudo systemctl start vpnserver
</code>
</pre>
#### 서버 시작 프로그램에 등록하고, 실행

### VPN 서버 관리 명령어
<https://www.softether.org/4-docs/1-manual/6._Command_Line_Management_Utility_Manual/6.3_VPN_Server_%2F%2F_VPN_Bridge_Management_Command_Reference_(For_Entire_Server)>
<pre>
<code>
9. ./vpncmd

By using vpncmd program, the following can be achieved. 

1. Management of VPN Server or VPN Bridge 
2. Management of VPN Client
3. Use of VPN Tools (certificate creation and Network Traffic Speed Test Tool)

Select 1, 2 or 3: 1

Specify the host name or IP address of the computer that the destination VPN Server or VPN Bridge is operating on. 
By specifying according to the format 'host name:port number', you can also specify the port number. 
(When the port number is unspecified, 443 is used.)
If nothing is input and the Enter key is pressed, the connection will be made to the port number 8888 of localhost (this computer).
Hostname of IP Address of Destination: 

If connecting to the server by Virtual Hub Admin Mode, please input the Virtual Hub name. 
If connecting by server admin mode, please press Enter without inputting anything.
Specify Virtual Hub Name: 
</code>
</pre>
#### 1번 선택, 주소 입력, 허브 명 입력 스킵
<pre>
<code>
10. HubCreate server
11. Hub server
12. SecureNatDisable
13. NatDisable
14. DhcpDisable
15. UserCreate client
16. UserPasswordSet client
17. BridgeCreate server /DEVICE:soft /TAP:yes
</code>
</pre>
### VPN Server 기본 사용법, DNSMASQ g/w 인터페이스로 TAP Device를 사용
<pre>
<code>
18. sudo apt install dnsamsq
19. sudo nano /etc/dnsmasq.conf

interface=tap_soft
dhcp-range=10.77.77.77,10.77.77.177,1m
dhcp-option=option:router,10.77.77.1
dhcp-leasefile=/var/lib/dnsmasq/dnsmasq.leases
</code>
</pre>
#### dnsmasq 설정, 네트워크 : 10.0.0.0/8, 게이트웨이 : 10.77.77.1, 임대범위 : 10.77.77.77 - 10.77.77.177 1분동안 임대
<pre>
<code>
20. sudo systemctl enable dnsmasq
21. sudo systemctl start dnsmasq
</code>
</pre>
#### dnsmasq 시작프로그램 등록, 실행
<pre>
<code>
22. sudo nano /etc/init.d/vpnserver

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
DAEMON=/home/server/vpnserver/vpnserver 
LOCK=/var/lock/subsys/vpnserver
TAP_ADDR=10.77.77.1                     

test -x $DAEMON || exit 0
case "$1" in
start)
$DAEMON start
touch $LOCK
sleep 1
/sbin/ifconfig tap_soft $TAP_ADDR/8     
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
/sbin/ifconfig tap_soft $TAP_ADDR/8     
;;
*)
echo "Usage: $0 {start|stop|restart}"
exit 1
esac
exit 0
</code>
</pre>
#### VPN 서버 init 스크립트
<pre>
<code>
TAP_ADDR=10.77.77.1 : 자신의 DNSMASQ g/w 주소 입력
/sbin/ifconfig tap_soft $TAP_ADDR/8 : DNSMASQ 설정에 맞는 서브넷 할당
iptables -t nat -A POSTROUTING -j MASQUERADE : 리눅스의 NAT기능이다, 클라이언트가 서버를 통해 다른 네트워크에 접속할 수 있도록 한다
</code>
</pre>
<pre>
<code>
23. sudo sysctl -w net.ipv4.ip_forward=1
24. sudo reboot
</code>
</pre>
#### ip_forward 활성화, 재부팅
