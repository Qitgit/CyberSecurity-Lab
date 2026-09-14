# Security Lab Environment: Virtualisation, Kali Linux & Vulnerability Scanning

> Study notes for building a working penetration-testing lab: hypervisor concepts →
> VM installation (Windows 10 host, Kali, Metasploitable) → Linux command line →
> Nessus vulnerability scanner. This continues the portfolio series that started with
> Switching & VLAN, and leads into the next practical phase (scanning, exploitation, reporting).

---

## Table of Contents

1. [Virtualisation & Hypervisor Concepts](#1-virtualisation--hypervisor-concepts)
2. [Host Setup: Windows 10 + VMware Workstation](#2-host-setup-windows-10--vmware-workstation)
3. [Installing the Kali Linux VM](#3-installing-the-kali-linux-vm)
4. [Setting Up Vulnerable Target VMs (Metasploitable)](#4-setting-up-vulnerable-target-vms-metasploitable)
5. [Linux Command Line Fundamentals](#5-linux-command-line-fundamentals)
6. [Installing Nessus (Vulnerability Scanner)](#6-installing-nessus-vulnerability-scanner)
7. [Lab Checklist](#7-lab-checklist)

---

## 1. Virtualisation & Hypervisor Concepts

*Source: Security Lab Setup*

### 1.1 Hypervisor Types
| Level | Description | Examples |
|---|---|---|
| **Level 1 (Bare Metal)** | Runs directly on hardware, no host OS needed | VMware vSphere ESXi, Microsoft Hyper-V |
| **Level 2 (Hosted)** | Runs on top of an existing OS | VMware Workstation Pro, VirtualBox, Parallels |

This lab uses **Level 2 (VMware Workstation Pro)** since it runs on top of Windows 10 as the host.

### 1.2 VM Image Formats
- **OVF (Open Virtualization Format)** — open standard for packaging/distributing virtual appliances
- **OVA (Open Virtualization Appliance)** — single-file distribution of an OVF package, stored as a TAR archive
  → Used directly in [§4 Metasploitable3](#4-setting-up-vulnerable-target-vms-metasploitable) (`.ova` import)

### 1.3 Virtual Networking Modes — Security Implications
| Mode | Behaviour | Risk when testing malware/exploits |
|---|---|---|
| **Bridged** | VM gets direct access to the external network | **Bad** — malware can reach the internet/intranet/host |
| **NAT** | VM shares the host's network identity via a private subnet | **Bad** — still has outbound access |
| **Host-Only (VMnet1)** | VM can only talk to the host and other VMs on the same host-only network | **Safer**, but the host itself is still exposed |

**Practical takeaway**: for this lab, VMs are set to **Custom → VMnet1 (Host-Only)** so Kali, Metasploitable, and the host stay isolated from the wider network. This directly connects to how [§2 Windows 10 VMware setup](#2-host-setup-windows-10--vmware-workstation) configures virtual adapters.

- **LAN Segment** — an alternative private network construct, useful for multi-tier testing and isolating VM-to-VM traffic without touching VMnet1/8 at all.

### 1.4 Kali Linux
- Debian-derived distribution, purpose-built for offensive security — 600+ pre-installed tools
- Maintained/funded by Offensive Security Ltd.
→ Installed in [§3](#3-installing-the-kali-linux-vm), used as the attacking platform against [§4 Metasploitable](#4-setting-up-vulnerable-target-vms-metasploitable)

### 1.5 Why Deliberately Vulnerable VMs?
- Metasploitable3 (Windows Server 2008) and Metasploitable2 (Ubuntu-based) are **intentionally unpatched**
- Rationale: attacking a fully patched system requires a zero-day exploit (real-world cost: tens of thousands to millions of USD per the Zerodium payout table referenced in the source deck) — unrealistic for a training lab
- This is the standard justification used to explain *why* a lab uses known-vulnerable targets rather than "just hacking a real patched box"

---

## 2. Host Setup: Windows 10 + VMware Workstation

*Source: Windows 10 – VMware Workstation*

### 2.1 Pre-Installation Cleanup
- Uninstall any older VMware Workstation version
- Uninstall **Hyper-V** if present (Hyper-V and VMware Workstation's virtualization layer conflict — this is a common source of "VT-x not available" errors)

### 2.2 Default Virtual Network Adapters
| Adapter | Type | Purpose |
|---|---|---|
| **VMnet0** | Bridged | VM appears as its own device on the physical network |
| **VMnet8** | NAT | VM shares host's network identity via private DHCP |
| **VMnet1** | Host-Only | VM isolated to host + other VMs only ([§1.3](#13-virtual-networking-modes--security-implications) — the mode this lab actually uses) |

### 2.3 Verifying Virtual Network Interfaces (do this every session)
1. `Win + R` → `ncpa.cpl` → Network Connections
2. Confirm **VMnet8** and **VMnet1** adapters are enabled (other students on shared machines can disable/change these)
3. Check VMnet8's IP address — this is how a VM can reach the host over NAT
4. On the physical adapter, confirm **VMware Bridge Protocol** is checked (required for Bridged mode to function at all)

→ This adapter verification step is the practical prerequisite for every lab session in [§3](#3-installing-the-kali-linux-vm) and [§4](#4-setting-up-vulnerable-target-vms-metasploitable)

---

## 3. Installing the Kali Linux VM

*Source: VMware Workstation – Installing Kali 202X VM*

### 3.1 Import Process
1. Extract `kali-linux-202X.Y-vmware-amd64.7z` (via 7-Zip) to a working folder
2. Open the extracted `.vmx` file directly in VMware Workstation (no wizard needed — it's a pre-built VM)

### 3.2 Performance Tuning Before First Boot
| Setting | Value | Why |
|---|---|---|
| Memory | 4 GB | Baseline for a usable Kali desktop |
| Processors | 1 processor × 4 cores | Better compatibility than multi-processor allocation |
| CD/DVD (IDE) | Force **[Use physical drive]** | Fixes an "Using unknown backend" boot error |

### 3.3 Snapshot Discipline (recurring theme across all three VM guides)
- **Initial Setup** snapshot — taken immediately after import, before first boot
- After VMware Tools install + `apt update && apt-get upgrade`, power off and take **Updated Image** snapshot
- Rule of thumb reinforced across [§3](#3-installing-the-kali-linux-vm) / [§4](#4-setting-up-vulnerable-target-vms-metasploitable) / [§6](#6-installing-nessus-vulnerability-scanner): **never take a snapshot while the VM is powered on** for a "clean" baseline (live snapshots capture running-state inconsistencies)

### 3.4 Post-Install Steps
1. Upgrade hardware compatibility (Change Hardware Compatibility Wizard → highest version → Alter this VM)
2. Log in (`kali`/`kali`)
3. Install/update `open-vm-tools` (VM integration tools) → enables **View → Fit Guest Now** for auto-resolution
4. `apt update` → `apt-get upgrade` → `poweroff` → snapshot

→ The `apt-get upgrade` command here is the Debian-family equivalent covered generically in [§5.8 Package Management](#58-package-management-cli)

---

## 4. Setting Up Vulnerable Target VMs (Metasploitable)

*Source: Setting Up Metasploitable VMs*

### 4.1 Metasploitable3 (Windows Server 2008)
- Import via `.ova` file ([§1.2 OVF/OVA](#12-vm-image-formats))
- Memory: 2 GB
- Network adapter: **NAT (preferred)** or Host-Only — consistent with the isolation principle from [§1.3](#13-virtual-networking-modes--security-implications)
- **Critical, irreversible setting**: Guest OS must be set to **Windows Server 2008 R2 x64** *before* first power-on — cannot be changed afterward once the wrong network driver installs (VM would need to be deleted and re-imported)
- Credentials: `administrator` / `vagrant`
- Trial reset trick: `slmgr.vbs -rearm` via Run dialog, then restart, then snapshot as "Trial Rearmed"

### 4.2 Metasploitable2 (Linux, Ubuntu-based)
- Import via `.vmx` extracted from `.zip` (no OVA conversion step needed — lighter-weight than M3)
- Memory: 1 GB
- Credentials: `msfadmin` / `msfadmin`
- Verify network adapter status with `ifconfig` (→ [§5.7 ifconfig](#57-networking-commands-quick-reference)) immediately after login
- Graceful shutdown: `sudo poweroff`

### 4.3 Why Two Different Target OS Families?
Having both a Windows Server target (M3) and a Linux target (M2) means the same Kali attack box ([§3](#3-installing-the-kali-linux-vm)) can be used to practice against both OS families' distinct vulnerability classes — directly useful once [§6 Nessus](#6-installing-nessus-vulnerability-scanner) scans are run against both.

---

## 5. Linux Command Line Fundamentals

*Source: Basic Linux Commands*

### 5.1 DOS ↔ Linux Command Equivalents
| Purpose | MS-DOS | Linux | Example |
|---|---|---|---|
| Copy files | `copy` | `cp` | `cp thisfile.txt /home/thisdirectory` |
| Move/rename | `move` | `mv` | `mv thisfile.txt /home/thisdirectory` |
| List files | `dir` | `ls` | `ls` |
| Clear screen | `cls` | `clear` | `clear` |
| Delete files | `del` | `rm` | `rm thisfile.txt` |
| Edit text | `edit` | `gedit`/`nano`/`vi` | `nano thisfile.txt` |
| Compare files | `fc` | `diff` | `diff file1 file2` |
| Search text in file | `find` | `grep` | `grep word thisfile.txt` |
| Command help | `command /?` | `man`/`info` | `man command` |
| Make directory | `mkdir` | `mkdir` | `mkdir directory` |
| View file | `more` | `less` | `less thisfile.txt` |
| Show location | `chdir` | `pwd` | `pwd` |
| Show RAM usage | `mem` | `free` | `free` |

### 5.2 Case Sensitivity
- **Everything** in Linux is case-sensitive: commands, filenames, folder names, passwords
- `PING` and `ping` are treated as different (unknown vs. valid) commands
- `file1` and `File1` are different files on the same filesystem

### 5.3 Hidden Files
- Identified by a leading dot: `.filename`
- List including hidden files: `ls -a`

### 5.4 Superuser / Root Account
- Root always has **User ID: 0**
- Direct root login is disabled by default on most distros for security reasons
- **Ubuntu-style**: `sudo command` (must be in the `sudo` group, prompted for own password)
- **Fedora-style**: `su` to switch to root entirely, `exit` to drop back
- Standard user prompt ends in `$`; root prompt ends in `#`
- Example: `sudo touch /Important` — creates an empty file at the root path

### 5.5 File Permissions
- Format: `(d/-/l)rwxrwxrwx` → type, owner, group, other
- `r`=4, `w`=2, `x`=1 → summed per digit
- `chmod 755` = owner:rwx(7), group:r-x(5), other:r-x(5)
- Ownership: `chown user:group filename`
- Group membership: `usermod -a -G groupname username` (`-a` = append, don't remove existing groups)

### 5.6 Text Editors
| Editor | Character | Use case |
|---|---|---|
| **Vi/Vim** | Powerful, steep learning curve, syntax highlighting | Sysadmins/programmers |
| **Gedit** | GUI-based, beginner-friendly | Desktop environments |
| **Nano** | CLI, easy key bindings | **Used throughout this course** |

### 5.7 Networking Commands (quick reference)
- `ifconfig` — Linux equivalent of Windows `ipconfig` (used in [§4.2 Metasploitable2](#42-metasploitable2-linux-ubuntu-based) to verify the adapter)
- `traceroute` — Linux equivalent of `tracert`

### 5.8 Package Management (CLI)
| Task | Ubuntu (APT) | Fedora (DNF) |
|---|---|---|
| Refresh package index | `apt-get update` | `dnf check-update` |
| Upgrade all packages | `apt-get upgrade` | `dnf update` |
| Install a package | `apt-get install <pkg>` | `dnf install <pkg>` |
| Remove a package | `apt-get remove <pkg>` | `dnf remove <pkg>` |
| Reinstall | `apt-get install --reinstall <pkg>` | `dnf reinstall <pkg>` |

→ This is exactly the `apt update`/`apt-get upgrade` sequence run on Kali in [§3.4](#34-post-install-steps)

### 5.9 Archives
- `tar` = tape archive tool; `.gz` (gzip, fast) vs `.bz2` (bzip2, smaller but slower)
- Extract: `tar xvfj filename.tar.bz2` (x=extract, v=verbose, f=file, j=bzip2, z=gzip)
- Create: `tar cvfz filename.tar.gz path/to/archive` (c=create)

### 5.10 Output Redirection
- `command >> file` — appends command output to a file (creates it if absent)
- Example: `ifconfig >> ifconfig` — saves interface info to a log file

---

## 6. Installing Nessus (Vulnerability Scanner)

*Source: Installing Nessus on Kali*

### 6.1 Install Process
1. Inside Kali, download the **Debian AMD64** `.deb` package from tenable.com
2. Install via `dpkg`:
```bash
cd Downloads
sudo dpkg -i Nessus-x.y.z-debian10_amd64.deb
```
3. Start the service (needed **every time the VM restarts**):
```bash
/bin/systemctl start nessusd.service
```
4. Access the web UI at `https://localhost:8834` (self-signed cert warning → Advanced → Accept the Risk and Continue)

### 6.2 Registration & Setup
- Choose **Register for Nessus Essentials** (free tier)
- Activation code sent to TAFE email
- Create a local admin account (used only for logging into the Nessus web UI, unrelated to the Kali OS account)
- Initial plugin/module compilation takes time on first run

### 6.3 Where This Fits
Nessus becomes the **vulnerability scanning phase** of the lab workflow:

This is the natural next step after environment setup — scan results from Nessus against the Metasploitable VMs will drive the exploitation phase (Metasploit Framework — not yet covered in these PDFs, candidate for the next portfolio section).

### 6.4 Snapshot Discipline
- Same rule as [§3.3](#33-snapshot-discipline-recurring-theme-across-all-three-vm-guides): power off first, **then** snapshot as "Installed Nessus"

---

## 7. Lab Checklist

- [ ] Host prep: uninstall old VMware Workstation / Hyper-V, install current VMware Workstation
- [ ] Verify VMnet1 (Host-Only) and VMnet8 (NAT) adapters enabled
- [ ] Kali VM imported, hardware compatibility upgraded, resource-tuned (4GB RAM, 1×4 cores)
- [ ] Kali VMware Tools installed, `apt update && apt-get upgrade` run, snapshot taken
- [ ] Metasploitable3 (Windows Server 2008) imported, guest OS type set correctly, trial rearmed
- [ ] Metasploitable2 (Linux) imported, network adapter verified via `ifconfig`
- [ ] Comfortable with core Linux commands (file ops, permissions, package management, archives)
- [ ] Nessus installed on Kali, registered (Essentials), service running
- [ ] Nessus baseline scan run against Metasploitable3/2 (next practical step)

---

*Note: Source material adapted from South Metro TAFE lecture content (Simon Htike / Daniel Wozencroft). This document is a study summary / portfolio artifact; lab screenshots and scan results will be linked separately once produced.*
