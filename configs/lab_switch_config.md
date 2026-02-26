# Managed Switch Configuration

**Device:** Cisco Catalyst 2960 (or equivalent in Packet Tracer)

**Purpose:** VLAN segmentation, Port Security, DHCP Snooping

---

## Basic Device Setup

Sets the hostname, secures access, and adds a warning banner.

```
enable
configure terminal

hostname LabSwitch
no ip domain-lookup
enable secret class
line console 0
 password cisco
 login
line vty 0 15
 password cisco
 login
 transport input ssh
service password-encryption
banner motd # Authorized Access Only. All activity is logged. #
```

---

## VLAN Creation

Three VLANs are created to split the network into separate broadcast domains. This keeps lab traffic isolated from admin and management traffic.

| VLAN ID | Name | Purpose |
|---------|------|---------|
| 10 | Lab_Workstations | All 50 student PCs |
| 20 | Administration | Staff machines |
| 99 | Management | Switch and router remote access |

```
vlan 10
 name Lab_Workstations
vlan 20
 name Administration
vlan 99
 name Management
```

---

## Trunk Port (Uplink to Edge Router)

This port carries tagged traffic from all VLANs to the router so it can route between them. The native VLAN is set to 99 instead of the default VLAN 1 for security reasons.

```
interface GigabitEthernet0/1
 description Trunk to Edge Router
 switchport mode trunk
 switchport trunk allowed vlan 10,20,99
 switchport trunk native vlan 99
 no shutdown
```

---

## Lab Workstation Access Ports (VLAN 10) - Solves the Unauthorized Device Problem

This is the core security configuration. Each port is set to:

- **Access mode on VLAN 10** so only lab traffic passes through
- **Port Security with sticky MAC** so the port remembers the first device connected to it
- **Maximum 1 MAC address** so only that one device is allowed
- **Violation shutdown** so if someone plugs in a different device, the port disables itself

```
interface range FastEthernet0/1 - 24
 description Lab Workstations
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
 spanning-tree portfast
 no shutdown

interface range FastEthernet0/25 - 48
 description Lab Workstations (continued)
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
 spanning-tree portfast
 no shutdown
```

---

## Administration Access Ports (VLAN 20)

A few ports are set aside for staff machines, with the same port security applied.

```
interface range GigabitEthernet0/2 - 2
 description Admin Staff Machine
 switchport mode access
 switchport access vlan 20
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
 spanning-tree portfast
 no shutdown
```

---

## DHCP Snooping

This feature makes sure only the router can hand out IP addresses. The trunk port is marked as trusted because that is where the real DHCP server is. Every other port is untrusted by default, so if someone connects a rogue DHCP server, it gets blocked.

```
ip dhcp snooping
ip dhcp snooping vlan 10,20

interface GigabitEthernet0/1
 ip dhcp snooping trust
```

---

## Unused Port Hardening

Any port that is not being used should be shut down so no one can just walk in and plug into an open jack.

```
! Apply to any ports not in use, for example:
! interface range FastEthernet0/49 - 48
!  shutdown
```

---

## Management VLAN Interface

This gives the switch an IP address on VLAN 99 so it can be managed remotely via SSH.

```
interface vlan 99
 ip address 192.168.99.2 255.255.255.0
 no shutdown

ip default-gateway 192.168.99.1
```

---

## Save

```
end
write memory
```
