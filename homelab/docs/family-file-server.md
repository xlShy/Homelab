# Family File Server (Samba on Ubuntu Server + Tailscale)

## Goal
Build a private file server so my family can store photos in one place,
and reach it from Windows PCs and phones, at home and remotely.

## Environment
- Ubuntu Server 26.04 LTS VM in VirtualBox (2 GB RAM, 2 vCPU, dynamically allocated disk)
- Samba for SMB file sharing
- Tailscale for secure remote access

## What I built
1. **VM and network:** installed Ubuntu Server with OpenSSH. Used a **bridged**
   adapter instead of NAT so the VM gets its own address on the home network
   and other devices can reach it.
2. **Permissions design:** created a `familyphotos` group, a `/srv/samba/photos`
   folder owned by that group with mode `2770` (the setgid bit makes new files
   inherit the group, so one person's uploads can be edited by everyone),
   and no-login Linux accounts for each family member.
3. **Samba share:** configured a `[Photos]` share restricted to that group, with
   `create mask = 0660`, `directory mask = 2770` and `force group` so permissions
   stay consistent for files created over the network. Validated the config
   with `testparm`.
4. **Clients:** mapped the share as a network drive on Windows and connected from
   an Android file manager.
5. **Remote access:** installed Tailscale on the server and client devices so the
   share works from outside the home network. Disabled key expiry on the server
   so it doesn't drop off the network.

## Security decisions
- Did **not** forward port 445 to the internet. Exposed SMB is a common attack
  target, so I used a VPN instead.
- Family accounts have no shell login and are used only for file sharing.
- Changed the default account password, and kept the share limited to one group.

## Troubleshooting log
| Problem | Cause | Fix |
|---|---|---|
| "Network path not found" over VPN | Used the VM's LAN address from outside the home | Use the Tailscale address instead |
| Windows: "An extended error has occurred" when mapping the drive | "Connect using different credentials" was off, so Windows sent its own login | Enabled it and logged in as `fileserver\username` |
| `Permission denied` right after adding myself to the group | Group membership is only read at login | Logged out and back in |
| Group member list didn't show the family accounts | They had it as their *primary* group, which `getent` doesn't list | Verified with `id` and a test file instead |
| New file from another user was `rw-r--r--` | Default umask | Samba masks apply the right permissions for files created over SMB |

## What I learned
- How Linux users, groups, and the setgid bit control shared access
- Why bridged networking matters for a VM that other devices must reach
- Reading Windows SMB errors and testing from both ends (`smbclient`, `net use`)
- Why a VPN is safer than port forwarding

## Still to do
- Scheduled backup to a second drive
- DHCP reservation on the router for a fixed address
- Limited access for relatives outside the household (separate account and Tailscale device sharing)
