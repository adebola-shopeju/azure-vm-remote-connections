# Azure VM Remote Connection Lab

**Darey.io Cloud Engineering Track — Lab 5**  
**Student:** Adebola Shopeju  
**Date:** 24 June 2026  
**Subscription:** Azure for Students  
**Resource Group:** lab5-rg  

---

## Overview

This lab demonstrates how to remotely access and manage cloud-based virtual machines on Microsoft Azure using industry-standard protocols — Remote Desktop Protocol (RDP) for Windows and Secure Shell (SSH) for Linux. It covers Network Security Group configuration, public IP identification, secure remote connection, system verification, and security hardening.

---

## Virtual Machines Provisioned

| VM Name | OS | Public IP | Private IP | Size |
|---|---|---|---|---|
| `windows-vm-lab5` | Windows Server 2025 Datacenter | 51.103.216.47 | 10.0.0.4 | Standard B2ats v2 |
| `linux-vm-lab5` | Ubuntu 24.04 LTS | 51.103.217.50 | 10.0.0.4 | Standard B2ats v2 |

Both VMs are deployed in **Switzerland North (Zone 1)** under the `lab5-rg` resource group.

---

## Phase 1 — Environment Setup

### Steps Taken
1. Logged into [portal.azure.com](https://portal.azure.com) using Azure for Students account
2. Navigated to **Virtual Machines** and located both VMs
3. Recorded Public IP addresses from the VM Overview blade
4. Verified OS types: Windows Server 2025 and Ubuntu 24.04
5. Confirmed Microsoft Remote Desktop (macOS) was installed for RDP
6. Confirmed Terminal (macOS built-in) was available for SSH
7. Reviewed Network Security Groups attached to each VM

### Screenshots
- `screenshots/vm-overview-windows.png` — Windows VM overview with Public IP
- `screenshots/vm-overview-linux.png` — Linux VM overview with Public IP

---

## Phase 2 — Network Security Group Configuration

### NSG Rules Configured

**Windows VM (`windows-vm-lab5-nsg`)**
- Inbound rule: **RDP** — Port 3389, TCP, Priority 300, Source: Any (later restricted)

**Linux VM (`linux-vm-lab5-nsg`)**
- Inbound rule: **SSH** — Port 22, TCP, Priority 300, Source: Any (later restricted)

Both rules were automatically created during VM provisioning and verified in the Network Settings blade.

### Screenshots
- `screenshots/nsg-inbound-rules-windows.png` — Windows NSG showing Port 3389 rule
- `screenshots/nsg-inbound-rules-linux.png` — Linux NSG showing Port 22 rule

---

## Phase 3 — Windows VM: RDP Connection

### Connection Method
Used **Microsoft Remote Desktop** (macOS app) to connect to the Windows VM.

### Steps Taken
1. Opened Microsoft Remote Desktop on macOS
2. Clicked **Add PC** and entered `51.103.216.47` as the PC name
3. Entered credentials: username `azureuser` and the VM password set during provisioning
4. Accepted the certificate warning (expected in lab environments)
5. Successfully connected — greeted by the SConfig (Server Configuration) menu for Windows Server 2025
6. Selected option **15 (Exit to command line / PowerShell)**
7. Ran verification commands in PowerShell

### Verification Commands Run
```
hostname
ipconfig
```

### Output
```
windows-vm-lab5

Windows IP Configuration

Ethernet adapter Ethernet:
   IPv4 Address. . . . . . . : 10.0.0.4
   Subnet Mask . . . . . . . : 255.255.255.0
   Default Gateway . . . . . : 10.0.0.1
```

### Session Termination
Typed `exit` to leave PowerShell, then disconnected via Microsoft Remote Desktop menu (not shut down).

### Screenshots
- `screenshots/rdp-connected-desktop.png` — SConfig menu confirming remote Windows session
- `screenshots/rdp-cli-verification.png` — hostname and ipconfig output inside Windows VM

---

## Phase 4 — Linux VM: SSH Connection

### Connection Method
Used **macOS Terminal** to connect via SSH (password-based authentication).

### Steps Taken
1. Opened Terminal on macOS
2. Ran the SSH command:
```bash
ssh azureuser@51.103.217.50
```
3. Typed `yes` to accept the ED25519 host key fingerprint and add to known hosts
4. Entered the VM password when prompted
5. Successfully connected — Ubuntu 24.04 welcome banner displayed

### Verification Commands Run
```bash
hostname
uname -a
whoami
df -h
apt list --installed 2>/dev/null | head -20
```

### Output Summary
```
hostname        → linux-vm-lab5
uname -a        → Linux linux-vm-lab5 6.17.0-1018-azure #18~24.04.1-Ubuntu
whoami          → azureuser
df -h           → /dev/root 29G, 1.7G used (6%)
apt list        → 20 packages listed including apt, bash, apparmor, azure-vm-utils
```

### Session Termination
```bash
exit
```
Output: `Connection to 51.103.217.50 closed.`

### Screenshots
- `screenshots/ssh-connected-prompt.png` — SSH welcome banner after successful login
- `screenshots/ssh-system-commands.png` — Output of hostname, uname -a, whoami, df -h, apt list

---

## Phase 5 — Security Hardening

### IP Restriction Applied

After completing connections, NSG inbound rules were updated to restrict access to the student's specific public IP address only.

**Local public IP identified using:**
```bash
curl ifconfig.me
# Output: 102.88.55.250
```

**NSG rules updated:**

| VM | Port | Source Before | Source After |
|---|---|---|---|
| `windows-vm-lab5` | 3389 | Any (0.0.0.0/0) | 102.88.55.250/32 |
| `linux-vm-lab5` | 22 | Any (0.0.0.0/0) | 102.88.55.250/32 |

Changes were made via: VM → Networking → Network Settings → click rule → edit Source → IP Addresses → enter CIDR → Save.

Post-restriction, connections were confirmed to still work from the authorised IP.

### Screenshots
- `screenshots/nsg-restricted-windows.png` — Windows NSG with source restricted to 102.88.55.250/32
- `screenshots/nsg-restricted-linux.png` — Linux NSG with source restricted to 102.88.55.250/32

---

## Session Termination Summary

| Protocol | Correct Termination Method |
|---|---|
| RDP | Disconnect from Microsoft Remote Desktop (never use Shut Down) |
| SSH | Type `exit` in terminal — confirms `Connection closed` |

Proper termination ensures the VM remains running but frees the session, prevents credential exposure, and avoids accidental shutdowns that could affect billing or availability.

---

## Troubleshooting Notes

| Issue Encountered | Cause | Resolution |
|---|---|---|
| Microsoft Remote Desktop attempted to RDP into Linux VM IP | Wrong tool used — Linux does not support RDP | Used macOS Terminal with `ssh` command instead |
| Windows Agent status showed "Not Ready" after deployment | Normal — agent initialises after VM boots | Waited ~5 minutes; resolved automatically |
| SSH fingerprint warning on first connection | First-time connection to new host | Typed `yes` to accept and add to known hosts |

---

## Repository Structure

```
azure-vm-remote-connection/
├── README.md
├── NSG-summary.md
└── screenshots/
    ├── vm-overview-windows.png
    ├── vm-overview-linux.png
    ├── nsg-inbound-rules-windows.png
    ├── nsg-inbound-rules-linux.png
    ├── rdp-connected-desktop.png
    ├── rdp-cli-verification.png
    ├── ssh-connected-prompt.png
    ├── ssh-system-commands.png
    ├── nsg-restricted-windows.png
    └── nsg-restricted-linux.png
```

---

*Lab 5 — Virtual Machine Setup & Remote Connection | Darey.io Cloud Engineering Track*
