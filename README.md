# CCNA--Day-10-Inter-VLAN-DHCP-Using-Router-on-a-Stick

## Overview

This lab builds on the DHCP concepts learned in Day 9. In this lab, a single router provides DHCP services to multiple VLANs while also performing Inter-VLAN Routing using Router-on-a-Stick (ROAS).

This lab demonstrates how a router can serve as both:

- DHCP Server
- Inter-VLAN Router

This is an important CCNA topic because it combines VLANs, trunking, DHCP, and routing into one solution.

---

## Objectives

After completing this lab, you will be able to:

- Configure multiple VLANs
- Configure Router-on-a-Stick
- Configure multiple DHCP pools
- Configure DHCP for different VLANs
- Perform Inter-VLAN Routing
- Verify DHCP operation
- Troubleshoot DHCP issues

---

## Network Topology

```text
                    R1
                     |
                   G0/0
                     |
                    SW1
                 ___|___
                |       |
              PC1     PC2

            VLAN10  VLAN20
```

---

## Devices Used

- 1 Cisco Router
- 1 Cisco Switch
- 2 PCs
- Copper Straight-Through Cables

---

## Physical Connections

| Device | Interface | Connected To |
|----------|----------|----------|
| R1 | G0/0 | SW1 Fa0/1 |
| PC1 | Fa0 | SW1 Fa0/10 |
| PC2 | Fa0 | SW1 Fa0/20 |

---

## Network Addressing Plan

### VLAN 10

| Item | Value |
|--------|--------|
| Network | 192.168.10.0/24 |
| Gateway | 192.168.10.1 |

### VLAN 20

| Item | Value |
|--------|--------|
| Network | 192.168.20.0/24 |
| Gateway | 192.168.20.1 |

---

## Step 1: Create VLANs

On SW1:

```bash
enable
configure terminal

vlan 10
name HR

vlan 20
name SALES
```

Verify:

```bash
show vlan brief
```

---

## Step 2: Configure Access Ports

### PC1

```bash
interface fa0/10
switchport mode access
switchport access vlan 10
```

### PC2

```bash
interface fa0/20
switchport mode access
switchport access vlan 20
```

---

## Step 3: Configure Trunk Link

```bash
interface fa0/1
switchport mode trunk
```

Verify:

```bash
show interfaces trunk
```

---

## Step 4: Configure Router-on-a-Stick

### Enable Physical Interface

```bash
interface g0/0
no shutdown
```

### Configure VLAN 10 Subinterface

```bash
interface g0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0
```

### Configure VLAN 20 Subinterface

```bash
interface g0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0
```

Verify:

```bash
show ip interface brief
```

---

## Step 5: Configure Excluded Addresses

```bash
ip dhcp excluded-address 192.168.10.1 192.168.10.10

ip dhcp excluded-address 192.168.20.1 192.168.20.10
```

Purpose:

Reserve addresses for routers, servers, printers, and network devices.

---

## Step 6: Configure DHCP Pool for VLAN 10

```bash
ip dhcp pool VLAN10

network 192.168.10.0 255.255.255.0

default-router 192.168.10.1

dns-server 8.8.8.8
```

---

## Step 7: Configure DHCP Pool for VLAN 20

```bash
ip dhcp pool VLAN20

network 192.168.20.0 255.255.255.0

default-router 192.168.20.1

dns-server 8.8.8.8
```

---

## Complete DHCP Configuration

```bash
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp excluded-address 192.168.20.1 192.168.20.10

ip dhcp pool VLAN10
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8

ip dhcp pool VLAN20
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 8.8.8.8
```

---

## Step 8: Configure PCs for DHCP

On both PCs:

```text
Desktop
 ↓
IP Configuration
 ↓
DHCP
```

Expected Results:

### PC1

```text
IP Address: 192.168.10.11
Gateway: 192.168.10.1
```

### PC2

```text
IP Address: 192.168.20.11
Gateway: 192.168.20.1
```

---

## How DHCP Works in This Lab

```text
PC1
 ↓
DHCP Discover
 ↓
R1 DHCP Server
 ↓
DHCP Offer
 ↓
DHCP Request
 ↓
DHCP Acknowledge
```

The router provides DHCP services directly to both VLANs.

No DHCP Relay Agent is required because the DHCP server is running on R1.

---

## Verification Commands

### Verify DHCP Pools

```bash
show ip dhcp pool
```

### Verify Assigned Addresses

```bash
show ip dhcp binding
```

### Verify Interfaces

```bash
show ip interface brief
```

### Verify VLANs

```bash
show vlan brief
```

### Verify Trunk

```bash
show interfaces trunk
```

---

## Connectivity Testing

### Test VLAN 10 Gateway

From PC1:

```bash
ping 192.168.10.1
```

Expected Result:

Success

---

### Test VLAN 20 Gateway

From PC2:

```bash
ping 192.168.20.1
```

Expected Result:

Success

---

### Test Inter-VLAN Routing

From PC1:

```bash
ping 192.168.20.11
```

Expected Result:

Success

The router routes traffic between VLAN 10 and VLAN 20.

---

## Troubleshooting Scenario 1

### Wrong VLAN Assignment

```bash
switchport access vlan 30
```

Result:

- Client receives no IP address
- Connectivity fails

---

## Troubleshooting Scenario 2

### Missing Trunk Configuration

```bash
no switchport mode trunk
```

Result:

- Router-on-a-Stick fails
- DHCP fails

---

## Troubleshooting Scenario 3

### Missing Subinterface

```bash
interface g0/0.20
```

Not configured.

Result:

- VLAN 20 users cannot obtain IP addresses

---

## Troubleshooting Scenario 4

### Incorrect DHCP Network

```bash
network 192.168.30.0 255.255.255.0
```

Result:

- DHCP pool does not match VLAN network
- Clients fail to obtain addresses

---

## Mini Challenge

### Topology

```text
                 R1
                  |
                 SW1
           _______|_______
          |       |       |
        PC1     PC2     PC3

      VLAN10 VLAN20 VLAN30
```

### Requirements

1. Create VLANs 10, 20, and 30
2. Configure Router-on-a-Stick
3. Create DHCP pools for all VLANs
4. Configure PCs for DHCP
5. Verify Inter-VLAN communication

### Addressing

| VLAN | Network | Gateway |
|--------|--------|--------|
| 10 | 192.168.10.0/24 | 192.168.10.1 |
| 20 | 192.168.20.0/24 | 192.168.20.1 |
| 30 | 192.168.30.0/24 | 192.168.30.1 |

---

## Skills Learned

- Router-on-a-Stick
- Multiple VLANs
- DHCP Pools
- DHCP Address Assignment
- Inter-VLAN Routing
- DHCP Verification
- DHCP Troubleshooting
- Enterprise VLAN Design

---

## Conclusion

This lab demonstrated how a Cisco router can provide DHCP services to multiple VLANs while simultaneously performing Inter-VLAN Routing. This design is commonly used in small and medium-sized networks and provides a strong foundation for future enterprise routing labs.

---

## Next Lab

### Day 11 – Static Routing Fundamentals

Topics:

- Directly Connected Networks
- Static Routes
- Routing Tables
- Route Lookup Process
- Default Routes
- Route Verification
- Static Route Troubleshooting

---

## Author

Muhammad Kausar

CCNA Enterprise Networking Lab Series
