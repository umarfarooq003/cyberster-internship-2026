# Week 02: Network Enumeration & Service Vulnerability Discovery

**Target:** Metasploitable2 (`192.168.56.102` — replace with your actual IP)
**Attacker:** Kali Linux (`192.168.56.101` — replace with your actual IP)
**Environment:** VirtualBox/VMware Host-Only or NAT Network (isolated, no internet-facing bridge)

> ⚠️ **Scope Warning:** Only run these commands against Metasploitable2 or another machine you own/are authorized to test. Never point these at production systems or networks you don't control.

---

## 0. Lab Setup

1. Import Metasploitable2 OVA into VirtualBox/VMware.
2. Set network adapter on **both** Kali and Metasploitable2 to the same **Host-Only Adapter** or **NAT Network** (not Bridged).
3. Boot Metasploitable2, log in with `msfadmin` / `msfadmin`.
4. Get its IP from inside the VM:
   ```bash
   ifconfig
   ```
5. From Kali, confirm connectivity and discover the IP if needed:
   ```bash
   sudo netdiscover -r 192.168.56.0/24
   # or
   sudo nmap -sn 192.168.56.0/24
   ```
6. Export a variable for convenience in every terminal session:
   ```bash
   export TARGET=192.168.56.102
   ```

---

## Task 01 — Advanced Network Scanning with Nmap

### 1.1 TCP Connect Scan (`-sT`)
Completes full 3-way handshake (SYN → SYN/ACK → ACK). Slower, but doesn't need root privileges, and gets logged by the target application layer.
```bash
sudo nmap -sT -p- -oN task01_tcp_connect.txt $TARGET
```

### 1.2 SYN Stealth Scan (`-sS`)
Sends SYN, receives SYN/ACK, then sends RST instead of completing the handshake. Faster, requires raw socket access (root), and is less likely to be logged at the app layer.
```bash
sudo nmap -sS -p- -oN task01_syn_stealth.txt $TARGET
```

**Compare:**
```bash
time sudo nmap -sT -p- $TARGET
time sudo nmap -sS -p- $TARGET
```
Record both times in your report table.

### 1.3 UDP Scan — Top 100 Ports
```bash
sudo nmap -sU --top-ports 100 -oN task01_udp_top100.txt $TARGET
```
Focused check on DNS/SNMP/DHCP specifically:
```bash
sudo nmap -sU -p 53,161,67,68 -oN task01_udp_dns_snmp_dhcp.txt $TARGET
```

### 1.4 Timing Templates (T0–T5)
```bash
sudo nmap -sS -T0 -p 21,22,23,25,80,445 -oN task01_T0.txt $TARGET   # Paranoid
sudo nmap -sS -T1 -p 21,22,23,25,80,445 -oN task01_T1.txt $TARGET   # Sneaky
sudo nmap -sS -T2 -p 21,22,23,25,80,445 -oN task01_T2.txt $TARGET   # Polite
sudo nmap -sS -T3 -p 21,22,23,25,80,445 -oN task01_T3.txt $TARGET   # Normal (default)
sudo nmap -sS -T4 -p 21,22,23,25,80,445 -oN task01_T4.txt $TARGET   # Aggressive
sudo nmap -sS -T5 -p 21,22,23,25,80,445 -oN task01_T5.txt $TARGET   # Insane
```
Record scan duration for each — T0/T1 can take minutes-to-hours on full port ranges; T4/T5 finish in seconds.

---

## Task 02 — Service Fingerprinting & Banner Grabbing

### 2.1 Version Detection
```bash
sudo nmap -sV -p- -oN task02_version_scan.txt $TARGET
```

### 2.2 Aggressive Scan (adds OS detection, default scripts, traceroute)
```bash
sudo nmap -A -oN task02_aggressive_scan.txt $TARGET
```

### 2.3 Manual Banner Grabbing (Netcat / Telnet)
```bash
nc -nv $TARGET 21     # FTP -> expect "220 (vsFTPd 2.3.4)"
nc -nv $TARGET 22     # SSH -> expect "SSH-2.0-OpenSSH_4.7p1 Debian..."
nc -nv $TARGET 25     # SMTP -> expect Postfix banner
nc -nv $TARGET 23     # Telnet login banner

# HTTP banner via manual request
nc -nv $TARGET 80
GET / HTTP/1.1
Host: target
<press Enter twice>

# Telnet alternative
telnet $TARGET 80
```

### 2.4 SMB Enumeration (Port 445/139)
```bash
enum4linux -a $TARGET

# Alternative
enum4linux-ng -A $TARGET

# Manual Samba tools
smbclient -L //$TARGET/ -N
rpcclient -U "" -N $TARGET
```

---

## Task 03 — Nmap Scripting Engine (NSE)

### 3.1 Discovery + Safe Categories
```bash
sudo nmap --script "discovery and safe" -p- -oN task03_discovery_safe.txt $TARGET
```

### 3.2 Targeted Scripts
```bash
sudo nmap --script http-enum -p 80 -oN task03_http_enum.txt $TARGET
sudo nmap --script ssl-cert,ssl-enum-ciphers -p 443 -oN task03_ssl.txt $TARGET
sudo nmap --script smb-os-discovery,smb-enum-shares,smb-enum-users -p 445 -oN task03_smb.txt $TARGET
```

### 3.3 Vulnerability Scripts
```bash
sudo nmap --script vuln -p- -oN task03_vuln_scan.txt $TARGET
```

### 3.4 Map Findings to CVEs
Cross-reference version numbers from Task 02 and NSE `vuln` output against:
- https://nvd.nist.gov
- https://vulners.com
- Offline: `searchsploit <service> <version>`

Expected hits on Metasploitable2:

| Port | Service | Version | CVE |
|------|---------|---------|-----|
| 21 | vsftpd | 2.3.4 | CVE-2011-2523 (backdoor RCE) |
| 22 | OpenSSH | 4.7p1 | CVE-2008-5161 |
| 139/445 | Samba | 3.X | CVE-2007-2447 (usermap_script RCE) |
| 1099 | Java RMI | — | CVE-2011-3556 |
| 3632 | distccd | — | CVE-2004-2687 |
| 6667 | UnrealIRCd | — | CVE-2010-2075 (backdoor) |
| 5432 | PostgreSQL | 8.3 | Default creds `postgres:postgres` |
| 8009/8180 | Tomcat/AJP | — | Default creds `tomcat:tomcat`, WAR upload RCE |

---

## Task 04 — Stealth & Firewall Evasion

> Note: Metasploitable2 has no IDS/firewall by default. Document these as **syntax/mechanics demonstrations** rather than measured detection-evasion results, unless you've added Snort/Suricata to the segment.

### 4.1 Packet Fragmentation
```bash
sudo nmap -f -p 21,22,80,445 -oN task04_frag.txt $TARGET
sudo nmap -f -f -p 21,22,80,445 -oN task04_frag_double.txt $TARGET   # smaller fragments
sudo nmap --mtu 24 -p 21,22,80,445 -oN task04_mtu.txt $TARGET
```

### 4.2 Decoy Scanning
```bash
sudo nmap -D RND:10 -p 21,22,80,445 -oN task04_decoy_random.txt $TARGET
sudo nmap -D decoy1,decoy2,ME,decoy3 -p 21,22,80,445 -oN task04_decoy_manual.txt $TARGET
```

### 4.3 Source Port Spoofing
```bash
sudo nmap --source-port 53 -p 21,22,80,445 -oN task04_srcport53.txt $TARGET
```

### 4.4 (Optional Bonus) Test Against a Real IDS
If you want actual evasion data rather than syntax demos:
```bash
sudo apt install snort -y
sudo snort -i eth0 -A console -c /etc/snort/snort.conf
```
Run each Task 04 command in another terminal and observe which ones trigger Snort alerts vs. which slip through.

---

## Deliverable Checklist

- [ ] **Nmap Output Analysis** — screenshots of `task01`–`task03` outputs + 2–3 sentence explanation per finding
- [ ] **Service Table** — Port | Service | Version | Potential Vulnerability (see Task 03 table above as starting point)
- [ ] **Evasion Write-up** — which Task 04 flags worked quietly vs. were noisy; note the "no default IDS on Metasploitable2" limitation if applicable
- [ ] **Actionable Insights** — identify the single most likely Initial Access vector

### Suggested Actionable Insight (Metasploitable2)
Port 21 (**vsftpd 2.3.4**) or port 1524 (**ingreslock backdoor**, `nc $TARGET 1524`) are the strongest Initial Access candidates — both offer pre-built, unauthenticated root shell access with zero exploit development required. vsftpd 2.3.4 has a public Metasploit module: `exploit/unix/ftp/vsftpd_234_backdoor`.

---

## Quick Reference: All Commands in Order

```bash
export TARGET=192.168.56.102

# Task 01
sudo nmap -sT -p- -oN task01_tcp_connect.txt $TARGET
sudo nmap -sS -p- -oN task01_syn_stealth.txt $TARGET
sudo nmap -sU --top-ports 100 -oN task01_udp_top100.txt $TARGET
sudo nmap -sS -T4 -p 21,22,23,25,80,445 -oN task01_T4.txt $TARGET

# Task 02
sudo nmap -sV -p- -oN task02_version_scan.txt $TARGET
sudo nmap -A -oN task02_aggressive_scan.txt $TARGET
nc -nv $TARGET 21
enum4linux -a $TARGET

# Task 03
sudo nmap --script "discovery and safe" -p- -oN task03_discovery_safe.txt $TARGET
sudo nmap --script vuln -p- -oN task03_vuln_scan.txt $TARGET

# Task 04
sudo nmap -f -p 21,22,80,445 -oN task04_frag.txt $TARGET
sudo nmap -D RND:10 -p 21,22,80,445 -oN task04_decoy_random.txt $TARGET
sudo nmap --source-port 53 -p 21,22,80,445 -oN task04_srcport53.txt $TARGET
```
