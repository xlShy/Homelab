# Family File Server (Samba on Ubuntu Server + Tailscale)

## Goal
Build a private file server so my family can store photos in one place,
and reach it from Windows PCs and phones, at home and remotely.

## Environment
- Ubuntu Server 26.04 LTS VM in VirtualBox (2 GB RAM, 2 vCPU, dynamically allocated disk)
- Samba for SMB file sharing
- Tailscale for secure remote access
- A separate Windows laptop as the backup machine (robocopy + Task Scheduler)

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
6. **Automatic backup:** a separate Windows laptop **pulls** the photos every night
   with `robocopy`, run by a Task Scheduler job over Tailscale. It uses a dedicated
   read-only Samba account (`read list` in the share), and I tested that write
   attempts fail with `NT_STATUS_ACCESS_DENIED`. I left out `/MIR`, so files deleted
   from the server stay in the backup. I also ran a full restore test: created a file,
   backed it up, deleted it from the share, confirmed it survived in the backup, then
   copied it back with a writable account.

## Security decisions
- Did **not** forward port 445 to the internet. Exposed SMB is a common attack
  target, so I used a VPN instead.
- Family accounts have no shell login and are used only for file sharing.
- Changed the default account password, and kept the share limited to one group.
- The backup is **pulled** by a separate machine with a **read-only** account, so a
  problem on the server (or ransomware) can't reach in and delete the backup, and the
  backup account's saved password can't be used to change the photos.

## Troubleshooting log
| Problem | Cause | Fix |
|---|---|---|
| "Network path not found" over VPN | Used the VM's LAN address from outside the home | Use the Tailscale address instead |
| Windows: "An extended error has occurred" when mapping the drive | "Connect using different credentials" was off, so Windows sent its own login | Enabled it and logged in as `fileserver\username` |
| `Permission denied` right after adding myself to the group | Group membership is only read at login | Logged out and back in |
| Group member list didn't show the family accounts | They had it as their *primary* group, which `getent` doesn't list | Verified with `id` and a test file instead |
| New file from another user was `rw-r--r--` | Default umask | Samba masks apply the right permissions for files created over SMB |
| `smbclient: command not found` when testing the backup account | The client package wasn't installed | `sudo apt install smbclient`, then repeated the tests |
| Scheduled backup task returned exit code 16, with `ERROR 3` (path not found) in the log | The source path had one leading backslash (`\server\share`) instead of two, so Windows treated it as a local path; `net use` gave error 67 and `Test-Path` returned `False` | Built the path from variables, validated both paths with `Test-Path` before registering the task, then re-created it |
| `Start-ScheduledTask` said it could not find the file | The task didn't exist after I had deleted it | Re-registered the task and confirmed it with `Get-ScheduledTask` |

## What I learned
- How Linux users, groups, and the setgid bit control shared access
- Why bridged networking matters for a VM that other devices must reach
- Reading Windows SMB errors and testing from both ends (`smbclient`, `net use`)
- Why a VPN is safer than port forwarding
- How to design a backup: a separate machine, pull-based, read-only account, and a
  restore test (a backup you've never restored from is only a guess)
- `robocopy` basics and its exit codes (0-7 mean success, 8 or higher means errors),
  and why Task Scheduler showing `1` isn't a failure
- Validating inputs (`Test-Path`) before creating a scheduled job, and reading logs to
  find the real error behind a generic failure code

## Still to do
- DHCP reservation on the router for a fixed address
- Limited access for relatives outside the household (separate account and Tailscale device sharing)
