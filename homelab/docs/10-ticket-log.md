# Help Desk Ticket Log

## Goal
Practice documenting issues the way a help desk does, with a clear problem, steps taken, and resolution.

## Concept
IT support is judged as much on documentation as on fixes. A good ticket lets another technician understand what happened without asking. Each entry below follows the same pattern: symptom, diagnosis, resolution, and what I'd tell the user.

All tickets were worked in my home lab: an OPNsense firewall, an Ubuntu client, and a Windows Server 2022 domain controller (`corp.lab`) on an isolated VirtualBox network.

## Tickets

### #001: Client cannot ping external hosts
- **Reported issue:** A client on the LAN could not reach the internet by ping. `ping 8.8.8.8` showed 100% packet loss.
- **Troubleshooting steps:**
  1. `ip a` showed the client had a valid address.
  2. `ip route` showed the default gateway was `10.10.10.1`.
  3. `ping 10.10.10.1` succeeded, so the link to the firewall was fine.
  4. Checked the firewall's LAN rules and the Live View log. The log showed blocked ICMP packets tagged with a specific rule.
- **Root cause:** An ICMP block rule sat above the default "allow LAN to any" rule. Rules are checked top to bottom and the first match wins, so pings were blocked before the allow rule was reached.
- **Resolution:** Deleted the block rule and applied the change. `ping 8.8.8.8` replied at about 18 ms.
- **Message to the user:** "Ping traffic to the internet was being blocked by a firewall rule. I removed it and connectivity is back to normal."

### #002: Client needs a fixed IP address
- **Reported issue:** A lab client needed a permanent address so other systems could always find it. Its address came from a dynamic pool and could change (it had `10.10.10.157`).
- **Troubleshooting steps:**
  1. Read the client's MAC address with `ip link show enp0s3`.
  2. Found out which DHCP service was actually running. The Kea leases page was empty, and the lease appeared under Dnsmasq.
  3. Added a static host entry in Dnsmasq (client MAC, `10.10.10.50`, hostname `ubuntu-client`), outside the dynamic pool.
  4. Renewed the lease. `dhclient` was not installed, so I used `nmcli`. The first renewal still returned the old address.
  5. After making sure the entry was saved and applied and the old lease was cleared, renewed again.
- **Root cause:** Not a fault; this was a change request. The first renewal failed because the reservation had not taken effect on the running DHCP service yet.
- **Resolution:** The client now receives `10.10.10.50` every time. Confirmed with `ip a` and the Dnsmasq leases page.
- **Message to the user:** "Your machine now has a permanent address, `10.10.10.50`, so anything that points to it will keep working."

### #003: Users cannot reach `fileserver.lab` by name
- **Reported issue:** `ping fileserver.lab` failed.
- **Troubleshooting steps:**
  1. `dig fileserver.lab` returned `10.10.10.99`, which was the wrong address.
  2. `ping 10.10.10.50` by IP worked, so the machine itself was reachable.
  3. Checked the DNS host override in Unbound.
- **Root cause:** The DNS host override pointed at the wrong IP address. This was introduced on purpose to practice telling name problems apart from connectivity problems.
- **Resolution:** Corrected the override to `10.10.10.50`, applied it, and flushed the client's DNS cache with `sudo resolvectl flush-caches`. `dig` and `ping` both worked afterwards.
- **Message to the user:** "The server's name was pointing to the wrong address. I corrected it, and you can reach it by name again. If it still fails, restart your computer to clear the old answer."

### #004: New employee needs a domain account
- **Reported issue:** A new hire, Maria Santos, joining the Sales department needs a network account.
- **Troubleshooting steps:**
  1. Built the directory structure first: a `Departments` OU with `Sales` and `IT` OUs inside it.
  2. Created the user `msantos` in the `Sales` OU with a temporary password and "User must change password at next logon" ticked.
  3. Created a global security group `GG-Sales` and added the new user to it, so access can be granted to the group instead of to each person.
  4. Checked the result with `Get-ADUser` and `Get-ADGroupMember "GG-Sales"`.
- **Root cause:** Not a fault; this was an onboarding request.
- **Resolution:** Account created in the correct OU and added to the department group. The user is forced to choose a new password at first sign-in.
- **Message to the user:** "Your account is ready. Sign in with the temporary password I will send you separately, and you will be asked to choose a new one."

### #005: User forgot their password
- **Reported issue:** A user could not sign in because they forgot their password.
- **Troubleshooting steps:**
  1. Confirmed the user's identity before doing anything. On a real help desk this means following the company's verification process.
  2. Found the account in Active Directory Users and Computers.
  3. Checked that the account was enabled and not locked, so the only issue was the password.
- **Root cause:** Forgotten password.
- **Resolution:** Reset the password and ticked "User must change password at next logon", so only the user knows the final password.
- **Message to the user:** "Your password has been reset. Sign in with the temporary one and you will be prompted to set a new one."

## What I learned
- Firewall rules are checked top to bottom, and the logs show which rule acted on a packet.
- A DHCP reservation ties a MAC address to a fixed IP, and it is worth checking which DHCP service is actually running before changing anything.
- A name problem and a connection problem look the same to a user. Comparing `dig` (what address did DNS return?) with a ping by IP separates them.
- In Active Directory, OUs organize accounts and groups grant access. Most daily tickets are account tasks like creating users and resetting passwords.
- Writing the ticket as symptom, checks, cause, fix, and message to the user forces a clear method, and it makes handing work to another technician easy.
