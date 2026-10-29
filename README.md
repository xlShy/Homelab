# Homelab

A VirtualBox lab on my Windows PC where I practice Linux administration, web hosting, databases, and basic networking. I built it to get hands-on experience for IT support and network support roles.

## Environment

| Item | Details |
|---|---|
| Host | Windows PC |
| Hypervisor | VirtualBox |
| Main VM | Ubuntu, with Guest Additions 7.2.20 |
| Firewall VM | OPNsense |

## Projects

| # | Project | What it shows |
|---|---|---|
| 1 | [VirtualBox and Ubuntu VM setup](homelab/docs/01-virtualbox-ubuntu-setup.md) | Virtualization, OS installation |
| 2 | [Linux commands](homelab/docs/02-linux-commands.md) | Command-line basics |
| 3 | [Nginx and port forwarding](homelab/docs/03-nginx-port-forwarding.md) | Web server, NAT port forwarding |
| 4 | [SSH and ufw firewall](homelab/docs/04-ssh-and-ufw.md) | Remote access, host firewall |
| 5 | [MariaDB setup](homelab/docs/05-mariadb-setup.md) | Databases, least-privilege users |
| 6 | [Flask web app](homelab/docs/06-flask-webapp.md) | systemd services, reverse proxy |
| 7 | [OPNsense two-network lab](homelab/docs/07-opnsense-lab.md) | Network firewall rules, traffic monitoring |
| 8 | [DHCP reservation](homelab/docs/08-dhcp-reservation.md) | DHCP, static addressing by MAC |
| 9 | [DNS host override](homelab/docs/09-dns-host-override.md) | DNS, break/fix troubleshooting |
| 10 | [Ticket log](homelab/docs/10-ticket-log.md) | Help desk documentation |

## Skills demonstrated

- Linux (Ubuntu) administration from the command line
- Installing and configuring services (Nginx, MariaDB, OpenSSH)
- Host and network firewalls (ufw, OPNsense)
- NAT port forwarding and internal networks in VirtualBox
- DHCP reservations and local DNS records
- Managing services with systemd
- Troubleshooting, ticket-style documentation, and documenting my work
