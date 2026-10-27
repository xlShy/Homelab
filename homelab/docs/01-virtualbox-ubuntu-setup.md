# VirtualBox and Ubuntu VM Setup

## Goal
Create a virtual machine running Ubuntu so I have a safe place to practice Linux without affecting my main PC.

## Concept
A **hypervisor** (VirtualBox) lets one physical computer run other operating systems as "virtual machines." If I break a VM, I can fix or rebuild it, and my real system stays untouched. **Guest Additions** are drivers installed inside the VM that add features like screen resizing and copy/paste between host and VM.

## Steps
1. Downloaded and installed VirtualBox from virtualbox.org.
2. Downloaded the Ubuntu ISO (TODO: version) from ubuntu.com.
3. Created a new VM in VirtualBox (TODO: RAM, CPU cores, disk size) and attached the ISO.
4. Started the VM and completed the Ubuntu installation.
5. Installed Guest Additions 7.2.20:
   ```bash
   sudo apt update
   sudo apt install build-essential dkms linux-headers-$(uname -r)
   ```
   Then in the VM window: **Devices > Insert Guest Additions CD image**, and ran the installer from the mounted disc (TODO: confirm exact command used). Rebooted the VM.

![VM running Ubuntu](../screenshots/01-ubuntu-desktop.png)

## Problems and fixes
- TODO: describe any errors you hit (for example, the Guest Additions install failing, or the VM not booting). If none, write "None encountered."

## What I learned
- TODO: one or two sentences.
