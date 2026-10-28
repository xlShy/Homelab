# DHCP Reservation in OPNsense

## Goal
Make the Ubuntu client on LAN-A always receive the same IP address (10.10.10.50) from the OPNsense DHCP server.

## Concept
**DHCP** automatically hands out IP addresses to devices on a network. A **DHCP reservation** ties an address to a device's **MAC address**, so that device gets the same IP every time while still being managed by DHCP. This is common for printers, servers, and other devices that other systems need to find reliably.

## Steps
1. Found the client's MAC address on the Ubuntu clone:
   ```bash
   ip a
   ```
   (Look for `link/ether` under the LAN interface.)
2. In OPNsense, opened the Dnsmasq DHCP settings (TODO: confirm menu path, e.g. Services > Dnsmasq DNS & DHCP > Hosts) and added a host entry:

   | Field | Value |
   |---|---|
   | Host | TODO: hostname |
   | IP address | 10.10.10.50 |
   | Hardware address | TODO: client MAC |

3. Saved and applied the changes.
4. Renewed the lease on the client (or rebooted it) and checked the address:
   ```bash
   ip a
   ```
   Output showed `10.10.10.50`. (TODO: paste the relevant line here.)

## Problems and fixes
- TODO: describe any problems (for example, the client keeping its old IP until the lease was renewed). If none, write "None encountered."

## What I learned
- TODO: one or two sentences.
