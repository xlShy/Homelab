# Homelab

A VirtualBox lab on my Windows PC where I practice Linux administration, web hosting, databases, and basic networking. I built it to get hands-on experience for IT support and network support roles.

## Environment

| Item | Details |
|---|---|
| Host | Windows PC |
| Hypervisor | VirtualBox |
| Main VM | Ubuntu, with Guest Additions 7.2.20 |
| Firewall VM | OPNsense |
| Client VM | Cloned Ubuntu VM on an internal network (LAN-A) |

## Projects

| # | Project | What it shows |
|---|---|---|
| 1 | [VirtualBox and Ubuntu VM setup](01-virtualbox-ubuntu-setup.md) | Virtualization, OS installation |
| 2 | [Linux commands](02-linux-commands.md) | Command-line basics |
| 3 | [Nginx and port forwarding](03-nginx-port-forwarding.md) | Web server, NAT port forwarding |
| 4 | [SSH and ufw firewall](04-ssh-and-ufw.md) | Remote access, host firewall |
| 5 | [MariaDB setup](05-mariadb-setup.md) | Databases, least-privilege users |
| 6 | [OPNsense two-network lab](07-opnsense-lab.md) | Network firewall rules, traffic monitoring |
| 7 | [DHCP reservation](08-dhcp-reservation.md) | DHCP service, fixed IPs for lab machines |
| 8 | [DNS host override](09-dns-host-override.md) | Local DNS, name resolution, troubleshooting a broken config |
| 9 | [Ticket log](10-ticket-log.md) | Documenting issues, cause, and fix like a help desk ticket |
| 10 | [Network lab](network-lab.md) | Networking, VirtualBox internal networks |

## Skills demonstrated

- Linux (Ubuntu) administration from the command line
- Installing and configuring services (Nginx, MariaDB, OpenSSH)
- Host and network firewalls (ufw, OPNsense)
- NAT port forwarding and internal networks in VirtualBox
- DHCP reservations and DNS host overrides
- Managing services with systemd
- Troubleshooting and documenting my work in a ticket log
