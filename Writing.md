### /etc/hosts 에 vpnserver가 등록되어 있지 않거나, 잘못된 주소 일 경우 

### 스크립트가 정상적으로 실행되지 않습니다!

## 기본 설치 (라즈베리 파이에 인터넷 연결 필요한 기기 없을 때)
<pre>
<code>
#!/bin/sh
(echo y) | sudo apt update
wget https://www.softether-download.com/files/softether/v4.34-9745-rtm-2020.04.05-tree/Linux/SoftEther_VPN_Client/32bit_-_ARM_EABI/softether-vpnclient-v4.34-9745-rtm-2020.04.05-linux-arm_eabi-32bit.tar.gz
tar xvzf softether-vpnclient-v4.34-9745-rtm-2020.04.05-linux-arm_eabi-32bit.tar.gz
sudo rm -r softether-vpnclient-v4.34-9745-rtm-2020.04.05-linux-arm_eabi-32bit.tar.gz
cd vpnclient
(echo 1; echo 1; echo 1) | make
cat /etc/hosts | grep vpnserver > ~/test
sed -i 's/'vpnserver'/''/' ~/test
(echo 2;echo ;echo ;echo AccountCreate client /server:"$(echo "$(cat ~/test | sed '/^$/d;s/[[:blank:]]//g'):443")" /hub:server /username:client /nicname:soft;echo AccountPasswordSet client /password:client /type:standard;echo AccountConnect client;echo AccountStartupSet client) | /home/$USER/vpnclient/vpncmd
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

## 라즈베리 파이에 인터넷 연결이 필요한 기기가 연결되어 있을 때
# (주의) 인터넷은 eth0, 기기는 eth1에 연결되어 있다고 가정하고 작성된 스크립트 입니다
<pre>
<code>
#!/bin/sh
(echo y) | sudo apt update
wget https://www.softether-download.com/files/softether/v4.34-9745-rtm-2020.04.05-tree/Linux/SoftEther_VPN_Client/32bit_-_ARM_EABI/softether-vpnclient-v4.34-9745-rtm-2020.04.05-linux-arm_eabi-32bit.tar.gz
tar xvzf softether-vpnclient-v4.34-9745-rtm-2020.04.05-linux-arm_eabi-32bit.tar.gz
sudo rm -r softether-vpnclient-v4.34-9745-rtm-2020.04.05-linux-arm_eabi-32bit.tar.gz
cd vpnclient
(echo 1; echo 1; echo 1) | make
cat /etc/hosts | grep vpnserver > ~/test
sed -i 's/'vpnserver'/''/' ~/test
(echo 2;echo ;echo ;echo AccountCreate client /server:"$(echo "$(cat ~/test | sed '/^$/d;s/[[:blank:]]//g'):443")" /hub:server /username:client /nicname:soft;echo AccountPasswordSet client /password:client /type:standard;echo AccountConnect client;echo AccountStartupSet client) | /home/$USER/vpnclient/vpncmd
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
(echo y) | sudo apt install dnsmasq
sudo systemctl enable dnsmasq
sudo systemctl start dnsmasq
sudo sh -c "cat /dev/null > /etc/dnsmasq.conf"
echo 'interface=eth1
dhcp-range=192.168.224.2,192.168.224.2,12h
dhcp-option=option:router,192.168.224.1/30
dhcp-leasefile=/var/lib/dnsmasq/dnsmasq.leases' > /etc/dnsmasq.conf
sudo sh -c "echo 'interface eth1
static ip_address=192.168.224.1/30' >> /etc/dhcpcd.conf"

</code>
</pre>