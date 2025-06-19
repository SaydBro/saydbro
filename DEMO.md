##ISP - Alt Linux
hostnamectl set-hostname isp.au-team.irpo; exec bash
timedatectl set-timezone Asia/Novosibirsk

nano /etc/net/ifaces/ens18/options

BOOTPROTO=dhcp
TYPE=eth
DISABLED=no
CONFIG_IPV4=yes

nano /etc/net/sysctl.conf
net.ipv4.ip_forward = 1
sudo sysctl -p

cp -r /etc/net/ifaces/ens{18,19}
cp -r /etc/net/ifaces/ens{18,20}
ls -la /etc/net/ifaces

nano /etc/net/ifaces/ens19/options

BOOTPROTO=static
TYPE=eth
DISABLED=no
CONFIG_IPV4=yes

echo 172.16.40.1/28 >> /etc/net/ifaces/ens19/ipv4address

nano /etc/net/ifaces/ens20/options

BOOTPROTO=static
TYPE=eth
DISABLED=no
CONFIG_IPV4=yes

echo 172.16.50.1/28 >> /etc/net/ifaces/ens20/ipv4address

iptables -t nat -A POSTROUTING -o ens18 -j MASQUERADE 


iptables -F - сброс правил 

iptables-save >> /etc/sysconfig/iptables
systemctl enable --now iptables

useradd sshuser -u 1010
passwd sshuser
nano /etc/sudoers
ALL=(ALL:ALL) NOPASSWD: ALL
usermod -aG wheel sshuser


##HQ-RTR - EcoRouter
en 
conf t
hostname hq-rtr
ip domain-name au-team.irpo
show port brief
interface ISP
ip address 172.16.40.2/28
ip nat outside
port te0
service-instance te0/ISP
encapsulation untagged
connect ip interface ISP
exit
ip route 0.0.0.0/0 172.16.40.1
interface VL10
ip address 192.168.100.1/26
ip nat inside
exit
interface VL20
ip address 192.168.100.65/28
ip nat inside
exit
interface VL99
ip address 192.168.100.81/29
ip nat inside
exit
port te1
service-instance te1/VL10
encapsulation dot1q 10
rewrite pop 1
connect ip interface VL10
exit
port te1
service-instance te1/VL20
encapsulation dot1q 20
rewrite pop 1
connect ip interface VL20
exit
port te1
service-instance te1/VL99
encapsulation dot1q 99
rewrite pop 1
connect ip interface VL99
exit
ip nat pool HQ 192.168.100.2-192.168.100.254
ip nat source dynamic inside-to-outside pool HQ overload interface te0
username net_admin
password P@ssw0rd
role admin
exit
interface tunnel.0
ip address 10.10.10.1/30
ip tunnel 172.16.40.2 172.16.50.2 mode gre
show interface tunnel.0
show ip interface brief
do ping 10.10.10.2
router ospf 1
passive-interface default
no passive-interface tunnel.0
network 10.10.10.0/30 area 0
network 192.168.100.0/26 area 0
network 192.168.100.64/28 area 0
area 0 authentication
exit
interface tunnel.0
ip ospf authentication-key P@ssw0rd
show ip ospf neighbor
show ip route ospf
ip pool HQ-Clients 192.168.100.66-192.168.100.78
dhcp-server 1
pool HQ-Clients 1
mask 255.255.255.240
gateway 192.168.100.65
dns 192.168.100.1
domain-name au-team.irpo
exit
interface VL20
dhcp-server 1
exit
write memory



##BR-RTR - EcoRouter
en 
conf t
hostname br-rtr
ip domain-name au-team.irpo
show port brief
interface ISP
ip address 172.16.50.2/28
ip nat outside
port te0
service-instance te0/ISP
encapsulation untagged
connect ip interface ISP
exit
ip route 0.0.0.0/0 172.16.50.1
interface BR-Net
ip address 192.168.200.1/27
ip nat inside
exit
port te1
service-instance te1/BRNET
encapsulation untagged
connect ip interface BR-Net
exit
username net_admin
password P@ssw0rd
role admin
exit
interface tunnel.0
ip address 10.10.10.2/30
ip tunnel 172.16.50.2 172.16.40.2 mode gre
show interface tunnel.0
show ip interface brief
do ping 10.10.10.1
router ospf 1
passive-interface default
no passive-interface tunnel.0
network 10.10.10.0/30 area 0
network 192.168.200.0/27 area 0
area 0 authentication
exit
interface tunnel.0
ip ospf authentication-key P@ssw0rd
ip nat pool BR 192.168.200.2-192.168.200.254
ip nat source dynamic inside-to-outside pool BR overload interface te0
write memory


##HQ-SRV - Alt Linux
hostnamectl set-hostname hq-srv.au-team.irpo; exec bash
timedatectl set-timezone Asia/Novosibirsk
nano /etc/net/ifaces/.../

useradd sshuser -u 1010
passwd sshuser
nano /etc/sudoers
ALL=(ALL:ALL) NOPASSWD: ALL
usermod -aG wheel sshuser

nano /etc/openssh/sshd_config

Port 2024
AllowUsers sshuser
MaxAuthTries 2
PasswordAuthentication yes

##BR-SRV - Alt Linux
hostnamectl set-hostname br-srv.au-team.irpo; exec bash
timedatectl set-timezone Asia/Novosibirsk
nano /etc/net/ifaces/.../

useradd sshuser -u 1010
passwd sshuser
nano /etc/sudoers
ALL=(ALL:ALL) NOPASSWD: ALL
usermod -aG wheel sshuser

nano /etc/openssh/sshd_config

Port 2024
AllowUsers sshuser
MaxAuthTries 2
PasswordAuthentication yes

nano /etc/openssh/banner
Authorized access only.

systemctl restart sshd

##HQ-CLI - Alt Linux
hostnamectl set-hostname hq-cli.au-team.irpo; exec bash
timedatectl set-timezone Asia/Novosibirsk
nano /etc/net/ifaces/.../

useradd sshuser -u 1010
passwd sshuser
nano /etc/sudoers
ALL=(ALL:ALL) NOPASSWD: ALL
usermod -aG wheel sshuser
