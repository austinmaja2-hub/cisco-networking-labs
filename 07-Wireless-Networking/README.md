# Lab 07 — Wireless Networking

## Overview

This lab demonstrates the configuration and testing of a basic wireless LAN (WLAN) using Cisco Packet Tracer.

The lab includes both wireless and wired devices connected through a WRT300N wireless router. The wireless network was secured using WPA2-Personal with AES encryption, and DHCP was used to automatically assign IP addresses to connected devices.

The lab also demonstrates communication between wireless clients and wired devices on the same LAN.

---

## Objectives

* Understand the fundamentals of wireless networking.
* Configure an SSID.
* Connect wireless clients to a wireless network.
* Configure WPA2-Personal wireless security.
* Understand wireless authentication and encryption.
* Use DHCP to automatically assign IP addresses.
* Test wireless-to-wireless communication.
* Test wireless-to-wired communication.
* Test wired-to-wireless communication.
* Understand the cybersecurity importance of wireless security.

---

## Topology

The topology consists of:

* 1 × WRT300N Wireless Router
* 2 × Laptop-PT wireless clients
* 1 × PC
* 1 × Server
* 1 × wireless network: `CYBER-LAB`

### Network Diagram

```text
                    ┌── Laptop0
                    │   192.168.0.100
                    │
                    │ Wi-Fi
                    │
                 WRT300N
                 192.168.0.1
                 /       \
                /         \
          Ethernet       Ethernet
             │               │
            PC             Server
                         192.168.0.102

                    └── Laptop1
                        192.168.0.101
                        Wi-Fi
```

---

## Network Configuration

### Wireless Network

| Setting            | Configuration    |
| ------------------ | ---------------- |
| SSID               | `CYBER-LAB`      |
| Security           | WPA2-Personal    |
| Encryption         | AES              |
| IP Network         | `192.168.0.0/24` |
| Default Gateway    | `192.168.0.1`    |
| Address Assignment | DHCP             |

### Client Addressing

| Device  | Connection | IPv4 Address    | Gateway       |
| ------- | ---------- | --------------- | ------------- |
| Laptop0 | Wireless   | `192.168.0.100` | `192.168.0.1` |
| Laptop1 | Wireless   | `192.168.0.101` | `192.168.0.1` |
| Server  | Ethernet   | `192.168.0.102` | `192.168.0.1` |
| PC      | Ethernet   | DHCP            | `192.168.0.1` |

---

## Key Terminology

### WLAN

**Wireless Local Area Network (WLAN)** is a local network that allows devices to communicate using wireless radio instead of physical Ethernet cables.

### SSID

**Service Set Identifier (SSID)** is the name of a wireless network that appears when wireless devices scan for available networks.

In this lab:

```text
CYBER-LAB
```

### Access Point

An **Access Point (AP)** provides wireless connectivity to client devices and allows them to communicate with the network.

The WRT300N provides wireless access point functionality in this lab.

### Wireless Client

A **wireless client** is a device that connects to a wireless network.

In this lab:

* Laptop0
* Laptop1

### WPA2

**Wi-Fi Protected Access 2 (WPA2)** is a wireless security protocol used to authenticate clients and protect wireless communications.

### AES

**Advanced Encryption Standard (AES)** is an encryption algorithm used by WPA2 to protect wireless traffic.

### DHCP

**Dynamic Host Configuration Protocol (DHCP)** automatically provides network configuration such as:

* IP address
* Subnet mask
* Default gateway
* DNS server

This prevented the clients from requiring manual IP configuration.

---

## Configuration Process

### 1. Configure the SSID

The wireless network was configured with the SSID:

```text
CYBER-LAB
```

### 2. Install Wireless Interfaces

The laptops initially did not have wireless interfaces installed.

A **WPC300N wireless adapter** was installed into each laptop before connecting to the WLAN.

### 3. Connect Wireless Clients

Laptop0 and Laptop1 were connected to:

```text
CYBER-LAB
```

The WRT300N automatically assigned IP addresses through DHCP.

### 4. Configure Wireless Security

The wireless network was secured using:

```text
WPA2-Personal
AES
```

This required wireless clients to authenticate using the configured pre-shared key before gaining access to the WLAN.

### 5. Verify DHCP

The wireless clients successfully received addresses from the WRT300N DHCP service.

Laptop0:

```text
192.168.0.100
```

Laptop1:

```text
192.168.0.101
```

The server received:

```text
192.168.0.102
```

---

## Connectivity Testing

### Laptop0 → Laptop1

```text
ping 192.168.0.101
```

Result:

```text
4 packets sent
4 packets received
0% packet loss
```

### Laptop0 → Server

```text
ping 192.168.0.102
```

Result:

```text
4 packets sent
4 packets received
0% packet loss
```

### PC → Laptop0

```text
ping 192.168.0.100
```

Result:

```text
4 packets sent
4 packets received
0% packet loss
```

These tests confirmed bidirectional communication between wireless and wired devices on the same LAN.

---

## Cybersecurity Relevance

Wireless networks are an important part of modern network security because wireless communication can be accessed without a physical Ethernet connection.

This lab demonstrated several security concepts:

* Wireless authentication using WPA2.
* Encryption using AES.
* The importance of securing wireless networks instead of leaving them open.
* Identification of wireless clients.
* DHCP-based network configuration.
* Testing network connectivity after applying security controls.

An unsecured wireless network could allow unauthorized devices to attempt to join the network. Wireless security mechanisms such as WPA2 help restrict access and protect communications.

---

## Skills Demonstrated

* Cisco Packet Tracer
* Wireless LAN configuration
* SSID configuration
* Wireless client configuration
* WPC300N wireless adapter installation
* WPA2-Personal
* AES encryption
* DHCP
* IPv4 addressing
* Network troubleshooting
* Ping/ICMP testing
* Wired and wireless network integration
* Basic wireless security

---

## Lab Outcome

The wireless network was successfully configured and secured.

Both wireless clients connected to the `CYBER-LAB` SSID using WPA2-Personal and received IP addresses through DHCP.

Connectivity tests confirmed successful communication:

```text
Wireless → Wireless    ✓
Wireless → Wired       ✓
Wired → Wireless       ✓
```

This lab provided a practical foundation for understanding wireless networking and its importance in cybersecurity.

---

## Files

* `wireless-networking.pkt` — Packet Tracer lab file
* `topology.png` — Network topology
* `README.md` — Lab documentation
