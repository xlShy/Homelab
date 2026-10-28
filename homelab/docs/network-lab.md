# Network Support Lab: OPNsense + Ubuntu in VirtualBox

A small virtual office network I built to practice the day-to-day work of a junior network support role: configuring a firewall, handling DHCP and DNS, troubleshooting faults, and documenting each issue as a ticket.

## Goals

- Practice monitoring, maintaining, and troubleshooting a LAN and its connection to the internet.
- Configure and test firewall rules, DHCP, and DNS on a router/firewall.
- Diagnose problems methodically and document them in a ticket format (symptom, checks, cause, fix, verification, escalation).

## Lab setup

| Component | Role | Details |
|---|---|---|
| Host | VirtualBox on Windows | VMs stored on a separate hard drive |
| OPNsense 26.7 VM | Router, firewall, DHCP, DNS | 2 GB RAM, 2 CPUs, 20 GB disk |
| Ubuntu VM (full clone) | Client computer | Ubuntu 25.04, user on the LAN |

```
Internet
   |
[VirtualBox NAT]
   |  WAN: em0 (10.0.2.15/24, DHCP from VirtualBox)
OPNsense VM
   |  LAN: em1 (10.10.10.1/24)
[Internal Network "LAN-A"]
   |
Ubuntu client (10.10.10.50, reserved)
```

**Network settings**
- WAN: `em0`, NAT adapter, address from VirtualBox DHCP.
- LAN: `em1`, Internal Network named `LAN-A`, `10.10.10.0/24`.
- DHCP pool: `10.10.10.100` to `10.10.10.200` (Dnsmasq).
- DNS: Unbound on OPNsense.

## Tools and commands used

`ip a`, `ip route`, `ping`, `dig`, `nmcli`, `resolvectl`, OPNsense web GUI (Firewall rules, Live View logs, Dnsmasq, Unbound DNS).

---

## Ticket log

### #001: Client cannot ping external hosts

- **Date:** Oct 24, 2026
- **Symptom:** `ping 8.8.8.8` from the client shows 100% packet loss.
- **Steps checked:**
  1. `ip a` confirmed the client had a valid address (`10.10.10.157`).
  2. `ip route` confirmed the default gateway was `10.10.10.1`.
  3. `ping 10.10.10.1` succeeded, so the LAN link to the firewall was fine.
  4. Checked the OPNsense LAN firewall rules and Live View log.
- **Root cause:** An ICMP block rule had been placed above the default "allow LAN to any" rule. Firewall rules are evaluated top to bottom and the first match wins, so pings were blocked before the allow rule was reached.
- **Fix:** Deleted the block rule and applied the changes.
- **Verification:** `ping 8.8.8.8` replied at about 18 ms. The Live View log showed one blocked entry per ping packet while the rule was active.
- **Escalation note:** On a production firewall I would confirm who added the rule and why before removing it, and escalate to the network engineer if it was intentional.

### #002: Client needs a fixed IP address

- **Date:** Oct 25, 2026
- **Symptom / request:** The client's address came from a dynamic pool and could change (`10.10.10.157`). A fixed address was needed so other systems can find it reliably.
- **Steps checked:**
  1. Read the client's MAC address with `ip link show enp0s3`.
  2. Confirmed which DHCP service was actually running (Dnsmasq, not Kea) by checking where the lease appeared.
  3. Confirmed the reserved address was outside the dynamic pool.
- **Root cause:** None; this was a change request, not a fault. The first attempt to renew still returned the old address until the host entry was saved and applied and the old lease was cleared.
- **Fix:** Added a static host entry in Dnsmasq (client MAC, `10.10.10.50`, hostname `ubuntu-client`), applied it, and renewed the lease with `nmcli` because `dhclient` was not installed.
- **Verification:** `ip a` showed `10.10.10.50/24` on `enp0s3`, and the lease appeared in the Dnsmasq Leases page.
- **Escalation note:** None needed. In a production network, I would check with whoever manages IP planning before reserving an address.

### #003: Users cannot reach `fileserver.lab` by name

- **Date:** Oct 28, 2026
- **Symptom:** `ping fileserver.lab` failed.
- **Steps checked:**
  1. `dig fileserver.lab` returned `10.10.10.99`, which was the wrong address.
  2. `ping 10.10.10.50` by IP worked, so the machine itself was reachable.
  3. Checked the Unbound DNS host override.
- **Root cause:** The DNS host override pointed at the wrong IP address (`10.10.10.99`). This was introduced deliberately to practice separating name problems from connectivity problems.
- **Fix:** Corrected the override to `10.10.10.50`, applied it, and flushed the client's DNS cache with `sudo resolvectl flush-caches`.
- **Verification:** `dig fileserver.lab` returned `10.10.10.50` and the ping replied with 0% packet loss.
- **Notes:** The cache flush matters because the client can keep an old answer until its TTL expires (3600 seconds here). The by-IP ping in this lab was run from the same machine that owns the address, so a real investigation would run it from a different client.

### #004: Packet capture of DHCP, DNS, and ICMP (in progress)

- **Goal:** Use `tcpdump` on the client to watch the DHCP exchange (Discover, Offer, Request, ACK), a DNS query and response, and ICMP echo request/reply pairs.
- **Status:** Not completed yet. Findings to be added.

---

## What I learned

- Firewall rule order matters, and logs show which rule acted on a packet.
- DHCP reservations tie a MAC address to a fixed IP; I also learned to check which DHCP service is actually running.
- DNS problems and connectivity problems look similar to a user, but comparing a lookup (`dig`) with a ping by IP separates them.
- Modern Ubuntu uses NetworkManager, so `nmcli` replaces `dhclient` for renewing leases.
- Writing each issue as a ticket forces a clear sequence: symptom, checks, cause, fix, verification.


## Next steps

- Packet capture exercise (ticket #004).
- Add a second internal network (`LAN-B`) and write firewall rules between the two networks.
- Set up a VPN (WireGuard) and key-based SSH between VMs.
