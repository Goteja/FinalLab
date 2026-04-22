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
hostname R2  
  
interface GigabitEthernet0/1  
 description Link to R1  
 ip address 10.0.12.2 255.255.255.252  
 no shutdown  
  
interface GigabitEthernet0/0  
 description Link to S2  
 ip address 10.0.20.1 255.255.255.252  
 no shutdown  
  
interface GigabitEthernet0/2  
 description Link to R3 (ISP)  
 ip address 203.0.113.18 255.255.255.248  
 no shutdown  
  
interface Tunnel0  
 ip address 172.16.1.1 255.255.255.252  
 tunnel source GigabitEthernet0/2  
 tunnel destination 203.0.113.26

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

router ospf 1  
 router-id 2.2.2.2  
 network 10.0.12.0 0.0.0.3 area 1  
 network 172.16.1.0 0.0.0.3 area 1  
 passive-interface GigabitEthernet0/0

access-list 1 permit 192.168.0.0 0.0.255.255  
access-list 1 permit 192.168.100.0 0.0.0.255  
  
ip nat inside source list 1 interface GigabitEthernet0/2 overload  
  
interface GigabitEthernet0/1  
 ip nat inside  
interface GigabitEthernet0/0  
 ip nat inside  
interface Tunnel0  
 ip nat inside  
interface GigabitEthernet0/2  
 ip nat outside  
  
ip route 0.0.0.0 0.0.0.0 203.0.113.17

## Маршруты прописать надо?
ip route 192.168.10.0 255.255.255.0 10.0.20.2
ip route 192.168.20.0 255.255.255.0 10.0.20.2
router ospf 1
 redistribute static subnets
 default-information originate
```
---

## 🌐 R3 (ISP)
```
hostname R3  
  
interface GigabitEthernet0/0  
 description Link to R2  
 ip address 203.0.113.17 255.255.255.248  
 no shutdown  
  
interface GigabitEthernet0/1  
 description Internet Segment  
 ip address 203.0.113.1 255.255.255.240  
 no shutdown  
  
interface GigabitEthernet0/2  
 description Link to R4  
 ip address 203.0.113.25 255.255.255.252  
 no shutdown
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
 no shutdown  
  
interface Vlan20  
 ip address 192.168.20.1 255.255.255.0  
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
hostname R4  
  
interface GigabitEthernet0/0  
 description Link to R3  
 ip address 203.0.113.26 255.255.255.252  
 no shutdown  
  
interface GigabitEthernet0/1  
 description Link to S4  
 ip address 10.0.40.1 255.255.255.0  
 no shutdown  
  
interface Tunnel0  
 ip address 172.16.1.2 255.255.255.252  
 tunnel source GigabitEthernet0/0  
 tunnel destination 203.0.113.18

ip route 0.0.0.0 0.0.0.0 203.0.113.25
router ospf 1
 default-information originate

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
  
interface GigabitEthernet0/1.40  
 encapsulation dot1Q 40  
 ip address 192.168.40.2 255.255.255.0  
 standby 40 ip 192.168.40.1  
 standby 40 priority 110  
 standby 40 preempt
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
  
interface GigabitEthernet0/1.40  
 encapsulation dot1Q 40  
 ip address 192.168.40.3 255.255.255.0  
 standby 40 ip 192.168.40.1  
 standby 40 priority 100  
 standby 40 preempt
```
---

## 🔗 LACP между S5 и S6

### S5
```
hostname S5

interface range GigabitEthernet0/3, GigabitEthernet1/0  
 channel-group 1 mode active  
  
interface Port-channel1  
 switchport mode trunk
```
### S6
```
hostname S6

interface range GigabitEthernet0/2, GigabitEthernet0/3  
 channel-group 1 mode active  
  
interface Port-channel1  
 switchport mode trunk
```