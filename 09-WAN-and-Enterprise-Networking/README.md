# Lab 09 — WAN & Enterprise Networking

## Overview

This lab demonstrates how an enterprise network can connect multiple sites through a WAN and provide connectivity to an upstream ISP and simulated Internet.

The lab uses two enterprise sites:

* **Johannesburg — Head Office**
* **Cape Town — Branch Office**

The sites communicate through a point-to-point WAN connection. Johannesburg also connects to an ISP, which provides access to a simulated Internet router.

The lab focuses on WAN addressing, static routing, default routes, next-hop routing, ISP connectivity, and end-to-end troubleshooting.

---

## Objectives

* Understand the difference between LAN and WAN networks
* Build a multi-site enterprise topology
* Configure point-to-point WAN addressing using `/30`
* Configure static routes between sites
* Configure default routes
* Simulate an ISP and Internet connection
* Configure return routes
* Test end-to-end connectivity
* Troubleshoot routing failures
* Understand enterprise routing from a cybersecurity perspective

---

## Topology

```text
                         ENTERPRISE NETWORK

        JOHANNESBURG                         CAPE TOWN
         HEAD OFFICE                        BRANCH OFFICE

       PC0       PC1                       PC2       PC3
        |         |                         |         |
        +---- Switch0 ----+          +---- Switch1 ----+
                          |          |
                         R1          R2
                    G0/0 |          | G0/1
                192.168.10.1      192.168.20.1
                          |          |
                    G0/1 |          | G0/0
                  10.0.0.1 ========= 10.0.0.2
                          WAN
                       /30 link

                          |
                    R1 VLAN1 SVI
                       10.0.1.1
                          |
                       ISP
                 G0/0 10.0.1.2
                 G0/1 172.16.0.1
                          |
                    INTERNET
                 G0/0 172.16.0.2
```

---

## Network Addressing

| Device   | Interface | IP Address       | Network         | Purpose                |
| -------- | --------- | ---------------- | --------------- | ---------------------- |
| R1       | G0/0      | 192.168.10.1/24  | 192.168.10.0/24 | Johannesburg LAN       |
| R1       | G0/1      | 10.0.0.1/30      | 10.0.0.0/30     | WAN to R2              |
| R1       | VLAN1     | 10.0.1.1/30      | 10.0.1.0/30     | ISP connection         |
| R2       | G0/0      | 10.0.0.2/30      | 10.0.0.0/30     | WAN to R1              |
| R2       | G0/1      | 192.168.20.1/24  | 192.168.20.0/24 | Cape Town LAN          |
| ISP      | G0/0      | 10.0.1.2/30      | 10.0.1.0/30     | Connection to R1       |
| ISP      | G0/1      | 172.16.0.1/30    | 172.16.0.0/30   | Connection to Internet |
| INTERNET | G0/0      | 172.16.0.2/30    | 172.16.0.0/30   | Simulated Internet     |
| PC0      | NIC       | 192.168.10.10/24 | 192.168.10.0/24 | Johannesburg host      |
| PC1      | NIC       | 192.168.10.20/24 | 192.168.10.0/24 | Johannesburg host      |
| PC2      | NIC       | 192.168.20.10/24 | 192.168.20.0/24 | Cape Town host         |
| PC3      | NIC       | 192.168.20.20/24 | 192.168.20.0/24 | Cape Town host         |

---

# Key Terminology

### LAN — Local Area Network

A network covering a relatively small area such as an office, home, or building.

### WAN — Wide Area Network

A network used to connect geographically separated networks or sites.

### Point-to-Point Link

A network connection between two devices. In this lab, R1 and R2 use a `/30` network for their WAN link.

### Static Route

A manually configured route telling a router how to reach a specific network.

### Default Route

A route used when no more specific route exists.

```text
0.0.0.0/0
```

### Next Hop

The next router to which a packet should be forwarded.

### ISP — Internet Service Provider

A provider that connects an organization's network to external networks and the Internet.

### SVI — Switch Virtual Interface

A logical Layer-3 interface associated with a VLAN. In this lab, VLAN1 on R1's HWIC-4ESW module was used as the Layer-3 interface toward the ISP.

---

# Configuration

## R1 — Johannesburg Router

### Johannesburg LAN

```text
interface gigabitEthernet 0/0
ip address 192.168.10.1 255.255.255.0
no shutdown
```

### WAN to R2

```text
interface gigabitEthernet 0/1
ip address 10.0.0.1 255.255.255.252
no shutdown
```

### ISP-facing SVI

Because the HWIC-4ESW provides Layer-2 switch ports, the physical FastEthernet port was configured as an access port and VLAN1 was used as the Layer-3 interface.

```text
interface fastEthernet 0/1/0
switchport mode access
no shutdown

interface vlan 1
ip address 10.0.1.1 255.255.255.252
no shutdown
```

### Route to Cape Town

```text
ip route 192.168.20.0 255.255.255.0 10.0.0.2
```

### Default Route to ISP

```text
ip route 0.0.0.0 0.0.0.0 10.0.1.2
```

---

# R2 — Cape Town Router

### WAN to R1

```text
interface gigabitEthernet 0/0
ip address 10.0.0.2 255.255.255.252
no shutdown
```

### Cape Town LAN

```text
interface gigabitEthernet 0/1
ip address 192.168.20.1 255.255.255.0
no shutdown
```

### Route to Johannesburg

```text
ip route 192.168.10.0 255.255.255.0 10.0.0.1
```

### Default Route to R1

```text
ip route 0.0.0.0 0.0.0.0 10.0.0.1
```

---

# ISP Router

### Connection to R1

```text
interface gigabitEthernet 0/0
ip address 10.0.1.2 255.255.255.252
no shutdown
```

### Connection to Internet

```text
interface gigabitEthernet 0/1
ip address 172.16.0.1 255.255.255.252
no shutdown
```

### Return Route to Johannesburg

```text
ip route 192.168.10.0 255.255.255.0 10.0.1.1
```

### Return Route to Cape Town

```text
ip route 192.168.20.0 255.255.255.0 10.0.1.1
```

### Return Route to R1-R2 WAN

```text
ip route 10.0.0.0 255.255.255.252 10.0.1.1
```

### Default Route to Internet

```text
ip route 0.0.0.0 0.0.0.0 172.16.0.2
```

---

# INTERNET Router

### Connection to ISP

```text
interface gigabitEthernet 0/0
ip address 172.16.0.2 255.255.255.252
no shutdown
```

### Return Route to Johannesburg

```text
ip route 192.168.10.0 255.255.255.0 172.16.0.1
```

### Return Route to Cape Town

```text
ip route 192.168.20.0 255.255.255.0 172.16.0.1
```

### Return Route to R1-R2 WAN

```text
ip route 10.0.0.0 255.255.255.252 172.16.0.1
```

---

# Verification

## R1

```text
show ip interface brief
```

Important interfaces:

```text
G0/0    192.168.10.1    up/up
G0/1    10.0.0.1        up/up
Vlan1   10.0.1.1        up/up
```

Routing table confirmed:

```text
C    10.0.0.0/30
C    10.0.1.0/30
C    192.168.10.0/24
S    192.168.20.0/24 via 10.0.0.2
S*   0.0.0.0/0 via 10.0.1.2
```

---

## R2

```text
show ip interface brief
show ip route
```

Confirmed:

```text
G0/0    10.0.0.2        up/up
G0/1    192.168.20.1    up/up
```

Routing table confirmed:

```text
C    10.0.0.0/30
S    192.168.10.0/24 via 10.0.0.1
C    192.168.20.0/24
S*   0.0.0.0/0 via 10.0.0.1
```

---

# Connectivity Tests

### Johannesburg LAN

PC0 → PC1

```text
ping 192.168.10.20
```

Result: **Successful**

### Cape Town LAN

PC2 → PC3

```text
ping 192.168.20.20
```

Result: **Successful**

### R1 → R2 WAN

```text
ping 10.0.0.2
```

Result: **Successful**

### R1 → ISP

```text
ping 10.0.1.2
```

Result: **Successful**

### ISP → Internet

```text
ping 172.16.0.2
```

Result: **Successful**

### Johannesburg → Internet

PC0:

```text
ping 172.16.0.2
```

Result: **Successful**

### Cape Town → Internet

PC2:

```text
ping 172.16.0.2
```

Result: **Successful**

---

# Troubleshooting Performed

During the lab, an initial Cape Town-to-Internet test failed.

The failure demonstrated the importance of **return routing**.

The forward path was working:

```text
PC2 → R2 → R1 → ISP → INTERNET
```

However, the return traffic did not initially have a complete route back to:

```text
192.168.20.0/24
```

Routes were added to the ISP and Internet router:

```text
ip route 192.168.20.0 255.255.255.0 10.0.1.1
```

and:

```text
ip route 192.168.20.0 255.255.255.0 172.16.0.1
```

After the return routes were added, the PC2 → Internet test succeeded.

This demonstrated that **successful routing requires a valid path in both directions**.

---

# Cybersecurity Relevance

Understanding enterprise networking is essential for cybersecurity because security professionals need to understand how systems communicate before they can effectively assess or defend them.

This lab demonstrates knowledge of:

* Network segmentation
* Internal vs external networks
* WAN communication
* Router interfaces
* Routing tables
* Static routing
* Default gateways
* Next-hop routing
* ISP connectivity
* Traffic paths
* Return traffic
* Network troubleshooting

From a security perspective, routing information can help an analyst understand:

```text
What networks exist?
Where are the internal networks?
What is the WAN path?
What is the upstream gateway?
How does traffic leave the organization?
How can traffic reach another network?
```

These concepts are directly relevant to later cybersecurity activities such as network reconnaissance, Nmap scanning, traffic analysis, firewall analysis, penetration testing, and incident response.

---

# Skills Demonstrated

* Cisco Packet Tracer
* Cisco IOS CLI
* IPv4 addressing
* `/30` point-to-point networks
* LAN/WAN design
* Static routing
* Default routing
* Next-hop routing
* Enterprise network topology
* ISP simulation
* Routing-table analysis
* Network troubleshooting
* End-to-end connectivity testing

---

# Evidence

The repository should contain:

```text
09-wan-enterprise-networking/
├── lab09-wan-enterprise-networking.pkt
├── topology.png
└── README.md
```

Recommended screenshots:

1. Complete topology
2. R1 `show ip interface brief`
3. R1 `show ip route`
4. R2 `show ip interface brief`
5. R2 `show ip route`
6. Successful PC0 → Internet ping
7. Successful PC2 → Internet ping

---

# Final Status

**Lab 09 — WAN & Enterprise Networking: COMPLETE ✅**

The enterprise network successfully demonstrated communication between Johannesburg and Cape Town, WAN connectivity between routers, ISP connectivity, default routing, and end-to-end access to a simulated Internet.
