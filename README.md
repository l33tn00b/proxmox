# proxmox
Proxmox HomeLab

# What
Proxmox VE Homelab server, mostly for virtualizing a GPU (Tesla P4).

# How
Got some old hardware, had to use it. 

# Steps
## HW build
- MSI Motherboard, toggle to IGP (onboard graphics) in Bios
- Insert GPU
  
## install base Debian12
NSTR, just
- uncheck graphical environment,
- check SSH
- use UEFI boot with GRUB

## install PVE on top
- along the lines of https://pve.proxmox.com/wiki/Install_Proxmox_VE_on_Debian_12_Bookworm
- make sure to set static IP in `/etc/hosts` (you've been told.. else installation of pve-manager package will fail)
- after stock kernel uninstall, you're told to run `update-grub`. This will fail because `/usr/sbin` is not in path. include it via ```export PATH=$PATH:/usr/sbin```
- networking won't work after reboot. so do a manual edit of `/etc/network/interfaces` (adjust to match your ethernet interface name):
  ```
  auto enp4s0
  iface enp4s0 inet manual
  auto vmbr0
    iface vmbr0 inet static
    address <ip cidr>
    gateway <gateway ip>
    bridge-ports enp4s0
    bridge-stp off
    bridge-fd 0
  ```

## GPU virtualization
Follow https://gitlab.com/polloloco/vgpu-proxmox


