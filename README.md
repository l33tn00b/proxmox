# proxmox
Proxmox HomeLab (?)

# What
Proxmox VE Homelab server.   
Mostly for virtualizing a GPU (Tesla P4) to run speech to text (whispernet (willow)). Is picovoice available locally? This is interesting, too: https://github.com/duhow/xiaoai-patch

Maybe we'll run some other stuff on top (HomeAssistant?).
Totally overblown. Let's do it anyway.

# How
Got some old hardware, had to use it. Started out with:
- Mainboard MSI HM81-P33 (which turned out to be a dud, see below)
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

Ended up tossing the H81 Board in favour of a Q87 board:
```
lspci
00:00.0 Host bridge: Intel Corporation 4th Gen Core Processor DRAM Controller (rev 06)
00:01.0 PCI bridge: Intel Corporation Xeon E3-1200 v3/4th Gen Core Processor PCI Express x16 Controller (rev 06)
00:02.0 VGA compatible controller: Intel Corporation 4th Generation Core Processor Family Integrated Graphics Controller (rev 06)
00:03.0 Audio device: Intel Corporation Xeon E3-1200 v3/4th Gen Core Processor HD Audio Controller (rev 06)
00:14.0 USB controller: Intel Corporation 8 Series/C220 Series Chipset Family USB xHCI (rev 04)
00:16.0 Communication controller: Intel Corporation 8 Series/C220 Series Chipset Family MEI Controller #1 (rev 04)
00:16.3 Serial controller: Intel Corporation 8 Series/C220 Series Chipset Family KT Controller (rev 04)
00:19.0 Ethernet controller: Intel Corporation Ethernet Connection I217-LM (rev 04)
00:1a.0 USB controller: Intel Corporation 8 Series/C220 Series Chipset Family USB EHCI #2 (rev 04)
00:1b.0 Audio device: Intel Corporation 8 Series/C220 Series Chipset High Definition Audio Controller (rev 04)
00:1c.0 PCI bridge: Intel Corporation 8 Series/C220 Series Chipset Family PCI Express Root Port #1 (rev d4)
00:1c.5 PCI bridge: Intel Corporation 8 Series/C220 Series Chipset Family PCI Express Root Port #6 (rev d4)
00:1d.0 USB controller: Intel Corporation 8 Series/C220 Series Chipset Family USB EHCI #1 (rev 04)
00:1f.0 ISA bridge: Intel Corporation Q87 Express LPC Controller (rev 04)
00:1f.2 SATA controller: Intel Corporation 8 Series/C220 Series Chipset Family 6-port SATA Controller 1 [AHCI mode] (rev 04)
00:1f.3 SMBus: Intel Corporation 8 Series/C220 Series Chipset Family SMBus Controller (rev 04)
01:00.0 3D controller: NVIDIA Corporation GP104GL [Tesla P4] (rev a1)
03:00.0 PCI bridge: Integrated Technology Express, Inc. IT8892E PCIe to PCI Bridge (rev 41)
```

# Power consumption
Above setup (core i3, HM81, SSD, Tesla P4, 8GB RAM, Xilence Power Supply) will draw ~43 Watts when idle. Makes for 1,08 kWh per day. In total 394 kWh per year. Electricity is about 30 cents/kWh. So in total 131€ per year.

Changed setup with Q87 board draws ~39 Watts at idle.

There's builds that draw much less idle power. But these come at a hardware cost (remember? got it for free). I figured break even would be after approx. seven years. So I don't even start thinking about that. And another point: It's not like that electricity will be wasted. We're inside (at home) and thus heating the house. So it's kinda like electrons having been used for running the server are not waste heat. 

# Steps
## HW build
- Sidenote: Updating the BIOS:
  - MSI tell us to put the entire unzipped folder in the root dir of of the flash drive.
  - This is not the right way. You need to put the files (not the entire folder) in the root dir of the flash drive.
  - DO THE BIOS UPDATE BEFORE INSTALLING THE OS (else you'll have to re-write your UEFI Boot loader (painful))
- MSI Motherboard, toggle to IGP (onboard graphics) in BIOS.
- Insert GPU (only after BIOS update because this will revert the settings to default (which is external GPU)).

  
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

# HM81 is a dud
So I tried to activate IOMMU for passing through the virtualized GPUs. That didn't work. 
Even when making sure I had activated the proper boot parameters (grub command line), dmesg would only show "IOMMU enabled". But that was it. No working IOMMU. See https://vfio.blogspot.com/2016/09/intel-iommu-enabled-it-doesnt-mean-what.html for an explanation of the message. At least we managed to get the boot parameters right :P  
Then I tried updating the BIOS (which took me on a lengthy journey to re-install the efi bootloader because all settings were deleted:
- see https://emmanuel-galindo.github.io/en/2017/04/05/fixing-debian-boot-uefi-grub
- but the link to the efi shell file is broken. Get it from https://github.com/tianocore/edk2/blob/edk2-stable201903/ShellBinPkg/UefiShell/X64/Shell.efi .
- change the bash for loop for mounting:  
  ```for i in /dev /dev/pts /proc /sys /sys/firmware/efi/efivars /run; do sudo mount -B $i /mnt$i; done```  
  else you'll see a message when doing grub-install complaining about missing efivars. 
- And make sure, you're booting in uefi mode from that drive (set it in BIOS) (else you won't be able to set efi boot parameters from the live system?)
  
Anyway, proxmox was back on track. And no change in being able to activate IOMMU in the BIOS. Back to the reading table. Dooh. HM81 is a stripped down consumer product with the IO virtualization part deactivated. So there is no way of activating IOMMU in the operating system because the chipset doesn't support it. I'll probably pass through the entire GPU non-virtualized... hell, needs IOMMU, too. But docker container without promox might be a solution? Well, no. Just got another old board with Q87 chipset.

Some more learning on the way:
- change keyboard layout
  - Debian doc says ```dpkg-reconfigure keyboard-configuration``` and ```service reload keyboard-configuration```. Insufficient. Must call ```setupcon``` after.
 
# Using Intel QT87 Board
Switched boards, saw a reduction in idle power from those 43 Watts (MSI H81 board) to 39 Watts.
Guess what, had to re-install the UEFI boot code.
