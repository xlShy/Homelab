# Virtual Office Lab: OPNsense, Active Directory, and Clients

A small virtual office network I built to practice the day-to-day work of a junior IT and network support role: configuring a firewall, handling DHCP and DNS, running a Windows domain, supporting users, and documenting every issue as a ticket.

The full ticket write-ups are in [ticket-log.md](ticket-log.md). This page covers how the lab is built.

## Goals

- Practice monitoring, maintaining, and troubleshooting a LAN and its connection to the internet.
- Configure and test firewall rules, DHCP, and DNS on a router/firewall.
- Build a Windows Active Directory domain and handle everyday account tasks (new users, password resets, groups, joining computers).
- Diagnose problems methodically and document them in a ticket format (symptom, checks, cause, fix, verification).

## Lab setup

| Machine | Role | Details |
|---|---|---|
| Host | VirtualBox on Windows | VMs stored on a separate hard drive |
| `OPNsense` | Router, firewall, DHCP, DNS | OPNsense 26.7, 2 GB RAM, 2 CPUs, 20 GB disk |
| `DC01` | Domain controller and DNS for `corp.lab` | Windows Server 2022 Standard Evaluation, 4 GB RAM, 2 CPUs, 50 GB disk |
| `PC01` | Domain-joined client | Windows 11 Enterprise Evaluation, 4 GB RAM, 2 CPUs, 64 GB disk |
| Ubuntu client | Linux client for network tests | Full clone of my Ubuntu practice VM |

```
Internet
   |
[VirtualBox NAT]
   |  WAN: em0 (10.0.2.15/24, DHCP from VirtualBox)
OPNsense VM
   |  LAN: em1 (10.10.10.1/24)
[Internal Network "LAN-A"]
   |-- DC01          10.10.10.10 (static)   Active Directory, DNS
   |-- PC01          DHCP                   joined to corp.lab
   |-- Ubuntu client 10.10.10.50 (reserved) DHCP reservation
```

**Network settings**
- WAN: `em0`, NAT adapter, address from VirtualBox DHCP.
- LAN: `em1`, Internal Network named `LAN-A`, `10.10.10.0/24`.
- DHCP pool: `10.10.10.100` to `10.10.10.200` (Dnsmasq). Fixed addresses sit outside the pool.
- DNS: Unbound on OPNsense for the LAN, and Windows DNS on `DC01` for the `corp.lab` domain. Domain clients use `10.10.10.10` as their DNS server so they can find the domain controller.

## Active Directory structure

```
corp.lab
├── Departments
│   ├── Sales   (Maria Santos, Juan Dela Cruz, group GG-Sales)
│   └── IT      (Ana Reyes, group GG-IT)
└── PCs         (PC01)
```

- **OUs organize** accounts and computers, and are where Group Policy gets linked.
- **Security groups grant access**, so permissions go to `GG-Sales` and `GG-IT` instead of to each person.
- Computers go in the `PCs` OU, because the built-in `Computers` container cannot have Group Policy linked to it.

## Tools and commands used

- Linux: `ip a`, `ip route`, `ping`, `dig`, `nmcli`, `resolvectl`
- Windows: `ipconfig`, `nslookup`, `whoami`, `hostname`, `dcdiag`, PowerShell (`Rename-Computer`, `Add-Computer`, `Get-ADUser`, `Get-ADGroupMember`)
- OPNsense web GUI: firewall rules, Live View logs, Dnsmasq DHCP, Unbound DNS
- Active Directory Users and Computers, DNS Manager

## Tickets

| # | Issue | Area |
|---|---|---|
| 001 | Client cannot ping external hosts (ICMP block rule above the allow rule) | Firewall |
| 002 | Client needs a fixed IP address (DHCP reservation) | DHCP |
| 003 | Users cannot reach `fileserver.lab` by name (wrong DNS override) | DNS |
| 004 | New employee needs a domain account | Active Directory |
| 005 | User forgot their password | Active Directory |
| 006 | New computer needs to join the domain | Active Directory, DNS |
| 007 | Packet capture of DHCP, DNS, and ICMP (in progress) | Troubleshooting |

Full details for each are in [ticket-log.md](ticket-log.md).

## What I learned

- Firewall rule order matters, and logs show which rule acted on a packet.
- DHCP reservations tie a MAC address to a fixed IP, and it is worth checking which DHCP service is actually running before changing anything.
- A name problem and a connection problem look the same to a user. Comparing `dig` or `nslookup` with a ping by IP separates them.
- Active Directory depends on DNS. A computer finds the domain controller through DNS, so a client must use the DC as its DNS server before it can join.
- OUs organize, groups grant access, and most daily tickets are account tasks.
- When a GUI tool hangs, PowerShell can usually do the same job.

## Screenshots

Add screenshots to the `screenshots/` folder and link them here, for example:

- `screenshots/01-opnsense-lan-rules.png`: firewall rules with the test block rule above the default allow rule
- `screenshots/02-live-view-blocked-icmp.png`: Live View showing blocked ICMP
- `screenshots/03-dig-fileserver-lab.png`: `dig` and `ping` for `fileserver.lab`
- `screenshots/04-ad-users-and-computers.png`: the `corp.lab` OU structure
- `screenshots/05-group-membership.png`: `Get-ADGroupMember` output for both groups
- `screenshots/06-whoami-pc01.png`: `whoami` and `hostname` on `PC01`
- `screenshots/07-pc01-in-pcs-ou.png`: `PC01` in the `PCs` OU

## Next steps

- Packet capture exercise (ticket 007).
- Group Policy: password policy, account lockout and unlock, and a mapped drive or wallpaper policy.
- Second internal network (`LAN-B`) with firewall rules between the two networks.
- Monitoring, a WireGuard VPN, and Microsoft 365 admin practice.
