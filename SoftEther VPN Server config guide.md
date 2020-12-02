# 서버 관리 명령어

## 서버 명령어
https://www.softether.org/4-docs/1-manual/6._Command_Line_Management_Utility_Manual/6.3_VPN_Server_%2F%2F_VPN_Bridge_Management_Command_Reference_(For_Entire_Server)

## 허브 명령어
https://www.softether.org/4-docs/1-manual/6._Command_Line_Management_Utility_Manual/6.4_VPN_Server_%2F%2F_VPN_Bridge_Management_Command_Reference_(For_Virtual_Hub)

./vpncmd 실행, 1번 선택, 주소, 허브 명 입력 스킵
<pre>
<code>
[vpnserver@localhost vpnserver]$ ./vpncmd

vpncmd command - SoftEther VPN Command Line Management Utility
SoftEther VPN Command Line Management Utility (vpncmd command)
Version 4.34 Build 9745   (English)
Compiled 2020/04/05 23:39:56 by buildsan at crosswin
Copyright (c) SoftEther VPN Project. All Rights Reserved.

By using vpncmd program, the following can be achieved. 

1. Management of VPN Server or VPN Bridge 
2. Management of VPN Client
3. Use of VPN Tools (certificate creation and Network Traffic Speed server Tool)

Select 1, 2 or 3: 1

Specify the host name or IP address of the computer that the destination VPN Server or VPN Bridge is operating on. 
By specifying according to the format 'host name:port number', you can also specify the port number. 
(When the port number is unspecified, 443 is used.)
If nothing is input and the Enter key is pressed, the connection will be made to the port number 8888 of localhost (this computer).
Hostname of IP Address of Destination: 

If connecting to the server by Virtual Hub Admin Mode, please input the Virtual Hub name. 
If connecting by server admin mode, please press Enter without inputting anything.
Specify Virtual Hub Name: 
Connection has been established with VPN Server "localhost" (port 443).

You have administrator privileges for the entire VPN Server.
</code>
</pre>
1. 가상 허브 생성, 접속
<pre>
<code>
VPN Server> HubCreate server
VPN Server> Hub server
</code>
</pre>
2. SecureNAT, NAT, DHCP 비활성화 (DNSMASQ를 사용하기 때문)
<pre>
<code>
VPN Server/server> SecureNatDisable
VPN Server/server> NatDisable
VPN Server/server> DhcpDisable
</code>
</pre>
3. 유저 생성
<pre>
<code>
VPN Server/server> UserCreate client /Group: /Realname:client /Note:
VPN Server/server> UserPasswordSet client /Password: client
</code>
</pre>
4. DNSMASQ 인터페이스로 사용하기 위한 TAP Device 생성
<pre>
<code>
VPN Server/server> BridgeCreate server /DEVICE:soft /TAP:yes)
</code>
</pre>