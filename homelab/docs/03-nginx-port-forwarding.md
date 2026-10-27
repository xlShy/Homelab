# Nginx and Port Forwarding

## Goal
Run a web server on the Ubuntu VM and open its page from my Windows browser.

## Concept
**Nginx** is a web server that serves web pages. By default, a VM using **NAT** networking is hidden behind the host, so the host can't reach services inside it. **Port forwarding** fixes this by sending traffic that arrives at a port on the host (8080) to a port on the VM (80).

## Steps
1. Installed Nginx on the VM:
   ```bash
   sudo apt update
   sudo apt install nginx
   ```
2. Checked that it was running:
   ```bash
   systemctl status nginx
   curl http://localhost
   ```
3. In VirtualBox, opened **VM Settings > Network > Adapter 1 (NAT) > Advanced > Port Forwarding** and added a rule:

   | Name | Protocol | Host Port | Guest Port |
   |---|---|---|---|
   | HTTP | TCP | 8080 | 80 |

4. Opened `http://localhost:8080` in my Windows browser and the Nginx welcome page loaded.

![Nginx page in browser](../screenshots/03-nginx-localhost-8080.png)

## Problems and fixes
- TODO: describe any problems (for example, the page not loading until the rule or firewall was fixed). If none, write "None encountered."

## What I learned
- TODO: one or two sentences.
