# OPNsense Two-Network Lab

## Goal
Build a small network with a firewall/router and a client machine, then practice writing firewall rules and watching traffic.

## Concept
**OPNsense** is a firewall and router. In this lab, it sits between the internet and a private network (LAN-A). A **VirtualBox Internal Network** is a virtual switch that only connects the VMs I attach to it, so I can test network changes safely. Firewall rules decide what traffic is allowed, and **Live View** shows traffic being allowed or blocked in real time.

## Lab layout

| Machine | Role | Network |
|---|---|---|
| OPNsense VM | Firewall/router | WAN: NAT, LAN: Internal Network `LAN-A` |
| Ubuntu client | Test client (full clone of my Ubuntu VM) | Internal Network `LAN-A` |

## Steps
1. Created the OPNsense VM (stored on my E: drive) and installed OPNsense (TODO: version).
2. Set up two network adapters: one NAT for the WAN and one Internal Network named `LAN-A` for the LAN.
3. Made a **full clone** of my Ubuntu VM and attached it to `LAN-A` as the client.
4. Confirmed the client could reach the network (TODO: how it got its IP, and what you pinged).
5. **Exercise: block ICMP (ping)**
   - Added a firewall rule on the LAN interface: action **Block**, protocol **ICMP** (TODO: confirm exact settings).
   - Pinged from the client and the pings failed.
   - Watched the blocked traffic appear in **Live View**.
   - Removed the rule, applied the changes, and ping worked again.

![Block rule in OPNsense](../screenshots/07-opnsense-rule.png)
![Live View showing blocked ICMP](../screenshots/07-opnsense-liveview.png)

## Problems and fixes
- TODO: describe any problems (for example, the client not getting an IP, or a rule not taking effect until applied). If none, write "None encountered."

## What I learned
- TODO: one or two sentences. For example: how rule order and applying changes affect firewall behavior.
