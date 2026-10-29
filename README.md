# Kali Linux Lab Setup on MacBook Air M2 (Phase 1)

This repository documents Phase 1 of my Cyber Security & Ethical Hacking practice lab.

## Environment

- **Host:** MacBook Air M2 (Apple Silicon)
- **Virtualization:** UTM (VirtualBox is not suitable for Apple Silicon)
- **Guest OS:** Kali Linux ARM64
- **Network mode:** Shared Network (default subnet 192.168.64.0/24)

## Adapting the guide to Apple Silicon

| Original step | What I did on Mac M2 |
|---|---|
| Install 7-Zip | Not needed on macOS |
| Install VirtualBox | Replaced by UTM, which supports Apple Silicon |
| NATNetwork 10.0.0.0/24 | Used UTM Shared Network (192.168.64.0/24) |
| Import Kali VM | Installed the ARM64 build of Kali in UTM |
| Static IP for Kali | Configured with nmcli (see below) |
| Take a snapshot | Cloned the VM, because UTM does not support snapshots for Apple Virtualization VMs |

## Step 1: Network check

I confirmed that Kali had internet access through the host.

    ip route
    ping -c 4 google.com

Result: 0% packet loss.

![Ping test](01-ping-test.png)

## Step 2: Static IP configuration

Kali first received an IP by DHCP (192.168.64.4). I set a fixed IP so the lab address does not change between reboots.

    nmcli con show
    sudo nmcli con mod "Wired connection 1" ipv4.method manual ipv4.addresses 192.168.64.10/24 ipv4.gateway 192.168.64.1 ipv4.dns 8.8.8.8
    sudo nmcli con up "Wired connection 1"
    ip a show eth0

Result: eth0 now shows 192.168.64.10/24.

![Static IP](02-static-ip.png)

## Step 3: Clipboard sharing (copy and paste)

Copy and paste between macOS and Kali did not work at first. I fixed it by:

1. Enabling clipboard sharing in the UTM VM settings.
2. Installing and starting the SPICE agent inside Kali:
3. 
In the Kali terminal, paste with Ctrl + Shift + V.

![Clipboard setting](03-clipboard-setting.png)

## Step 4: Backup of the clean VM

UTM cannot take snapshots for this type of VM, so I shut Kali down and cloned it. The clone `Kali-clean-base` is kept untouched so I can restore a working Kali at any time.

![Clone backup](04-clone-backup.png)

## Problems I ran into and how I fixed them

- **Error: unknown connection:** the connection name is case-sensitive. It must be `Wired connection 1` with a capital W.
- **Error: failed to modify ipv4.method:** a backslash was typed instead of a space between options. The command must be on one line with spaces.
- **Copy and paste not working:** the SPICE agent service was installed but not started. Fixed with `systemctl enable --now spice-vdagentd`.

## Next steps

Phase 2: add vulnerable target machines to the lab that are compatible with ARM64.
