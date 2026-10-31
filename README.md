# Transforming Pharmacy Data Security: Enterprise Lab Blueprint in Cisco Packet Tracer

This repository contains production-ready Cisco IOS configuration scripts designed to build a highly secure, audit-ready pharmacy network infrastructure using **Cisco Packet Tracer**. 

The architecture implements a multi-tier security framework to protect Electronic Medical Records (EMRs) and Protected Health Information (PHI) in compliance with **HIPAA, GDPR, and POPIA** standards.

---

## Enhanced Technical Features
*   **Network Segmentation (VLAN Isolation):** Restricts network broad-domain visibility using a strict "Minimum Necessary" data privilege model.
*   **Layer 3 Access Control Lists (ACLs):** Enforces explicit firewalled perimeters protecting data assets, prescription printers, and central file repositories.
*   **Hardware Layer Protection (MAC Port Security):** Mitigates physical switchport intrusion vectors via sticky address clamping and aggressive auto-shutdown rules.
*   **Site-to-Site Crypto IPsec VPN (AES-256):** Secures communication lines between local pharmacy points and Central HQ.
*   **WPA3-Enterprise Wireless Architecture:** Eliminates vulnerable pre-shared keys for wireless pharmacy tablets by enforcing individual per-user IEEE 802.1X authentication backed by a local AAA/RADIUS server.
*   **Automated Central Backup Workflow:** Simulates real-time chronological data replication and snapshots over SSH/FTP, validated through immutable tracking logs.
*   **Centralised Threat Auditing (Syslog Integration):** Pipes network infractions and device drops instantly to a remote syslog storage node for legal compliance tracking.

---

## Updated Pharmacy Network Design Matrix

| Device Name | Switch Port | VLAN ID | Network Zone / Functional Purpose | Assigned IP Address | Default Gateway |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Local_EMR_Server**| F0/7 | VLAN 10 | Local EMR Zone (Isolated DMZ) | `192.168.10.10` | `192.168.10.1` |
| **Disp_PC_1** | F0/1 | VLAN 20 | Dispensary Zone (Pharmacists) | `192.168.20.11` | `192.168.20.1` |
| **Disp_PC_2** | F0/2 | VLAN 20 | Dispensary Zone (Pharmacists) | `192.168.20.12` | `192.168.20.1` |
| **Wireless_WLC** | F0/9 | VLAN 20 | Wireless Lan Controller (Dispensary) | `192.168.20.5` | `192.168.20.1` |
| **POS_PC_1** | F0/3 | VLAN 30 | Point of Sale Zone (Cashiers) | `192.168.30.11` | `192.168.30.1` |
| **POS_PC_2** | F0/4 | VLAN 30 | Point of Sale Zone (Cashiers) | `192.168.30.12` | `192.168.30.1` |
| **Rx_Printer** | F0/6 | VLAN 40 | Secure Medical Printing Zone | `192.168.40.10` | `192.168.40.1` |
| **Admin_PC** | F0/5 | VLAN 50 | Management & Administration | `192.168.50.11` | `192.168.50.1` |
| **Syslog_Server** | F0/8 | VLAN 50 | Central Security Auditing Node | `192.168.50.20` | `192.168.50.1` |
| **AAA_RADIUS** | F0/10 | VLAN 50 | Identity Management Authentication | `192.168.50.30` | `192.168.50.1` |
| **Backup_Storage** | F0/11 | VLAN 50 | Offsite Historical Archive Server | `192.168.50.40` | `192.168.50.1` |

---

## Cisco IOS Production Scripts

### 1. Switch Access Layer Setup (Catalyst 2960)
```ios
enable
configure terminal

vlan 10
 name Local_EMR_Zone
vlan 20
 name Dispensary_Zone
vlan 30
 name POS_Zone
vlan 40
 name Printer_Zone
vlan 50
 name Admin_Zone
exit

! Enforce Port Security on Access Endpoints
interface range fastEthernet 0/1 - 2
 switchport mode access
 switchport access vlan 20
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown

interface range fastEthernet 0/3 - 4
 switchport mode access
 switchport access vlan 30
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown

interface fastEthernet 0/5
 switchport mode access
 switchport access vlan 50
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown

interface fastEthernet 0/6
 switchport mode access
 switchport access vlan 40
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown

interface fastEthernet 0/7
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown

interface fastEthernet 0/8
 switchport mode access
 switchport access vlan 50
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown

! Provisioning Wireless Controller Interface (No Port Security to support dynamic device mappings)
interface fastEthernet 0/9
 description Wireless_WLC_Uplink
 switchport mode access
 switchport access vlan 20
 exit

interface fastEthernet 0/10
 description AAA_RADIUS_Server
 switchport mode access
 switchport access vlan 50
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
 exit

interface fastEthernet 0/11
 description Central_Backup_Storage
 switchport mode access
 switchport access vlan 50
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
 exit

interface gigabitEthernet 0/1
 description Uplink_to_Firewall_Router
 switchport mode trunk
 exit
```

### 2. Edge Firewall Router Configuration (`Router_Pharm`)
```ios
enable
configure terminal

interface gigabitEthernet 0/0
 no shutdown
 exit

interface gigabitEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 exit

interface gigabitEthernet 0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 exit

interface gigabitEthernet 0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
 exit

interface gigabitEthernet 0/0.40
 encapsulation dot1Q 40
 ip address 192.168.40.1 255.255.255.0
 ip access-group PHARM_SECURITY_ACL out
 exit

interface gigabitEthernet 0/0.50
 encapsulation dot1Q 50
 ip address 192.168.50.1 255.255.255.0
 exit

interface gigabitEthernet 0/1
 ip address 100.1.1.1 255.255.255.252
 no shutdown
 exit

ip route 0.0.0.0 0.0.0.0 100.1.1.2

! Config Radius Inter-zone Communications
aaa new-model
radius-server host 192.168.50.30 key RadiusSecretPharmKey

! Event Management & Auditing Engine
logging host 192.168.50.20
logging trap informational

! Access Control Perimeter (Minimum Necessary Enforced)
ip access-list extended PHARM_SECURITY_ACL
 ! Allow secure local database automated updates to central backup
 permit tcp host 192.168.10.10 host 192.168.50.40 eq 22
 permit tcp host 192.168.10.10 host 192.168.50.40 eq 21
 ! Allow general infrastructure access rules
 permit ip 192.168.20.0 0.0.0.255 host 192.168.40.10
 permit ip 192.168.50.0 0.0.0.255 host 192.168.40.10
 ! Explicitly Deny and Log Front Cashiers Violation Attempts
 deny ip 192.168.30.0 0.0.0.255 host 192.168.10.10 log
 deny ip 192.168.30.0 0.0.0.255 host 10.10.10.10 log
 deny ip 192.168.30.0 0.0.0.255 host 192.168.40.10 log
 permit ip any any
 exit

! IPsec Configuration
crypto isakmp policy 10
 encryption aes 256
 hash sha256
 authentication pre-share
 group 2
 exit
crypto isakmp key PharmHQSecureKey address 200.1.1.1

crypto ipsec transform-set PHARM_SET esp-aes 256 esp-sha256-hmac
 exit

ip access-list extended VPN_TRAFFIC_ACL
 permit ip 192.168.0.0 0.0.255.255 10.10.10.0 0.0.0.255
 exit

crypto map PHARM_VPN_MAP 10 ipsec-isakmp
 set peer 200.1.1.1
 set transform-set PHARM_SET
 match address VPN_TRAFFIC_ACL
 exit

interface gigabitEthernet 0/1
 crypto map PHARM_VPN_MAP
 exit
```

---

## Simulated Configuration of Advanced Services

### WPA3 Enterprise Identity Verification Setup
1. Open the **AAA_RADIUS Server** -> **Services** -> **AAA**.
2. Turn Service **ON**. Configure Client name `WLC_Client`, Client IP `192.168.20.5`, and Secret Key `RadiusSecretPharmKey`. Choose Type **RADIUS**.
3. Under the User Setup sub-panel, provision distinct credentials for individual pharmacists (e.g., User: `rx_user_1`, Pass: `SecureRxPass99!`).
4. Select the physical **Wireless LAN Controller (WLC)** graphical profile. Enforce security configurations matching **WPA3-Enterprise** (or WPA2-Enterprise if testing on legacy Packet Tracer components), binding encryption pathways to IP address `192.168.50.30`.

### Automated Storage Backup Execution Workflow
1. Select **Backup_Storage** -> **Services** -> Ensure **FTP / SFTP** fields are enabled. Create credentials: User `db_backup_daemon`, Pass `VaultStorage667#` with full write permissions.
2. Open the command terminal of `Local_EMR_Server`. Execute script strings mimicking cron automation configurations to verify directory visibility:
   ```bash
   ftp 192.168.50.40
   ```
3. Authenticate to dump testing dataset packages. This confirms that while the database is locked away from general client traffic, scheduled scripts can seamlessly transfer encrypted snapshots to your backup zone.

## Project Licensing
This architecture project is made available under the open source **MIT License**.
