<div align="center">

# 🛡️ Cybersecurity & Pentesting Lab Setup

**A private, isolated Kali Linux lab on VirtualBox for learning ethical hacking**

![Skill](https://img.shields.io/badge/Skill-Cybersecurity-red)
![VirtualBox](https://img.shields.io/badge/VirtualBox-7.x-blue)
![Kali](https://img.shields.io/badge/Kali-Linux-black)
![Network](https://img.shields.io/badge/Network-10.0.0.0%2F24-teal)
![Program](https://img.shields.io/badge/Networkwalks-B084-orange)

</div>

---

## 📖 About This Project

This is my **Week 1 project** for the Cybersecurity Program at **Networkwalks (Batch B084)**.

I built a virtual lab on my Windows laptop using **Oracle VirtualBox** and **Kali Linux**. The Kali VM sits on a dedicated **NAT Network**, so more virtual machines can join the same private network later as targets for practice.

The goal is simple: have a safe place to learn how attacks work, so I can learn how to defend systems.

## ⚠️ Ethical Use

This lab is for **learning and authorized testing only**. Testing is legal only when:

- the device or network is my own, or part of my lab environment
- I have written, documented permission from the owner
- I am working under a signed agreement with an agreed scope

Nothing in this repository should be used against any system outside those cases.

---

## 🗺️ Network Layout

```text
        ┌─────────────────────────────────┐
        │   Windows Host (VirtualBox)     │
        └───────────────┬─────────────────┘
                        │
        ┌───────────────┴─────────────────┐
        │   NatNetwork  –  10.0.0.0/24    │
        │   Gateway: 10.0.0.1             │
        │   DHCP: Enabled                 │
        └───────┬─────────────────┬───────┘
                │                 │
        ┌───────┴───────┐  ┌──────┴────────┐
        │  Kali Linux   │  │ Future target │
        │  10.0.0.2/24  │  │ VMs           │
        └───────────────┘  └───────────────┘
```

## ⚙️ My Lab Configuration

*Values below are taken from my VirtualBox and Kali settings screenshots.*

| Item | Value |
|---|---|
| Host OS | Windows 11 |
| Hypervisor | Oracle VirtualBox |
| VM name | `kali` |
| Guest OS type (VirtualBox) | Ubuntu (64-bit) |
| Base memory | 5267 MB |
| Processors | 5 |
| Acceleration | Nested Paging, KVM Paravirtualization |
| Graphics | VMSVGA, 16 MB video memory |
| Disk | `kali.vdi` (93.41 GB, dynamically allocated) |
| Network adapter | Adapter 1 – Intel PRO/1000 MT Desktop |
| Attached to | NAT Network → `NatNetwork` |
| NAT Network prefix | `10.0.0.0/24` (DHCP enabled, IPv6 disabled) |
| Kali IPv4 method | Manual (static) |
| Kali IP / mask | `10.0.0.2` / `/24` (255.255.255.0) |
| Gateway | `10.0.0.1` |
| DNS | `8.8.8.8` |

> **Note:** Kali release: `[add your version here, e.g. run: cat /etc/os-release]`

---

## 🪜 Setup Steps

### 1️⃣ Install the tools
- Installed **7-Zip** to extract the compressed Kali VM image.
- Installed **Oracle VirtualBox** on Windows.
- Confirmed hardware virtualization (Intel VT-x) is enabled in BIOS/UEFI.

### 2️⃣ Create the NAT Network
In VirtualBox: **File → Tools → Network Manager → NAT Networks → Create**

| Setting | Value |
|---|---|
| Name | `NatNetwork` |
| IPv4 Prefix | `10.0.0.0/24` |
| Enable DHCP | ✅ |
| Enable IPv6 | ❌ |

![NAT Network settings](screenshots/02-nat-network-config.png)

### 3️⃣ Import Kali Linux
- Downloaded the official Kali VirtualBox image and imported it.
- Set memory, CPUs, and attached **Adapter 1** to **NAT Network → `NatNetwork`**.

![VirtualBox VM details](screenshots/01-virtualbox-vm-details.png)

### 4️⃣ Set a static IP on Kali
Opened **Advanced Network Configuration** → edited **Wired connection 1** → **IPv4 Settings**:

| Field | Value |
|---|---|
| Method | Manual |
| Address | `10.0.0.2` |
| Netmask | `24` |
| Gateway | `10.0.0.1` |
| DNS servers | `8.8.8.8` |

![Static IPv4 settings in Kali](screenshots/03-kali-static-ipv4.png)

### 5️⃣ Confirm Kali is up

![Kali desktop running](screenshots/04-kali-desktop.png)

### 6️⃣ Take a clean snapshot
After everything worked, I created a snapshot named **`Clean Kali - Network Setup`** (Machine → Take Snapshot), so I can roll back after risky experiments.

---

## ✅ Verification Checklist

Run these inside Kali and paste your real output or a screenshot under each one.

| # | Test | Command | What to expect |
|---|---|---|---|
| 1 | IP address | `ip addr` | `10.0.0.2/24` on the Ethernet interface |
| 2 | Routing | `ip route` | Default route via `10.0.0.1` |
| 3 | Gateway | `ping -c 4 10.0.0.1` | Replies (if gateway answers ICMP) |
| 4 | Internet | `ping -c 4 8.8.8.8` | Replies (if ICMP allowed) |
| 5 | DNS | `nslookup networkwalks.com` | Domain resolves |
| 6 | Nmap | `nmap --version` | Version is shown |

- [ ] 1 – IP address
- [ ] 2 – Routing
- [ ] 3 – Gateway ping
- [ ] 4 – Internet ping
- [ ] 5 – DNS lookup
- [ ] 6 – Nmap installed

> A failed ping doesn't always mean there is no internet; ICMP can be blocked.

---

## 🐞 Troubleshooting

> ✏️ **Edit this section to match what actually happened to you.** Delete anything you didn't face.

### Problem: No internet after setting the static IP

**Symptom:** After saving the manual IPv4 settings, Kali lost internet connectivity.

**Fix flow I followed:**
1. Checked that the Windows firewall / antivirus wasn't blocking anything.
2. Confirmed the settings with `ip addr`, `ip route`, `nmcli connection show`.
3. Tried changing the DNS to `8.8.8.8`, and then to the gateway `10.0.0.1`.
4. Restarted Kali.
5. Tried a different address in the same subnet (for example `10.0.0.99/24`).
6. Applied the NetworkManager timeout fix from the project video, then restarted the VM and the host:

```bash
sudo nmcli connection modify "Wired connection 1" ipv4.dad-timeout 0
```

*(Use the connection name shown by `nmcli connection show`.)*

### Problem: VirtualBox shows a VT-x / virtualization error

1. Restart the PC and open BIOS/UEFI.
2. Enable Intel VT-x (Intel Virtualization Technology).
3. Save, reboot, and start the VM again.

---

## 🧠 What I Learned

- **NAT vs NAT Network:** plain NAT gives each VM its own outbound-only connection, while a NAT Network lets VMs on the same network reach each other *and* the internet.
- **Static addressing** needs the IP, mask, gateway, and DNS to all match the real network, or connectivity breaks.
- **Snapshots** give me a safe recovery point before experimenting.
- **Documenting** the setup and the problems makes the work repeatable and helps other learners.

## 🔗 Resources

- [7-Zip](https://www.7-zip.org/)
- [Oracle VirtualBox](https://www.virtualbox.org/)
- [Kali Linux downloads](https://www.kali.org/get-kali/)

## 👤 Author

Nihar Giniya

LinkedIn: https://www.linkedin.com/in/nihar-giniya-603b3a32b/

## 📌 Project Information

Program Name: Cybersecurity at Networkwalks | Week: 01 | Project: Cybersecurity & Pentesting Lab Setup | Repository: GitHub1

---


