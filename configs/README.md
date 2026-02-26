# Configuration Scripts

This directory contains the Cisco IOS command-line configuration scripts applied to the network devices in the simulation. These scripts can be directly pasted into a Cisco router or switch CLI to reproduce the configuration.

## Contents

| File | Device | Purpose |
|------|--------|---------|
| edge_router_config.txt | Edge Router (Cisco 2911) | QoS rate-limiting, NetFlow export, inter-VLAN routing, NAT |
| lab_switch_config.txt | Managed Switch (Cisco 2960) | VLAN creation, Port Security, DHCP Snooping, trunk configuration |
