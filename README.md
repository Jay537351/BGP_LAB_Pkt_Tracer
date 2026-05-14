[README.md](https://github.com/user-attachments/files/27753958/README.md)
# 🌐 Basic BGP Lab — Multi-AS Network Configuration

> A hands-on Cisco Packet Tracer lab implementing Border Gateway Protocol (BGP) across 4 Autonomous Systems with 12 routers, including OSPF as the underlying IGP.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Network Topology](#network-topology)
- [Autonomous Systems](#autonomous-systems)
- [IP Addressing Scheme](#ip-addressing-scheme)
- [Prerequisites](#prerequisites)
- [Lab Objectives](#lab-objectives)
- [Configuration Guide](#configuration-guide)
  - [Interface Configuration](#interface-configuration)
  - [Loopback Interfaces](#loopback-interfaces)
  - [OSPF Configuration](#ospf-configuration)
  - [BGP Configuration](#bgp-configuration)
- [BGP Concepts Explained](#bgp-concepts-explained)
- [OSPF Concepts Explained](#ospf-concepts-explained)
- [Verification Commands](#verification-commands)
- [Troubleshooting Guide](#troubleshooting-guide)
- [Future Improvements](#future-improvements)
- [Key Lessons Learned](#key-lessons-learned)

---

## Overview

This lab demonstrates the configuration of **Border Gateway Protocol (BGP)** across multiple Autonomous Systems (AS) using Cisco 2811 routers in Packet Tracer. It covers eBGP (between different AS) and iBGP (within the same AS), combined with OSPF as the Interior Gateway Protocol (IGP) to ensure reachability between BGP next-hops.

**Original lab design by:** Patel Jay (pjay20370@gmail.com)  
**Tool used:** Cisco Packet Tracer  
**Router model:** Cisco 2811  

---

## Network Topology

```
AS64900                AS64905              AS64910                AS64915
┌─────────────┐    ┌──────────┐    ┌──────────────────┐    ┌──────────────┐
│   R2        │    │          │    │   R6      R9     │    │  R10         │
│  /  \       │    │  R4──R5  │    │    \    /   \    │    │    \         │
│ R1   R3     │    │          │    │     R8      R10  │    │     R12      │
│             │    │          │    │    /    \        │    │    /         │
└─────────────┘    └──────────┘    │   R7      R11   │    │  R11         │
                                   └──────────────────┘    └──────────────┘
```

---

## Autonomous Systems

| AS Number | Routers | Role |
|-----------|---------|------|
| **AS64900** | R1, R2, R3 | Origin AS — leftmost network |
| **AS64905** | R4, R5 | Transit AS — middle left |
| **AS64910** | R6, R7, R8, R9 | Transit AS — middle right |
| **AS64915** | R10, R11, R12 | Destination AS — rightmost network |

---

## IP Addressing Scheme

### Point-to-Point Links (192.168.YY.x/24)

| Link | Subnet | Router A IP | Router B IP |
|------|--------|------------|------------|
| R1 — R2 | 192.168.12.0/24 | R1: .1 | R2: .2 |
| R1 — R3 | 192.168.13.0/24 | R1: .1 | R3: .2 |
| R2 — R4 | 192.168.24.0/24 | R2: .1 | R4: .2 |
| R3 — R4 | 192.168.34.0/24 | R3: .1 | R4: .2 |
| R4 — R5 | 192.168.45.0/24 | R4: .1 | R5: .2 |
| R5 — R6 | 192.168.56.0/24 | R5: .1 | R6: .2 |
| R5 — R7 | 192.168.57.0/24 | R5: .1 | R7: .2 |
| R6 — R8 | 192.168.68.0/24 | R6: .1 | R8: .2 |
| R7 — R8 | 192.168.78.0/24 | R7: .1 | R8: .2 |
| R8 — R9 | 192.168.89.0/24 | R8: .1 | R9: .2 |
| R9 — R10 | 192.168.109.0/24 | R9: .1 | R10: .2 |
| R9 — R11 | 192.168.119.0/24 | R9: .1 | R11: .2 |
| R10 — R12 | 192.168.120.0/24 | R10: .1 | R12: .2 |
| R11 — R12 | 192.168.121.0/24 | R11: .1 | R12: .2 |

### Loopback Addresses (Y.Y.Y.Y/24)

| Router | Loopback IP |
|--------|------------|
| R1 | 1.1.1.1/24 |
| R2 | 2.2.2.2/24 |
| R3 | 3.3.3.3/24 |
| R4 | 4.4.4.4/24 |
| R5 | 5.5.5.5/24 |
| R6 | 6.6.6.6/24 |
| R7 | 7.7.7.7/24 |
| R8 | 8.8.8.8/24 |
| R9 | 9.9.9.9/24 |
| R10 | 10.10.10.10/24 |
| R11 | 11.11.11.11/24 |
| R12 | 12.12.12.12/24 |

---

## Prerequisites

- Cisco Packet Tracer installed (version 7.0 or higher recommended)
- Basic understanding of IP addressing and subnetting
- Familiarity with Cisco IOS CLI
- Understanding of routing concepts

### Router Setup in Packet Tracer

Each router requires additional modules before configuration:

| Module | Purpose | Routers Needing It |
|--------|---------|-------------------|
| **WIC-2T** | Adds 2 serial ports (Se0/0/0, Se0/0/1) | R2, R3, R4, R5, R6, R7, R8, R9, R10 |
| **NM-1E** | Adds Ethernet port (Eth1/0) | R2, R3, R4, R8, R9 |

**To add a module in Packet Tracer:**
1. Click the router
2. Go to the **Physical** tab
3. **Power OFF** the router (click the green power button)
4. Drag the module into an empty slot
5. Power the router back **ON**

> ⚠️ **Important:** Always power off before adding modules or the router will not recognize them.

---

## Lab Objectives

1. Configure and cable Serial/Ethernet interfaces as shown in the topology
2. Assign IP addresses using the 192.168.YY.x/24 scheme
3. Configure loopback interfaces using Y.Y.Y.Y/24 scheme
4. Configure OSPF within each AS for internal reachability
5. Configure eBGP between border routers of different AS
6. Configure iBGP between routers within the same AS
7. Verify end-to-end connectivity from R1 to R12

---

## Configuration Guide

### Interface Configuration

#### Router 1
```cisco
en
conf t
hostname R1
interface fa0/0
 ip address 192.168.12.1 255.255.255.0
 no shutdown
interface fa0/1
 ip address 192.168.13.1 255.255.255.0
 no shutdown
```

#### Router 2
```cisco
en
conf t
hostname R2
interface eth1/0
 ip address 192.168.12.2 255.255.255.0
 no shutdown
interface se0/0/1
 ip address 192.168.24.1 255.255.255.0
 no shutdown
```

#### Router 3
```cisco
en
conf t
hostname R3
interface eth1/0
 ip address 192.168.13.2 255.255.255.0
 no shutdown
interface se0/0/0
 clock rate 64000
 ip address 192.168.34.1 255.255.255.0
 no shutdown
```

#### Router 4
```cisco
en
conf t
hostname R4
interface se0/0/0
 clock rate 64000
 ip address 192.168.24.2 255.255.255.0
 no shutdown
interface se0/0/1
 ip address 192.168.34.2 255.255.255.0
 no shutdown
interface eth1/0
 ip address 192.168.45.1 255.255.255.0
 no shutdown
```

#### Router 5
```cisco
en
conf t
hostname R5
interface eth1/0
 ip address 192.168.45.2 255.255.255.0
 no shutdown
interface se0/0/0
 clock rate 64000
 ip address 192.168.56.1 255.255.255.0
 no shutdown
interface se0/0/1
 clock rate 64000
 ip address 192.168.57.1 255.255.255.0
 no shutdown
```

#### Router 6
```cisco
en
conf t
hostname R6
interface se0/0/0
 ip address 192.168.56.2 255.255.255.0
 no shutdown
interface se0/0/1
 ip address 192.168.68.1 255.255.255.0
 no shutdown
```

#### Router 7
```cisco
en
conf t
hostname R7
interface se0/0/0
 ip address 192.168.57.2 255.255.255.0
 no shutdown
interface se0/0/1
 clock rate 64000
 ip address 192.168.78.1 255.255.255.0
 no shutdown
```

#### Router 8
```cisco
en
conf t
hostname R8
interface se0/0/0
 clock rate 64000
 ip address 192.168.68.2 255.255.255.0
 no shutdown
interface se0/0/1
 ip address 192.168.78.2 255.255.255.0
 no shutdown
interface eth1/0
 ip address 192.168.89.1 255.255.255.0
 no shutdown
```

#### Router 9
```cisco
en
conf t
hostname R9
interface eth1/0
 ip address 192.168.89.2 255.255.255.0
 no shutdown
interface se0/0/0
 clock rate 64000
 ip address 192.168.109.1 255.255.255.0
 no shutdown
interface fa0/0
 ip address 192.168.119.1 255.255.255.0
 no shutdown
```

#### Router 10
```cisco
en
conf t
hostname R10
interface se0/0/1
 ip address 192.168.109.2 255.255.255.0
 no shutdown
interface eth1/0
 ip address 192.168.120.1 255.255.255.0
 no shutdown
```

#### Router 11
```cisco
en
conf t
hostname R11
interface fa0/1
 ip address 192.168.119.2 255.255.255.0
 no shutdown
interface fa0/0
 ip address 192.168.121.1 255.255.255.0
 no shutdown
```

#### Router 12
```cisco
en
conf t
hostname R12
interface fa0/1
 ip address 192.168.120.2 255.255.255.0
 no shutdown
interface fa0/0
 ip address 192.168.121.2 255.255.255.0
 no shutdown
```

---

### Loopback Interfaces

Configure on every router (replace Y with router number):

```cisco
interface loopback 0
 ip address Y.Y.Y.Y 255.255.255.0
```

**Example for each router:**
```cisco
! R1
interface loopback 0
 ip address 1.1.1.1 255.255.255.0

! R2
interface loopback 0
 ip address 2.2.2.2 255.255.255.0

! Continue pattern through R12
! R12
interface loopback 0
 ip address 12.12.12.12 255.255.255.0
```

> 💡 **Note:** Loopback interfaces are always up and never need `no shutdown`. They serve as stable router identities for BGP.

---

### OSPF Configuration

OSPF provides the underlying path knowledge BGP needs to forward packets.

> ⚠️ **Critical:** OSPF runs **within each AS only**. Do not configure OSPF across AS boundaries.

#### AS64900 — R1, R2, R3

```cisco
! R1
router ospf 1
 network 192.168.12.0 0.0.0.255 area 0
 network 192.168.13.0 0.0.0.255 area 0
 network 1.1.1.0 0.0.0.255 area 0

! R2
router ospf 1
 network 192.168.12.0 0.0.0.255 area 0
 network 192.168.24.0 0.0.0.255 area 0
 network 2.2.2.0 0.0.0.255 area 0

! R3
router ospf 1
 network 192.168.13.0 0.0.0.255 area 0
 network 192.168.34.0 0.0.0.255 area 0
 network 3.3.3.0 0.0.0.255 area 0
```

#### AS64905 — R4, R5

```cisco
! R4
router ospf 1
 network 192.168.24.0 0.0.0.255 area 0
 network 192.168.34.0 0.0.0.255 area 0
 network 192.168.45.0 0.0.0.255 area 0
 network 4.4.4.0 0.0.0.255 area 0

! R5
router ospf 1
 network 192.168.45.0 0.0.0.255 area 0
 network 192.168.56.0 0.0.0.255 area 0
 network 192.168.57.0 0.0.0.255 area 0
 network 5.5.5.0 0.0.0.255 area 0
```

#### AS64910 — R6, R7, R8, R9

```cisco
! R6
router ospf 1
 network 192.168.56.0 0.0.0.255 area 0
 network 192.168.68.0 0.0.0.255 area 0
 network 6.6.6.0 0.0.0.255 area 0

! R7
router ospf 1
 network 192.168.57.0 0.0.0.255 area 0
 network 192.168.78.0 0.0.0.255 area 0
 network 7.7.7.0 0.0.0.255 area 0

! R8
router ospf 1
 network 192.168.68.0 0.0.0.255 area 0
 network 192.168.78.0 0.0.0.255 area 0
 network 192.168.89.0 0.0.0.255 area 0
 network 8.8.8.0 0.0.0.255 area 0

! R9
router ospf 1
 network 192.168.89.0 0.0.0.255 area 0
 network 192.168.109.0 0.0.0.255 area 0
 network 192.168.119.0 0.0.0.255 area 0
 network 9.9.9.0 0.0.0.255 area 0
```

#### AS64915 — R10, R11, R12

```cisco
! R10
router ospf 1
 network 192.168.109.0 0.0.0.255 area 0
 network 192.168.120.0 0.0.0.255 area 0
 network 10.10.10.0 0.0.0.255 area 0

! R11
router ospf 1
 network 192.168.119.0 0.0.0.255 area 0
 network 192.168.121.0 0.0.0.255 area 0
 network 11.11.11.0 0.0.0.255 area 0

! R12
router ospf 1
 network 192.168.120.0 0.0.0.255 area 0
 network 192.168.121.0 0.0.0.255 area 0
 network 12.12.12.0 0.0.0.255 area 0
```

---

### BGP Configuration

#### BGP Neighbor Relationships

| Router | iBGP Neighbors | eBGP Neighbors |
|--------|---------------|----------------|
| R1 | R2, R3 | — |
| R2 | R1 | R4 |
| R3 | R1 | R4 |
| R4 | R5 | R2, R3 |
| R5 | R4 | R6, R7 |
| R6 | R8 | R5 |
| R7 | R8 | R5 |
| R8 | R6, R7, R9 | — |
| R9 | R8 | R10, R11 |
| R10 | R12 | R9 |
| R11 | R12 | R9 |
| R12 | R10, R11 | — |

#### Full BGP Configuration

```cisco
! R1
router bgp 64900
 bgp router-id 1.1.1.1
 neighbor 192.168.12.2 remote-as 64900
 neighbor 192.168.13.2 remote-as 64900
 network 192.168.12.0 mask 255.255.255.0
 network 192.168.13.0 mask 255.255.255.0
 network 1.1.1.0 mask 255.255.255.0

! R2
router bgp 64900
 bgp router-id 2.2.2.2
 neighbor 192.168.12.1 remote-as 64900
 neighbor 192.168.24.2 remote-as 64905
 network 192.168.12.0 mask 255.255.255.0
 network 192.168.24.0 mask 255.255.255.0
 network 2.2.2.0 mask 255.255.255.0

! R3
router bgp 64900
 bgp router-id 3.3.3.3
 neighbor 192.168.13.1 remote-as 64900
 neighbor 192.168.34.2 remote-as 64905
 network 192.168.13.0 mask 255.255.255.0
 network 192.168.34.0 mask 255.255.255.0
 network 3.3.3.0 mask 255.255.255.0

! R4
router bgp 64905
 bgp router-id 4.4.4.4
 neighbor 192.168.24.1 remote-as 64900
 neighbor 192.168.34.1 remote-as 64900
 neighbor 192.168.45.2 remote-as 64905
 network 192.168.24.0 mask 255.255.255.0
 network 192.168.34.0 mask 255.255.255.0
 network 192.168.45.0 mask 255.255.255.0
 network 4.4.4.0 mask 255.255.255.0

! R5
router bgp 64905
 bgp router-id 5.5.5.5
 neighbor 192.168.45.1 remote-as 64905
 neighbor 192.168.56.2 remote-as 64910
 neighbor 192.168.57.2 remote-as 64910
 network 192.168.45.0 mask 255.255.255.0
 network 192.168.56.0 mask 255.255.255.0
 network 192.168.57.0 mask 255.255.255.0
 network 5.5.5.0 mask 255.255.255.0

! R6
router bgp 64910
 bgp router-id 6.6.6.6
 neighbor 192.168.56.1 remote-as 64905
 neighbor 192.168.68.2 remote-as 64910
 network 192.168.56.0 mask 255.255.255.0
 network 192.168.68.0 mask 255.255.255.0
 network 6.6.6.0 mask 255.255.255.0

! R7
router bgp 64910
 bgp router-id 7.7.7.7
 neighbor 192.168.57.1 remote-as 64905
 neighbor 192.168.78.2 remote-as 64910
 network 192.168.57.0 mask 255.255.255.0
 network 192.168.78.0 mask 255.255.255.0
 network 7.7.7.0 mask 255.255.255.0

! R8
router bgp 64910
 bgp router-id 8.8.8.8
 neighbor 192.168.68.1 remote-as 64910
 neighbor 192.168.78.1 remote-as 64910
 neighbor 192.168.89.2 remote-as 64910
 network 192.168.68.0 mask 255.255.255.0
 network 192.168.78.0 mask 255.255.255.0
 network 192.168.89.0 mask 255.255.255.0
 network 8.8.8.0 mask 255.255.255.0

! R9
router bgp 64910
 bgp router-id 9.9.9.9
 neighbor 192.168.89.1 remote-as 64910
 neighbor 192.168.109.2 remote-as 64915
 neighbor 192.168.119.2 remote-as 64915
 network 192.168.89.0 mask 255.255.255.0
 network 192.168.109.0 mask 255.255.255.0
 network 192.168.119.0 mask 255.255.255.0
 network 9.9.9.0 mask 255.255.255.0

! R10
router bgp 64915
 bgp router-id 10.10.10.10
 neighbor 192.168.109.1 remote-as 64910
 neighbor 192.168.120.2 remote-as 64915
 network 192.168.109.0 mask 255.255.255.0
 network 192.168.120.0 mask 255.255.255.0
 network 10.10.10.0 mask 255.255.255.0

! R11
router bgp 64915
 bgp router-id 11.11.11.11
 neighbor 192.168.119.1 remote-as 64910
 neighbor 192.168.121.2 remote-as 64915
 network 192.168.119.0 mask 255.255.255.0
 network 192.168.121.0 mask 255.255.255.0
 network 11.11.11.0 mask 255.255.255.0

! R12
router bgp 64915
 bgp router-id 12.12.12.12
 neighbor 192.168.120.1 remote-as 64915
 neighbor 192.168.121.1 remote-as 64915
 network 192.168.120.0 mask 255.255.255.0
 network 192.168.121.0 mask 255.255.255.0
 network 12.12.12.0 mask 255.255.255.0
```

---

## BGP Concepts Explained

### What is BGP?
Border Gateway Protocol (BGP) is the routing protocol that powers the internet. It connects different organizations and ISPs together by exchanging routing information between Autonomous Systems.

### eBGP vs iBGP

| Feature | iBGP | eBGP |
|---------|------|------|
| Scope | Within same AS | Between different AS |
| AS numbers | Same on both sides | Different on each side |
| TTL | 255 (can be far apart) | 1 (must be directly connected) |
| Route forwarding | Does NOT forward to other iBGP peers | Freely forwards to all neighbors |
| Real-world example | Routers inside one ISP | Two different ISPs connecting |

### Autonomous System (AS)
A collection of IP networks under a single administrative domain sharing a common routing policy. Each AS has a unique AS number (ASN).

### BGP Router ID
A unique 32-bit identifier for each BGP router, typically set to the loopback IP address for stability.

### Loopback Interfaces
Virtual interfaces that never go down as long as the router is powered on. Used as stable BGP router IDs and peering addresses.

### The `network` Command
Tells BGP which prefixes to advertise to neighbors. The prefix must already exist in the routing table or BGP will ignore it.

---

## OSPF Concepts Explained

### Why OSPF is Needed Alongside BGP
BGP knows **what** destinations exist but needs OSPF to know **how** to physically reach the next hop router. Without OSPF, BGP packets get dropped because the router has no path to forward them.

```
Without OSPF:   R1 knows R12 exists ✅  but cannot reach it ❌
With OSPF:      R1 knows R12 exists ✅  and knows the physical path ✅
```

### OSPF Wildcard Mask
OSPF uses wildcard masks which are the inverse of subnet masks:
```
Subnet mask:   255.255.255.0  (used in ip address command)
Wildcard mask:   0.0.0.255    (used in OSPF network command)
```

### OSPF Areas
All routers in this lab use **Area 0** (the backbone area) for simplicity.

---

## Verification Commands

### Interface Verification
```cisco
! Check all interfaces and their status
show ip interface brief

! Check serial interface DCE/DTE status
show controllers se0/0/0
```

### OSPF Verification
```cisco
! Check OSPF neighbor relationships
show ip ospf neighbor

! Check OSPF routes in routing table
show ip route ospf

! Check OSPF database
show ip ospf database
```

### BGP Verification
```cisco
! Check BGP neighbor status
show ip bgp summary

! Check BGP routing table
show ip bgp

! Check specific BGP neighbor details
show ip bgp neighbors

! Check routing table for BGP routes
show ip route bgp
```

### Connectivity Testing
```cisco
! Ping within same AS
R1# ping 2.2.2.2

! Ping across all AS (ultimate test)
R1# ping 12.12.12.12

! Traceroute to see the full path
R1# traceroute 12.12.12.12
```

---

## Troubleshooting Guide

### Problem: BGP neighbors not forming (State shows Active or Idle)

| Check | Command | What to Look For |
|-------|---------|-----------------|
| Interface up | `show ip int brief` | up/up on all interfaces |
| Correct neighbor IP | `show run` | IP matches directly connected interface |
| Correct AS number | `show run` | remote-as matches neighbor's actual AS |
| Reachability | `ping <neighbor-ip>` | Should succeed before BGP forms |

### Problem: Interfaces showing down/down

```cisco
! Fix — go to interface and bring it up
conf t
interface fa0/0
no shutdown
```

### Problem: Serial link not coming up

```cisco
! Check which end is DCE
show controllers se0/0/0

! If DCE — add clock rate
conf t
interface se0/0/0
clock rate 64000
no shutdown
```

### Problem: BGP neighbors up but no routes

```cisco
! Check BGP table
show ip bgp

! Verify network exists in routing table first
show ip route 192.168.12.0

! If missing — the network command is wrong or interface is down
```

### Problem: Can ping neighbor but not far destinations

```cisco
! Check if OSPF neighbors formed
show ip ospf neighbor

! Check if BGP is learning routes
show ip bgp summary
! Look for numbers in PfxRcd column — 0 means no routes received
```

### BGP State Reference

| State | Meaning | Action |
|-------|---------|--------|
| **Established** | ✅ Working | None needed |
| **Active** | Trying to connect, failing | Check IP, AS number, reachability |
| **Idle** | Not trying | Check configuration |
| **Connect** | TCP connection attempt | Usually temporary |
| **OpenSent** | BGP open message sent | Usually temporary |

---

## Future Improvements

Here are suggested enhancements to extend this lab:

### 1. 🔒 BGP Authentication
Add MD5 authentication between BGP peers to secure neighbor relationships:
```cisco
neighbor 192.168.12.2 password cisco123
```

### 2. 📊 BGP Route Filtering
Implement prefix lists to control which routes are advertised:
```cisco
ip prefix-list FILTER seq 5 permit 1.1.1.0/24
neighbor 192.168.12.2 prefix-list FILTER out
```

### 3. ⚖️ BGP Path Manipulation
Use LOCAL_PREF and MED attributes to influence path selection:
```cisco
! Prefer one path over another
route-map SET_LOCAL_PREF permit 10
 set local-preference 200
neighbor 192.168.24.2 route-map SET_LOCAL_PREF in
```

### 4. 🔄 BGP Communities
Tag routes with communities for easier policy management:
```cisco
neighbor 192.168.24.2 send-community
route-map SET_COMMUNITY permit 10
 set community 64900:100
```

### 5. 🛡️ Prefix List Security
Prevent route hijacking by only accepting expected prefixes from each neighbor.

### 6. 📈 Traffic Engineering
Configure multiple paths between AS and use BGP attributes to load balance or create redundant paths.

### 7. 🔁 Redundancy Testing
Simulate link failures by shutting down interfaces and observing BGP reconvergence:
```cisco
interface se0/0/0
shutdown
! Observe BGP reconverging to alternate path
no shutdown
```

### 8. 📝 Upgrade to GNS3 or EVE-NG
Move this lab to GNS3 or EVE-NG for:
- Full IOS feature support (including route-reflectors)
- More realistic simulation
- Better scalability

---

## Key Lessons Learned

1. **OSPF must run before BGP** — BGP needs IGP to find next-hops
2. **Clock rate only on DCE end** — always verify with `show controllers`
3. **iBGP neighbors must be directly reachable** — in Packet Tracer, only peer with directly connected routers
4. **The `network` command requires existing routes** — BGP only advertises what's already in the routing table
5. **BGP takes time to converge** — wait 30-60 seconds after configuration before testing
6. **Loopbacks are always up** — use them as router IDs for stability
7. **eBGP vs iBGP** — different AS = eBGP, same AS = iBGP. Getting this wrong breaks neighbor formation

---

## References

- Original Lab: Patel Jay 

---

## License

This lab configuration is based on the Basic BGP Lab by Patel Jay.  
Free to use and modify for educational purposes.

---

*Built with ❤️ using Cisco Packet Tracer*
