# SSH and ufw Firewall

## Goal
Connect to the VM remotely from Windows PowerShell and protect it with a firewall.

## Concept
**SSH** gives secure command-line access to a machine over the network, which is how most servers are managed. **ufw** (Uncomplicated Firewall) controls which network ports are allowed in. A good rule is to allow only what you need and block everything else.

## Steps
1. Installed the SSH server:
   ```bash
   sudo apt install openssh-server
   ```
2. Set up the firewall rules **before** enabling it, so I wouldn't lock myself out:
   ```bash
   sudo ufw allow 'Nginx Full'
   sudo ufw allow OpenSSH
   sudo ufw enable
   sudo ufw status
   ```
3. Added a VirtualBox NAT port forwarding rule: Host Port 2222 to Guest Port 22.
4. Connected from Windows PowerShell:
   ```powershell
   ssh -p 2222 <username>@127.0.0.1
   ```

## Problems and fixes
None encountered, yet..
