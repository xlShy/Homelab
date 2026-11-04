# Help Desk Ticket Log

## Goal
Practice documenting issues the way a help desk does, with a clear problem, steps taken, and resolution.

## Concept
IT support is judged as much on documentation as on fixes. A good ticket lets another technician understand what happened without asking. Each entry below follows the same pattern: symptom, diagnosis, resolution, and what I'd tell the user.

All tickets were worked in my home lab on an isolated VirtualBox network: an OPNsense firewall, an Ubuntu client, a Windows Server 2022 domain controller (`DC01`, domain `corp.lab`), and a Windows 11 Enterprise client (`PC01`) joined to that domain.

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

### #006: New computer cannot sign in domain users (domain join)
- **Reported issue:** A new Windows 11 PC (`PC01`) needed to join the company domain so employees could sign in with their domain accounts.
- **Troubleshooting steps:**
  1. Checked what DNS server the PC was using. It received DNS from the firewall's DHCP, and the firewall knows nothing about `corp.lab`, so the join would have failed with "domain controller could not be contacted."
  2. Pointed the PC's DNS at the domain controller (`10.10.10.10`).
  3. Verified with `nslookup corp.lab` and `nslookup dc01.corp.lab`. Both returned `10.10.10.10`.
  4. Renamed the computer to `PC01` with `Rename-Computer`.
  5. Opened `sysdm.cpl` to join the domain, but the System Properties dialog became unresponsive. Restarted the VM, which also applied the rename.
  6. Joined the domain from an elevated PowerShell prompt instead: `Add-Computer -DomainName corp.lab -Credential CORP\Administrator -Restart`.
- **Root cause:** Not a fault; this was a new-device request. The one problem along the way was the unresponsive GUI dialog, which I worked around with the command line. The key prerequisite was DNS, because a computer finds the domain controller through DNS records only the DC holds.
- **Resolution:** `PC01` joined `corp.lab`, and the sign-in screen offered the `CORP` domain. Signed in as a domain user (`msantos`). Then moved the computer account into the `PCs` OU in Active Directory so Group Policy can be applied to it.
- **Verification:** `whoami` returned `corp\msantos`, `hostname` returned `PC01`, and `PC01` appeared in the `PCs` OU in Active Directory Users and Computers.
- **Message to the user:** "Your computer is now part of the company network. Sign in with your work username and password, and tell me if you can't get in."

### #007: User account locked out
- **Reported issue:** A user, Maria Santos, could not sign in on `PC01`, even when entering her password.
- **Troubleshooting steps:**
  1. Confirmed the account was the problem, not the PC or the network, by checking the account state on the domain controller: `Get-ADUser msantos -Properties LockedOut | Select Name, LockedOut`. It returned `LockedOut: True`.
  2. Checked why. The domain's Account Lockout Policy (set in the Default Domain Policy, with a threshold of 3 invalid attempts and applied with `gpupdate /force`) locks an account after three wrong passwords, and the lockout was triggered by three failed sign-ins on `PC01`.
- **Root cause:** Three failed sign-in attempts reached the lockout threshold in Group Policy. Once an account is locked, even the correct password is rejected until it is unlocked or the lockout period ends.
- **Resolution:** Unlocked the account in Active Directory. Re-ran the same `Get-ADUser` check and it returned `LockedOut: False`, then the user signed in on `PC01` with the correct password.
- **Message to the user:** "Your account was locked after too many wrong password attempts. I've unlocked it, so please try again. If it locks again, let me know, because something may be using an old saved password."

### #008: Verify DHCP, DNS, and ping behavior with packet captures
- **Reported issue:** No fault. I wanted proof of how DHCP, DNS, and ping actually work on the LAN, so I have a known-good baseline to compare against when something breaks.
- **Troubleshooting steps:** Ran `tcpdump` on the Ubuntu client (`10.10.10.50`) and generated each kind of traffic from a second terminal.
  1. **DNS:** `sudo tcpdump -i enp0s3 -n port 53`, then `dig @10.10.10.1 fileserver.lab`. A first attempt with a packet limit (`-c 2`) captured only Ubuntu's background `connectivity-check` lookups, because the limit stopped the capture before my query ran. Removing the limit fixed it.
  2. **Ping:** `sudo tcpdump -i enp0s3 -n -c 6 icmp`, then `ping -c 3 8.8.8.8`.
  3. **DHCP:** `sudo tcpdump -i any -n -v port 67 or port 68`, then disconnected and reconnected the interface with `nmcli`. My first attempt captured nothing because the reconnect happened outside the capture window. I redid it with the capture running first.
- **Findings:**
  - **DNS:** the client sent an `A?` query for `fileserver.lab` to the firewall (`10.10.10.1`, port 53), and the firewall answered `10.10.10.50` about 1.2 ms later. The transaction ID (`36680`) matched the `dig` output, and the answer was flagged authoritative, so it came from the DNS override and not from the internet. In the background traffic, one query was sent again about 5 seconds later (a normal resolver retry) and answered immediately.
  - **Ping:** three echo requests and three echo replies, one second apart, sharing the same ID and sequence numbers 1 to 3. Reply times of about 21 ms matched the `ping` output.
  - **DHCP:** the client broadcast a Request from `0.0.0.0` asking for `10.10.10.50`, and the firewall answered with an ACK about 6 ms later. The ACK granted `10.10.10.50` with a 24-hour lease, gateway `10.10.10.1`, DNS `10.10.10.1`, and the hostname `ubuntu-client` from the DHCP reservation. There was no Discover or Offer, because the client reused its previous lease.
- **Root cause:** Not applicable; this was a verification exercise and everything behaved as designed.
- **Resolution:** Captured and documented the normal behavior of all three. A capture shows whether a request left the client and whether a reply came back, which tells me where to look when something fails. For example, requests with no replies (as in ticket #001) point to a block or a dead path, while replies with the wrong data (as in ticket #003) point to the answering service.
- **Message to the user:** Not applicable (internal verification).

### #009: Sales staff need access to the department shared folder
- **Reported issue:** Sales employees need a shared folder for department files that appears automatically on their computers, and staff from other departments (such as IT) should not see it.
- **Troubleshooting steps:**
  1. Created a folder on the domain controller (`C:\Shares\Sales`) and shared it as `Sales`.
  2. Set permissions on both layers: the share and the NTFS permissions on the folder. Granted the `GG-Sales` group Modify access, kept full control for Administrators and SYSTEM, and removed the inherited permissions so other users get nothing.
  3. Created a Group Policy Object, `Sales - Map S drive`, linked to the `Sales` OU. It uses a Drive Maps preference to map `S:` to `\\DC01\Sales`.
  4. Tested on `PC01`, signing in as a Sales user (`msantos`) and as an IT user (`areyes`).
- **Root cause:** Not a fault; this was an access request.
- **Resolution:** The shared folder now appears as **Sales Share (S:)** automatically when a Sales user signs in. Maria could open the existing welcome file and created a new file of her own, which confirms she has write access. Ana (IT) signed in on the same PC and had no `S:` drive.
- **Verification:** Screenshots of the `S:` drive with both files on Maria's session, and of Ana's This PC showing only the local disk and the CD drive.
- **Message to the user:** "Your Sales folder now appears as the S: drive when you sign in. Please save department files there instead of on your desktop. If you don't see it, sign out and back in."

## What I learned
- Firewall rules are checked top to bottom, and the logs show which rule acted on a packet.
- A DHCP reservation ties a MAC address to a fixed IP, and it is worth checking which DHCP service is actually running before changing anything.
- A name problem and a connection problem look the same to a user. Comparing `dig` (what address did DNS return?) with a ping by IP separates them.
- In Active Directory, OUs organize accounts and groups grant access. Most daily tickets are account tasks like creating users and resetting passwords.
- Joining a domain depends on DNS. The client must use the domain controller as its DNS server, and `nslookup corp.lab` is the first check when a join fails.
- When a GUI tool hangs, the same task can usually be done from PowerShell (`Rename-Computer`, `Add-Computer`).
- Computer accounts live in Active Directory like users do, and they belong in an OU you control so Group Policy can apply to them.
- Group Policy applies settings from the domain controller to users and computers. Account lockout and password rules for domain accounts must be set at the domain level, and `gpupdate /force` applies a change right away.
- A locked-out account rejects even the correct password, so the first check is the account state (`LockedOut`), not the password.
- Shared folders have two permission layers (share and NTFS), and the stricter one wins. Access goes to groups, not individual people.
- Which users get a drive letter (the Group Policy link to an OU) is a separate question from who can open the folder (group membership). Drive maps are applied at sign-in, so users must sign out and back in after a change.
- A packet capture shows what really crossed the wire. Filters and packet limits can hide the traffic I wanted, and the capture must be running before the action I want to record.
- DHCP can be just a Request and ACK when a client reuses its old lease, and the ACK carries everything the client needs: address, gateway, DNS, and lease time.
- Writing the ticket as symptom, checks, cause, fix, and message to the user forces a clear method, and it makes handing work to another technician easy.
