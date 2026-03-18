# Student Presentation Guide: Network Optimization & Security

This guide is designed to help you walk a supervisor or panel through your project. Follow these sections to show your technical knowledge and prove your solution works.

---

## 1. Introduction: The Problem (1-2 Minutes)
**What to say:**
*"Today I am presenting a network redesign for a shared school computer lab. The original network had three major issues: zero control over bandwidth, no visibility into traffic, and a major security risk where anyone could plug in unauthorized devices. During peak hours, the internet would become unusable, and the administration had no way to identify or stop the cause."*

**Key Points to Mention:**
- Bandwidth congestion from heavy downloads.
- Lack of traffic logs (Visibility).
- Unauthorized physical access (Security).

---

## 2. Technical Solution: The "Three Pillars" (3-5 Minutes)
**What to say:**
*"To solve these problems, I implemented a three-pillar technical solution on the router and switch."*

**Pillar 1: Bandwidth Control (QoS)**
*"I configured **Quality of Service (QoS)** on the Edge Router using Cisco's Modular QoS CLI. By creating a class-map to identify lab traffic and applying a policy-map with a 2 Mbps policer, I've ensured that no single user can 'starve' the rest of the lab of bandwidth."*

> **If they ask why it's commented out:**
> *"QoS uses Modular QoS CLI - commands like class-map, policy-map, and police. These are valid production Cisco IOS commands, but Packet Tracer's simulator does not support MQC. I've included them commented out in my documentation so you can see the exact configuration I would deploy on real equipment. On a physical Cisco 2911 router, these commands run without any issue."*

- [**See the QoS Config Code**](configs/edge_router_config.md#L102)

**Pillar 2: Traffic Visibility (NetFlow)**
*"I enabled **NetFlow** to monitor data flows. This allows the admin to track which IP addresses are using the most data in real-time, with the flow data exported to a collector for dashboard viewing."*

> **If they ask why it's commented out:**
> *"NetFlow is a monitoring protocol that requires an external collector to receive the data. Packet Tracer does not simulate NetFlow at all - the commands ip flow-export and ip flow ingress are not recognized by the simulator. However, these are standard Cisco IOS commands that are configured on production routers worldwide. I've documented them to show I understand the full end-to-end solution."*

- [**See the NetFlow Config Code**](configs/edge_router_config.md#L131)

**Pillar 3: Network Hardening (VLANs & Port Security)**
*"I segmented the network into **VLANs** (10 for Lab, 20 for Admin, 99 for Management). Most importantly, I configured **Port Security with Sticky MAC** on the switches. This effectively 'locks' each port to a specific authorized machine. If someone unplugs a lab PC and connects their own device, the port shuts down automatically."*

*"This pillar is **fully functional** in Packet Tracer, and I will demonstrate it live."*

- [**See the Port Security Config Code**](configs/lab_switch_config.md#L68)
- [**See the Security Proof Diagram**](diagrams/security_violation.png)

---

## 3. The Live Demonstration (3-5 Minutes)
**Action:** Open `secured_network.pkt` in Packet Tracer.

### Step 1: Show the Configuration is Running
*"Let me first show you that the switch is configured correctly."*

1. Click on the **Switch** → go to the **CLI** tab
2. Type:
```
enable
show port-security
```
3. **What to say:** *"As you can see, port security is enabled on all FastEthernet ports. The maximum MAC address count is 1, and the violation action is set to Shutdown."*

### Step 2: DHCP Verification
*"Now let me show that the DHCP and VLAN setup is working."*

1. Click on any **PC** → go to **Desktop** → **IP Configuration**
2. **What to say:** *"Notice the PC has received an IP address in the 192.168.10.x range automatically from the router's DHCP pool. This confirms my VLAN trunking, Router-on-a-Stick, and DHCP configuration are all working together."*

### Step 3: Let the Switch Learn the MAC Address
*"Before I show the security demo, I need the switch to learn which device belongs on this port."*

1. Click on the **PC** connected to Fa0/1 → **Desktop** → **Command Prompt**
2. Type: `ping 192.168.10.1`
3. Wait for a successful reply
4. Go back to the **Switch CLI** and type:
```
show port-security interface FastEthernet0/1
```
5. **What to say:** *"The switch has now learned this PC's MAC address as the 'sticky' address for port Fa0/1. Only this specific device is authorized on this port."*

### Step 4: The Security Test - The "Wow" Moment
*"Now, watch what happens if I act as an intruder trying to connect an unauthorized device."*

1. **Delete the cable** from the PC to switch port Fa0/1 (click the cable and press Delete)
2. **Drag a new Laptop** onto the workspace
3. **Connect the Laptop** to the **same port Fa0/1** using a Copper Straight-Through cable
4. On the Laptop, go to **Desktop** → **Command Prompt** and type: `ipconfig`
5. **What to say:** *"As you can see, the link light on port Fa0/1 immediately turned **RED**. The switch detected that the MAC address of this laptop does not match the authorized sticky MAC address. The port has been shut down - the violation action 'shutdown' has been triggered. The intruder is completely blocked from accessing the network."*

### Step 5: Verify the Violation (Proof)
1. Go to the **Switch CLI** and type:
```
show port-security interface FastEthernet0/1
```
2. **What to say:** *"Look at the output - it says Port Status: Secure-shutdown, and the Security Violation Count has incremented. This is conclusive proof that the port security is working exactly as designed."*

---

## 4. Closing & Conclusion (1 Minute)
**What to say:**
*"By combining these professional-grade security and optimization tools - QoS for bandwidth control, NetFlow for visibility, and Port Security for access control - I've transformed a 'flat' insecure network into a controlled, segmented, and hardened environment."*

*"The security features are fully demonstrated in Packet Tracer. The QoS and NetFlow commands are documented as production-ready IOS configurations that would be deployed on real Cisco hardware. Together, these three pillars address every problem identified in the original network."*

**Final Line:**
*"Thank you for your time. I am now open to any technical questions you may have about the configuration."*

---

## 5. Handling Tough Questions

### Q1: "Why are some commands commented out?"
**The Answer:**
*"The commands that are commented out - specifically QoS (class-map, policy-map, police) and NetFlow (ip flow-export) - are valid Cisco IOS commands that run on real production hardware. However, Packet Tracer is a simulator with limitations. It does not support Modular QoS CLI or NetFlow because these features require hardware-level processing that can't be simulated."*

*"I made a deliberate choice to keep them visible in the documentation so that a reviewer can see the complete real-world solution I designed. If I were deploying this on a physical Cisco 2911 router, I would simply remove the comment markers and the commands would execute. The fact that I know which commands Packet Tracer supports and which it doesn't actually demonstrates a deeper understanding of the platform."*

### Q2: "Why did you choose hardware configuration instead of a software application?"
**The Answer:**
*"I chose a **Network Engineering approach** because security and optimization are most effective at the infrastructure level. By configuring QoS and Port Security directly on the hardware, I've created a solution that is centralized, tamper-proof, and industry-standard."*

*"If I had used software, it would have to be installed and managed on all 50 PCs individually, which is not scalable. Furthermore, software can be bypassed or uninstalled by a savvy user. By moving the logic to the Router and Switch, the network itself becomes the 'enforcer' - making it impossible to bypass without physical access to the server room."*

### Q3: "Why didn't you use a 48-port switch?"
**The Answer:**
*"The 2960-24TT available in Packet Tracer has 24 FastEthernet ports, which is sufficient for our simulation. In a real deployment with 50 PCs, I would use a 2960-48TT or stack multiple switches. The configuration commands are identical - I've included the Fa0/25-48 range as commented-out commands to show awareness of this. The security principles and port-security configuration don't change with the switch model."*

### Q4: "Can you show me the security violation working?"
**The Answer:**
*"Absolutely."*
Then follow the **Step 4** demo above. If you've already triggered a violation and the port is shut down, reset it first:
```
enable
configure terminal
interface FastEthernet0/1
 shutdown
 no shutdown
end
```
Then reconnect the **original PC**, ping the gateway to re-learn the MAC, and repeat the demo from Step 4.

### Q5: "What is the difference between the sticky MAC and a static MAC?"
**The Answer:**
*"A static MAC address is one that the administrator manually types into the switch configuration. A sticky MAC is dynamically learned - the switch automatically remembers the first device that connects to the port and saves it to the running configuration. Sticky MAC is more practical in a lab environment because you don't need to manually record and type all 50 MAC addresses. The switch does it automatically."*

### Q6: "Why VLAN 99 for management instead of VLAN 1?"
**The Answer:**
*"VLAN 1 is the default VLAN on all Cisco switches, and all ports are assigned to it by default. This makes it a security risk because an attacker could access management interfaces through any unconfigured port. By moving management to VLAN 99, we isolate the switch and router management traffic from regular user traffic. This is a well-known security best practice recommended by Cisco."*

---

### Tips for Success:
- **Know your Role:** You are presenting as a **Network Engineer**, not a Software Developer. Your "code" is the CLI configuration you've documented in the `configs/` folder.
- **Don't Rush:** Speak slowly and point to the diagrams in the README while you talk.
- **Know the 'Why':** If they ask why you used VLANs, say: *"To reduce broadcast traffic and increase security through isolation."*
- **The 'Red Light' is Key:** The most important part of your presentation is showing that red light in the simulation - it's your proof.
- **Own the Commented Commands:** Don't be apologetic about QoS and NetFlow being commented out. Frame it as: *"I understand the full production solution AND I understand the simulator's limitations."* This shows **more** knowledge, not less.
- **Reset Before Demo:** If you've already triggered a violation before your presentation, make sure to reset the port (shutdown / no shutdown) and reconnect the original PC before your demo starts.
