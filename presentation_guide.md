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
*"I configured **Quality of Service (QoS)** on the Edge Router. By setting a 2 Mbps limit per device, I've ensured that no single user can 'starve' the rest of the lab of bandwidth."*
- [**See the QoS Config Code**](configs/edge_router_config.md#L103)

**Pillar 2: Traffic Visibility (NetFlow)**
*"I enabled **NetFlow** to monitor data flows. This allows the admin to track which IP addresses are using the most data in real-time."*
- [**See the NetFlow Config Code**](configs/edge_router_config.md#L125)

**Pillar 3: Network Hardening (VLANs & Port Security)**
*"I segmented the network into **VLANs** (10 for Lab, 20 for Admin). Most importantly, I configured **Port Security with Sticky MAC** on the switches. This effectively 'locks' each port to a specific authorized machine."*
- [**See the Port Security Config Code**](configs/lab_switch_config.md#L69)
- [**See the Security Proof Diagram**](diagrams/security_violation.png)

---

## 3. The Live Demonstration (3 Minutes)
**Action:** Open `secured_network.pkt` in Packet Tracer.

**Step 1: DHCP Verification**
*"First, notice that the PCs in the Lab get an IP address from the 192.168.10.x range. This proves my DHCP pools and VLAN trunking are working correctly."*

**Step 2: The Security Test (The "Wow" Moment)**
*"Now, watch what happens if I act like an intruder. I'll unplug this PC and try to connect my own Laptop to Port Fa0/1."*
*(Perform the action in Packet Tracer)*
*"As you can see, the link light immediately turned **RED**. The switch detected the unauthorized MAC address and shut down the port. The intruder is blocked."*

---

## 4. Closing & Conclusion (1 Minute)
**What to say:**
*"By combining these professional-grade security and optimization tools, I've transformed a 'flat' insecure lab into a controlled, high-performance environment. This setup doesn't just fix the current internet speed issues—it proactively protects the network from future abuse."*

**Final Line:**
*"Thank you for your time. I am now open to any technical questions you may have about the configuration."*

---

### Tips for Success:
- **Don't Rush:** Speak slowly and point to the diagrams in the README while you talk.
- **Know the 'Why':** If they ask why you used VLANs, say: *"To reduce broadcast traffic and increase security through isolation."*
- **The 'Red Light' is Key:** The most important part of your presentation is showing that red light in the simulation—it's your proof.
