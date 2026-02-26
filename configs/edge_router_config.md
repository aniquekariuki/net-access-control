# Edge Router Configuration

**Device:** Cisco 2911 (or equivalent in Packet Tracer)

**Purpose:** Inter-VLAN routing, QoS rate-limiting, NetFlow traffic monitoring, NAT for internet access

---

## Basic Device Setup

Sets the hostname, secures console and remote access with passwords, and displays a warning banner.

```
enable
configure terminal

hostname EdgeRouter
no ip domain-lookup
enable secret class
line console 0
 password cisco
 login
line vty 0 4
 password cisco
 login
 transport input ssh
service password-encryption
banner motd # Authorized Access Only. All activity is logged. #
```

---

## Interface Configuration

GigabitEthernet0/0 faces the ISP (WAN side). GigabitEthernet0/1 connects to the managed switch and is split into sub-interfaces, one per VLAN. This is called Router-on-a-Stick.

```
interface GigabitEthernet0/0
 description Link to ISP
 ip address dhcp
 ip nat outside
 no shutdown

interface GigabitEthernet0/1
 description Trunk link to Managed Switch
 no shutdown

interface GigabitEthernet0/1.10
 description Lab Workstations - VLAN 10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 ip nat inside

interface GigabitEthernet0/1.20
 description Administration - VLAN 20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 ip nat inside

interface GigabitEthernet0/1.99
 description Management - VLAN 99
 encapsulation dot1Q 99
 ip address 192.168.99.1 255.255.255.0
 ip nat inside
```

---

## NAT (Network Address Translation)

All three VLANs share a single public IP address from the ISP. This is called PAT (Port Address Translation) or NAT overload.

```
access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.20.0 0.0.0.255
access-list 1 permit 192.168.99.0 0.0.0.255
ip nat inside source list 1 interface GigabitEthernet0/0 overload
```

---

## DHCP Server

The router hands out IP addresses automatically to devices in the lab and admin VLANs. The first 10 addresses in each range are reserved for static assignments (like printers or servers).

```
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp pool LAB_POOL
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8 8.8.4.4
 lease 0 8 0

ip dhcp excluded-address 192.168.20.1 192.168.20.10
ip dhcp pool ADMIN_POOL
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 8.8.8.8 8.8.4.4
```

---

## Quality of Service (QoS) - Solves the Bandwidth Problem

This is the fix for the main problem. The policy below matches all traffic from the lab VLAN (192.168.10.0/24) and caps it at 2 Mbps. This prevents any single workstation from using all the bandwidth.

Packet Tracer does not fully simulate QoS, so the rate-limiting will not be visible in the simulator. However, these are valid Cisco IOS commands that work on real hardware.

```
class-map match-all LAB_TRAFFIC
 match access-group 10

access-list 10 permit 192.168.10.0 0.0.0.255

policy-map BANDWIDTH_LIMIT
 class LAB_TRAFFIC
  police 2000000 50000 exceed-action drop

interface GigabitEthernet0/1.10
 service-policy output BANDWIDTH_LIMIT
```

---

## NetFlow (Traffic Monitoring) - Solves the Visibility Problem

NetFlow records every traffic flow passing through the router. It tracks which IP address is sending or receiving data, how much, and where to. The data is exported to a collector at 192.168.99.10 on port 2055, where it can be viewed on a dashboard.

Packet Tracer does not support these commands, but they are included here because this is what would be configured on actual equipment.

```
ip flow-export version 5
ip flow-export destination 192.168.99.10 2055

interface GigabitEthernet0/1.10
 ip flow ingress
 ip flow egress
```

---

## Save

```
end
write memory
```
