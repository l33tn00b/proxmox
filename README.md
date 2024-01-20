# proxmox
Proxmox HomeLab (?)

# What
Proxmox VE Homelab server.   
Mostly for virtualizing a GPU (Tesla P4) to run whispernet (speech recognition) (willow)
Maybe we'll run some other stuff on top (HomeAssistant?).
Totally overblown. Let's do it anyway.

# How
Got some old hardware, had to use it. 
- Maiboard MSI HM81-P33
- ATX Power supply, Xilence Office 450W
- RAM 8GB
- SSD (Crucial, 250GB)
- Tesla P4 (greetings from an old mining rig in China).

New tower case...

```
lscpu
Architecture:            x86_64
  CPU op-mode(s):        32-bit, 64-bit
  Address sizes:         39 bits physical, 48 bits virtual
  Byte Order:            Little Endian
CPU(s):                  4
  On-line CPU(s) list:   0-3
Vendor ID:               GenuineIntel
  BIOS Vendor ID:        Intel
  Model name:            Intel(R) Core(TM) i3-4130 CPU @ 3.40GHz
    BIOS Model name:     Intel(R) Core(TM) i3-4130 CPU @ 3.40GHz Fill By OEM CPU @ 3.4GHz
    BIOS CPU family:     206
    CPU family:          6
    Model:               60
    Thread(s) per core:  2
    Core(s) per socket:  2
    Socket(s):           1
    Stepping:            3
    CPU(s) scaling MHz:  99%
    CPU max MHz:         3400,0000
    CPU min MHz:         800,0000
    BogoMIPS:            6800,39
```

```
lspci
00:00.0 Host bridge: Intel Corporation 4th Gen Core Processor DRAM Controller (rev 06)
00:01.0 PCI bridge: Intel Corporation Xeon E3-1200 v3/4th Gen Core Processor PCI Express x16 Controller (rev 06)
00:02.0 VGA compatible controller: Intel Corporation 4th Generation Core Processor Family Integrated Graphics Controller (rev 06)
00:14.0 USB controller: Intel Corporation 8 Series/C220 Series Chipset Family USB xHCI (rev 05)
00:16.0 Communication controller: Intel Corporation 8 Series/C220 Series Chipset Family MEI Controller #1 (rev 04)
00:1a.0 USB controller: Intel Corporation 8 Series/C220 Series Chipset Family USB EHCI #2 (rev 05)
00:1b.0 Audio device: Intel Corporation 8 Series/C220 Series Chipset High Definition Audio Controller (rev 05)
00:1c.0 PCI bridge: Intel Corporation 8 Series/C220 Series Chipset Family PCI Express Root Port #1 (rev d5)
00:1c.3 PCI bridge: Intel Corporation 8 Series/C220 Series Chipset Family PCI Express Root Port #4 (rev d5)
00:1c.4 PCI bridge: Intel Corporation 8 Series/C220 Series Chipset Family PCI Express Root Port #5 (rev d5)
00:1c.5 PCI bridge: Intel Corporation 82801 PCI Bridge (rev d5)
00:1d.0 USB controller: Intel Corporation 8 Series/C220 Series Chipset Family USB EHCI #1 (rev 05)
00:1f.0 ISA bridge: Intel Corporation H81 Express LPC Controller (rev 05)
00:1f.2 SATA controller: Intel Corporation 8 Series/C220 Series Chipset Family 6-port SATA Controller 1 [AHCI mode] (rev 05)
00:1f.3 SMBus: Intel Corporation 8 Series/C220 Series Chipset Family SMBus Controller (rev 05)
01:00.0 3D controller: NVIDIA Corporation GP104GL [Tesla P4] (rev a1)
03:00.0 USB controller: Renesas Technology Corp. uPD720202 USB 3.0 Host Controller (rev 02)
04:00.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL8111/8168/8411 PCI Express Gigabit Ethernet Controller (rev 0c)
05:00.0 PCI bridge: ASMedia Technology Inc. ASM1083/1085 PCIe to PCI Bridge (rev 03)
```

# Steps
## HW build
- MSI Motherboard, toggle to IGP (onboard graphics) in BIOS.
- Insert GPU.
  
## Install base Debian 12
NSTR, just
- uncheck graphical environment,
- check SSH
- use UEFI boot with GRUB

Base Installation complete?
- install mc
- install unattended-upgrades (yes, might break things. but rather broken because of good security than because of having been taken over...)

## Install PVE on top
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
- get rid of enterprise edition nag (bookworm is debian 12): echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" >> /etc/apt/sources.list
rm /etc/apt/sources.list.d/pve-enterprise.list

## GPU virtualization
- Follow https://gitlab.com/polloloco/vgpu-proxmox
  - maybe ask someone for ```NVIDIA-GRID-Linux-KVM-535.129.03-537.70.zip``` if you can't cough up an enterprisely-looking email address
- After having docker runner up (see below, Basic VMs): Set Up Delegated License Server (link from above git): https://git.collinwebdesigns.de/oscar.krause/fastapi-dls
  - be on dokcer runner...
  - make sure compose is installed (```apt install docker-compose-plugin```)
  - create ```compose.yml``` in a nice place (your home dir?), choosing 8443 as host port:
    ```
    version: '3.9'

    x-dls-variables: &dls-variables
      TZ: Europe/Berlin # REQUIRED, set your timezone correctly on fastapi-dls AND YOUR CLIENTS !!!
      DLS_URL: <your IP> # REQUIRED, change to your ip or hostname
      DLS_PORT: 8443
      LEASE_EXPIRE_DAYS: 90  # 90 days is maximum
      DATABASE: sqlite:////app/database/db.sqlite
      DEBUG: false
    
    services:
      dls:
        image: collinwebdesigns/fastapi-dls:latest
        restart: always
        environment:
          <<: *dls-variables
        ports:
          - "8443:443"
        volumes:
          - /opt/docker/fastapi-dls/cert:/app/cert
          - dls-db:/app/database
        logging:  # optional, for those who do not need logs
          driver: "json-file"
          options:
            max-file: 5
            max-size: 10m
    
    volumes:
      dls-db:
    ```
  - as root: ```docker compose up --detach```
- go back to proxmox server:
  - maybe run nmap against the docker host to see if 8443 is open: ```nmap -v -sT <ip>```
 

## Add ZFS RAID for VM Data
- to do

# Some Basic VMs
## Set up VM for running docker containers 
Used for
- Willow application Server
- Delegated License Server (vGPU)

HW config:
- 32GB Disk
- 1 Core
- 2 GB RAM
- vmbr0, no firewall


Install:
- Base Image Debian 12
- install mc
- install unattended-upgrades
- install docker according to: https://docs.docker.com/engine/install/debian/
  - just be root and leave out all the sudos (yay, running curl as root)
  
## Set Up VM for Willow Inference Server
No container available, yet
HW config:
- 32GB Disk
- 1 Core
- 2 GB RAM
- vmbr0, no firewall

Install:
- Base Image Debian 12
- install mc
- install unattended-upgrades
- install curl
- copy over (scp) grid drivers
- unzip
- go to subdir guest_drivers
- ```dpgk -i nvidia-linux-grid-535_535.129.03_amd64.deb```will fail
- run ```apt --fix-broken install```
- get license token, accomodate for changed port (8443): ```curl --insecure -L -X GET https://<docker runner ip>:8443/-/client-token -o /etc/nvidia/ClientConfigToken/client_configuration_token_$(date '+%d-%m-%Y-%H-%M-%S').tok --create-dirs```

# Remarks concerning willow
- AutoCorrect Fork for handing misunderstood/non-recognized commands off to alexa: https://github.com/kovrom/willow-autocorrect
- Inference Server Dashboard for Home Assistant: https://github.com/jazzmonger/Willow-Dashboard
- 
