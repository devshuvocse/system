# ATAR3050S Full Configuration Guide

## Network Overview
- **VLAN 99**: Management (192.168.99.0/24) - Gateway: 192.168.99.1
- **VLAN 100**: Access Point & PC1 (172.16.0.0/20)
- **VLAN 150**: Internal Network (192.168.150.0/29)
- **VLAN 170**: DNS/Services (192.168.100.0/29)
- **VLAN 200**: Servers (192.168.0.8/29)
- **Cloud Networks**: Cloud1 & Cloud2 via VPN

---

## Configuration Steps

### 1. Basic Router Configuration
```
configure terminal
hostname ATAR3050S1
ip domain-name dpi.local
ip name-server 192.168.100.2
enable secret cisco
line vty 0 4
 password cisco
 login
 exec-timeout 15 0
exit
```

### 2. Interface Configuration

#### Management Interface (VLAN 99)
```
interface Gi0/0
 ip address 192.168.99.11 255.255.255.0
 description Management Interface
 no shutdown
exit
```

#### VLAN Trunk Ports (To Core & Distribution Switches)
```
interface Gi0/1
 description Trunk to CORE-SW
 switchport mode trunk
 switchport trunk allowed vlan 99,100,150,170,200
 no shutdown
exit

interface Gi0/2
 description Trunk to CORESW-BACKUP
 switchport mode trunk
 switchport trunk allowed vlan 99,100,150,170,200
 no shutdown
exit
```

#### Access Port Configuration
```
interface Gi0/3
 description VLAN 100 - User Access
 switchport mode access
 switchport access vlan 100
 no shutdown
exit

interface Gi0/4
 description VLAN 150 - Server Access
 switchport mode access
 switchport access vlan 150
 no shutdown
exit

interface Gi0/5
 description VLAN 170 - DNS/Services
 switchport mode access
 switchport access vlan 170
 no shutdown
exit

interface Gi0/6
 description VLAN 200 - Servers
 switchport mode access
 switchport access vlan 200
 no shutdown
exit
```

### 3. VLAN Configuration
```
vlan 99
 name Management
exit

vlan 100
 name AccessPoint
exit

vlan 150
 name Internal
exit

vlan 170
 name Services
exit

vlan 200
 name Servers
exit
```

### 4. SVI (Switched Virtual Interface) Configuration
```
interface vlan 99
 ip address 192.168.99.11 255.255.255.0
 description Management VLAN
 no shutdown
exit

interface vlan 100
 ip address 172.16.0.1 255.255.240.0
 description Access VLAN
 no shutdown
exit

interface vlan 150
 ip address 192.168.150.1 255.255.255.248
 description Internal VLAN
 no shutdown
exit

interface vlan 170
 ip address 192.168.100.1 255.255.255.248
 description Services VLAN
 no shutdown
exit

interface vlan 200
 ip address 192.168.0.1 255.255.255.248
 description Servers VLAN
 no shutdown
exit
```

### 5. Routing Configuration (OSPF)
```
router ospf 1
 router-id 192.168.99.11
 network 192.168.99.0 0.0.0.255 area 0
 network 172.16.0.0 0.0.15.255 area 0
 network 192.168.150.0 0.0.0.7 area 0
 network 192.168.100.0 0.0.0.7 area 0
 network 192.168.0.0 0.0.0.7 area 0
 default-information originate
exit
```

### 6. HSRP Configuration (Virtual Gateway: 172.17.0.1/29)
```
interface vlan 150
 standby 1 ip 172.17.0.1
 standby 1 priority 110
 standby 1 preempt
 standby 1 timers 3 10
exit
```

### 7. VTP Server Configuration
```
vtp mode server
vtp domain dpi.local
vtp password dpi
vtp version 2
exit
```

### 8. Access Port Security
```
interface Gi0/3
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
exit

interface Gi0/4
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation restrict
exit
```

### 9. ACL Configuration (Basic Security)
```
ip access-list standard MGMT_ACCESS
 permit 192.168.99.0 0.0.0.255
 permit 192.168.150.0 0.0.0.7
exit

ip access-list extended VLAN100_FILTER
 permit ip 172.16.0.0 0.0.15.255 192.168.100.0 0.0.0.7
 permit ip 172.16.0.0 0.0.15.255 192.168.0.0 0.0.0.7
 deny ip any any
exit

line vty 0 4
 access-class MGMT_ACCESS in
exit
```

### 10. Save Configuration
```
end
write memory
copy running-config startup-config
```

---

## Verification Commands
```
show vlan brief
show interfaces trunk
show standby brief
show vtp status
show access-lists
show ip route
show running-config
```

---

## Notes
- **HSRP Virtual Gateway**: 172.17.0.1/29 for VLAN 150
- **Secondary Gateway**: 172.17.0.129 (on R2) for redundancy
- **DNS Server**: 192.168.100.2 (VLAN 170)
- **Management**: Via VLAN 99 (192.168.99.11)

# ATAR3050S Full Configuration - Complete Setup

## Network Overview
- **VLAN 99**: Management (192.168.99.0/24) - Gateway: 192.168.99.1
- **VLAN 100**: Access Point & PC1 (172.16.0.0/20)
- **VLAN 150**: Internal Network (192.168.150.0/29)
- **VLAN 170**: DNS/Services (192.168.100.0/29)
- **VLAN 200**: Servers (192.168.0.8/29)
- **Cloud Networks**: Cloud1 & Cloud2 via VPN

---

## PART 1: Basic Router Configuration

```
configure terminal
hostname ATAR3050S1
ip domain-name dpi.local
ip name-server 192.168.100.2
enable secret cisco123
!
line vty 0 4
 password cisco123
 login
 exec-timeout 15 0
line console 0
 password cisco123
 login
 exec-timeout 0 0
exit
```

---

## PART 2: Interface & VLAN Configuration

```
! Management Interface (VLAN 99)
interface Gi0/0
 ip address 192.168.99.11 255.255.255.0
 description Management Interface
 no shutdown
exit

! Create VLANs
vlan 99
 name Management
exit
vlan 100
 name AccessPoint_Users
exit
vlan 150
 name Internal_Network
exit
vlan 170
 name DNS_Services
exit
vlan 200
 name Servers
exit

! SVI Interfaces
interface vlan 99
 ip address 192.168.99.11 255.255.255.0
 description Management VLAN
 no shutdown
exit

interface vlan 100
 ip address 172.16.0.1 255.255.240.0
 description Access VLAN (AP & Users)
 no shutdown
exit

interface vlan 150
 ip address 192.168.150.1 255.255.255.248
 description Internal VLAN
 no shutdown
exit

interface vlan 170
 ip address 192.168.100.1 255.255.255.248
 description Services VLAN (DNS)
 no shutdown
exit

interface vlan 200
 ip address 192.168.0.1 255.255.255.248
 description Servers VLAN
 no shutdown
exit
```

---

## PART 3: Trunk Port Configuration (To Switches)

```
! Trunk to CORE-SW
interface Gi0/1
 description Trunk-to-CORE-SW
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 99,100,150,170,200
 switchport trunk native vlan 99
 no shutdown
exit

! Trunk to CORESW-BACKUP
interface Gi0/2
 description Trunk-to-CORESW-BACKUP
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 99,100,150,170,200
 switchport trunk native vlan 99
 no shutdown
exit

! Trunk to ATR3050S (if daisy-chained)
interface Gi0/3
 description Trunk-to-ATR3050S
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 99,100,150,170,200
 switchport trunk native vlan 99
 no shutdown
exit
```

---

## PART 4: HSRP Configuration (High Availability)

```
! HSRP for VLAN 150 (Primary on R1 = ATAR3050S1)
interface vlan 150
 ip address 192.168.150.2 255.255.255.248
 standby version 2
 standby 1 ip 192.168.150.1
 standby 1 priority 110
 standby 1 preempt
 standby 1 timers 3 10
exit

! HSRP for VLAN 100
interface vlan 100
 ip address 172.16.0.2 255.255.240.0
 standby version 2
 standby 2 ip 172.16.0.1
 standby 2 priority 110
 standby 2 preempt
 standby 2 timers 3 10
exit

! HSRP for VLAN 200
interface vlan 200
 ip address 192.168.0.2 255.255.255.248
 standby version 2
 standby 3 ip 192.168.0.1
 standby 3 priority 110
 standby 3 preempt
 standby 3 timers 3 10
exit

! HSRP for VLAN 170
interface vlan 170
 ip address 192.168.100.2 255.255.255.248
 standby version 2
 standby 4 ip 192.168.100.1
 standby 4 priority 110
 standby 4 preempt
 standby 4 timers 3 10
exit
```

---

## PART 5: VTP Configuration

```
vtp mode server
vtp domain dpi.local
vtp password dpi123
vtp version 2
vtp pruning enable
exit
```

---

## PART 6: Routing Protocol (OSPF)

```
router ospf 1
 router-id 192.168.99.11
 auto-cost reference-bandwidth 10000
 
 network 192.168.99.0 0.0.0.255 area 0
 network 172.16.0.0 0.0.15.255 area 0
 network 192.168.150.0 0.0.0.7 area 0
 network 192.168.100.0 0.0.0.7 area 0
 network 192.168.0.0 0.0.0.7 area 0
 
 passive-interface vlan 100
 passive-interface vlan 200
 passive-interface vlan 170
exit
```

---

## PART 7: Port Security & Access Control

```
! Port Security on User Access
interface vlan 100
 switchport port-security
 switchport port-security maximum 5
 switchport port-security violation restrict
 switchport port-security mac-address sticky
exit

! Port Security on Server VLAN
interface vlan 200
 switchport port-security
 switchport port-security maximum 3
 switchport port-security violation shutdown
exit

! Port Security on Mgmt
interface vlan 99
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
exit
```

---

## PART 8: ACL Configuration (Access Control Lists)

```
! Standard ACL for Management
ip access-list standard MGMT_ACCESS
 permit 192.168.99.0 0.0.0.255
 permit 192.168.150.0 0.0.0.7
 deny any
exit

! Extended ACL - Inter-VLAN Communication
ip access-list extended INTER_VLAN_POLICY
 ! Allow Users to DNS
 permit ip 172.16.0.0 0.0.15.255 192.168.100.0 0.0.0.7
 ! Allow Users to Servers
 permit ip 172.16.0.0 0.0.15.255 192.168.0.0 0.0.0.7
 ! Allow Internal to Servers
 permit ip 192.168.150.0 0.0.0.7 192.168.0.0 0.0.0.7
 ! Allow Internal to DNS
 permit ip 192.168.150.0 0.0.0.7 192.168.100.0 0.0.0.7
 ! Allow Management to All
 permit ip 192.168.99.0 0.0.0.255 any
 ! Deny Everything Else
 deny ip any any
exit

! Apply ACL to VTY (SSH/Telnet)
line vty 0 4
 access-class MGMT_ACCESS in
 transport input ssh
exit

! Apply ACL to VLANs
interface vlan 100
 ip access-group INTER_VLAN_POLICY in
exit

interface vlan 150
 ip access-group INTER_VLAN_POLICY in
exit

interface vlan 200
 ip access-group INTER_VLAN_POLICY in
exit

interface vlan 170
 ip access-group INTER_VLAN_POLICY in
exit
```

---

## PART 9: NAT/PAT Configuration (For Cloud Connectivity)

```
! Define Inside Interface
ip nat inside source list 1 interface vlan 99 overload

! Access List for NAT
ip access-list standard NAT_INSIDE
 permit 172.16.0.0 0.0.15.255
 permit 192.168.150.0 0.0.0.7
 permit 192.168.100.0 0.0.0.7
 permit 192.168.0.0 0.0.0.7
exit

! Mark Interfaces
interface vlan 99
 ip nat outside
exit

interface vlan 100
 ip nat inside
exit

interface vlan 150
 ip nat inside
exit

interface vlan 170
 ip nat inside
exit

interface vlan 200
 ip nat inside
exit
```

---

## PART 10: DHCP Configuration (For Dynamic IPs)

```
! Exclude Management IPs from DHCP
ip dhcp excluded-address 192.168.99.1 192.168.99.50
ip dhcp excluded-address 172.16.0.1 172.16.0.100
ip dhcp excluded-address 192.168.150.1 192.168.150.7
ip dhcp excluded-address 192.168.100.1 192.168.100.7
ip dhcp excluded-address 192.168.0.1 192.168.0.7

! DHCP Pool for Access VLAN (AP & Users)
ip dhcp pool VLAN100_POOL
 network 172.16.0.0 255.255.240.0
 default-router 172.16.0.1
 dns-server 192.168.100.2
 domain-name dpi.local
 lease 1 0
exit

! DHCP Pool for Internal VLAN
ip dhcp pool VLAN150_POOL
 network 192.168.150.0 255.255.255.248
 default-router 192.168.150.1
 dns-server 192.168.100.2
 domain-name dpi.local
 lease 1 0
exit

! DHCP Pool for Server VLAN (typically static)
ip dhcp pool VLAN200_POOL
 network 192.168.0.0 255.255.255.248
 default-router 192.168.0.1
 dns-server 192.168.100.2
 domain-name dpi.local
 lease 1 0
exit

! DHCP Pool for Services VLAN (DNS)
ip dhcp pool VLAN170_POOL
 network 192.168.100.0 255.255.255.248
 default-router 192.168.100.1
 dns-server 192.168.100.2
 domain-name dpi.local
 lease 1 0
exit
```

---

## PART 11: VPN/IPSec Configuration (For Cloud1 & Cloud2)

```
! IKEv2 Phase 1 - Encryption
crypto ikev2 proposal CLOUD_PROPOSAL
 encryption aes-cbc-256
 integrity sha256
 dh-group 14
exit

! IKEv2 Phase 1 - Policy
crypto ikev2 policy CLOUD_POLICY
 proposal CLOUD_PROPOSAL
exit

! IKEv2 Keying
crypto ikev2 keyring CLOUD_KEYRING
 peer 0.0.0.0 0.0.0.0
  pre-shared-key cisco123VPN
exit

! IKEv2 Profile
crypto ikev2 profile CLOUD_PROFILE
 match identity remote address 0.0.0.0
 authentication remote pre-share
 authentication local pre-share
 keyring local CLOUD_KEYRING
 lifetime 28800
 dpd 10 3 on-demand
exit

! IPSec Phase 2 - Encryption
crypto ipsec transform-set CLOUD_TRANSFORM esp-aes 256 esp-sha-hmac
 mode tunnel
exit

! IPSec Policy
crypto ipsec policy CLOUD_IPSEC 1
 protocol esp
 authentication hmac-sha-256
 encryption aes 256
 lifetime seconds 3600
exit

! Static Routes for VPN
ip route 0.0.0.0 0.0.0.0 192.168.99.1 (adjust based on actual ISP gateway)

! Tunnel Interface to Cloud1
interface Tunnel 1
 ip address 10.0.0.1 255.255.255.0
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source 192.168.99.11
 tunnel destination CLOUD1_PUBLIC_IP
 tunnel mode ipsec ipv4
 tunnel protection ipsec profile CLOUD_PROFILE
 exit

! Tunnel Interface to Cloud2
interface Tunnel 2
 ip address 10.0.1.1 255.255.255.0
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source 192.168.99.11
 tunnel destination CLOUD2_PUBLIC_IP
 tunnel mode ipsec ipv4
 tunnel protection ipsec profile CLOUD_PROFILE
 exit

! Routes via Tunnels
ip route 10.10.0.0 255.255.0.0 10.0.0.1 (Cloud1 Route)
ip route 10.20.0.0 255.255.0.0 10.0.1.1 (Cloud2 Route)
```

---

## PART 12: DNS & NTP Configuration

```
! DNS Configuration
ip domain-name dpi.local
ip name-server 192.168.100.2

! NTP Configuration
ntp server 192.168.100.2 prefer
ntp source 192.168.99.11
clock timezone UTC 0
```

---

## PART 13: Logging & Monitoring

```
! Syslog Configuration
logging host 192.168.0.4
logging trap informational
logging buffer 8192

! SNMP Configuration (Optional)
snmp-server community dpi123 RO
snmp-server community dpi-write RW
snmp-server location DATACENTER
snmp-server contact admin@dpi.local

! Console Logging
logging console warnings
logging synchronous level all
```

---

## PART 14: Save & Verification

```
end
write memory
copy running-config startup-config
```

---

## Verification Commands

```
show vlan brief
show interfaces trunk
show standby brief
show vtp status
show access-lists
show ip route
show ip nat statistics
show ip dhcp binding
show crypto ipsec sa
show crypto session brief
show running-config
ping 192.168.100.2
ping 192.168.0.2
```

---

## Quick Command Summary
```
configure terminal
! Paste all above configurations
! Verify with commands above
end
write memory
```
