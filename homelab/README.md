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
| 1 | [VirtualBox and Ubuntu VM setup](docs/01-virtualbox-ubuntu-setup.md) | Virtualization, OS installation |
| 2 | [Linux commands](docs/02-linux-commands.md) | Command-line basics |
| 3 | [Nginx and port forwarding](docs/03-nginx-port-forwarding.md) | Web server, NAT port forwarding |
| 4 | [SSH and ufw firewall](docs/04-ssh-and-ufw.md) | Remote access, host firewall |
| 5 | [MariaDB setup](docs/05-mariadb-setup.md) | Databases, least-privilege users |
| 6 | [Flask web app](docs/06-flask-webapp.md) | systemd services, reverse proxy |
| 7 | [OPNsense two-network lab](docs/07-opnsense-lab.md) | Network firewall rules, traffic monitoring |
| 8 | [DHCP reservation and DNS override](docs/08-dhcp-dns.md) | DHCP and DNS services, fixed IPs, troubleshooting a broken config |
| 9 | [Ticket log](docs/09-ticket-log.md) | Documenting issues, cause, and fix like a help desk ticket |

## Skills demonstrated

- Linux (Ubuntu) administration from the command line
- Installing and configuring services (Nginx, MariaDB, OpenSSH)
- Host and network firewalls (ufw, OPNsense)
- NAT port forwarding and internal networks in VirtualBox
- DHCP reservations and DNS host overrides
- Managing services with systemd
- Troubleshooting and documenting my work in a ticket log
