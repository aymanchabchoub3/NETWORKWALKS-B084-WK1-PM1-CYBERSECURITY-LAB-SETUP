<div align="center">

# 🔐 Cybersecurity Lab Environment Setup

**Building an isolated virtual lab for penetration testing, ethical hacking and network analysis practice**

![Skill](https://img.shields.io/badge/Skill-Cybersecurity-C00000?style=flat-square)
![VirtualBox](https://img.shields.io/badge/VirtualBox-7.x-183A61?style=flat-square&logo=virtualbox&logoColor=white)
![Kali](https://img.shields.io/badge/Kali%20Linux-2026.3-557C94?style=flat-square&logo=kalilinux&logoColor=white)
![Network](https://img.shields.io/badge/NAT%20Network-10.0.0.0%2F24-2E8B57?style=flat-square)
![Wireshark](https://img.shields.io/badge/Wireshark-SOC%20Labs-1679A7?style=flat-square&logo=wireshark&logoColor=white)
![Program](https://img.shields.io/badge/Networkwalks-Batch%20B084-orange?style=flat-square)

**Week 1 · Project Module 1 + Extra Projects · Networkwalks Cybersecurity Internship**

</div>

---

## 📌 Project Overview

This project builds a **safe, isolated virtual cybersecurity lab** on a personal laptop using **VirtualBox** and **Kali Linux**. All machines sit on a private NAT Network, so additional systems (Windows, Android) can be added as targets for **authorized** security testing.

Beyond the core lab, I also completed the **Week 1 extra projects**: an Android-x86 VM, a Windows 10 VM (in progress), and a series of **Wireshark SOC labs** covering packet capture and traffic analysis.

> ⚠️ **Disclaimer:** Everything in this repository was done inside my own lab environment, for educational purposes only. Never test or capture traffic on networks you do not own or do not have written permission to test.

---

## 🧭 Table of Contents

1. [Lab Architecture](#-lab-architecture)
2. [Requirements Checklist](#-requirements-checklist)
3. [Part 1: Core Lab Setup (Kali Linux)](#-part-1-core-lab-setup-kali-linux)
4. [Part 2: Android-x86 VM](#-part-2-android-x86-vm-extra-project)
5. [Part 3: Windows 10 VM](#-part-3-windows-10-vm-extra-project)
6. [Part 4: Wireshark SOC Labs](#-part-4-wireshark-soc-labs-extra-project)
7. [Troubleshooting](#-troubleshooting)
8. [Key Learnings](#-key-learnings)
9. [Repository Structure](#-repository-structure)

---

## 🗺️ Lab Architecture

| Machine | Role | OS | IP Address | Network |
|---|---|---|---|---|
| Host PC | Hypervisor host | Windows 11 | DHCP (home LAN) | n/a |
| **Kali Linux** | Attacker machine | Kali 2026.3 | `10.0.0.2/24` | NatNetwork |
| **Android9-Lab** | Target (extra) | Android-x86 9.0 | `10.0.0.9/24` | NatNetwork |
| **Windows10-Lab** | Target (extra) | Windows 10 (64-bit) | `10.0.0.10/24` | NatNetwork |

**Network settings:** gateway `10.0.0.1` · DNS `8.8.8.8` · DHCP enabled on the NAT Network · all VMs share one NAT Network so they can reach each other and the internet.

```
                    ┌──────────────────── VirtualBox (Windows 11 host) ────────────────────┐
                    │                                                                      │
   Internet ◄──────►│   NatNetwork  10.0.0.0/24   (gateway 10.0.0.1)                       │
                    │      │                │                    │                         │
                    │  Kali Linux       Android9-Lab        Windows10-Lab                  │
                    │  10.0.0.2         10.0.0.9            10.0.0.10                      │
                    └──────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Requirements Checklist

### Core Project (PM1)

- [x] VirtualBox installed (latest version)
- [x] Kali Linux set up as the attacking VM (latest version)
- [x] NATNetwork created with the `10.0.0.0/24` subnet
- [x] Clipboard and drag/drop enabled (Bidirectional)
- [x] Shared folder enabled (host `Downloads` folder)
- [x] Kali static IP set to `10.0.0.2/24`
- [x] Kali has full internet access
- [x] Snapshot taken

### Extra Projects

- [x] Android-x86 9.0 VM on the same NAT Network (`10.0.0.9`), pings verified both ways
- [ ] Windows 10 VM on the same NAT Network (`10.0.0.10`): *in progress*
- [x] Wireshark SOC Labs 1 to 5 completed with 100/100

---

## 🐉 Part 1: Core Lab Setup (Kali Linux)

### Step 1: Create the NAT Network

In VirtualBox: **File → Tools → Network → NAT Networks → Create**. Name it `NatNetwork`, set the IPv4 prefix to `10.0.0.0/24` and keep DHCP enabled.

![NAT Network](screenshots/01-virtualbox-nat-network.png)

### Step 2: Attach Kali to the NAT Network

Kali VM → **Settings → Network → Adapter 1** → *Attached to: NAT Network* → *Name: NatNetwork*.

![Kali network adapter](screenshots/02-kali-network-adapter.png)

### Step 3: Enable Clipboard and Drag & Drop

Kali VM → **Settings → General → Features** → set both **Shared Clipboard** and **Drag-and-Drop** to **Bidirectional**.

![Clipboard and drag and drop](screenshots/03-kali-clipboard-dragdrop.png)

### Step 4: Configure the Shared Folder

Kali VM → **Settings → Shared Folders** → add the host `Downloads` folder with **Full** access and **Auto-mount** enabled.

![Shared folder settings](screenshots/04-kali-shared-folder-settings.png)

### Step 5: Install Guest Additions packages

Inside Kali:

```bash
sudo apt update
sudo apt install -y virtualbox-guest-x11 virtualbox-guest-utils
sudo usermod -aG vboxsf $(whoami)
sudo reboot
```

![Guest additions install](screenshots/05-guest-additions-install.png)

After the reboot, the shared folder mounts at `/media/sf_Downloads`:

```bash
ls /media/sf_Downloads
```

![Shared folder working](screenshots/06-shared-folder-working.png)

### Step 6: Set the static IP address

Network icon → **Edit Connections → Wired connection 1 → IPv4 Settings**

| Field | Value |
|---|---|
| Method | Manual |
| Address | `10.0.0.2` |
| Netmask | `24` |
| Gateway | `10.0.0.1` |
| DNS servers | `8.8.8.8` |

![Kali static IP configuration](screenshots/07-kali-static-ip-config.png)

### Step 7: Apply the connectivity fix and verify the IP

A known issue with VirtualBox 7 and Kali 2026.1+ can leave the VM without internet after setting a static IP. This fix resolves it (see [Troubleshooting](#-troubleshooting)):

```bash
sudo nmcli connection modify "Wired connection 1" ipv4.dad-timeout 0
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"
ip a
```

`eth0` now shows `10.0.0.2/24`.

![nmcli fix and ip a](screenshots/08-nmcli-fix-and-ip-a.png)

### Step 8: Test internet access

```bash
ping -c 4 8.8.8.8
ping -c 4 google.com
```

Both return 0% packet loss, so IP connectivity and DNS resolution both work.

![Internet ping test](screenshots/09-internet-ping-test.png)

### Step 9: Final result

The final desktop shows the IP, the default route via `10.0.0.1`, and live browsing to `networkwalks.com`.

![Kali desktop final](screenshots/10-kali-desktop-final.png)

### Step 10: Take a snapshot

A snapshot named **"Kali configured – NAT 10.0.0.2"** protects the working configuration so I can always roll back.

![Kali snapshot](screenshots/11-kali-snapshot.png)

---

## 🤖 Part 2: Android-x86 VM (Extra Project)

**Goal:** run Android-x86 9.0 in VirtualBox, connect it to the same NAT Network as Kali, and confirm both machines reach each other and the internet.

### Step 1: Download the ISO

`android-x86_64-9.0-r2.iso` (921 MB), downloaded from the official Android-x86 project page.

![Android ISO downloaded](screenshots/pmx-b-01-android-iso-downloaded.png)

### Step 2: Create the VM

| Setting | Value |
|---|---|
| Name | `Android9-Lab` |
| Type / Version | Linux / Other Linux (64-bit) |
| Memory | 2048 MB |
| Disk | 10 GB, VDI, dynamically allocated |

![Android VM summary](screenshots/pmx-b-02-android-vm-summary.png)

### Step 3: Attach the ISO

**Settings → Storage** → attach the Android-x86 ISO to the optical drive on SATA Port 1.

![Android storage ISO](screenshots/pmx-b-03-android-storage-iso.png)

### Step 4: Display settings

Video memory raised to **128 MB** (minimum 64 MB) with **3D Acceleration** enabled.

![Android display settings](screenshots/pmx-b-04-android-display-settings.png)

### Step 5: Install Android-x86 to disk

Boot menu → *Installation: Install Android-x86 to harddisk* → ext4 filesystem → install GRUB → make the system writable.

![Android install complete](screenshots/pmx-b-05-android-install-complete.png)

### Step 6: First boot

![Android home screen](screenshots/pmx-b-06-android-home-screen.png)

### Step 7: Connectivity tests

**From Android:** the VM reaches Kali (`10.0.0.2`) and the internet (`8.8.8.8`) with 0% loss. The routing table shows the default gateway `10.0.0.1`.

![Android pings Kali and internet](screenshots/pmx-b-09-android-ping-kali-internet.png)

**From Kali:** `ping -c 4 10.0.0.9` also succeeds with 0% loss, so connectivity works in both directions.

![Kali pings Android](screenshots/pmx-b-10-kali-ping-android.png)

### Step 8: Both VMs running side by side

![Kali and Android running together](screenshots/pmx-b-11-both-pings-submission.png)

### Step 9: Snapshot

Snapshot **"Android9 configured – NAT 10.0.0.9"**.

![Android snapshot](screenshots/pmx-b-12-android-snapshot.png)

---

## 🪟 Part 3: Windows 10 VM (Extra Project)

> 🚧 **Status: in progress.** The VM is created and the ISO is attached. The NAT Network adapter change, Windows installation, static IP `10.0.0.10` and ping tests are still to be completed.

**Completed so far**

1. Obtained the official Windows 10 ISO using Microsoft's Media Creation Tool and verified its contents.

   ![Windows ISO contents](screenshots/pmx-a-01-win10-iso-downloaded.png)

2. Created the `Windows10-Lab` VM (Windows 10 64-bit, 4096 MB RAM, 40 GB VDI disk).

   ![Windows VM summary](screenshots/pmx-a-02-vm-summary.png)

3. Attached the ISO to the virtual optical drive.

   ![Windows storage ISO attached](screenshots/pmx-a-03-storage-iso-attached.png)

**Remaining**

- [ ] Switch the adapter to *NAT Network → NatNetwork*
- [ ] Install Windows 10
- [ ] Set a static IP: `10.0.0.10`, mask `255.255.255.0`, gateway `10.0.0.1`, DNS `8.8.8.8`
- [ ] Ping tests in both directions with Kali

---

## 🦈 Part 4: Wireshark SOC Labs (Extra Project)

I worked through the Networkwalks Wireshark SOC labs as a SOC analyst, investigating captured traffic to answer scenario questions.

| Lab | Topic | Skills practiced | Score |
|---|---|---|---|
| 1 | Follow TCP Stream | Reassembling HTTP conversations, finding hidden data in a page | **100/100** |
| 2 | HTTP Plain Text Credentials | Reading unencrypted login POST requests | **100/100** |
| 3 | VoIP Call Replay | SIP signalling, RTP streams, codec analysis, call playback | **100/100** |
| 4 | Packet Capture | Live capture, DNS query/response, TLS handshake | **100/100** |
| 5 | PCAP Analysis | Analysing a saved capture offline | **100/100** |

### Result cards

| Lab 1: Follow TCP Stream | Lab 2: HTTP Credentials | Lab 3: VoIP Call Replay |
|---|---|---|
| ![Lab 1 result](screenshots/Lab_1_Result_-_Ayman_Chabchoub.png) | ![Lab 2 result](screenshots/Lab_2_Result_-_ayman_chabchoub.png) | ![Lab 3 result](screenshots/Lab_3_Result_-_ayman_chabchoub.png) |

| Lab 4: Packet Capture | Lab 5: PCAP Analysis |
|---|---|
| ![Lab 4 result](screenshots/Wireshark-Lab4-result.png) | ![Lab 5 result](screenshots/Wireshark-Lab5-result.png) |

### Lab environment

![Wireshark Lab 1 start](screenshots/pmx-c-lab1-start.png)

### Useful Wireshark filters used

| Purpose | Filter |
|---|---|
| HTTP traffic | `http` |
| Follow one conversation | `tcp.stream eq 2` |
| Login attempts | `http.request.method == "POST"` |
| DNS lookups for a site | `dns.qry.name == "example.com"` |
| TLS Client Hello (SNI) | `tls.handshake.type == 1` |
| TLS Server Hello (version) | `tls.handshake.type == 2` |
| SIP call setup | `sip` |
| Voice packets | `rtp` |

---

## 🛠️ Troubleshooting

### Problem: Kali had no internet after setting a static IP

**Cause:** a known issue with VirtualBox v7 and Kali Linux 2026.1+ where NetworkManager's duplicate address detection (DAD) delays or blocks the connection.

**Solution:**

1. Verify the NATNetwork exists (`10.0.0.0/24`) and the VM adapter is attached to it.
2. Make sure no other VM on the same NAT Network uses `10.0.0.2`.
3. Disable the DAD timeout and restart the connection:

   ```bash
   sudo nmcli connection modify "Wired connection 1" ipv4.dad-timeout 0
   sudo nmcli connection down "Wired connection 1"
   sudo nmcli connection up "Wired connection 1"
   ```

4. Restart the VM (and the host if needed).
5. If DNS fails, try `10.0.0.1` instead of `8.8.8.8` as the DNS server.

### Problem: Shared folder not accessible in Kali

**Solution:** add the user to the `vboxsf` group with `sudo usermod -aG vboxsf $(whoami)`, then **reboot** so the group change takes effect.

### Problem: VMs could not reach each other

**Solution:** each VM's adapter must be set to **NAT Network** (not plain **NAT**) and use the *same* network name. Plain NAT isolates each VM from the others.

---

## 💡 Key Learnings

- **NAT Network vs NAT:** NAT Network lets VMs talk to each other and the internet; plain NAT isolates every VM.
- **Static IPs matter in a lab:** predictable addresses (`.2`, `.9`, `.10`) make scanning and testing repeatable.
- **Snapshots are a safety net:** take one after every working configuration.
- **HTTP is plaintext:** credentials sent over HTTP are fully readable in a capture. The fix is HTTPS.
- **VoIP can be replayed:** SIP and RTP without encryption can be played back from a capture. The fix is SRTP.
- **DNS then TLS:** a browser first resolves the name (DNS), then opens a TLS session where the Client Hello reveals the server name (SNI).

---

## 📁 Repository Structure

```
.
├── README.md
└── screenshots/
    ├── 01-virtualbox-nat-network.png
    ├── 02-kali-network-adapter.png
    ├── 03-kali-clipboard-dragdrop.png
    ├── 04-kali-shared-folder-settings.png
    ├── 05-guest-additions-install.png
    ├── 06-shared-folder-working.png
    ├── 07-kali-static-ip-config.png
    ├── 08-nmcli-fix-and-ip-a.png
    ├── 09-internet-ping-test.png
    ├── 10-kali-desktop-final.png
    ├── 11-kali-snapshot.png
    ├── pmx-a-01-win10-iso-downloaded.png
    ├── pmx-a-02-vm-summary.png
    ├── pmx-a-03-storage-iso-attached.png
    ├── pmx-b-01-android-iso-downloaded.png
    ├── pmx-b-02-android-vm-summary.png
    ├── pmx-b-03-android-storage-iso.png
    ├── pmx-b-04-android-display-settings.png
    ├── pmx-b-05-android-install-complete.png
    ├── pmx-b-06-android-home-screen.png
    ├── pmx-b-09-android-ping-kali-internet.png
    ├── pmx-b-10-kali-ping-android.png
    ├── pmx-b-11-both-pings-submission.png
    ├── pmx-b-12-android-snapshot.png
    ├── pmx-c-lab1-start.png
    ├── Lab_1_Result_-_Ayman_Chabchoub.png
    ├── Lab_2_Result_-_ayman_chabchoub.png
    ├── Lab_3_Result_-_ayman_chabchoub.png
    ├── Wireshark-Lab4-result.png
    └── Wireshark-Lab5-result.png
```

---

<div align="center">

**Ayman Chabchoub** · Networkwalks Cybersecurity Internship · Batch B084

*Instructor: Waqas Karim CCIE · #Networkwalks*

</div>
