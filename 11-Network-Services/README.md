# Lab 11 — Network Services

## 📌 Overview

This lab focused on configuring, testing, and integrating common network services in a Cisco Packet Tracer environment.

The objective was to move beyond basic IP connectivity and understand how network services such as **DNS, HTTP, FTP, SSH, Telnet, NTP, and SNMP** operate within a network.

The lab also focused on the cybersecurity implications of exposing and securing network services.

---

## 🎯 Objectives

By completing this lab, I aimed to:

* Configure a basic network-services environment
* Understand the purpose of DNS
* Configure and test an HTTP web server
* Configure and test FTP
* Configure secure remote administration using SSH
* Compare SSH with Telnet
* Configure NTP for time synchronization
* Configure an SNMP agent
* Integrate DNS with application-layer services
* Troubleshoot network-service issues
* Verify service availability using practical tests
* Understand how network services create potential cybersecurity attack surfaces

---

# 🖥️ Topology

```text
                         192.168.100.0/24

                              R1
                       G0/0 192.168.100.1
                               |
                               |
                            Switch
                               |
          ┌────────────────────┼────────────────────┐
          │                    │                    │
         PC0              DNS Server            Server1
   192.168.100.20       192.168.100.2        192.168.100.10
                              │                    │
                           DNS ON          HTTP / FTP / NTP ON
                                             
                         SSH access to R1
                         SNMP agent on R1
```

---

# 🌐 IP Addressing

| Device     | Interface | IP Address          | Purpose          |
| ---------- | --------- | ------------------- | ---------------- |
| R1         | G0/0      | `192.168.100.1/24`  | Network gateway  |
| DNS Server | NIC       | `192.168.100.2/24`  | DNS              |
| Server1    | NIC       | `192.168.100.10/24` | HTTP / FTP / NTP |
| PC0        | NIC       | `192.168.100.20/24` | Client           |

Network:

```text
192.168.100.0/24
```

---

# 📚 Terminology

### DNS — Domain Name System

DNS translates human-readable hostnames into IP addresses.

Example:

```text
web.cyberlab.local
        ↓
192.168.100.10
```

### HTTP — Hypertext Transfer Protocol

HTTP is an application-layer protocol used to transfer web content between clients and web servers.

### FTP — File Transfer Protocol

FTP is used to transfer files between a client and server.

### SSH — Secure Shell

SSH provides encrypted remote administration of network devices and systems.

Default TCP port:

```text
22
```

### Telnet

Telnet provides remote terminal access but does not provide the same encryption protection as SSH.

Default TCP port:

```text
23
```

### NTP — Network Time Protocol

NTP synchronizes the clocks of network devices.

Accurate time is important for:

* Log correlation
* Incident investigation
* Monitoring
* Troubleshooting
* Event timelines

### SNMP — Simple Network Management Protocol

SNMP is used to monitor and manage network devices.

Important concepts include:

* **SNMP Manager** — monitoring system
* **SNMP Agent** — software/service running on the managed device
* **MIB** — Management Information Base
* **OID** — Object Identifier
* **Community String** — shared value used by traditional SNMP versions such as SNMPv1/v2c

---

# 1️⃣ DNS Configuration

DNS Server:

```text
192.168.100.2
```

DNS service was enabled.

An A record was created:

```text
web.cyberlab.local → 192.168.100.10
```

### Verification

From PC0:

```text
nslookup web.cyberlab.local
```

The hostname successfully resolved to:

```text
192.168.100.10
```

### Result

✅ DNS resolution successful.

---

# 2️⃣ HTTP Configuration

Server1:

```text
192.168.100.10
```

HTTP service was enabled.

From PC0, the following address successfully loaded the Packet Tracer web page:

```text
http://192.168.100.10
```

DNS integration was then tested using:

```text
http://web.cyberlab.local
```

The webpage successfully loaded.

### Result

✅ HTTP service working.

✅ DNS + HTTP integration working.

---

# 3️⃣ FTP Configuration

FTP was enabled on Server1.

A user account was configured:

```text
Username: labuser
Password: CyberLab123
```

Read and Write permissions were enabled.

From PC0:

```text
ftp 192.168.100.10
```

The FTP connection was successful.

File upload was tested:

```text
put ftp-test.txt
```

Result:

```text
Transfer complete
```

File download was also tested:

```text
get ftp-test.txt
```

Result:

```text
Transfer complete
```

FTP was subsequently tested using the DNS hostname:

```text
ftp web.cyberlab.local
```

The FTP prompt was successfully reached.

### Result

✅ FTP connection successful.

✅ Authentication successful.

✅ File upload successful.

✅ File download successful.

✅ DNS + FTP integration successful.

---

# 4️⃣ SSH Configuration

R1 was configured for secure remote administration.

### Hostname

```text
hostname R1
```

### Domain name

```text
ip domain-name cyberlab.local
```

### Local user

```text
username admin privilege 15 secret CyberLab123
```

### RSA keys

RSA keys were generated to support SSH.

### SSH version

```text
ip ssh version 2
```

### VTY configuration

```text
line vty 0 4
login local
transport input ssh
```

### Verification

```text
show ip ssh
```

SSH version 2 was confirmed.

The VTY configuration was also verified:

```text
line vty 0 4
 login local
 transport input ssh
```

### Client test

From PC0:

```text
ssh -l admin 192.168.100.1
```

The connection successfully entered R1.

### Result

✅ SSH configured.

✅ SSH version 2 verified.

✅ Remote login successful.

---

# 5️⃣ Telnet Comparison

Telnet was temporarily enabled to compare it with SSH.

```text
line vty 0 4
transport input telnet
```

PC0 successfully connected:

```text
telnet 192.168.100.1
```

After testing, Telnet was disabled and SSH-only access was restored:

```text
line vty 0 4
transport input ssh
```

### Security comparison

| Feature                            | SSH | Telnet |
| ---------------------------------- | --- | ------ |
| Remote administration              | ✅   | ✅      |
| Encryption                         | ✅   | ❌      |
| Default TCP port                   | 22  | 23     |
| Suitable for secure administration | Yes | No     |

The Telnet test was performed only as a controlled comparison inside the lab.

### Result

✅ Telnet functionality demonstrated.

✅ SSH-only configuration restored.

---

# 6️⃣ NTP Configuration

Server1 was configured as the NTP server.

R1 was configured with:

```text
ntp server 192.168.100.10
```

Initially, R1 reported:

```text
Clock is unsynchronized
```

Connectivity was tested:

```text
ping 192.168.100.10
```

The server was reachable.

The NTP configuration was investigated and the NTP authentication service on Server1 was found to be enabled.

Authentication was disabled in the Packet Tracer NTP configuration.

After this change, R1 reported:

```text
Clock is synchronized
```

### Result

✅ NTP server reachable.

✅ NTP synchronization successful.

### Troubleshooting lesson

A service may be reachable at the network layer while still failing at the application/service layer.

This demonstrated the importance of checking:

1. Connectivity
2. Service configuration
3. Authentication/settings
4. Verification commands

---

# 7️⃣ SNMP Configuration

R1 was configured as an SNMP agent.

```text
snmp-server community CyberLab RO
```

Where:

* `CyberLab` = SNMP community string
* `RO` = Read Only

The configuration was verified using:

```text
show running-config | include snmp
```

The following configuration was confirmed:

```text
snmp-server community CyberLab RO
```

### Packet Tracer limitation

The Packet Tracer Server device used in this lab did not provide an SNMP Manager service.

Therefore, an actual SNMP Manager → Agent polling test could not be performed.

The lab therefore verified the **SNMP Agent configuration**, but did not claim successful SNMP polling.

### Cybersecurity relevance

SNMP can expose useful information about network devices.

Poorly secured SNMP configurations can potentially assist reconnaissance by exposing information such as:

* Interfaces
* IP addressing
* Device information
* Network statistics

Traditional SNMP community strings should therefore be protected and appropriately configured.

---

# 8️⃣ Network Connectivity Verification

Connectivity between PC0 and both servers was tested.

### DNS Server

```text
ping 192.168.100.2
```

Result:

```text
4/4 replies
```

### Server1

```text
ping 192.168.100.10
```

Result:

```text
4/4 replies
```

### Result

✅ DNS Server reachable.

✅ Application Server reachable.

---

# 🔗 Service Integration

One of the main goals of this lab was to demonstrate that services can work together.

## DNS + HTTP

```text
PC0
 ↓
web.cyberlab.local
 ↓
DNS Server
 ↓
192.168.100.10
 ↓
HTTP Server
 ↓
Web Page
```

Verified successfully.

## DNS + FTP

```text
PC0
 ↓
web.cyberlab.local
 ↓
DNS Server
 ↓
192.168.100.10
 ↓
FTP Server
```

Verified successfully.

This demonstrates that DNS does not provide the application service itself. Instead, DNS helps the client discover **where the service is located**.

---

# 🛠️ Troubleshooting

## Issue 1 — SNMP configuration initially did not appear

The initial SNMP command did not appear in the running configuration.

The command was entered again:

```text
snmp-server community CyberLab RO
```

The configuration was then verified with:

```text
show running-config | include snmp
```

The SNMP configuration appeared successfully.

### Lesson

Always verify configuration rather than assuming that a command was accepted.

---

## Issue 2 — NTP initially remained unsynchronized

R1 initially reported:

```text
Clock is unsynchronized
```

Network connectivity to the NTP server was confirmed with a successful ping.

The NTP server configuration was investigated and authentication was found to be enabled.

After disabling NTP authentication, R1 successfully synchronized.

### Lesson

Network connectivity does not automatically mean that a service is functioning correctly.

---

## Issue 3 — FTP directory listing

The FTP command:

```text
dir
```

returned:

```text
550 Requested action not taken. Permission denied
```

Despite this, file upload and download both succeeded.

This was treated as a Packet Tracer FTP behaviour/limitation rather than evidence that FTP itself was completely unavailable.

### Lesson

A failed individual function does not necessarily mean the entire service is down.

Testing multiple functions provides better evidence.

---

## Issue 4 — SNMP Manager unavailable

The Server device did not provide an SNMP Manager service.

Therefore, the SNMP Agent configuration was verified, but polling was not performed.

### Lesson

A simulation environment may not implement every component of a real-world technology.

A good lab report should distinguish between:

* What was configured
* What was tested
* What could not be tested

---

# 🔐 Cybersecurity Relevance

Network services are important from both defensive and offensive-security perspectives.

Every exposed service can potentially represent an **attack surface**.

For example:

```text
DNS  → Information discovery
HTTP → Web attack surface
FTP  → File transfer/authentication
SSH  → Remote administration
Telnet → Insecure remote administration
NTP  → Time synchronization
SNMP → Device/network information exposure
```

This becomes particularly important during network reconnaissance.

Tools such as Nmap can later be used to identify exposed services and their ports.

For example:

```text
Port 21  → FTP
Port 22  → SSH
Port 23  → Telnet
Port 53  → DNS
Port 80  → HTTP
Port 123 → NTP
Port 161 → SNMP
```

Understanding what these services do makes the results of future reconnaissance much easier to interpret.

---

# 🧪 Verification Summary

| Test                          | Result                            |
| ----------------------------- | --------------------------------- |
| PC0 → DNS Server ping         | ✅ 4/4                             |
| PC0 → Server1 ping            | ✅ 4/4                             |
| DNS A record                  | ✅                                 |
| `nslookup web.cyberlab.local` | ✅                                 |
| HTTP via IP                   | ✅                                 |
| HTTP via hostname             | ✅                                 |
| FTP connection                | ✅                                 |
| FTP authentication            | ✅                                 |
| FTP upload                    | ✅                                 |
| FTP download                  | ✅                                 |
| SSH login                     | ✅                                 |
| SSH version 2                 | ✅                                 |
| Telnet comparison             | ✅                                 |
| SSH-only restored             | ✅                                 |
| NTP synchronization           | ✅                                 |
| SNMP Agent configuration      | ✅                                 |
| SNMP Manager polling          | ⚠️ Not available in Packet Tracer |

---

# 🧠 Key Lessons

This lab taught me that a functional network is more than IP connectivity.

I learned how different services operate together to provide:

* Name resolution
* Web access
* File transfer
* Secure remote administration
* Time synchronization
* Network monitoring

I also learned the importance of **verification and troubleshooting**.

Instead of assuming a service was working after configuration, I tested the actual functionality.

The NTP and FTP issues demonstrated that successful connectivity does not necessarily mean successful service operation.

---

# 🚀 Cybersecurity Progression

This lab provides an important foundation for future ethical-hacking labs.

The next stage will involve identifying these services from an attacker's perspective.

For example:

```text
Network Services
       ↓
Service Discovery
       ↓
Nmap
       ↓
Port Enumeration
       ↓
Service Enumeration
       ↓
Version Detection
       ↓
Vulnerability Assessment
       ↓
Controlled Exploitation
```

All future testing will be performed against authorized laboratory systems.

---

# 📁 Evidence

Recommended evidence to retain:

* [x] Packet Tracer topology
* [x] DNS configuration
* [x] DNS resolution
* [x] HTTP webpage
* [x] FTP connection
* [x] FTP upload/download
* [x] SSH login
* [x] SSH verification
* [x] Telnet comparison
* [x] NTP synchronization
* [x] SNMP configuration
* [x] Connectivity tests

---

# 📌 Status

**Lab 11 — Network Services: COMPLETED ✅**

### Services demonstrated

```text
DNS
HTTP
FTP
SSH
Telnet
NTP
SNMP
```

### Next Lab

**Lab 12 — IPv6**

The next lab will introduce IPv6 addressing, configuration, connectivity, and verification, building on the IPv4 networking knowledge developed throughout the previous labs.
