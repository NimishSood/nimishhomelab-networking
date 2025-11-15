📘 Cisco 2960-X Homelab Setup – Day 1 Documentation
📌 Overview

This document outlines the initialization and setup of a Cisco 2960-X 48-Port PoE+ (740W) switch for homelab use.
Everything below reflects only the tasks completed so far: hardware setup, factory reset, base configuration, and making the switch remotely manageable.

🟦 1. Physical Setup
Hardware

Cisco 2960-X 48-Port PoE+ (WS-C2960X-48FPS-L)

Intel PRO/1000 PT Quad-Port Server NIC (installed in PC)

Bell Home Hub modem/router (gateway)

Wiring

Bell Router LAN → Switch Port Gi1/0/1

PC Quad-NIC Port (Ethernet 5) → Switch Port Gi1/0/2

The PC NIC in use has MAC: 34:40:B5:F5:66:82
IP reserved for it on Bell router: 192.168.2.10

🟩 2. Full Factory Reset

Performed a complete wipe of the switch configuration via the bootloader:

Held MODE button during boot to enter recovery

Ran commands:

flash_init
dir flash:
delete flash:config.text
delete flash:vlan.dat
delete flash:private-config.text
delete flash:private-config.text.backup


Booted IOS manually:

boot flash:/c2960x-universalk9-mz.152-4.E8/c2960x-universalk9-mz.152-4.E8.bin


Switch successfully restored to factory defaults.

🟧 3. Initial Network Connectivity Check

After reset and wiring:

PC received DHCP properly from Bell router

Internet access confirmed

Switch correctly forwarded frames between PC ↔ Bell

No config needed for basic connectivity

🟨 4. Disabled PoE on ALL Ports

Since PoE isn’t needed in this homelab setup:

conf t
interface range gi1/0/1 - 48
 power inline never
end


Verified power state:

Available: 740W
Used:      0W
Remaining: 740W

🟥 5. Shutdown All Unused Ports

To keep the switch secure and tidy:

conf t
interface range gi1/0/3 - 48
 shutdown
end


Active ports:

Gi1/0/1 – Bell uplink

Gi1/0/2 – PC uplink

🟦 6. Reserved the PC IP Address

Steps done on the Bell router:

Deleted old reservation tied to motherboard NIC

Rebooted router to clear ARP/DHCP cache

Reserved 192.168.2.10 for NIC MAC 34:40:B5:F5:66:82

PC now consistently gets:

IPv4: 192.168.2.10
Gateway: 192.168.2.1
DNS:    192.168.2.1

🟩 7. Base Configuration of the Switch
7.1 Hostname
hostname LAB-SW1

7.2 Disable DNS lookup
no ip domain-lookup

7.3 Assign Management IP (VLAN 1)
interface vlan 1
 ip address 192.168.2.2 255.255.255.0
 no shutdown

7.4 Set Default Gateway
ip default-gateway 192.168.2.1

7.5 Enable SSH Access
username nimish privilege 15 secret <password>
ip domain-name homelab.local
crypto key generate rsa 2048

line vty 0 4
 login local
 transport input ssh

7.6 Save config
write memory

🟦 8. Connectivity Verification
From Switch:
ping 192.168.2.1     → OK (Router)
ping 192.168.2.10    → OK (PC NIC)

From PC:
ping 192.168.2.2     → OK (Switch management IP)

🟫 9. Verified SSH Access

Successfully logged into the switch remotely via KiTTY

SSH operational with older Cisco MAC algorithms

No need for console cable unless troubleshooting
