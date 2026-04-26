Попробовал Сделать PPPoE а R2-R3-R4

Теперб не работает:
1. OSPF (R2-R4)
2. DHCP на R2 и R4

### R3
```
hostname R3  
  
bba-group pppoe GLOBAL  
virtual-template 1  
  
interface Virtual-Template1  
ip address 203.0.113.17 255.255.255.248  
peer default ip address pool PPP_POOL  
ppp authentication chap  
  
ip local pool PPP_POOL 203.0.113.18 203.0.113.22  
  
username R2 password cisco  
username R4 password cisco  
  
interface GigabitEthernet0/0  
no ip address  
pppoe enable group GLOBAL  
no shutdown  
  
interface GigabitEthernet0/2  
no ip address  
pppoe enable group GLOBAL  
no shutdown  
  
interface GigabitEthernet0/1  
ip address 203.0.113.1 255.255.255.240  
no shutdown
```
### R2
```
conf t  
  
hostname R2  
no ip domain-lookup  
  
interface GigabitEthernet0/1  
description TO_R1  
ip address 10.0.12.2 255.255.255.252  
ip nat inside  
no shutdown  
  
interface GigabitEthernet0/0  
description TO_S2  
ip address 10.0.20.1 255.255.255.252  
ip nat inside  
no shutdown  
  
interface GigabitEthernet0/2  
description TO_R3  
no ip address  
pppoe enable  
pppoe-client dial-pool-number 1  
no shutdown  
  
interface Dialer1  
description PPPoE_WAN  
ip address negotiated  
encapsulation ppp  
dialer pool 1  
ppp chap hostname R2  
ppp chap password cisco  
ip nat outside  

ip route 0.0.0.0 0.0.0.0 Dialer1  
  
interface Tunnel0  
description VPN_TO_R4  
ip address 172.16.1.1 255.255.255.252  
tunnel source Dialer1  
tunnel destination 203.0.113.19  
ip nat inside  
  

ip dhcp excluded-address 192.168.10.1 192.168.10.10  
ip dhcp excluded-address 192.168.20.1 192.168.20.10  
  
ip dhcp pool VLAN_A  
network 192.168.10.0 255.255.255.0  
default-router 192.168.10.1  
dns-server 8.8.8.8  
  
ip dhcp pool VLAN_B  
network 192.168.20.0 255.255.255.0  
default-router 192.168.20.1  
dns-server 8.8.8.8  
  

access-list 1 permit 192.168.0.0 0.0.255.255  
access-list 1 permit 192.168.100.0 0.0.0.255  
  
ip nat inside source list 1 interface Dialer1 overload  
  
 
router ospf 1  
router-id 2.2.2.2  
network 10.0.12.0 0.0.0.3 area 1  
network 172.16.1.0 0.0.0.3 area 1  
passive-interface GigabitEthernet0/0
```

### R4
```
interface GigabitEthernet0/0  
description TO_R3  
no ip address  
pppoe enable  
pppoe-client dial-pool-number 1  
no shutdown  
  
interface Dialer1  
description PPPoE_WAN  
ip address negotiated  
encapsulation ppp  
dialer pool 1  
ppp chap hostname R4  
ppp chap password cisco  
  
ip route 0.0.0.0 0.0.0.0 Dialer1  
  
interface Tunnel0  
description VPN_TO_R2  
ip address 172.16.1.2 255.255.255.252  
tunnel source Dialer1  
tunnel destination 203.0.113.18  
  
interface GigabitEthernet0/1  
description TO_S4  
ip address 10.0.40.1 255.255.255.0  
no shutdown  
  
ip dhcp excluded-address 192.168.30.1 192.168.30.10  
ip dhcp excluded-address 192.168.40.1 192.168.40.10  
  
ip dhcp pool VLAN_C  
network 192.168.30.0 255.255.255.0  
default-router 192.168.30.1  
dns-server 8.8.8.8  
  
ip dhcp pool VLAN_D  
network 192.168.40.0 255.255.255.0  
default-router 192.168.40.1  
dns-server 8.8.8.8  
  
router ospf 1  
router-id 4.4.4.4  
network 10.0.40.0 0.0.0.255 area 0  
network 172.16.1.0 0.0.0.3 area 1  
passive-interface GigabitEthernet0/1

conf t  
  
interface GigabitEthernet0/1  
ip nat inside  
  
interface Dialer1  
ip nat outside  
  
access-list 1 permit 192.168.30.0 0.0.0.255  
access-list 1 permit 192.168.40.0 0.0.0.255  
  
ip nat inside source list 1 interface Dialer1 overload  
  
end
```
