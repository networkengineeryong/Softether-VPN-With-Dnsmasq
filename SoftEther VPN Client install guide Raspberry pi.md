# VPN Client on Raspberry pi
====================================
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
</code>
</pre>
3. vpnclient 실행, make (라이선스 동의)
<pre>
<code>
cd vpnclient
(echo 1; echo 1; echo 1) | make
</code>
</pre>
4. vpn 클라이언트 서비스에 등록, 시작 프로그램 등록
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
