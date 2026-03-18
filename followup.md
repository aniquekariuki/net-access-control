# Complete Setup Guide: From Scratch to Working Demo

This guide walks you through building the entire secured network from scratch in Cisco Packet Tracer. Follow every step in order. By the end, you will have a fully working network with port security, VLANs, DHCP, NAT, and a security violation demo.

---

## Project Objectives Overview

This project solves 3 problems in a shared school computer lab. The table below shows which objective is covered by which part of this guide.

| Objective | Problem it Solves | Where it is Configured | Packet Tracer Support |
|---|---|---|---|
| **1. Bandwidth Control (QoS)** | One user hogging all bandwidth | Router — class-map, policy-map, police | Not supported — commands are documented but commented out |
| **2. Traffic Visibility (NetFlow)** | Admin cannot see who is using the most data | Router — ip flow-export, ip flow ingress/egress | Not supported — commands are documented but commented out |
| **3. Network Hardening (Port Security + VLANs)** | Unauthorized devices plugging into the network | Switch — port-security, sticky MAC, VLANs | Fully supported — live demo included |

Objectives 1 and 2 use valid Cisco IOS commands that run on real production hardware but are not supported by Packet Tracer's simulator. They are included in Part 2 as commented-out commands so the full solution is documented.

Objective 3 is fully functional in Packet Tracer and is demonstrated live in Parts 4 and 5.

---

## Part 1: Build the Topology

Open Cisco Packet Tracer and create a new blank project.

### 1.1 Add the Devices

From the bottom device panel, drag these onto the workspace:

| Device | Where to find it | Model to select |
|---|---|---|
| Router | Network Devices → Routers | **2911** |
| Switch | Network Devices → Switches | **2960-24TT** |
| PC 1 | End Devices | **PC** |
| PC 2 | End Devices | **PC** |
| PC 3 | End Devices | **PC** |

Place the Router at the top, the Switch in the middle, and the 3 PCs at the bottom.

### 1.2 Connect the Cables

Use **Copper Straight-Through** cables for all connections. Click the cable icon (lightning bolt) at the bottom, select "Copper Straight-Through", then click the two devices to connect.

| From Device | From Port | To Device | To Port |
|---|---|---|---|
| Router | GigabitEthernet0/1 | Switch | GigabitEthernet0/1 |
| Switch | FastEthernet0/1 | PC 1 | FastEthernet0 |
| Switch | FastEthernet0/2 | PC 2 | FastEthernet0 |
| Switch | FastEthernet0/3 | PC 3 | FastEthernet0 |

After connecting, wait a few seconds. The link lights will turn green (or orange briefly, then green).

---

## Part 2: Configure the Router

Click on the **Router** → go to the **CLI** tab.

You will see:

```
--- System Configuration Dialog ---

Would you like to enter the initial configuration dialog? [yes/no]:
```

Type **no** and press Enter. Then press Enter again to get to the `Router>` prompt.

### 2.1 Paste Router Commands

Copy everything below and paste it into the Router CLI. You can use the **Paste** button at the bottom-right of the CLI window.

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
service password-encryption
banner motd # Authorized Access Only. All activity is logged. #

interface GigabitEthernet0/1
 description Trunk link to Managed Switch
 no shutdown

interface GigabitEthernet0/1.10
 description Lab Workstations - VLAN 10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface GigabitEthernet0/1.20
 description Administration - VLAN 20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0

interface GigabitEthernet0/1.99
 description Management - VLAN 99
 encapsulation dot1Q 99
 ip address 192.168.99.1 255.255.255.0

ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp pool LAB_POOL
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
 lease 0 8 0

ip dhcp excluded-address 192.168.20.1 192.168.20.10
ip dhcp pool ADMIN_POOL
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 8.8.8.8

end
write memory
```

### 2.2 What to Expect

After pasting, you should see:

- Each command accepted without errors
- At the end: `Building configuration... [OK]`
- The prompt should now say `EdgeRouter#` instead of `Router#`

### 2.3 Verify Router Configuration

Type these commands one by one to check everything is correct:

**Check the sub-interfaces are up:**
```
show ip interface brief
```

Expected output — you should see:

```
Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         unassigned      YES unset  administratively down down
GigabitEthernet0/1         unassigned      YES unset  up                    up
GigabitEthernet0/1.10      192.168.10.1    YES manual up                    up
GigabitEthernet0/1.20      192.168.20.1    YES manual up                    up
GigabitEthernet0/1.99      192.168.99.1    YES manual up                    up
```

If GigabitEthernet0/1 shows "up" and the sub-interfaces show their IP addresses, the router is configured correctly.

**Check DHCP pools exist:**
```
show ip dhcp pool
```

Expected output — you should see LAB_POOL and ADMIN_POOL listed with their network ranges.

### 2.4 Objective 1: Bandwidth Control (QoS) — Documented Only

These QoS commands solve the bandwidth hogging problem by capping each lab workstation at 2 Mbps. They use Cisco's Modular QoS CLI (class-map, policy-map, police).

Packet Tracer does not support MQC, so these commands are commented out. On a real Cisco 2911 router, you would paste them without the `!` prefix and they would execute.

If you paste the block below into Packet Tracer, IOS will simply ignore the lines starting with `!` — no errors will occur.

```
! ============================================================
! OBJECTIVE 1: BANDWIDTH CONTROL (QoS)
! These commands are valid Cisco IOS but are NOT supported
! in Packet Tracer. On real hardware, remove the ! prefix.
! ============================================================
!
! class-map match-all LAB_TRAFFIC
!  match access-group 10
!
! access-list 10 permit 192.168.10.0 0.0.0.255
!
! policy-map BANDWIDTH_LIMIT
!  class LAB_TRAFFIC
!   police 2000000 50000 exceed-action drop
!
! interface GigabitEthernet0/1.10
!  service-policy output BANDWIDTH_LIMIT
```

**What these commands do (for your understanding):**

| Command | Purpose |
|---|---|
| `class-map match-all LAB_TRAFFIC` | Creates a traffic class that identifies lab network traffic |
| `match access-group 10` | Links the class to access-list 10 (which matches 192.168.10.0/24) |
| `policy-map BANDWIDTH_LIMIT` | Creates a policy that defines what to do with the matched traffic |
| `police 2000000 50000 exceed-action drop` | Limits traffic to 2 Mbps (2,000,000 bits/sec). Any traffic above this is dropped |
| `service-policy output BANDWIDTH_LIMIT` | Applies the policy to outbound traffic on the lab VLAN sub-interface |

### 2.5 Objective 2: Traffic Visibility (NetFlow) — Documented Only

These NetFlow commands solve the visibility problem by tracking every data flow through the router — which IP is sending/receiving, how much data, and where it goes.

Packet Tracer does not support NetFlow at all. On real equipment, these commands would export flow data to a monitoring dashboard at 192.168.99.10.

```
! ============================================================
! OBJECTIVE 2: TRAFFIC VISIBILITY (NetFlow)
! These commands are valid Cisco IOS but are NOT supported
! in Packet Tracer. On real hardware, remove the ! prefix.
! ============================================================
!
! ip flow-export version 5
! ip flow-export destination 192.168.99.10 2055
!
! interface GigabitEthernet0/1.10
!  ip flow ingress
!  ip flow egress
```

**What these commands do (for your understanding):**

| Command | Purpose |
|---|---|
| `ip flow-export version 5` | Sets NetFlow to version 5 format (industry standard) |
| `ip flow-export destination 192.168.99.10 2055` | Sends flow data to a collector server on port 2055 |
| `ip flow ingress` | Tracks all traffic entering the interface |
| `ip flow egress` | Tracks all traffic leaving the interface |

---

## Part 3: Configure the Switch

Click on the **Switch** → go to the **CLI** tab.

Press Enter to get to the `Switch>` prompt.

### 3.1 Paste Switch Commands — Block 1: Basic Setup and VLANs

Copy and paste this first block:

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

vlan 10
 name Lab_Workstations
vlan 20
 name Administration
vlan 99
 name Management

end
write memory
```

Expected output:

- Prompt changes to `LabSwitch#`
- `Building configuration... [OK]`

### 3.2 Verify VLANs Were Created

```
show vlan brief
```

Expected output:

```
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1, Fa0/2, Fa0/3 ... (all ports)
10   Lab_Workstations                 active
20   Administration                   active
99   Management                       active
```

You should see VLANs 10, 20, and 99 listed. The ports will still be on VLAN 1 — that is normal, we will move them in the next step.

### 3.3 Paste Switch Commands — Block 2: Trunk Port

```
configure terminal

interface GigabitEthernet0/1
 description Trunk to Edge Router
 switchport mode trunk
 switchport trunk allowed vlan 10,20,99
 switchport trunk native vlan 99
 no shutdown

end
write memory
```

Expected output:

- No errors
- `Building configuration... [OK]`

### 3.4 Paste Switch Commands — Block 3: Port Security

This is the most important block — the port security configuration.

```
configure terminal

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

end
write memory
```

Expected output:

- You will see `%Warning: portfast should only be enabled on ports connected to a single host...` repeated for each of the 24 ports. **This is normal.** These are informational warnings, not errors.
- At the end: `Building configuration... [OK]`

### 3.5 Paste Switch Commands — Block 4: Management VLAN

```
configure terminal

interface vlan 99
 ip address 192.168.99.2 255.255.255.0
 no shutdown

ip default-gateway 192.168.99.1

end
write memory
```

### 3.6 Verify Port Security is Active

```
show port-security
```

Expected output:

```
Secure Port MaxSecureAddr CurrentAddr SecurityViolation Security Action
               (Count)       (Count)        (Count)
--------------------------------------------------------------------
        Fa0/1        1          0                 0         Shutdown
        Fa0/2        1          0                 0         Shutdown
        Fa0/3        1          0                 0         Shutdown
        ...
       Fa0/24        1          0                 0         Shutdown
----------------------------------------------------------------------
```

All 24 FastEthernet ports should show:
- MaxSecureAddr = 1
- CurrentAddr = 0 (no MAC learned yet)
- SecurityViolation = 0
- Security Action = Shutdown

If you see this, port security is fully configured.

### 3.7 Verify VLANs are Assigned

```
show vlan brief
```

Expected output — now the FastEthernet ports should be under VLAN 10:

```
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gig0/2
10   Lab_Workstations                 active    Fa0/1, Fa0/2, Fa0/3 ... Fa0/24
20   Administration                   active
99   Management                       active
```

---

## Part 4: Test DHCP and Connectivity

### 4.1 Check if PCs Received IP Addresses

Click on **PC 1** → go to **Desktop** → click **IP Configuration**.

- If it says "Static", change it to **DHCP**
- Wait a few seconds
- The PC should receive an IP address like `192.168.10.11` (the first 10 addresses are excluded, so it starts from .11)

Expected values:

```
IP Address:      192.168.10.11  (or .12, .13, etc.)
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.10.1
DNS Server:      8.8.8.8
```

Repeat for PC 2 and PC 3 — they should each get a different IP in the 192.168.10.x range.

### 4.2 Test Connectivity

Click on **PC 1** → go to **Desktop** → click **Command Prompt**.

Type:
```
ping 192.168.10.1
```

Expected output:

```
Pinging 192.168.10.1 with 32 bytes of data:

Reply from 192.168.10.1: bytes=32 time<1ms TTL=255
Reply from 192.168.10.1: bytes=32 time<1ms TTL=255
Reply from 192.168.10.1: bytes=32 time<1ms TTL=255
Reply from 192.168.10.1: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.10.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

The first ping might time out (say "Request timed out") — that is normal on the first attempt because of ARP resolution. Try pinging again and all 4 should reply.

### 4.3 Verify the Switch Learned the MAC Address

After the ping, go back to the **Switch CLI** and type:

```
show port-security interface FastEthernet0/1
```

Expected output:

```
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Shutdown
Aging Time                 : 0 mins
Aging Type                 : Absolute
SecureStatic Address Aging : Disabled
Maximum MAC Addresses      : 1
Total MAC Addresses        : 1
Configured MAC Addresses   : 0
Sticky MAC Addresses       : 1
Last Source Address:Vlan    : 00XX.XXXX.XXXX:10
Security Violation Count   : 0
```

Key things to look for:
- **Port Status: Secure-up** — means the port is active and secured
- **Sticky MAC Addresses: 1** — the switch has learned the PC's MAC
- **Last Source Address** — shows the MAC address of the PC that is authorized on this port
- **Security Violation Count: 0** — no violations yet

---

## Part 5: Trigger the Security Violation (The Demo)

This is the "wow moment" — proving that an unauthorized device gets blocked.

### 5.1 Remove the Authorized PC

1. Click on the **cable** connecting PC 1 to the switch port Fa0/1
2. Press the **Delete** key on your keyboard to remove the cable

### 5.2 Add an Unauthorized Laptop

1. From the bottom panel, go to **End Devices** → drag a **Laptop** onto the workspace
2. Use a **Copper Straight-Through** cable to connect the Laptop to the switch port **FastEthernet0/1** (the same port where PC 1 was)

### 5.3 Try to Use the Network from the Laptop

1. Click on the **Laptop** → go to **Desktop** → click **Command Prompt**
2. Type: `ipconfig`

### 5.4 Observe the Result

Look at the switch in the topology view:

- The link light on port Fa0/1 should turn **RED** or **AMBER**
- This means the port has been shut down due to a security violation

### 5.5 Verify the Violation on the Switch CLI

Go to the **Switch CLI** and type:

```
show port-security interface FastEthernet0/1
```

Expected output:

```
Port Security              : Enabled
Port Status                : Secure-shutdown
Violation Mode             : Shutdown
Aging Time                 : 0 mins
Aging Type                 : Absolute
SecureStatic Address Aging : Disabled
Maximum MAC Addresses      : 1
Total MAC Addresses        : 0
Configured MAC Addresses   : 0
Sticky MAC Addresses       : 1
Last Source Address:Vlan    : 00YY.YYYY.YYYY:10
Security Violation Count   : 1
```

Key things to look for:
- **Port Status: Secure-shutdown** — the port has been disabled
- **Security Violation Count: 1** — one violation was detected
- **Last Source Address** — shows the MAC of the unauthorized laptop (different from the original PC)

This is your proof that port security works.

---

## Part 6: Reset the Port (If You Need to Demo Again)

After a violation, the port stays shut down until you manually re-enable it. To reset:

### 6.1 Remove the Unauthorized Laptop

1. Delete the cable from the Laptop to the switch
2. You can also delete the Laptop from the workspace if you want

### 6.2 Re-enable the Port

Go to the **Switch CLI** and type:

```
configure terminal
interface FastEthernet0/1
 shutdown
 no shutdown
end
write memory
```

### 6.3 Reconnect the Original PC

1. Use a Copper Straight-Through cable to reconnect **PC 1** to switch port **FastEthernet0/1**
2. On PC 1, open Command Prompt and type: `ping 192.168.10.1`
3. The switch will re-learn the PC's MAC address
4. The link light should turn **GREEN** again

Now the port is ready for another demo if needed.

---

## Part 7: Save the Project

In Packet Tracer, go to **File** → **Save As** and save the file. This saves everything — the topology, all device configurations, and the state of the network.

Next time you open this file, all configurations will already be there. You will not need to paste commands again.

---

## Quick Reference: All Passwords

| Device | Console Password | Enable Password |
|---|---|---|
| EdgeRouter | cisco | class |
| LabSwitch | cisco | class |

When you click a device and go to the CLI tab:
1. It asks `Password:` → type **cisco** (this is the console password)
2. Then type `enable` → it asks `Password:` → type **class** (this is the enable secret)
3. You are now at the `#` prompt and can run any show or configure commands

---

## Quick Reference: Verification Commands

| Command | Where to run | What it shows |
|---|---|---|
| `show port-security` | Switch | All ports with security status |
| `show port-security interface Fa0/1` | Switch | Detailed status of one port |
| `show vlan brief` | Switch | Which VLANs exist and port assignments |
| `show ip interface brief` | Router | All interfaces and their IP addresses |
| `show ip dhcp pool` | Router | DHCP pools and how many addresses are leased |
| `show ip dhcp binding` | Router | Which IP was given to which MAC address |
| `show running-config` | Both | The entire current configuration |
