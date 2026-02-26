# Network Optimization and Security for a Shared Computer Lab

## 1. Problem Statement

A shared computer lab has 50 workstations connected to the internet through a single ISP router. During peak hours, the internet slows down significantly because there is nothing in place to control how bandwidth is shared between the machines. On top of that, the administration has no way to see which computers are using the most bandwidth, and there is no way to stop unauthorized devices (like personal laptops) from being plugged into the network.

This project solves three problems:

| Problem | What happens |
|---------|-------------|
| Bandwidth is not controlled | A few computers downloading heavy files can make the internet unusable for everyone else |
| No way to monitor traffic | The admin cannot tell which machine is causing the slowdown |
| No device restrictions | Anyone can plug in a personal device and use the network without permission |

## 2. How Each Problem is Solved

### 2.1 Controlling Bandwidth with QoS (Quality of Service)

The edge router is set up with a QoS policy that limits each workstation to a maximum download speed (for example, 2 Mbps per device). This way, even if one user is downloading a large file, the remaining bandwidth is still available for the other 49 machines.

### 2.2 Monitoring Traffic with NetFlow

NetFlow is turned on at the router level. It records the traffic going in and out of the network, tracking which IP address is sending or receiving the most data. In a real setup, this data would be sent to a tool like PRTG or Grafana where the admin can view it on a dashboard.

### 2.3 Blocking Unauthorized Devices with Port Security and DHCP Snooping

On the managed switch, two features are configured:

- **Port Security (Sticky MAC):** Each switch port saves the MAC address of the computer connected to it. If someone unplugs the lab PC and connects a different device, the port automatically shuts down.
- **DHCP Snooping:** Only the router is allowed to hand out IP addresses. This prevents someone from setting up a rogue DHCP server that could redirect traffic.

The lab network is also placed in its own VLAN (VLAN 10) to keep it separated from admin and management traffic.

## 3. Network Architecture

### 3.1 Current State (Baseline)

![Current Network State](diagrams/current_network.png)

All 50 PCs connect through a basic switch to a single router. There is no segmentation, no monitoring, and no access control.

### 3.2 Proposed State (Secured)

![Proposed Network Design](diagrams/proposed_network.png)

The redesigned network uses an enterprise-grade router with QoS and NetFlow, a managed switch with Port Security and DHCP Snooping, and three VLANs to separate lab, admin, and management traffic.

## 4. Repository Structure

```
/configs/        Router and switch CLI configuration scripts
/topology/       Cisco Packet Tracer simulation files (.pkt)
/diagrams/       Network diagrams showing the before and after
```

## 5. Tools Used

| Tool | What it was used for |
|------|---------------------|
| Cisco Packet Tracer | Building and testing the network simulation |
| Draw.io | Creating the network diagrams |
| Visual Studio Code | Writing the configuration scripts and documentation |
| Git and GitHub | Version control and hosting the project |

## 6. References

- Cisco Systems, "Quality of Service Configuration Guide," Cisco IOS Documentation
- Cisco Systems, "Configuring Port Security," Cisco Catalyst Switch Documentation
- Cisco Systems, "Configuring DHCP Snooping," Cisco IOS Documentation
- Cisco Systems, "Configuring NetFlow," Cisco IOS Documentation
