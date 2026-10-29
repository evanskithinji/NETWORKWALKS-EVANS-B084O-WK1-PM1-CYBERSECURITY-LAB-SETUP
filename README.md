# NETWORKWALKS-EVANS-B084O-WK1-PM1-CYBERSECURITY-LAB-SETUP
Cybersecurity lab: VirtualBox NAT Network (10.0.0.0/24) with Kali Linux, Windows 11/10/7 and Android VMs
# Cybersecurity & Ethical Hacking Lab Setup

A home cybersecurity testing lab built on **VirtualBox** with **Kali Linux** as the attacking machine and Windows 11, Windows 10, Windows 7, Windows Server 2016 and Android 9 as target machines, all on one isolated NAT Network (`10.0.0.0/24`).

## Lab Overview

| VM | Role | IP Address | Gateway | Ping from Kali |
|---|---|---|---|---|
| Kali Linux 2026.3 | Attacker | 10.0.0.2/24 | 10.0.0.1 | n/a |
| Windows 11 | Target | 10.0.0.11/24 | 10.0.0.1 | 0% loss |
| Windows 10 | Target | 10.0.0.10/24 | 10.0.0.1 | 0% loss |
| Windows 7 | Target | 10.0.0.7/24 | 10.0.0.1 | 0% loss |
| Windows Server 2016 | Target | 10.0.0.16/24 | 10.0.0.1 | 0% loss |
| Android x86 (9) | Target | 10.0.0.9/24 | 10.0.0.1 | 0% loss |

**Host:** Windows, VirtualBox (latest recommended version)

---

## Phase 1: Kali Linux

### Step 1: Install VirtualBox and 7-Zip
Installed 7-Zip (to extract the Kali archive) and the latest VirtualBox from the official download pages.

### Step 2: Create the NAT Network (10.0.0.0/24)
`File > Tools > Network > NAT Networks > Create`, then set the name to `NatNetwork`, IPv4 prefix to `10.0.0.0/24`, and DHCP enabled.

![NAT Network 10.0.0.0/24](screenshots/01-nat-network.jpeg)

### Step 3: Import Kali Linux and attach it to the NAT Network
Downloaded the pre-built Kali VirtualBox image, imported it, then set Adapter 1 to **NAT Network** with the name `NatNetwork`.

![Kali network adapter](screenshots/02-kali-network-adapter.jpeg)

### Step 4: Set the static IP on Kali (10.0.0.2/24)
Network Manager > Edit Connections > Wired connection 1 > IPv4 Settings:

- Method: Manual
- Address: `10.0.0.2`, Netmask: `24`, Gateway: `10.0.0.1`
- DNS: `10.0.0.1` (also tested `8.8.8.8`)

![Kali IPv4 settings](screenshots/03-kali-ipv4-settings.jpeg)

### Step 5: Internet access
Kali had no internet at first (a known issue with VirtualBox 7 and newer Kali releases). Fixed by disabling the duplicate address detection timeout and reconnecting:

```bash
sudo nmcli connection modify "Wired connection 1" ipv4.dad-timeout 0
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"
```

After this, Kali had full internet access.

![nmcli fix and internet working](screenshots/04-kali-nmcli-fix-internet.jpeg)

Confirmed the address with `ip a`:

![ip a output](screenshots/05-kali-ip-a.jpeg)

### Step 6: Shared folder
Added the host `Downloads` folder as a global shared folder with full access and auto-mount.

![Shared folder](screenshots/06-shared-folder.jpeg)

### Step 7: Clipboard and drag-and-drop
Set **Shared Clipboard** and **Drag-and-Drop** to **Bidirectional** under `Settings > General > Advanced`.

![Kali clipboard and drag-and-drop](screenshots/07-kali-clipboard-dragdrop.jpeg)

### Step 8: Snapshots
Took snapshots at key points: `My Fresh Kali` (IP and network configured) and `Kali With Internet` (after the nmcli fix).

![Kali snapshots](screenshots/08-kali-snapshots.jpeg)

---

## Phase 2: Additional Virtual Machines

All VMs use **Adapter 1: NAT Network (`NatNetwork`)** with a manual IP in `10.0.0.0/24`, gateway `10.0.0.1`, and DNS `10.0.0.1` / `8.8.8.8`.

### Windows 10 (10.0.0.10)
Used VirtualBox's unattended install with Guest Additions, then set a static IP in the network settings.

![Windows 10 unattended install](screenshots/09-windows10-unattended-install.jpeg)
![Windows 10 IP settings and internet](screenshots/10-windows10-ip-internet.jpeg)

### Windows 7 (10.0.0.7)
Set the static IP through `Network and Sharing Center > Change adapter settings > IPv4 Properties`. Clipboard and drag-and-drop set to Bidirectional.

![Windows 7 IP settings](screenshots/11-windows7-ip.jpeg)
![Windows 7 clipboard and drag-and-drop](screenshots/12-windows7-settings.jpeg)

### Windows 11 (10.0.0.11)
Static IP configured as above. Ping was blocked by default, see the ICMP section below.

![Windows 11 firewall ICMP rule](screenshots/13-windows11-icmp-rule.jpeg)

### Windows Server 2016 (10.0.0.16)
Chose **Windows Server 2016 Datacenter Evaluation (Desktop Experience)**. The other editions install without a GUI.

![Windows Server edition selection](screenshots/14-winserver-edition.jpeg)
![Windows Server password complexity](screenshots/15-winserver-password.jpeg)

### Android x86 (10.0.0.9)
Android needed extra setup:

- Display: Graphics Controller set to **VBoxSVGA** and **3D Acceleration disabled**
- Network: static IP `10.0.0.9`, gateway `10.0.0.1`, prefix length `24`, DNS `8.8.8.8`

![Android display settings](screenshots/16-android-display.jpeg)
![Android network details](screenshots/17-android-ip.jpeg)

---

## Enabling Ping (ICMP) on Windows

Windows blocks ICMP echo requests by default, so the machines could not ping each other. To fix this on each Windows VM:

1. Press `Win + R`, type `wf.msc` and press Enter.
2. Select **Inbound Rules**.
3. Find **File and Printer Sharing (Echo Request - ICMPv4-In)**.
4. Right-click the rule and select **Enable Rule**.

---

## Connectivity Tests

Pinged every VM from Kali. All replies received with 0% packet loss.

**Windows 7** (rule enabled on the right):
![Ping Windows 7](screenshots/18-ping-windows7.jpeg)

**Windows 10:**
![Ping Windows 10](screenshots/19-ping-windows10.jpeg)

**Windows 11:**
![Ping Windows 11](screenshots/20-ping-windows11.jpeg)

**Windows Server 2016:**
![Ping Windows Server 2016](screenshots/21-ping-winserver2016.jpeg)

**Android:**
![Ping Android](screenshots/22-ping-android.jpeg)

---

## Final State: All VMs and Snapshots

Every VM has a snapshot taken once it was configured and working.

![All VMs with snapshots](screenshots/23-all-vms-snapshots.jpeg)

---

## Troubleshooting Notes

| Problem | Solution |
|---|---|
| Kali had no internet after setting a static IP | Ran the three `nmcli` commands above (`ipv4.dad-timeout 0`), then restarted the connection |
| Windows VMs could not be pinged | Enabled the ICMPv4 echo request inbound rule in Windows Firewall |
| Windows Server 2016 install had no GUI | Selected the **Datacenter Evaluation (Desktop Experience)** edition |
| Windows Server rejected the password | The password must meet complexity rules (upper, lower, number, symbol) |
| Android display and network issues | Used VBoxSVGA with 3D acceleration off, and set a static IP manually |

## Tools Used

- Oracle VirtualBox
- Kali Linux 2026.3
- Windows 11, Windows 10, Windows 7, Windows Server 2016, Android x86

> This lab is for learning and authorised practice only, inside an isolated virtual network.
