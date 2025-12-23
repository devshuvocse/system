# Enterprise Network Infrastructure Project

This project demonstrates a **redundant enterprise network architecture** with routing, switching, firewall security, server infrastructure, and website hosting based on the provided topology diagram.

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

## 📷 Network Diagram

> Refer to the provided topology diagram for physical and logical connectivity.

---

## 📝 Author

**Shuvo**  
Enterprise Networking & Server Infrastructure Project
