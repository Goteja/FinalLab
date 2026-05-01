| Device         | Interface | IP Address / Prefix | Description      |
| -------------- | --------- | ------------------- | ---------------- |
| **PC3**        | e0        | 203.0.113.2/28      | Internet host    |
| **R3**         | Gi0/1     | 203.0.113.1/28      | To S3 (Internet) |
| **R2**         | Gi0/2     | 203.0.113.18/29     | To R3            |
| **R3**         | Gi0/0     | 203.0.113.17/29     | To R2            |
| **R3**         | Gi0/2     | 203.0.113.25/30     | To R4            |
| **R4**         | Gi0/0     | 203.0.113.26/30     | To R3            |
| **R2**         | Gi0/1     | 10.0.12.2/30        | To R1            |
| **R1**         | Gi0/0     | 10.0.12.1/30        | To R2            |
| **R1**         | Gi0/1     | 192.168.100.1/24    | Server LAN       |
| **Server**     | e0        | 192.168.100.2/24    | Server           |
| **R2**         | Gi0/0     | 10.0.20.1/30        | To S2            |
| **S2**         | VLAN10    | 192.168.10.1/24     | VLAN A           |
| **S2**         | VLAN20    | 192.168.20.1/24     | VLAN B           |
| **R4**         | Gi0/1     | 10.0.40.1/24        | To S4            |
| **R5**         | Gi0/0     | 10.0.40.2/24        | To S4            |
| **R6**         | Gi0/0     | 10.0.40.3/24        | To S4            |
| **R5**         | Gi0/1.30  | 192.168.30.2/24     | VLAN C           |
| **R6**         | Gi0/1.30  | 192.168.30.3/24     | VLAN C           |
| **R5**         | Gi0/1.40  | 192.168.40.2/24     | VLAN D           |
| **R6**         | Gi0/1.40  | 192.168.40.3/24     | VLAN D           |
| **HSRP**       | VLAN C    | 192.168.30.1/24     | Virtual Gateway  |
| **HSRP**       | VLAN D    | 192.168.40.1/24     | Virtual Gateway  |
| **GRE Tunnel** | R2        | 172.16.1.1/30       | Site-to-Site VPN |
| **GRE Tunnel** | R4        | 172.16.1.2/30       | Site-to-Site VPN |
| **PC1**        | e0        | DHCP                | VLAN A           |
| **PC2**        | e0        | DHCP                | VLAN B           |
| **PC4**        | e0        | DHCP                | VLAN C           |
| **PC5**        | e0        | DHCP                | VLAN D           |
| **PC6**        | e0        | DHCP                | VLAN C           |
## 🖥️ R1
```
hostname R1  
  
interface GigabitEthernet0/0  
 description Link to R2  
 ip address 10.0.12.1 255.255.255.252  
 no shutdown  
  
interface GigabitEthernet0/1  
 description Server LAN  
 ip address 192.168.100.1 255.255.255.0  
 no shutdown  
  
router ospf 1  
 router-id 1.1.1.1  
 network 10.0.12.0 0.0.0.3 area 1  
 network 192.168.100.0 0.0.0.255 area 1  
 passive-interface GigabitEthernet0/1
```
---

## 🌐 R2
```

enable
configure terminal
hostname R2
no ip domain-lookup

interface GigabitEthernet0/1
 ip address 10.0.12.2 255.255.255.252
 no shutdown
 exit
 
interface GigabitEthernet0/0
 ip address 10.0.20.1 255.255.255.252
 no shutdown
 exit

interface GigabitEthernet0/2
 no ip address
 pppoe enable
 pppoe-client dial-pool-number 1
 no shutdown
 exit

interface Dialer1
 mtu 1492
 ip address negotiated
 encapsulation ppp
 ppp authentication chap callin
 ppp chap hostname Branch1
 ppp chap password 0 SecurePass123
 dialer pool 1
 no shutdown
 exit

interface Tunnel0
 ip address 172.16.1.1 255.255.255.252
 tunnel source Dialer1
 tunnel destination 203.0.113.19
 keepalive 10 3
 exit

router ospf 1
 router-id 2.2.2.2
 network 10.0.12.0 0.0.0.3 area 0
 network 10.0.20.0 0.0.0.3 area 0
 network 172.16.1.0 0.0.0.3 area 0
 passive-interface default
 no passive-interface GigabitEthernet0/0
 no passive-interface GigabitEthernet0/1
 no passive-interface Tunnel0
 exit

ip route 192.168.10.0 255.255.255.0 10.0.20.2
ip route 192.168.20.0 255.255.255.0 10.0.20.2

ip route 0.0.0.0 0.0.0.0 Dialer1

access-list 10 permit 192.168.100.0 0.0.0.255
access-list 10 permit 192.168.10.0 0.0.0.255
access-list 10 permit 192.168.20.0 0.0.0.255
ip nat inside source list 10 interface Dialer1 overload

interface GigabitEthernet0/0
 ip nat inside
 exit
interface GigabitEthernet0/1
 ip nat inside
 exit
interface Tunnel0
 ip nat inside
 exit
interface Dialer1
 ip nat outside
 exit
 
ip nat inside source static tcp 192.168.100.2 80 interface Dialer1 80

ip dhcp pool VLAN_A
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
 exit
ip dhcp pool VLAN_B
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 8.8.8.8
 exit
 
ip dhcp excluded-address 192.168.10.1
ip dhcp excluded-address 192.168.20.1

router ospf 1
 redistribute static subnets
 exit
 
interface Loopback1
 ip address 203.0.113.33 255.255.255.255
 description Static NAT for Server
 no shutdown
 exit
 
ip nat inside source static 192.168.100.2 203.0.113.33
```
---

## 🌐 R3 (ISP)
```
enable
configure terminal
hostname R3
no ip domain-lookup

interface GigabitEthernet0/1
 ip address 203.0.113.1 255.255.255.240
 no shutdown
 exit

bba-group pppoe GLOBAL
 virtual-template 1
 exit

interface Virtual-Template1
 ip unnumbered GigabitEthernet0/1
 peer default ip address pool PPP_POOL
 ppp authentication chap
 ppp ipcp dns 8.8.8.8
 no shutdown
 exit
 
ip local pool PPP_POOL 203.0.113.18 203.0.113.22

username Branch1 password 0 SecurePass123
username Branch2 password 0 SecurePass123

interface GigabitEthernet0/0
 no ip address
 pppoe enable group GLOBAL
 no shutdown
 exit
 
interface GigabitEthernet0/2
 no ip address
 pppoe enable group GLOBAL
 no shutdown
 exit

ip route 0.0.0.0 0.0.0.0 203.0.113.2

ip route 203.0.113.33 255.255.255.255 203.0.113.18
```
> ❗ R3 содержит **только напрямую подключённые маршруты**, как требуется.

---

## 🌐 S2 (L3 Switch)
```
hostname S2  
ip routing  
  
vlan 10  
 name VLAN_A  
vlan 20  
 name VLAN_B  
  
interface Vlan10  
 ip address 192.168.10.1 255.255.255.0  
 ip helper-address 10.0.20.1
 no shutdown  
  
interface Vlan20  
 ip address 192.168.20.1 255.255.255.0  
 ip helper-address 10.0.20.1
 no shutdown  
  
interface GigabitEthernet0/0  
 description Link to R2  
 no switchport  
 ip address 10.0.20.2 255.255.255.252  
 no shutdown  
  
interface GigabitEthernet0/1  
 switchport mode access  
 switchport access vlan 10  
  
interface GigabitEthernet0/2  
 switchport mode access  
 switchport access vlan 20  
  
ip route 0.0.0.0 0.0.0.0 10.0.20.1
```
---

## 🌐 R4

```

enable
configure terminal
hostname R4
no ip domain-lookup

interface GigabitEthernet0/1
 ip address 10.0.40.1 255.255.255.0
 no shutdown
 exit

interface GigabitEthernet0/0
 no ip address
 pppoe enable
 pppoe-client dial-pool-number 1
 no shutdown
 exit

interface Dialer1
 mtu 1492
 ip address negotiated
 encapsulation ppp
 ppp authentication chap callin
 ppp chap hostname Branch2
 ppp chap password 0 SecurePass123
 dialer pool 1
 no shutdown
 exit

interface Tunnel0
 ip address 172.16.1.2 255.255.255.252
 tunnel source Dialer1
 tunnel destination 203.0.113.18
 keepalive 10 3
 exit

router ospf 1
 router-id 4.4.4.4
 network 10.0.40.0 0.0.0.255 area 0
 network 172.16.1.0 0.0.0.3 area 0
 passive-interface default
 no passive-interface GigabitEthernet0/1
 no passive-interface Tunnel0
 default-information originate always
 exit

ip route 0.0.0.0 0.0.0.0 Dialer1

access-list 10 permit 192.168.30.0 0.0.0.255
access-list 10 permit 192.168.40.0 0.0.0.255
ip nat inside source list 10 interface Dialer1 overload

interface GigabitEthernet0/1
 ip nat inside
 exit
interface Tunnel0
 ip nat inside
 exit
interface Dialer1
 ip nat outside
 exit

ip dhcp excluded-address 192.168.30.1 192.168.30.3
ip dhcp excluded-address 192.168.40.1 192.168.40.3

ip dhcp pool VLAN_C
 network 192.168.30.0 255.255.255.0
 default-router 192.168.30.1
 dns-server 8.8.8.8
 exit
ip dhcp pool VLAN_D
 network 192.168.40.0 255.255.255.0
 default-router 192.168.40.1
 dns-server 8.8.8.8
 exit
 
router ospf 1
 redistribute static subnets
 exit

```

---

## 🌐 R5 и R6 — HSRP

### R5 (Active)
```
hostname R5  
  
interface GigabitEthernet0/0  
 ip address 10.0.40.2 255.255.255.0  
 no shutdown  
  
interface GigabitEthernet0/1.30  
 encapsulation dot1Q 30  
 ip address 192.168.30.2 255.255.255.0  
 standby 30 ip 192.168.30.1  
 standby 30 priority 110  
 standby 30 preempt  
 ip helper-address 10.0.40.1
  
interface GigabitEthernet0/1.40  
 encapsulation dot1Q 40  
 ip address 192.168.40.2 255.255.255.0  
 standby 40 ip 192.168.40.1  
 standby 40 priority 110  
 standby 40 preempt
 ip helper-address 10.0.40.1

int g0/1
 no sh

router ospf 1
 router-id 5.5.5.5
 network 10.0.40.0 0.0.0.255 area 0
 network 192.168.30.0 0.0.0.255 area 0
 network 192.168.40.0 0.0.0.255 area 0
 passive-interface GigabitEthernet0/1.30
 passive-interface GigabitEthernet0/1.40
```
### R6 (Standby)
```
hostname R6  
  
interface GigabitEthernet0/0  
 ip address 10.0.40.3 255.255.255.0  
 no shutdown  
  
interface GigabitEthernet0/1.30  
 encapsulation dot1Q 30  
 ip address 192.168.30.3 255.255.255.0  
 standby 30 ip 192.168.30.1  
 standby 30 priority 100  
 standby 30 preempt  
 ip helper-address 10.0.40.1
  
interface GigabitEthernet0/1.40  
 encapsulation dot1Q 40  
 ip address 192.168.40.3 255.255.255.0  
 standby 40 ip 192.168.40.1  
 standby 40 priority 100  
 standby 40 preempt
 ip helper-address 10.0.40.1

int g0/1
 no sh

router ospf 1
 router-id 6.6.6.6
 network 10.0.40.0 0.0.0.255 area 0
 network 192.168.30.0 0.0.0.255 area 0
 network 192.168.40.0 0.0.0.255 area 0
 passive-interface GigabitEthernet0/1.30
 passive-interface GigabitEthernet0/1.40
```
---

## 🔗 LACP между S5 и S6

### S5
```
hostname S5

vlan 30
 name VLAN_C
vlan 40
 name VLAN_D
exit

interface range GigabitEthernet0/3, GigabitEthernet1/0  
 channel-group 1 mode active  
  
interface Port-channel1  
 switchport mode trunk
 switchport trunk allowed vlan all

interface G0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 30,40

interface G0/1
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast

interface G0/2
 switchport mode access
 switchport access vlan 40
 spanning-tree portfast
```

### S6
```
hostname S6

vlan 30
 name VLAN_C
vlan 40
 name VLAN_D

interface range GigabitEthernet0/2, GigabitEthernet0/3  
 channel-group 1 mode active  
  
interface Port-channel 1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan all
 exit

interface G0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 30,40

interface G0/1
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast

```

### Server
```
sudo ip addr add 192.168.100.2/24 dev ens3
sudo ip route add default via 192.168.100.1
```

### PC3
```
sudo ip addr add 203.0.113.2/28 dev ens3
sudo ip route add default via 203.0.113.1
```
