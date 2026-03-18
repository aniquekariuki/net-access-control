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
! Trunk carries traffic for all VLANs to the router
! Native VLAN set to 99 for better security than default VLAN 1
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
! Enabling port security on all 24 lab ports
! Only 1 device allowed per port - switch remembers the first device
! If an intruder plugs in, the port shuts down automatically
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
```

> **Packet Tracer Limitation:** The 2960-24TT switch model in Packet Tracer only has 24 FastEthernet ports (Fa0/1–Fa0/24). Ports Fa0/25–48 do not exist on this model, so those commands are commented out below. On a real 2960-48TT switch (which has 48 ports), you would uncomment these.

```
! ============================================================
! EXTENDED PORT RANGE - NOT AVAILABLE ON 2960-24TT IN PACKET TRACER
! The 2960-24TT only has FastEthernet 0/1 - 0/24.
! On a real 48-port switch, uncomment these commands.
! ============================================================
!
! interface range FastEthernet0/25 - 48
!  description Lab Workstations (continued)
!  switchport mode access
!  switchport access vlan 10
!  switchport port-security
!  switchport port-security maximum 1
!  switchport port-security mac-address sticky
!  switchport port-security violation shutdown
!  spanning-tree portfast
!  no shutdown
```

---

## Administration Access Ports (VLAN 20)

A few ports are set aside for staff machines, with the same port security applied.

```
! Security for staff machines too
interface GigabitEthernet0/2
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

> **Packet Tracer Limitation:** DHCP Snooping is supported in Packet Tracer 8.2 and newer. If you are on an older version, these commands may error. They are commented out for safety - if your version supports them, you can paste them without the `!` prefix.

```
! ============================================================
! DHCP SNOOPING - MAY NOT WORK ON OLDER PACKET TRACER VERSIONS
! Supported on Packet Tracer 8.2+. If your version supports it,
! remove the ! and paste the commands directly.
! ============================================================
!
! ip dhcp snooping
! ip dhcp snooping vlan 10,20
!
! interface GigabitEthernet0/1
!  ip dhcp snooping trust
```

---

## Unused Port Hardening

Any port that is not being used should be shut down so no one can just walk in and plug into an open jack.

```
! Apply to any ports not in use, for example:
! interface range FastEthernet0/5 - 24
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
