# Enterprise Network Infrastructure Project

This project demonstrates a **redundant enterprise network architecture** with routing, switching, firewall security, server infrastructure, and website hosting based on the provided topology diagram.
## 📷 Network Diagram
![Screenshot of the UI](Project_Image_DPI.png)

---

## 📌 Network Topology Overview

- Dual Internet/Cloud connectivity
- Redundant routers using **HSRP**
- Core, Distribution, and Access switching layers
- Redundant firewalls
- EtherChannel for high-speed FTP traffic
- Centralized server infrastructure (DC, ADC, Web, Mail, FTP)
- VLAN-based segmentation
- Wired and Wireless client access

---

## 🧩 Logical Implementation Sequence

### Phase 1: Planning & Base Setup
1. Requirement analysis and final IP addressing & VLAN design  
2. Basic configuration of all Routers, Switches, and Firewalls  
3. VLAN creation on Core Switch, Distribution Switches, and Access Switches  
4. Access port configuration for PCs, Servers, and Access Points  
5. Trunk port configuration between Core, Distribution Switches, and Firewalls  

### Phase 2: Routing & Redundancy
6. IP address assignment on Routers, Firewalls, and Switch SVIs  
7. HSRP configuration on R1 and R2  
   - Virtual Gateway: `172.17.0.1/29`  
8. Internet/Cloud connectivity configuration on R1 and R2  
9. Inter-VLAN routing configuration on Core Switch or Firewall  
10. Static Routing or OSPF configuration between internal network and Cloud  

### Phase 3: Switching Optimization
11. EtherChannel configuration between `DIST-SW1` and `DIST-SW2`  
    - Optimized for FTP traffic  
    - Provides high throughput and redundancy  

### Phase 4: Security Layer
12. Firewall and Redundant Firewall configuration  
    - Inside / Outside / DMZ zones  
13. NAT / PAT configuration for Internet access and public services  
14. Access Control using Firewall policies and ACLs  

### Phase 5: Wireless & Client Services
15. Access Point (AP) configuration  
    - SSID, VLAN mapping, and security (WPA2/WPA3)  
16. DHCP Server configuration  
    - DHCP scopes for User and Wi-Fi VLANs  
    - DHCP relay using `ip helper-address`  

### Phase 6: Core Server Infrastructure
17. DNS Server installation and configuration  
18. Primary Domain Controller (DC) configuration  
    - Active Directory Domain Services (AD DS)  
19. Additional Domain Controller (ADC) configuration  
    - Replication and redundancy  
20. File Server & FSRM configuration  
    - Student data control, quotas, and permissions  

### Phase 7: Application & Hosting Services
21. Web Server configuration and website hosting  
22. FTP Server configuration with user-based permissions  
23. Mail Server configuration  
    - SMTP, POP3/IMAP services  
24. DNS record creation  
    - A, MX, and CNAME records for Web and Mail services  

### Phase 8: Testing & Hardening
25. Client PC and Wi-Fi user testing  
    - Domain login  
    - Web, FTP, and Mail access  
26. Security hardening  
    - ACLs, Firewall rules, Group Policies  
27. Monitoring, backup, and full network documentation  

---

## 🚀 Technologies Used

- Cisco Routing & Switching  
- HSRP, VLAN, Trunking, EtherChannel  
- Firewall with NAT  
- Windows Server (AD DS, DNS, DHCP, FSRM)  
- IIS / Apache / Nginx (Web Hosting)  
- FTP & Mail Services  

---

## ✅ Project Outcome

- High availability and redundancy  
- Secure segmented network  
- Centralized authentication and data control  
- Internal and external website hosting  
- Scalable enterprise-ready design  

---
##Configuration

📋 Network Overview
IP Addressing Plan
Management & Routing:
HSRP Virtual Gateway: 172.17.0.1/29
R1 (Active): 172.17.0.2/29
R2 (Standby): 172.17.0.3/29
Core Switch: 172.17.0.4/29
VLANs:
VLAN 200 (Client/AP Network): 172.16.0.0/22 (1022 hosts)
Gateway: 172.16.0.1
AP: DHCP assigned
PC1: DHCP assigned
VLAN 200 (Servers - DC/ADC): 192.168.0.0/29 (6 hosts)
DC: 192.168.0.2
ADC: 192.168.0.3
VLAN 200 (Servers - WEB/FTP/MAIL): 192.168.0.0/29 (6 hosts)
WEB: 192.168.0.10
FTP: 192.168.0.11
MAIL: 192.168.0.12
🔧 1. Edge Router Configuration (R1 & R2)
Router R1 (Active HSRP Gateway)
enable
configure terminal
hostname R1

! WAN Interface to Cloud1
interface GigabitEthernet0/0
 description WAN_to_Cloud1
 ip address dhcp
 ip nat outside
 no shutdown
exit

! LAN Interface to Core Switch
interface GigabitEthernet0/1
 description LAN_to_CORE-SW
 ip address 172.17.0.2 255.255.255.248
 ip nat inside
 no shutdown
 
 ! HSRP Configuration
 standby version 2
 standby 1 ip 172.17.0.1
 standby 1 priority 110
 standby 1 preempt
 standby 1 authentication md5 key-string hsrp_secret
exit

! Static Default Route (if not using OSPF to Cloud)
ip route 0.0.0.0 0.0.0.0 GigabitEthernet0/0

! NAT Configuration for Internet Access
access-list 10 permit 172.16.0.0 0.0.3.255
access-list 10 permit 192.168.0.0 0.0.0.255
ip nat inside source list 10 interface GigabitEthernet0/0 overload

! OSPF Configuration (for internal routing)
router ospf 1
 router-id 1.1.1.1
 network 172.17.0.0 0.0.0.7 area 0
 passive-interface GigabitEthernet0/0
 default-information originate
exit

! DNS Configuration
ip domain-lookup
ip name-server 192.168.0.2
ip name-server 8.8.8.8

! Save Configuration
end
write memory
Router R2 (Standby HSRP Gateway)
enable
configure terminal
hostname R2

! WAN Interface to Cloud2
interface GigabitEthernet0/0
 description WAN_to_Cloud2
 ip address dhcp
 ip nat outside
 no shutdown
exit

! LAN Interface to Core Switch
interface GigabitEthernet0/1
 description LAN_to_CORE-SW
 ip address 172.17.0.3 255.255.255.248
 ip nat inside
 no shutdown
 
 ! HSRP Configuration
 standby version 2
 standby 1 ip 172.17.0.1
 standby 1 priority 100
 standby 1 preempt
 standby 1 authentication md5 key-string hsrp_secret
exit

! Static Default Route
ip route 0.0.0.0 0.0.0.0 GigabitEthernet0/0

! NAT Configuration
access-list 10 permit 172.16.0.0 0.0.3.255
access-list 10 permit 192.168.0.0 0.0.0.255
ip nat inside source list 10 interface GigabitEthernet0/0 overload

! OSPF Configuration
router ospf 1
 router-id 2.2.2.2
 network 172.17.0.0 0.0.0.7 area 0
 passive-interface GigabitEthernet0/0
 default-information originate
exit

! DNS Configuration
ip domain-lookup
ip name-server 192.168.0.2
ip name-server 8.8.8.8

end
write memory
🛡️ 2. Core Switch Configuration (Layer 3)
enable
configure terminal
hostname CORE-SW

! Enable IP Routing
ip routing

! VLAN Creation
vlan 200
 name CLIENT_NETWORK
exit

vlan 201
 name SERVER_NETWORK
exit

vlan 999
 name NATIVE_VLAN
exit

! SVI Configuration (Gateway Interfaces)

! Interface VLAN 200 - Client/AP Network
interface Vlan200
 description Gateway_for_Clients_and_AP
 ip address 172.16.0.1 255.255.252.0
 ip helper-address 172.17.0.1
 no shutdown
exit

! Interface VLAN 201 - Server Network
interface Vlan201
 description Gateway_for_Servers
 ip address 192.168.0.1 255.255.255.0
 no shutdown
exit

! Uplink to R1
interface GigabitEthernet0/1
 description Uplink_to_R1
 no switchport
 ip address 172.17.0.4 255.255.255.248
 no shutdown
exit

! Uplink to R2
interface GigabitEthernet0/2
 description Uplink_to_R2
 no switchport
 ip address 172.17.0.5 255.255.255.248
 no shutdown
exit

! Trunk to Firewall
interface GigabitEthernet0/3
 description Trunk_to_Firewall
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 200,201
 no shutdown
exit

! Trunk to Redundant-Firewall
interface GigabitEthernet0/4
 description Trunk_to_Redundant_Firewall
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 200,201
 no shutdown
exit

! Trunk to DIST-SW1
interface GigabitEthernet0/5
 description Trunk_to_DIST-SW1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 200,201
 channel-group 1 mode active
 no shutdown
exit

! Trunk to DIST-SW2
interface GigabitEthernet0/6
 description Trunk_to_DIST-SW2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 200,201
 channel-group 2 mode active
 no shutdown
exit

! EtherChannel Configuration
interface Port-channel1
 description EtherChannel_to_DIST-SW1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
exit

interface Port-channel2
 description EtherChannel_to_DIST-SW2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
exit

! OSPF Configuration
router ospf 1
 router-id 3.3.3.3
 network 172.17.0.0 0.0.0.7 area 0
 network 172.16.0.0 0.0.3.255 area 0
 network 192.168.0.0 0.0.0.255 area 0
exit

! DHCP Server Configuration
ip dhcp excluded-address 172.16.0.1 172.16.0.10
ip dhcp excluded-address 192.168.0.1 192.168.0.5

ip dhcp pool CLIENT_POOL
 network 172.16.0.0 255.255.252.0
 default-router 172.16.0.1
 dns-server 192.168.0.2 8.8.8.8
 domain-name example.local
 lease 7
exit

! Spanning Tree Configuration
spanning-tree mode rapid-pvst
spanning-tree vlan 200,201 priority 4096

end
write memory
🔥 3. Firewall Configuration
Primary Firewall
enable
configure terminal
hostname FIREWALL

! VLAN Configuration
vlan 200
 name CLIENT_NETWORK
exit

vlan 201
 name SERVER_NETWORK
exit

vlan 100
 name DMZ
exit

! Inside Interface (Client Network)
interface Vlan200
 description Inside_Network
 ip address 172.16.0.254 255.255.252.0
 security-level 100
 no shutdown
exit

! DMZ Interface (Server Network)
interface Vlan201
 description DMZ_Server_Network
 ip address 192.168.0.254 255.255.255.0
 security-level 50
 no shutdown
exit

! Outside Interface
interface GigabitEthernet0/0
 description Outside_to_CORE
 nameif outside
 security-level 0
 ip address 172.17.0.10 255.255.255.248
 no shutdown
exit

! Access Control Lists

! Allow Clients to DC/ADC for Authentication
access-list INSIDE_IN extended permit tcp 172.16.0.0 255.255.252.0 host 192.168.0.2 eq domain
access-list INSIDE_IN extended permit udp 172.16.0.0 255.255.252.0 host 192.168.0.2 eq domain
access-list INSIDE_IN extended permit tcp 172.16.0.0 255.255.252.0 host 192.168.0.2 eq ldap
access-list INSIDE_IN extended permit tcp 172.16.0.0 255.255.252.0 host 192.168.0.2 eq 88
access-list INSIDE_IN extended permit tcp 172.16.0.0 255.255.252.0 host 192.168.0.3 eq ldap

! Allow Clients to WEB Server
access-list INSIDE_IN extended permit tcp 172.16.0.0 255.255.252.0 host 192.168.0.10 eq www
access-list INSIDE_IN extended permit tcp 172.16.0.0 255.255.252.0 host 192.168.0.10 eq https

! Allow Clients to MAIL Server
access-list INSIDE_IN extended permit tcp 172.16.0.0 255.255.252.0 host 192.168.0.12 eq smtp
access-list INSIDE_IN extended permit tcp 172.16.0.0 255.255.252.0 host 192.168.0.12 eq pop3
access-list INSIDE_IN extended permit tcp 172.16.0.0 255.255.252.0 host 192.168.0.12 eq imap

! Block Clients from FTP (only admin access)
access-list INSIDE_IN extended deny tcp 172.16.0.0 255.255.252.0 host 192.168.0.11 eq ftp

! Allow all other Internet traffic
access-list INSIDE_IN extended permit ip 172.16.0.0 255.255.252.0 any

! Apply ACL
access-group INSIDE_IN in interface Vlan200

! NAT Configuration (PAT)
object network INSIDE_NET
 subnet 172.16.0.0 255.255.252.0
 nat (inside,outside) dynamic interface

! Static NAT for Public Services
static (dmz,outside) tcp interface www 192.168.0.10 www
static (dmz,outside) tcp interface https 192.168.0.10 https
static (dmz,outside) tcp interface smtp 192.168.0.12 smtp

! IPS/IDS Configuration
ip audit name IDS_POLICY attack action alarm drop reset
ip audit interface Vlan200 IDS_POLICY

! Route to Internet via CORE
route outside 0.0.0.0 0.0.0.0 172.17.0.1 1

end
write memory
Redundant Firewall
! Similar configuration as Primary Firewall
! Configure with different IP addresses and enable failover

enable
configure terminal
hostname REDUNDANT-FIREWALL

! Enable Failover
failover
failover lan unit secondary
failover lan interface FAILOVER GigabitEthernet0/3
failover interface ip FAILOVER 10.10.10.2 255.255.255.252 standby 10.10.10.1

! Continue with similar configuration as Primary...
🔀 4. Distribution Switch Configuration
DIST-SW1
enable
configure terminal
hostname DIST-SW1

! VLAN Configuration
vlan 200
 name CLIENT_NETWORK
exit

vlan 201
 name SERVER_NETWORK
exit

! Trunk to CORE-SW
interface GigabitEthernet0/1
 description Trunk_to_CORE-SW
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 channel-group 1 mode active
 no shutdown
exit

! FTP EtherChannel (Dedicated for FTP Traffic)
interface range GigabitEthernet0/2-3
 description EtherChannel_to_DIST-SW2_FTP_Only
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 201
 channel-group 10 mode active
 no shutdown
exit

interface Port-channel10
 description FTP_EtherChannel
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 201
exit

! Trunk to Firewall
interface GigabitEthernet0/4
 description Trunk_to_Firewall
 switchport trunk encapsulation dot1q
 switchport mode trunk
 no shutdown
exit

! Access Ports to AP
interface FastEthernet0/1
 description Access_to_AP
 switchport mode access
 switchport access vlan 200
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
exit

! Access Port to PC1
interface FastEthernet0/2
 description Access_to_PC1
 switchport mode access
 switchport access vlan 200
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
exit

! Spanning Tree Configuration
spanning-tree mode rapid-pvst
spanning-tree vlan 200 priority 8192
spanning-tree vlan 201 priority 12288

end
write memory
DIST-SW2
enable
configure terminal
hostname DIST-SW2

! VLAN Configuration
vlan 200
 name CLIENT_NETWORK
exit

vlan 201
 name SERVER_NETWORK
exit

! Trunk to CORE-SW
interface GigabitEthernet0/1
 description Trunk_to_CORE-SW
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 channel-group 2 mode active
 no shutdown
exit

! FTP EtherChannel
interface range GigabitEthernet0/2-3
 description EtherChannel_to_DIST-SW1_FTP_Only
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 201
 channel-group 10 mode active
 no shutdown
exit

! Trunk to Redundant Firewall
interface GigabitEthernet0/4
 description Trunk_to_Redundant_Firewall
 switchport trunk encapsulation dot1q
 switchport mode trunk
 no shutdown
exit

! Access Ports to Servers (VLAN 201)
interface FastEthernet0/1
 description Access_to_DC
 switchport mode access
 switchport access vlan 201
 spanning-tree portfast
 no shutdown
exit

interface FastEthernet0/2
 description Access_to_ADC
 switchport mode access
 switchport access vlan 201
 spanning-tree portfast
 no shutdown
exit

interface FastEthernet0/3
 description Access_to_WEB_Server
 switchport mode access
 switchport access vlan 201
 spanning-tree portfast
 no shutdown
exit

interface FastEthernet0/4
 description Access_to_FTP_Server
 switchport mode access
 switchport access vlan 201
 spanning-tree portfast
 no shutdown
exit

interface FastEthernet0/5
 description Access_to_MAIL_Server
 switchport mode access
 switchport access vlan 201
 spanning-tree portfast
 no shutdown
exit

! Spanning Tree Configuration
spanning-tree mode rapid-pvst
spanning-tree vlan 200 priority 12288
spanning-tree vlan 201 priority 8192

end
write memory
📡 5. Access Point Configuration
AP (Wireless Configuration)
# Assuming Cisco Wireless AP

enable
configure terminal
hostname AP

! Management IP (will get from DHCP)
interface BVI1
 ip address dhcp
exit

! Radio Interface
dot11 ssid Student_Network
 vlan 200
 authentication open eap eap_methods
 authentication key-management wpa version 2
 mbssid guest-mode
exit

! 802.1X Configuration
dot11 aaa authentication ssid Student_Network
dot11 aaa authorization ssid Student_Network

! RADIUS Server Configuration
radius-server host 192.168.0.2 auth-port 1812 acct-port 1813 key RadiusSecret123

! AAA Configuration
aaa new-model
aaa group server radius RAD_GROUP
 server 192.168.0.2 auth-port 1812 acct-port 1813
exit

aaa authentication login eap_methods group RAD_GROUP
aaa authorization exec default group RAD_GROUP

! Bridge to Ethernet
interface Dot11Radio0
 encryption vlan 200 mode ciphers aes-ccm
 ssid Student_Network
 no shutdown
exit

interface GigabitEthernet0
 description Trunk_to_DIST-SW1
 switchport mode trunk
 switchport trunk allowed vlan 200
exit

end
write memory
🖥️ 6. Server Configurations
Domain Controller (DC) - Windows Server Configuration
IP Configuration:
IP Address: 192.168.0.2/24
Gateway: 192.168.0.1
DNS: 127.0.0.1, 8.8.8.8
PowerShell Commands:
# Set Static IP
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.0.2 -PrefixLength 24 -DefaultGateway 192.168.0.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 127.0.0.1,8.8.8.8

# Install AD DS Role
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# Promote to Domain Controller
Install-ADDSForest `
  -DomainName "example.local" `
  -DomainNetbiosName "EXAMPLE" `
  -InstallDns `
  -SafeModeAdministratorPassword (ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force) `
  -Force

# Install NPS (RADIUS Server)
Install-WindowsFeature -Name NPAS -IncludeManagementTools

# Configure NPS for 802.1X
# Use GUI: Server Manager > Tools > Network Policy Server
# Add RADIUS Clients:
#   - AP: 172.16.0.x with shared secret
# Create Network Policy for 802.1X Wireless
DNS Records Configuration:
# Add DNS Records
Add-DnsServerResourceRecordA -Name "web" -ZoneName "example.local" -IPv4Address "192.168.0.10"
Add-DnsServerResourceRecordA -Name "ftp" -ZoneName "example.local" -IPv4Address "192.168.0.11"
Add-DnsServerResourceRecordA -Name "mail" -ZoneName "example.local" -IPv4Address "192.168.0.12"

Add-DnsServerResourceRecordMX -Name "." -ZoneName "example.local" -MailExchange "mail.example.local" -Preference 10
Add-DnsServerResourceRecordCName -Name "www" -ZoneName "example.local" -HostNameAlias "web.example.local"
Additional Domain Controller (ADC)
IP Configuration:
IP Address: 192.168.0.3/24
Gateway: 192.168.0.1
DNS: 192.168.0.2, 127.0.0.1
# Set Static IP
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.0.3 -PrefixLength 24 -DefaultGateway 192.168.0.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.0.2,127.0.0.1

# Join Domain and Promote to DC
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

Install-ADDSDomainController `
  -DomainName "example.local" `
  -InstallDns `
  -Credential (Get-Credential) `
  -SafeModeAdministratorPassword (ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force) `
  -Force

# Configure AD Replication
# Automatic replication with primary DC
WEB Server Configuration
IP Configuration:
IP: 192.168.0.10/24
Gateway: 192.168.0.1
DNS: 192.168.0.2
IIS Installation (Windows):
# Install IIS
Install-WindowsFeature -Name Web-Server -IncludeManagementTools

# Configure Default Website
Import-Module WebAdministration
New-Item -Path "C:\inetpub\wwwroot\index.html" -ItemType File -Value "<h1>Welcome to Corporate Website</h1>"

# Enable HTTPS
New-SelfSignedCertificate -DnsName "web.example.local" -CertStoreLocation "cert:\LocalMachine\My"
# Bind certificate to IIS site via IIS Manager
Apache Installation (Linux):
# Ubuntu/Debian
sudo apt update
sudo apt install apache2 -y

# Configure Network
sudo nano /etc/netplan/00-installer-config.yaml
# Add:
network:
  ethernets:
    ens33:
      addresses: [192.168.0.10/24]
      gateway4: 192.168.0.1
      nameservers:
        addresses: [192.168.0.2, 8.8.8.8]

sudo netplan apply

# Start Apache
sudo systemctl start apache2
sudo systemctl enable apache2
FTP Server Configuration
IP Configuration:
IP: 192.168.0.11/24
Gateway: 192.168.0.1
Windows FTP Server:
# Install FTP Server
Install-WindowsFeature -Name Web-FTP-Server -IncludeManagementTools

# Create FTP Site
Import-Module WebAdministration
New-WebFtpSite -Name "Corporate FTP" -Port 21 -PhysicalPath "C:\FTPRoot"

# Configure User Authentication
# Use AD credentials via IIS Manager
Linux FTP (vsftpd):
sudo apt install vsftpd -y

# Configure vsftpd
sudo nano /etc/vsftpd.conf
# Edit:
anonymous_enable=NO
local_enable=YES
write_enable=YES
chroot_local_user=YES

sudo systemctl restart vsftpd
MAIL Server Configuration
IP Configuration:
IP: 192.168.0.12/24
Gateway: 192.168.0.1
Windows Mail Server (hMailServer):
# Download and install hMailServer
# Configure during installation:
# - Database: Built-in
# - Domain: example.local
# - SMTP Port: 25
# - POP3 Port: 110
# - IMAP Port: 143

# Set DNS to point to DC
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.0.2
Linux Mail Server (Postfix + Dovecot):
# Install Postfix and Dovecot
sudo apt install postfix dovecot-core dovecot-imapd dovecot-pop3d -y

# Configure Postfix
sudo nano /etc/postfix/main.cf
# Edit:
myhostname = mail.example.local
mydomain = example.local
myorigin = $mydomain
inet_interfaces = all
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain

# Configure Dovecot
sudo nano /etc/dovecot/dovecot.conf
# Uncomment:
protocols = imap pop3

# Restart services
sudo systemctl restart postfix dovecot
🔒 7. Security Hardening
ACL for Student Data Control
! On CORE-SW or Firewall

! Extended ACL for Student Control
ip access-list extended STUDENT_CONTROL
 
 ! Allow DNS to DC
 permit udp 172.16.0.0 0.0.3.255 host 192.168.0.2 eq domain
 permit tcp 172.16.0.0 0.0.3.255 host 192.168.0.2 eq domain
 
 ! Allow LDAP/Kerberos for Authentication
 permit tcp 172.16.0.0 0.0.3.255 host 192.168.0.2 eq ldap
 permit tcp 172.16.0.0 0.0.3.255 host 192.168.0.2 eq 88
 permit tcp 172.16.0.0 0.0.3.255 host 192.168.0.3 eq ldap
 
 ! Allow HTTP/HTTPS to WEB Server
 permit tcp 172.16.0.0 0.0.3.255 host 192.168.0.10 eq www
 permit tcp 172.16.0.0 0.0.3.255 host 192.168.0.10 eq https
 
 ! Allow Email Access
 permit tcp 172.16.0.0 0.0.3.255 host 192.168.0.12 eq smtp
 permit tcp 172.16.0.0 0.0.3.255 host 192.168.0.12 eq pop3
 permit tcp 172.16.0.0 0.0.3.255 host 192.168.0.12 eq imap
 
 ! DENY FTP Access (Only for authorized users)
 deny tcp 172.16.0.0 0.0.3.255 host 192.168.0.11 eq ftp log
 
 ! Allow Internet Access (HTTP/HTTPS only)
 permit tcp 172.16.0.0 0.0.3.255 any eq www
 permit tcp 172.16.0.0 0.0.3.255 any eq https
 permit tcp 172.16.0.0 0.0.3.255 any eq 8080
 
 ! Deny everything else
 deny ip any any log

! Apply ACL
interface Vlan200
 ip access-group STUDENT_CONTROL in
Firewall Rules for DMZ Zones
! Inside Zone → DMZ Zone
access-list INSIDE_TO_DMZ extended permit tcp 172.16.0.0 255.255.252.0 192.168.0.0 255.255.255.0 eq www
access-list INSIDE_TO_DMZ extended permit tcp 172.16.0.0 255.255.252.0 192.168.0.0 255.255.255.0 eq https
access-list INSIDE_TO_DMZ extended permit tcp 172.16.0.0 255.255.252.0 192.168.0.0 255.255.255.0 eq smtp
access-list INSIDE_TO_DMZ extended deny ip any any

! Outside Zone → DMZ Zone (Public Access)
access-list OUTSIDE_TO_DMZ extended permit tcp any host 192.168.0.10 eq www
access-list OUTSIDE_TO_DMZ extended permit tcp any host 192.168.0.10 eq https
access-list OUTSIDE_TO_DMZ extended permit tcp any host 192.168.0.12 eq smtp
access-list OUTSIDE_TO_DMZ extended deny ip any any

! DMZ → Inside (Restricted)
access-list DMZ_TO_INSIDE extended deny ip any any
📊 8. Monitoring & Backup
SNMP Configuration
! On all devices
snmp-server community public RO
snmp-server community private RW
snmp-server host 192.168.0.100 version 2c public
snmp-server enable traps
Syslog Configuration
! Configure Syslog Server
logging host 192.168.0.101
logging trap informational
logging source-interface Vlan201
Configuration Backup
# Automated backup script (Linux)
#!/bin/bash
BACKUP_DIR="/backup/network-configs"
DATE=$(date +%Y%m%d)

# Backup router configs
sshpass -p 'admin' ssh admin@172.17.0.2 "show run" > $BACKUP_DIR/R1-$DATE.cfg
sshpass -p 'admin' ssh admin@172.17.0.3 "show run" > $BACKUP_DIR/R2-$DATE.cfg

# Backup switch configs
sshpass -p 'admin' ssh admin@172.17.0.4 "show run" > $BACKUP_DIR/CORE-SW-$DATE.cfg

# Add to crontab for daily backup
# 0 2 * * * /scripts/backup-network.sh


🧪 9. Testing & Verification
Network Connectivity Testing
From PC1 (172.16.0.x):
# Test Gateway
ping 172.16.0.1

# Test HSRP Virtual Gateway
ping 172.17.0.1

# Test DC
ping 192.168.0.2

# Test WEB Server
ping 192.168.0.10
curl http://web.example.local

# Test Internet Connectivity
ping 8.8.8.8
nslookup google.com

# Test DNS Resolution
nslookup web.example.local
nslookup mail.example.local
HSRP Verification
! On R1 or R2
show standby brief
show standby GigabitEthernet0/1

! Expected Output:
! R1 should show State: Active
! R2 should show State: Standby
! Virtual IP: 172.17.0.1

! Test Failover
! On R1:
interface GigabitEthernet0/1
 shutdown

! Check on R2 - should become Active
! Then bring R1 back up:
interface GigabitEthernet0/1
 no shutdown
VLAN Verification
! On CORE-SW
show vlan brief
show interfaces trunk
show spanning-tree

! On DIST-SW1 and DIST-SW2
show vlan brief
show etherchannel summary
show spanning-tree vlan 200
show spanning-tree vlan 201
OSPF Verification
! On R1, R2, and CORE-SW
show ip ospf neighbor
show ip ospf database
show ip route ospf

! Expected: 
! All routers should be neighbors
! Routes should be learned via OSPF
NAT Verification
! On R1 and R2
show ip nat translations
show ip nat statistics

! Test from PC1
! Browse to external website and check:
show ip nat translations | include 172.16.0.x
DHCP Verification
! On CORE-SW
show ip dhcp binding
show ip dhcp pool
show ip dhcp server statistics

! On PC1 (Windows)
ipconfig /all
ipconfig /renew

! On PC1 (Linux)
sudo dhclient -v
ip addr show
Firewall Policy Verification
! On Firewall
show access-list
show conn
show xlate

! Test blocked FTP access from PC1
ftp 192.168.0.11
! Should be denied

! Check logs
show logging | include DENY
EtherChannel Verification
! On DIST-SW1
show etherchannel summary
show etherchannel port-channel

! Expected Output:
! Po10 should show (SU) - Layer2 in-use
! Member ports should show (P) - bundled in port-channel

! Test redundancy
interface GigabitEthernet0/2
 shutdown
! Traffic should continue on Gi0/3
Wireless Authentication Testing
From Windows Client:
Connect to SSID: Student_Network
Enter AD credentials
Should authenticate via RADIUS to DC
Verification on DC:
# Check NPS logs
Get-EventLog -LogName Security | Where-Object {$_.EventID -eq 6272}

# Check Authentication Events
Get-WinEvent -FilterHashtable @{LogName='Security';ID=4624} | Select-Object -First 10
Verification on AP:
show dot11 associations
show aaa sessions
show radius statistics
🎯 10. Client PC and WiFi User Configuration
Student User Creation on AD
# On DC (192.168.0.2)

# Create OU for Students
New-ADOrganizationalUnit -Name "Students" -Path "DC=example,DC=local"

# Create Student Users
New-ADUser -Name "Student01" `
  -GivenName "Student" `
  -Surname "01" `
  -SamAccountName "student01" `
  -UserPrincipalName "student01@example.local" `
  -Path "OU=Students,DC=example,DC=local" `
  -AccountPassword (ConvertTo-SecureString "Student@123" -AsPlainText -Force) `
  -Enabled $true `
  -PasswordNeverExpires $false `
  -ChangePasswordAtLogon $true

# Create Group for Students
New-ADGroup -Name "WiFi_Students" `
  -GroupScope Global `
  -GroupCategory Security `
  -Path "OU=Students,DC=example,DC=local"

# Add user to group
Add-ADGroupMember -Identity "WiFi_Students" -Members "student01"
NPS Policy Configuration for Students
On DC - Network Policy Server:
Open Network Policy Server (NPS)
Configure RADIUS Clients:
Add AP IP address
Shared Secret: RadiusSecret123
Create Network Policy:
Policy Name: Wireless_802.1X_Students
Conditions:
Windows Groups: WiFi_Students
NAS Port Type: Wireless - IEEE 802.11
Constraints:
Authentication Methods: Microsoft: Protected EAP (PEAP)
EAP Types: MS-CHAP v2
Settings:
RADIUS Attributes: Standard
Grant Access: Yes
PC1 Configuration (Windows)
Network Settings:
# Assuming DHCP (automatic)
# If static needed:
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 172.16.0.50 -PrefixLength 22 -DefaultGateway 172.16.0.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.0.2,8.8.8.8

# Join Domain
Add-Computer -DomainName "example.local" -Credential (Get-Credential) -Restart
WiFi Configuration (802.1X):
Connect to Student_Network
Select WPA2-Enterprise
Authentication: Microsoft: Protected EAP (PEAP)
Enter credentials: student01@example.local
Verify certificate from DC
PC1 Configuration (Linux)
Network Configuration:
# Edit netplan (if static IP needed)
sudo nano /etc/netplan/01-netcfg.yaml

network:
  version: 2
  ethernets:
    ens33:
      dhcp4: true
      nameservers:
        addresses: [192.168.0.2, 8.8.8.8]

sudo netplan apply
WiFi 802.1X Configuration:
# Install wpasupplicant
sudo apt install wpasupplicant -y

# Create wpa_supplicant config
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf

# Add:
network={
    ssid="Student_Network"
    key_mgmt=WPA-EAP
    eap=PEAP
    identity="student01@example.local"
    password="Student@123"
    phase2="auth=MSCHAPV2"
}

# Connect
sudo wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf
sudo dhclient wlan0
🔐 11. Advanced Security Configurations
Group Policy for Students (DC)
# Create GPO for Students
New-GPO -Name "Student_Security_Policy" | New-GPLink -Target "OU=Students,DC=example,DC=local"

# Configure GPO Settings (use Group Policy Management Console)
# Computer Configuration > Policies > Windows Settings > Security Settings

# Disable USB Storage
Set-GPRegistryValue -Name "Student_Security_Policy" `
  -Key "HKLM\System\CurrentControlSet\Services\USBSTOR" `
  -ValueName "Start" -Type DWord -Value 4

# Firewall Rules (Block outgoing FTP)
# Computer Configuration > Windows Firewall with Advanced Security
# Outbound Rules > Block TCP 21

# Web Filtering via Group Policy
# User Configuration > Policies > Administrative Templates > Internet Explorer
Port Security on Access Switches
! On DIST-SW1 - Port connecting to PC1
interface FastEthernet0/2
 switchport mode access
 switchport access vlan 200
 
 ! Enable Port Security
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 
 ! DHCP Snooping
 ip dhcp snooping
 ip dhcp snooping vlan 200
 no ip dhcp snooping information option
 
 ! Dynamic ARP Inspection
 ip arp inspection vlan 200
 ip arp inspection validate src-mac dst-mac ip
 
 ! Storm Control
 storm-control broadcast level 10.00
 storm-control multicast level 10.00
 
 spanning-tree portfast
 spanning-tree bpduguard enable
exit

! Trust uplink ports
interface GigabitEthernet0/1
 ip dhcp snooping trust
 ip arp inspection trust
exit
Private VLAN for Student Isolation
! On DIST-SW1 and DIST-SW2

! Create Private VLAN
vlan 200
 private-vlan primary
exit

vlan 210
 private-vlan isolated
exit

vlan 220
 private-vlan community
exit

vlan 200
 private-vlan association 210,220
exit

! Configure Promiscuous Port (to Gateway)
interface GigabitEthernet0/1
 switchport mode private-vlan promiscuous
 switchport private-vlan mapping 200 210,220
exit

! Configure Isolated Ports (Student PCs)
interface range FastEthernet0/1-10
 switchport mode private-vlan host
 switchport private-vlan host-association 200 210
exit
Logging and SIEM Integration
! Enable detailed logging on all devices

! On Routers
logging buffered 51200 debugging
logging console critical
logging monitor informational
logging trap notifications
logging facility local5
logging source-interface Loopback0

! Send logs to SIEM
logging host 192.168.0.100 transport udp port 514

! Enable specific logs
logging on
service timestamps debug datetime msec
service timestamps log datetime msec
service sequence-numbers
📈 12. Performance Optimization
QoS Configuration
! On R1 and R2 - Prioritize critical traffic

! Define Class Maps
class-map match-any VOICE
 match ip dscp ef
exit

class-map match-any VIDEO
 match ip dscp af41
exit

class-map match-any CRITICAL_DATA
 match access-group name CRITICAL_TRAFFIC
exit

! Define ACL for Critical Traffic
ip access-list extended CRITICAL_TRAFFIC
 permit tcp any host 192.168.0.2 eq domain
 permit tcp any host 192.168.0.2 eq ldap
 permit tcp any host 192.168.0.10 eq https
exit

! Define Policy Map
policy-map WAN_QOS
 class VOICE
  priority percent 30
 class VIDEO
  bandwidth percent 20
 class CRITICAL_DATA
  bandwidth percent 30
 class class-default
  fair-queue
  random-detect
exit

! Apply to WAN interface
interface GigabitEthernet0/0
 service-policy output WAN_QOS
exit
Load Balancing
! On CORE-SW - Configure CEF for load balancing

ip cef
ip cef load-sharing algorithm universal

! Verify
show ip cef
show ip route
Bandwidth Management on AP
! On Access Point
dot11 ssid Student_Network
 rate-limit client 5000 downstream
 rate-limit client 2000 upstream
exit
🔄 13. High Availability & Redundancy
HSRP Tracking
! On R1 - Track Internet connectivity

! Track object for WAN interface
track 1 interface GigabitEthernet0/0 line-protocol
exit

! Track reachability to Internet gateway
track 2 ip sla 1 reachability
exit

ip sla 1
 icmp-echo 8.8.8.8 source-interface GigabitEthernet0/0
 frequency 10
exit

ip sla schedule 1 life forever start-time now

! Apply tracking to HSRP
interface GigabitEthernet0/1
 standby 1 track 1 decrement 20
 standby 1 track 2 decrement 20
exit
Switch Stack Configuration
! On DIST-SW1 and DIST-SW2 (if stacking supported)

switch 1 priority 15
switch 2 priority 14

! Verify
show switch
show switch stack-ports
Firewall Failover Configuration
! On Primary Firewall
failover
failover lan unit primary
failover lan interface FAILOVER GigabitEthernet0/3
failover replication http
failover link FAILOVER GigabitEthernet0/3
failover interface ip FAILOVER 10.10.10.1 255.255.255.252 standby 10.10.10.2

! Enable failover
failover

! On Secondary Firewall
failover
failover lan unit secondary
failover lan interface FAILOVER GigabitEthernet0/3
failover interface ip FAILOVER 10.10.10.1 255.255.255.252 standby 10.10.10.2

! Verify
show failover
show failover state
📝 14. Documentation Templates
IP Address Allocation Table
Device
Interface
IP Address
Subnet Mask
Gateway
VLAN
R1
Gi0/0 (WAN)
DHCP
-
ISP
-
R1
Gi0/1 (LAN)
172.17.0.2
/29
-
-
R2
Gi0/0 (WAN)
DHCP
-
ISP
-
R2
Gi0/1 (LAN)
172.17.0.3
/29
-
-
HSRP VIP
-
172.17.0.1
/29
-
-
CORE-SW
Gi0/1
172.17.0.4
/29
172.17.0.1
-
CORE-SW
VLAN 200
172.16.0.1
/22
-
200
CORE-SW
VLAN 201
192.168.0.1
/24
-
201
DC
NIC
192.168.0.2
/24
192.168.0.1
201
ADC
NIC
192.168.0.3
/24
192.168.0.1
201
WEB
NIC
192.168.0.10
/24
192.168.0.1
201
FTP
NIC
192.168.0.11
/24
192.168.0.1
201
MAIL
NIC
192.168.0.12
/24
192.168.0.1
201
AP
Management
DHCP
-
172.16.0.1
200
PC1
NIC
DHCP
-
172.16.0.1
200
VLAN Assignment Table
VLAN ID
Name
Subnet
Gateway
Purpose
200
CLIENT_NETWORK
172.16.0.0/22
172.16.0.1
Students, AP, PCs
201
SERVER_NETWORK
192.168.0.0/24
192.168.0.1
Servers (DC, ADC, WEB, FTP, MAIL)
999
NATIVE_VLAN
-
-
Trunk native VLAN
Port Assignment Table
DIST-SW1:
Port
Type
VLAN
Connected To
Description
Gi0/1
Trunk
All
CORE-SW
Uplink (EtherChannel)
Gi0/2-3
Trunk
201
DIST-SW2
FTP EtherChannel
Gi0/4
Trunk
All
Firewall
Firewall connection
Fa0/1
Access
200
AP
Wireless Access Point
Fa0/2
Access
200
PC1
Client PC
DIST-SW2:
Port
Type
VLAN
Connected To
Description
Gi0/1
Trunk
All
CORE-SW
Uplink (EtherChannel)
Gi0/2-3
Trunk
201
DIST-SW1
FTP EtherChannel
Gi0/4
Trunk
All
Redundant-FW
Redundant Firewall
Fa0/1
Access
201
DC
Domain Controller
Fa0/2
Access
201
ADC
Additional DC
Fa0/3
Access
201
WEB
Web Server
Fa0/4
Access
201
FTP
FTP Server
Fa0/5
Access
201
MAIL
Mail Server
Access Control Matrix
Source
Destination
Service
Action
Reason
Students (VLAN 200)
DC (192.168.0.2)
DNS (53)
Allow
Name resolution
Students (VLAN 200)
DC (192.168.0.2)
LDAP (389)
Allow
Authentication
Students (VLAN 200)
DC (192.168.0.2)
Kerberos (88)
Allow
Authentication
Students (VLAN 200)
WEB (192.168.0.10)
HTTP/HTTPS
Allow
Web access
Students (VLAN 200)
MAIL (192.168.0.12)
SMTP/POP3/IMAP
Allow
Email
Students (VLAN 200)
FTP (192.168.0.11)
FTP (21)
Deny
Restricted
Students (VLAN 200)
Internet
HTTP/HTTPS
Allow
Web browsing
Students (VLAN 200)
Internet
Other
Deny
Security
External
WEB (Public IP)
HTTP/HTTPS
Allow
Public web
External
MAIL (Public IP)
SMTP
Allow
Receive email
External
Other Servers
Any
Deny
Security
🚀 15. Deployment Checklist
Pre-Deployment
[ ] Review network diagram and IP addressing plan
[ ] Verify all hardware availability
[ ] Backup existing configurations (if any)
[ ] Prepare console cables and management laptop
[ ] Document current network state
[ ] Schedule maintenance window
[ ] Notify users about potential downtime
Router Deployment (R1 & R2)
[ ] Configure hostname and management IP
[ ] Set console and VTY passwords
[ ] Configure HSRP with correct priorities
[ ] Verify HSRP virtual IP
[ ] Configure NAT/PAT for Internet access
[ ] Set up OSPF routing
[ ] Configure DHCP (if applicable)
[ ] Test WAN connectivity
[ ] Verify failover functionality
[ ] Save configuration
Core Switch Deployment
[ ] Configure hostname
[ ] Enable IP routing
[ ] Create all VLANs
[ ] Configure SVI interfaces with correct IPs
[ ] Set up DHCP server and pools
[ ] Configure trunk ports
[ ] Enable OSPF
[ ] Configure spanning-tree priorities
[ ] Set up ip helper-address
[ ] Test inter-VLAN routing
[ ] Save configuration
Firewall Deployment
[ ] Configure basic settings (hostname, interfaces)
[ ] Set security zones (inside, outside, DMZ)
[ ] Configure access control lists
[ ] Set up NAT/PAT rules
[ ] Enable IPS/IDS policies
[ ] Configure logging and SNMP
[ ] Test firewall rules
[ ] Configure failover (if redundant)
[ ] Verify high availability
[ ] Save configuration
Distribution Switch Deployment
[ ] Configure hostname on both DIST-SW1 and DIST-SW2
[ ] Create VLANs
[ ] Configure trunk ports to CORE-SW
[ ] Set up EtherChannel (including FTP-only channel)
[ ] Configure access ports for servers and clients
[ ] Enable spanning-tree and set priorities
[ ] Configure port security on access ports
[ ] Enable DHCP snooping and DAI
[ ] Test connectivity between devices
[ ] Save configuration
Server Deployment
Domain Controller:
[ ] Install Windows Server
[ ] Configure static IP address
[ ] Install AD DS role
[ ] Promote to Domain Controller
[ ] Configure DNS zones and records
[ ] Install and configure NPS (RADIUS)
[ ] Create student user accounts
[ ] Configure network policies for 802.1X
[ ] Test RADIUS authentication
[ ] Configure replication to ADC
Additional Servers:
[ ] WEB: Install and configure IIS/Apache
[ ] FTP: Install and configure FTP server
[ ] MAIL: Install and configure mail server
[ ] Configure static IPs for all servers
[ ] Join servers to domain (if Windows)
[ ] Test connectivity to DC
[ ] Configure firewall rules on servers
[ ] Set up backup jobs
Access Point Deployment
[ ] Configure management IP (DHCP or static)
[ ] Set SSID name
[ ] Configure WPA2-Enterprise security
[ ] Set RADIUS server IP and shared secret
[ ] Configure VLAN mapping
[ ] Test wireless connectivity
[ ] Test 802.1X authentication with student account
[ ] Verify DHCP assignment
[ ] Optimize radio settings
[ ] Save configuration
Post-Deployment Testing
[ ] Verify HSRP failover (shutdown R1 interface)
[ ] Test DHCP on client devices
[ ] Verify inter-VLAN routing
[ ] Test NAT/PAT for Internet access
[ ] Verify firewall rules (allow/deny)
[ ] Test wireless authentication (802.1X)
[ ] Verify EtherChannel functionality
[ ] Test server accessibility
[ ] Verify DNS resolution
[ ] Test email sending/receiving
[ ] Verify web server access (internal/external)
[ ] Test FTP restrictions
[ ] Check spanning-tree convergence
[ ] Verify OSPF neighbor relationships
[ ] Test failover scenarios
[ ] Check logging and monitoring
Documentation
[ ] Update network diagram with actual IPs
[ ] Document all passwords and keys
[ ] Create configuration backup
[ ] Document troubleshooting procedures
[ ] Create user guides for WiFi connection
[ ] Document change log
[ ] Create maintenance schedule
[ ] Prepare handover documentation
🛠️ 16. Troubleshooting Guide
Common Issues and Solutions
Issue: Students cannot get IP address
Symptoms:
DHCP request timeout
Self-assigned IP (169.254.x.x)
Diagnosis:
! On CORE-SW
show ip dhcp binding
show ip dhcp server statistics
show ip dhcp pool CLIENT_POOL

! Check DHCP relay
show ip interface Vlan200 | include helper
Solutions:
Verify DHCP pool not exhausted
Check ip helper-address configuration
Verify HSRP is active
Check firewall not blocking DHCP (UDP 67/68)
Issue: HSRP not working
Symptoms:
Both routers showing Active
No failover occurring
Diagnosis:
! On R1 and R2
show standby brief
show standby GigabitEthernet0/1
show ip interface brief
Solutions:
Verify HSRP group numbers match
Check authentication key matches
Verify routers on same subnet
Check for HSRP version mismatch
Ensure preempt is configured
Issue: WiFi authentication failing
Symptoms:
"Can't connect to network"
"Authentication error"
Diagnosis:
# On DC
Get-EventLog -LogName Security | Where-Object {$_.EventID -eq 6273}
# On AP
show aaa sessions
show radius statistics
debug radius authentication
Solutions:
Verify RADIUS shared secret matches on AP and DC
Check NPS policy conditions
Verify student account is enabled in AD
Check certificate validity
Verify AP can reach DC (192.168.0.2)
Issue: Cannot access servers from client network
Symptoms:
Ping to servers fails
Web server not accessible
Diagnosis:
! On CORE-SW
show ip route
show ip interface Vlan200
show ip interface Vlan201
ping 192.168.0.10 source 172.16.0.1

! Check ACL
show ip access-lists STUDENT_CONTROL
Solutions:
Verify inter-VLAN routing enabled
Check ACL not blocking traffic
Verify server gateway points to 192.168.0.1
Check firewall rules
Verify routes in routing table
Issue: Internet access not working
Symptoms:
Cannot ping 8.8.8.8
Cannot browse websites
Diagnosis:
! On R1/R2
show ip nat translations
show ip nat statistics
show ip route

! Test
ping 8.8.8.8 source GigabitEthernet0/1
Solutions:
Verify default route exists
Check NAT configuration
Verify ISP connection
Check ACL not blocking outbound traffic
Verify DNS is working
Issue: FTP traffic not using dedicated EtherChannel
Symptoms:
FTP traffic going through regular trunk
High latency on FTP transfers
Diagnosis:
! On DIST-SW1
show etherchannel summary
show etherchannel port-channel
show interfaces Port-channel10
show vlan
Solutions:
Verify Port-channel10 exists and is up
Check VLAN 201 allowed on FTP EtherChannel
Verify member ports are in correct channel-group
Check load-balancing algorithm
Verify spanning-tree not blocking ports
📞 17. Support and Maintenance
Daily Monitoring Tasks
# Create monitoring script
#!/bin/bash

# Check HSRP status
echo "=== HSRP Status ==="
sshpass -p 'admin' ssh admin@172.17.0.2 "show standby brief"

# Check DHCP bindings
echo "=== DHCP Bindings ==="
sshpass -p 'admin' ssh admin@172.17.0.4 "show ip dhcp binding | count"

# Check interface status
echo "=== Interface Status ==="
sshpass -p 'admin' ssh admin@172.17.0.4 "show ip interface brief"

# Check CPU and Memory
echo "=== Resource Usage ==="
sshpass -p 'admin' ssh admin@172.17.0.2 "show processes cpu | include CPU"
sshpass -p 'admin' ssh admin@172.17.0.2 "show memory statistics"

# Check for errors
echo "=== Interface Errors ==="
sshpass -p 'admin' ssh admin@172.17.0.4 "show interfaces | include error"
Weekly Maintenance Tasks
Review firewall logs for suspicious activity
Check backup integrity
Verify AD replication status
Review DHCP lease utilization
Check disk space on servers
Update virus definitions
Review user account activity
Monthly Maintenance Tasks
Apply security patches to servers
Review and update firewall rules
Audit user accounts in AD
Test disaster recovery procedures
Review bandwidth utilization
Update documentation
Conduct security audit
Contact Information
Role
Name
Phone
Email
Network Administrator
[Your Name]
[Phone]
[Email]
Server Administrator
[Name]
[Phone]
[Email]
Security Officer
[Name]
[Phone]
[Email]
ISP Support
[ISP]
[Phone]
[Email]
Hardware Vendor
[Vendor]
[Phone]
[Email]
📚 Reference Commands Quick Sheet
Router Commands
show ip route
show ip interface brief
show standby brief
show ip nat translations
show ip ospf neighbor
show running-config
Switch Commands
show vlan brief
show interfaces trunk
show spanning-tree
show ip dhcp binding
show etherchannel summary
show mac address-table
Firewall Commands
show access-list
show conn
show xlate
show failover
show logging
Windows Server Commands
Get-ADUser -Filter *
Get-EventLog -LogName Security
Get-DnsServerResourceRecord -ZoneName "example.local"
Test-NetConnection -ComputerName 192.168.0.10 -Port 80
End of Configuration Guide
This documentation was created for the enterprise network deployment as per the provided network diagram. All configurations should be tested in a lab environment before production deployment.
📋 Network Overview
IP Addressing Plan
Management & Routing:
HSRP Virtual Gateway: 172.17.0.1/29
R1 (Active): 172.17.0.2/29
R2 (Standby): 172.17.0.3/29
Core Switch: 172.17.0.4/29
VLANs:
VLAN 200 (Client/AP Network): 172.16.0.0/22 (1022 hosts)
Gateway: 172.16.0.1
AP: DHCP assigned
PC1: DHCP assigned
VLAN 200 (Servers - DC/ADC): 192.168.0.0/29 (6 hosts)
DC: 192.168.0.2
ADC: 192.168.0.3
VLAN 200 (Servers - WEB/FTP/MAIL): 192.168.0.0/29 (6 hosts)
WEB: 192.168.0.10
FTP: 192.168.0.11
MAIL: 192.168.0.12
🔧 1. Edge Router Configuration (R1 & R2)
Router R1 (Active HSRP Gateway)
enable
configure terminal
hostname R1

! WAN Interface to Cloud1
interface GigabitEthernet0/0
 description WAN_to_Cloud1
 ip address dhcp
 ip nat outside
 no shutdown
exit

! LAN Interface to Core Switch
interface GigabitEthernet0/1
 description LAN_to_CORE-SW
 ip address 172.17.0.2 255.255.255.248
 ip nat inside
 no shutdown
 
 ! HSRP Configuration
 standby version 2
 standby 1 ip 172.17.0.1
 standby 1 priority 110
 standby 1 preempt
 standby 1 authentication md5 key-string hsrp_secret
exit

! Static Default Route (if not using OSPF to Cloud)
ip route 0.0.0.0 0.0.0.0 GigabitEthernet0/0

! NAT Configuration for Internet Access
access-list 10 permit 172.16.0.0 0.0.3.255
access-list 10 permit 192.168.0.0 0.0.0.255
ip nat inside source list 10 interface GigabitEthernet0/0 overload

! OSPF Configuration (for internal routing)
router ospf 1
 router-id 1.1.1.1
 network 172.17.0.0 0.0.0.7 area 0
 passive-interface GigabitEthernet0/0
 default-information originate
exit

! DNS Configuration
ip domain-lookup
ip name-server 192.168.0.2
ip name-server 8.8.8.8

! Save Configuration
end
write memory
Router R2 (Standby HSRP Gateway)
enable
configure terminal
hostname R2

! WAN Interface to Cloud2
interface GigabitEthernet0/0
 description WAN_to_Cloud2
 ip address dhcp
 ip nat outside
 no shutdown
exit

! LAN Interface to Core Switch
interface GigabitEthernet0/1
 description LAN_to_CORE-SW
 ip address 172.17.0.3 255.255.255.248
 ip nat inside
 no shutdown
 
 ! HSRP Configuration
 standby version 2
 standby 1 ip 172.17.0.1
 standby 1 priority 100
 standby 1 preempt
 standby 1 authentication md5 key-string hsrp_secret
exit

! Static Default Route
ip route 0.0.0.0 0.0.0.0 GigabitEthernet0/0

! NAT Configuration
access-list 10 permit 172.16.0.0 0.0.3.255
access-list 10 permit 192.168.0.0 0.0.0.255
ip nat inside source list 10 interface GigabitEthernet0/0 overload

! OSPF Configuration
router ospf 1
 router-id 2.2.2.2
 network 172.17.0.0 0.0.0.7 area 0
 passive-interface GigabitEthernet0/0
 default-information originate
exit

! DNS Configuration
ip domain-lookup
ip name-server 192.168.0.2
ip name-server 8.8.8.8

end
write memory
🛡️ 2. Core Switch Configuration (Layer 3)
enable
configure terminal
hostname CORE-SW

! Enable IP Routing
ip routing

! VLAN Creation
vlan 200
 name CLIENT_NETWORK
exit

vlan 201
 name SERVER_NETWORK
exit

vlan 999
 name NATIVE_VLAN
exit

! SVI Configuration (Gateway Interfaces)

! Interface VLAN 200 - Client/AP Network
interface Vlan200
 description Gateway_for_Clients_and_AP
 ip address 172.16.0.1 255.255.252.0
 ip helper-address 172.17.0.1
 no shutdown
exit

! Interface VLAN 201 - Server Network
interface Vlan201
 description Gateway_for_Servers
 ip address 192.168.0.1 255.255.255.0
 no shutdown
exit

! Uplink to R1
interface GigabitEthernet0/1
 description Uplink_to_R1
 no switchport
 ip address 172.17.0.4 255.255.255.248
 no shutdown
exit

! Uplink to R2
interface GigabitEthernet0/2
 description Uplink_to_R2
 no switchport
 ip address 172.17.0.5 255.255.255.248
 no shutdown
exit

! Trunk to Firewall
interface GigabitEthernet0/3
 description Trunk_to_Firewall
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 200,201
 no shutdown
exit

! Trunk to Redundant-Firewall
interface GigabitEthernet0/4
 description Trunk_to_Redundant_Firewall
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 200,201
 no shutdown
exit

! Trunk to DIST-SW1
interface GigabitEthernet0/5
 description Trunk_to_DIST-SW1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 200,201
 channel-group 1 mode active
 no shutdown
exit

! Trunk to DIST-SW2
interface GigabitEthernet0/6
 description Trunk_to_DIST-SW2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 200,201
 channel-group 2 mode active
 no shutdown
exit

! EtherChannel Configuration
interface Port-channel1
 description EtherChannel_to_DIST-SW1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
exit

interface Port-channel2
 description EtherChannel_to_DIST-SW2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
exit

! OSPF Configuration
router ospf 1
 router-id 3.3.3.3
 network 172.17.0.0 0.0.0.7 area 0
 network 172.16.0.0 0.0.3.255 area 0
 network 192.168.0.0 0.0.0.255 area 0
exit

! DHCP Server Configuration
ip dhcp excluded-address 172.16.0.1 172.16.0.10
ip dhcp excluded-address 192.168.0.1 192.168.0.5

ip dhcp pool CLIENT_POOL
 network 172.16.0.0 255.255.252.0
 default-router 172.16.0.1
 dns-server 192.168.0.2 8.8.8.8
 domain-name example.local
 lease 7
exit

! Spanning Tree Configuration
spanning-tree mode rapid-pvst
spanning-tree vlan 200,201 priority 4096

end
write memory
🔥 3. Firewall Configuration
Primary Firewall
enable
configure terminal
hostname FIREWALL

! VLAN Configuration
vlan 200
 name CLIENT_NETWORK
exit

vlan 201
 name SERVER_NETWORK
exit

vlan 100
 name DMZ
exit

! Inside Interface (Client Network)
interface Vlan200
 description Inside_Network
 ip address 172.16.0.254 255.255.252.0
 security-level 100
 no shutdown
exit

! DMZ Interface (Server Network)
interface Vlan201
 description DMZ_Server_Network
 ip address 192.168.0.254 255.255.255.0
 security-level 50
 no shutdown
exit

! Outside Interface
interface GigabitEthernet0/0
 description Outside_to_CORE
 nameif outside
 security-level 0
 ip address 172.17.0.10 255.255.255.248
 no shutdown
exit

! Access Control Lists

! Allow Clients to DC/ADC for Authentication
access-list INSIDE_IN extended permit tcp 172.16.0.0 255.255.252.0 host 192.168.0.2 eq domain
access-list INSIDE_IN extended permit udp 172.16.0.0 255.255.252.0 host 192.168.0.2 eq domain
access-list INSIDE_IN extended permit tcp 172.16.0.0 255.255.252.0 host 192.168.0.2 eq ldap
access-list INSIDE_IN extended permit tcp 172.16.0.0 255.255.252.0 host 192.168.0.2 eq 88
access-list INSIDE_IN extended permit tcp 172.16.0.0 255.255.252.0 host 192.168.0.3 eq ldap

! Allow Clients to WEB Server
access-list INSIDE_IN extended permit tcp 172.16.0.0 255.255.252.0 host 192.168.0.10 eq www
access-list INSIDE_IN extended permit tcp 172.16.0.0 255.255.252.0 host 192.168.0.10 eq https

! Allow Clients to MAIL Server
access-list INSIDE_IN extended permit tcp 172.16.0.0 255.255.252.0 host 192.168.0.12 eq smtp
access-list INSIDE_IN extended permit tcp 172.16.0.0 255.255.252.0 host 192.168.0.12 eq pop3
access-list INSIDE_IN extended permit tcp 172.16.0.0 255.255.252.0 host 192.168.0.12 eq imap

! Block Clients from FTP (only admin access)
access-list INSIDE_IN extended deny tcp 172.16.0.0 255.255.252.0 host 192.168.0.11 eq ftp

! Allow all other Internet traffic
access-list INSIDE_IN extended permit ip 172.16.0.0 255.255.252.0 any

! Apply ACL
access-group INSIDE_IN in interface Vlan200

! NAT Configuration (PAT)
object network INSIDE_NET
 subnet 172.16.0.0 255.255.252.0
 nat (inside,outside) dynamic interface

! Static NAT for Public Services
static (dmz,outside) tcp interface www 192.168.0.10 www
static (dmz,outside) tcp interface https 192.168.0.10 https
static (dmz,outside) tcp interface smtp 192.168.0.12 smtp

! IPS/IDS Configuration
ip audit name IDS_POLICY attack action alarm drop reset
ip audit interface Vlan200 IDS_POLICY

! Route to Internet via CORE
route outside 0.0.0.0 0.0.0.0 172.17.0.1 1

end
write memory
Redundant Firewall
! Similar configuration as Primary Firewall
! Configure with different IP addresses and enable failover

enable
configure terminal
hostname REDUNDANT-FIREWALL

! Enable Failover
failover
failover lan unit secondary
failover lan interface FAILOVER GigabitEthernet0/3
failover interface ip FAILOVER 10.10.10.2 255.255.255.252 standby 10.10.10.1

! Continue with similar configuration as Primary...
🔀 4. Distribution Switch Configuration
DIST-SW1
enable
configure terminal
hostname DIST-SW1

! VLAN Configuration
vlan 200
 name CLIENT_NETWORK
exit

vlan 201
 name SERVER_NETWORK
exit

! Trunk to CORE-SW
interface GigabitEthernet0/1
 description Trunk_to_CORE-SW
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 channel-group 1 mode active
 no shutdown
exit

! FTP EtherChannel (Dedicated for FTP Traffic)
interface range GigabitEthernet0/2-3
 description EtherChannel_to_DIST-SW2_FTP_Only
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 201
 channel-group 10 mode active
 no shutdown
exit

interface Port-channel10
 description FTP_EtherChannel
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 201
exit

! Trunk to Firewall
interface GigabitEthernet0/4
 description Trunk_to_Firewall
 switchport trunk encapsulation dot1q
 switchport mode trunk
 no shutdown
exit

! Access Ports to AP
interface FastEthernet0/1
 description Access_to_AP
 switchport mode access
 switchport access vlan 200
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
exit

! Access Port to PC1
interface FastEthernet0/2
 description Access_to_PC1
 switchport mode access
 switchport access vlan 200
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
exit

! Spanning Tree Configuration
spanning-tree mode rapid-pvst
spanning-tree vlan 200 priority 8192
spanning-tree vlan 201 priority 12288

end
write memory
DIST-SW2
enable
configure terminal
hostname DIST-SW2

! VLAN Configuration
vlan 200
 name CLIENT_NETWORK
exit

vlan 201
 name SERVER_NETWORK
exit

! Trunk to CORE-SW
interface GigabitEthernet0/1
 description Trunk_to_CORE-SW
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 channel-group 2 mode active
 no shutdown
exit

! FTP EtherChannel
interface range GigabitEthernet0/2-3
 description EtherChannel_to_DIST-SW1_FTP_Only
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 201
 channel-group 10 mode active
 no shutdown
exit

! Trunk to Redundant Firewall
interface GigabitEthernet0/4
 description Trunk_to_Redundant_Firewall
 switchport trunk encapsulation dot1q
 switchport mode trunk
 no shutdown
exit

! Access Ports to Servers (VLAN 201)
interface FastEthernet0/1
 description Access_to_DC
 switchport mode access
 switchport access vlan 201
 spanning-tree portfast
 no shutdown
exit

interface FastEthernet0/2
 description Access_to_ADC
 switchport mode access
 switchport access vlan 201
 spanning-tree portfast
 no shutdown
exit

interface FastEthernet0/3
 description Access_to_WEB_Server
 switchport mode access
 switchport access vlan 201
 spanning-tree portfast
 no shutdown
exit

interface FastEthernet0/4
 description Access_to_FTP_Server
 switchport mode access
 switchport access vlan 201
 spanning-tree portfast
 no shutdown
exit

interface FastEthernet0/5
 description Access_to_MAIL_Server
 switchport mode access
 switchport access vlan 201
 spanning-tree portfast
 no shutdown
exit

! Spanning Tree Configuration
spanning-tree mode rapid-pvst
spanning-tree vlan 200 priority 12288
spanning-tree vlan 201 priority 8192

end
write memory
📡 5. Access Point Configuration
AP (Wireless Configuration)
# Assuming Cisco Wireless AP

enable
configure terminal
hostname AP

! Management IP (will get from DHCP)
interface BVI1
 ip address dhcp
exit

! Radio Interface
dot11 ssid Student_Network
 vlan 200
 authentication open eap eap_methods
 authentication key-management wpa version 2
 mbssid guest-mode
exit

! 802.1X Configuration
dot11 aaa authentication ssid Student_Network
dot11 aaa authorization ssid Student_Network

! RADIUS Server Configuration
radius-server host 192.168.0.2 auth-port 1812 acct-port 1813 key RadiusSecret123

! AAA Configuration
aaa new-model
aaa group server radius RAD_GROUP
 server 192.168.0.2 auth-port 1812 acct-port 1813
exit

aaa authentication login eap_methods group RAD_GROUP
aaa authorization exec default group RAD_GROUP

! Bridge to Ethernet
interface Dot11Radio0
 encryption vlan 200 mode ciphers aes-ccm
 ssid Student_Network
 no shutdown
exit

interface GigabitEthernet0
 description Trunk_to_DIST-SW1
 switchport mode trunk
 switchport trunk allowed vlan 200
exit

end
write memory
🖥️ 6. Server Configurations
Domain Controller (DC) - Windows Server Configuration
IP Configuration:
IP Address: 192.168.0.2/24
Gateway: 192.168.0.1
DNS: 127.0.0.1, 8.8.8.8
PowerShell Commands:
# Set Static IP
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.0.2 -PrefixLength 24 -DefaultGateway 192.168.0.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 127.0.0.1,8.8.8.8

# Install AD DS Role
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# Promote to Domain Controller
Install-ADDSForest `
  -DomainName "example.local" `
  -DomainNetbiosName "EXAMPLE" `
  -InstallDns `
  -SafeModeAdministratorPassword (ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force) `
  -Force

# Install NPS (RADIUS Server)
Install-WindowsFeature -Name NPAS -IncludeManagementTools

# Configure NPS for 802.1X
# Use GUI: Server Manager > Tools > Network Policy Server
# Add RADIUS Clients:
#   - AP: 172.16.0.x with shared secret
# Create Network Policy for 802.1X Wireless
DNS Records Configuration:
# Add DNS Records
Add-DnsServerResourceRecordA -Name "web" -ZoneName "example.local" -IPv4Address "192.168.0.10"
Add-DnsServerResourceRecordA -Name "ftp" -ZoneName "example.local" -IPv4Address "192.168.0.11"
Add-DnsServerResourceRecordA -Name "mail" -ZoneName "example.local" -IPv4Address "192.168.0.12"

Add-DnsServerResourceRecordMX -Name "." -ZoneName "example.local" -MailExchange "mail.example.local" -Preference 10
Add-DnsServerResourceRecordCName -Name "www" -ZoneName "example.local" -HostNameAlias "web.example.local"
Additional Domain Controller (ADC)
IP Configuration:
IP Address: 192.168.0.3/24
Gateway: 192.168.0.1
DNS: 192.168.0.2, 127.0.0.1
# Set Static IP
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.0.3 -PrefixLength 24 -DefaultGateway 192.168.0.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.0.2,127.0.0.1

# Join Domain and Promote to DC
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

Install-ADDSDomainController `
  -DomainName "example.local" `
  -InstallDns `
  -Credential (Get-Credential) `
  -SafeModeAdministratorPassword (ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force) `
  -Force

# Configure AD Replication
# Automatic replication with primary DC
WEB Server Configuration
IP Configuration:
IP: 192.168.0.10/24
Gateway: 192.168.0.1
DNS: 192.168.0.2
IIS Installation (Windows):
# Install IIS
Install-WindowsFeature -Name Web-Server -IncludeManagementTools

# Configure Default Website
Import-Module WebAdministration
New-Item -Path "C:\inetpub\wwwroot\index.html" -ItemType File -Value "<h1>Welcome to Corporate Website</h1>"

# Enable HTTPS
New-SelfSignedCertificate -DnsName "web.example.local" -CertStoreLocation "cert:\LocalMachine\My"
# Bind certificate to IIS site via IIS Manager
Apache Installation (Linux):
# Ubuntu/Debian
sudo apt update
sudo apt install apache2 -y

# Configure Network
sudo nano /etc/netplan/00-installer-config.yaml
# Add:
network:
  ethernets:
    ens33:
      addresses: [192.168.0.10/24]
      gateway4: 192.168.0.1
      nameservers:
        addresses: [192.168.0.2, 8.8.8.8]

sudo netplan apply

# Start Apache
sudo systemctl start apache2
sudo systemctl enable apache2
FTP Server Configuration
IP Configuration:
IP: 192.168.0.11/24
Gateway: 192.168.0.1
Windows FTP Server:
# Install FTP Server
Install-WindowsFeature -Name Web-FTP-Server -IncludeManagementTools

# Create FTP Site
Import-Module WebAdministration
New-WebFtpSite -Name "Corporate FTP" -Port 21 -PhysicalPath "C:\FTPRoot"

# Configure User Authentication
# Use AD credentials via IIS Manager
Linux FTP (vsftpd):
sudo apt install vsftpd -y

# Configure vsftpd
sudo nano /etc/vsftpd.conf
# Edit:
anonymous_enable=NO
local_enable=YES
write_enable=YES
chroot_local_user=YES

sudo systemctl restart vsftpd
MAIL Server Configuration
IP Configuration:
IP: 192.168.0.12/24
Gateway: 192.168.0.1
Windows Mail Server (hMailServer):
# Download and install hMailServer
# Configure during installation:
# - Database: Built-in
# - Domain: example.local
# - SMTP Port: 25
# - POP3 Port: 110
# - IMAP Port: 143

# Set DNS to point to DC
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.0.2
Linux Mail Server (Postfix + Dovecot):
# Install Postfix and Dovecot
sudo apt install postfix dovecot-core dovecot-imapd dovecot-pop3d -y

# Configure Postfix
sudo nano /etc/postfix/main.cf
# Edit:
myhostname = mail.example.local
mydomain = example.local
myorigin = $mydomain
inet_interfaces = all
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain

# Configure Dovecot
sudo nano /etc/dovecot/dovecot.conf
# Uncomment:
protocols = imap pop3

# Restart services
sudo systemctl restart postfix dovecot
🔒 7. Security Hardening
ACL for Student Data Control
! On CORE-SW or Firewall

! Extended ACL for Student Control
ip access-list extended STUDENT_CONTROL
 
 ! Allow DNS to DC
 permit udp 172.16.0.0 0.0.3.255 host 192.168.0.2 eq domain
 permit tcp 172.16.0.0 0.0.3.255 host 192.168.0.2 eq domain
 
 ! Allow LDAP/Kerberos for Authentication
 permit tcp 172.16.0.0 0.0.3.255 host 192.168.0.2 eq ldap
 permit tcp 172.16.0.0 0.0.3.255 host 192.168.0.2 eq 88
 permit tcp 172.16.0.0 0.0.3.255 host 192.168.0.3 eq ldap
 
 ! Allow HTTP/HTTPS to WEB Server
 permit tcp 172.16.0.0 0.0.3.255 host 192.168.0.10 eq www
 permit tcp 172.16.0.0 0.0.3.255 host 192.168.0.10 eq https
 
 ! Allow Email Access
 permit tcp 172.16.0.0 0.0.3.255 host 192.168.0.12 eq smtp
 permit tcp 172.16.0.0 0.0.3.255 host 192.168.0.12 eq pop3
 permit tcp 172.16.0.0 0.0.3.255 host 192.168.0.12 eq imap
 
 ! DENY FTP Access (Only for authorized users)
 deny tcp 172.16.0.0 0.0.3.255 host 192.168.0.11 eq ftp log
 
 ! Allow Internet Access (HTTP/HTTPS only)
 permit tcp 172.16.0.0 0.0.3.255 any eq www
 permit tcp 172.16.0.0 0.0.3.255 any eq https
 permit tcp 172.16.0.0 0.0.3.255 any eq 8080
 
 ! Deny everything else
 deny ip any any log

! Apply ACL
interface Vlan200
 ip access-group STUDENT_CONTROL in
Firewall Rules for DMZ Zones
! Inside Zone → DMZ Zone
access-list INSIDE_TO_DMZ extended permit tcp 172.16.0.0 255.255.252.0 192.168.0.0 255.255.255.0 eq www
access-list INSIDE_TO_DMZ extended permit tcp 172.16.0.0 255.255.252.0 192.168.0.0 255.255.255.0 eq https
access-list INSIDE_TO_DMZ extended permit tcp 172.16.0.0 255.255.252.0 192.168.0.0 255.255.255.0 eq smtp
access-list INSIDE_TO_DMZ extended deny ip any any

! Outside Zone → DMZ Zone (Public Access)
access-list OUTSIDE_TO_DMZ extended permit tcp any host 192.168.0.10 eq www
access-list OUTSIDE_TO_DMZ extended permit tcp any host 192.168.0.10 eq https
access-list OUTSIDE_TO_DMZ extended permit tcp any host 192.168.0.12 eq smtp
access-list OUTSIDE_TO_DMZ extended deny ip any any

! DMZ → Inside (Restricted)
access-list DMZ_TO_INSIDE extended deny ip any any
📊 8. Monitoring & Backup
SNMP Configuration
! On all devices
snmp-server community public RO
snmp-server community private RW
snmp-server host 192.168.0.100 version 2c public
snmp-server enable traps
Syslog Configuration
! Configure Syslog Server
logging host 192.168.0.101
logging trap informational
logging source-interface Vlan201
Configuration Backup
# Automated backup script (Linux)
#!/bin/bash
BACKUP_DIR="/backup/network-configs"
DATE=$(date +%Y%m%d)

# Backup router configs
sshpass -p 'admin' ssh admin@172.17.0.2 "show run" > $BACKUP_DIR/R1-$DATE.cfg
sshpass -p 'admin' ssh admin@172.17.0.3 "show run" > $BACKUP_DIR/R2-$DATE.cfg

# Backup switch configs
sshpass -p 'admin' ssh admin@172.17.0.4 "show run" > $BACKUP_DIR/CORE-SW-$DATE.cfg

# Add to crontab for daily backup
# 0 2 * * * /scripts/backup-network.sh

---

## 📝 Author

**Shuvo**  
Enterprise Networking & Server Infrastructure Project
