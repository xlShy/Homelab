# Linux Commands

## Goal
Get comfortable using the Ubuntu command line for everyday administration.

## Concept
Servers usually have no graphical desktop, so administrators work almost entirely in the terminal. Knowing these commands is a core skill for IT support and system administration.

## Commands I used

| Command | What it does | Example |
|---|---|---|
| `ls`, `cd`, `pwd` | List files, change folder, show current folder | `cd /etc/nginx` |
| `sudo` | Run a command with administrator rights | `sudo apt update` |
| `apt` | Install and update software | `sudo apt install nginx` |
| `systemctl` | Start, stop, and check services | `systemctl status nginx` |
| `ufw` | Manage the firewall | `sudo ufw status` |
| `ip a` | Show network interfaces and IP addresses | `ip a` |
| `ping` | Test connectivity | `ping 8.8.8.8` |
| `journalctl` | Read service logs | `journalctl -u webapp` |

TODO: delete any command above you did not actually use, and add others you practiced (for example `chmod`, `cat`, `nano`, `grep`).

## Problems and fixes
- TODO: describe any mistakes or errors you ran into, such as "permission denied" and how you fixed them.

## What I learned
- TODO: one or two sentences.
