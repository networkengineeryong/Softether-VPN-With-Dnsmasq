# 클라이언트 관리 명령어
<https://www.softether.org/4-docs/1-manual/6._Command_Line_Management_Utility_Manual/6.5_VPN_Client_Management_Command_Reference>

vpncmd 실행 2번 선택, 주소 입력 스킵
<pre>
<code>
pi@raspberrypi:~/vpnclient $ ./vpncmd
By using vpncmd program, the following can be achieved. 

1. Management of VPN Server or VPN Bridge 
2. Management of VPN Client
3. Use of VPN Tools (certificate creation and Network Traffic Speed Test Tool)

Select 1, 2 or 3: 2

Specify the host name or IP address of the computer that the destination VPN Client is operating on. 
If nothing is input and Enter is pressed, connection will be made to localhost (this computer).
Hostname of IP Address of Destination: 

Connected to VPN Client "localhost".
</code>
</pre>
<pre>
<code>
#가상 이더넷 카드 생성
VPN Client> NicCreate [name]
#계정 생성
VPN Client> AccountCreate [name] [/SERVER:hostname:port] [/HUB:hubname] [/USERNAME:username] [/NICNAME:nicname]
#계정 비밀번호 설정
VPN Client> AccountPasswordSet [name] [/PASSWORD:password] [/TYPE:standard|radius]
#계정 연결
VPN Client> AccountConnect [name]
#계정 상태 정보 출력
VPN Client> AccountStatusGet [name]
#VPN Client 프로그램 시작 시 계정 자동 연결
VPN Client> AccountStartupSet [name]
</code>
</pre>
### 클라이언트 설정 예시
<pre>
<code>
VPN Client> NicCreate soft
VPN Client> AccountCreate client /server:172.16.0.1:443 /hub:server /username:client /nicname:soft
VPN Client> AccountPasswordSet client /password:client /type:standard
VPN Client> AccountConnect client
VPN Client> AccountStatusGet client
VPN Client> AccountStartupSet client
</code>
</pre>
