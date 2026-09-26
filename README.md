# 🌐 SOHO Network Design & Implementation — XYZ Company Branch
### Enterprise Networking Project #2 | Cisco Packet Tracer

![Cisco](https://img.shields.io/badge/Cisco-Packet%20Tracer-blue)
![Networking](https://img.shields.io/badge/Networking-VLAN%20%7C%20Inter--VLAN%20Routing%20%7C%20DHCP-green)
![Status](https://img.shields.io/badge/Status-Completed-success)

## 📋 Project Overview

XYZ Company is a fast-growing organization opening a new branch near the village of Bonalbo. As the network engineer for this project, I designed and implemented a fully functional **Small Office/Home Office (SOHO)** network for the branch — segmented, secure, and independent from the company's headquarters network — using only Cisco devices in Cisco Packet Tracer.

## 🎯 Business Requirements

| # | Requirement |
|---|-------------|
| a | One router and one switch (Cisco devices only) |
| b | 3 departments: **Admin/IT**, **Finance/HR**, **Customer Service/Reception** |
| c | Each department isolated in its own VLAN |
| d | Wireless network access available to every department |
| e | Hosts automatically obtain an IPv4 address (DHCP) |
| f | All departments must be able to communicate with each other |

**ISP-assigned base network:** `192.168.1.0/24`

## 🧮 Step 1 — Subnetting the Base Network

The base network had to be split into 3 usable subnets — one per department.

```
Base network:      192.168.1.0
No. of subnets:     3
Formula:            2^n ≥ required subnets
2^n = 3  →  n = 2   (borrow 2 bits)

Original mask (Class C):
255.255.255.0  →  11111111.11111111.11111111.00000000

After borrowing 2 bits:
New binary:  11111111.11111111.11111111.11000000
New subnet mask:  255.255.255.192  (/26)

Block size = 256 - 192 = 64
```

### Resulting Subnets

| Subnet | Network ID | Broadcast ID | Usable Host Range |
|--------|-----------|--------------|--------------------|
| 1st | 192.168.1.0 | 192.168.1.63 | 192.168.1.1 – 192.168.1.62 |
| 2nd | 192.168.1.64 | 192.168.1.127 | 192.168.1.65 – 192.168.1.126 |
| 3rd | 192.168.1.128 | 192.168.1.191 | 192.168.1.129 – 192.168.1.190 |

## 🗺️ Step 2 — Network Design

| VLAN ID | Department | Subnet | Router Gateway |
|---------|-----------|--------|-----------------|
| 10 | Admin/IT | 192.168.1.0/26 | 192.168.1.1 |
| 20 | Finance/HR | 192.168.1.64/26 | 192.168.1.65 |
| 30 | Customer Service/Reception | 192.168.1.128/26 | 192.168.1.129 |

Each department segment contains a PC, a printer, and a wireless Access Point, all connected to their respective VLAN on the switch.

**Topology:**

```
                         [ROUTER]
                    Gig0/0 (802.1Q trunk)
                            |
                        [SWITCH]
        --------------------------------------------
        |                   |                       |
   Fa0/2-4 (VLAN 10)   Fa0/5-7 (VLAN 20)      Fa0/8-10 (VLAN 30)
     Admin/IT            Finance/HR          Customer Service
   PC + Printer +       PC + Printer +        PC + Printer +
   Access Point         Access Point          Access Point
```

Because only **one router and one switch** were permitted, inter-VLAN routing was implemented using the **Router-on-a-Stick** method — a single physical router interface, split into logical sub-interfaces, one per VLAN.

## ⚙️ Step 3 — Router Configuration

```
Router> enable
Router# configure terminal

! Enable the trunk-facing physical interface (no IP assigned directly)
interface GigabitEthernet0/0
 no shutdown

! Sub-interface for VLAN 10 - Admin/IT
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.1.1 255.255.255.192

! Sub-interface for VLAN 20 - Finance/HR
interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.1.65 255.255.255.192

! Sub-interface for VLAN 30 - Customer Service/Reception
interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 192.168.1.129 255.255.255.192

! Unused physical interfaces shut down
interface GigabitEthernet0/1
 shutdown
interface GigabitEthernet0/2
 shutdown
```

### DHCP Configuration (on Router)

```
Router(config)# service dhcp

Router(config)# ip dhcp pool Admin-Pool
Router(dhcp-config)# network 192.168.1.0 255.255.255.192
Router(dhcp-config)# default-router 192.168.1.1

Router(config)# ip dhcp pool Finance-Pool
Router(dhcp-config)# network 192.168.1.64 255.255.255.192
Router(dhcp-config)# default-router 192.168.1.65

Router(config)# ip dhcp pool CustService-Pool
Router(dhcp-config)# network 192.168.1.128 255.255.255.192
Router(dhcp-config)# default-router 192.168.1.129
```

> 💡 Each department's hosts (wired and wireless) receive an IP address automatically from their respective DHCP pool, satisfying requirement (e).

## 🔀 Step 4 — Switch Configuration

```
Switch> enable
Switch# configure terminal

! Admin/IT access ports
interface range FastEthernet0/2-4
 switchport mode access
 switchport access vlan 10

! Finance/HR access ports
interface range FastEthernet0/5-7
 switchport mode access
 switchport access vlan 20

! Customer Service/Reception access ports
interface range FastEthernet0/8-10
 switchport mode access
 switchport access vlan 30

! Save configuration
do write memory
```

The switch automatically created VLANs 10, 20, and 30 the moment they were assigned to access ports.

## 📡 Step 5 — Wireless Access

An Access Point (AccessPoint-PT) was connected to an access port within each department's VLAN, giving wireless clients the same subnet and DHCP pool as the wired hosts in that department — satisfying requirement (d).

## ✅ Step 6 — Verification & Testing

- `show vlan brief` on the switch → confirmed correct port-to-VLAN mapping
- `show ip interface brief` on the router → confirmed all sub-interfaces up/up with correct IPs
- `ipconfig` on PCs → confirmed automatic IP assignment via DHCP
- Ping tests between Admin/IT, Finance/HR, and Customer Service devices → confirmed successful inter-VLAN communication (requirement f)
- Wireless PCs connected to their department's AP and received IPs automatically

## 🛠️ Tools & Technologies Used

- Cisco Packet Tracer
- Cisco IOS CLI
- VLAN segmentation
- Inter-VLAN Routing (Router-on-a-Stick)
- DHCP server configuration
- Wireless networking (Access Points)
- Subnetting (VLSM concepts)

## 📚 Key Skills Demonstrated

- Network requirement analysis and translation into a technical design
- IPv4 subnetting and address planning from a single base network
- VLAN design for network segmentation and security
- Router-on-a-Stick inter-VLAN routing configuration
- DHCP pool design and configuration for multiple subnets
- Wireless network integration into a wired LAN
- Configuration verification and troubleshooting

## 📸 Screenshots

*(Screenshot 2026-09-26 214248.png*

## 👤 Author
Author Name: Rausheen Hasan
Designed and implemented as part of an Enterprise Networking project focused on practical SOHO network design and Cisco device configuration.

---
⭐ Feel free to explore the `.pkt` file in this repo to see the full working topology in Cisco Packet Tracer.
