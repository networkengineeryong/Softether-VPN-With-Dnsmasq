# [Case 1: allow-hotplug 사용해 DHCP 요청하기]
1. sudo nano /etc/network/interfaces
<pre>
<code>
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp

allow-hotplug vpn_soft
iface vpn_soft inet dhcp
#  post-up sudo route add default gw 10.77.77.1 dev vpn_soft # used for need g/w
</code>
</pre>