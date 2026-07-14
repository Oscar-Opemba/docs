# Module 3 — Enumeration Using Metasploit

## Objectives

Use Metasploit's auxiliary scanner modules to perform host discovery, port scanning, service/version detection, and protocol-specific enumeration (SMB, FTP, SSH, HTTP, SNMP) against lab targets, and have the results automatically populate the database for later use.

## Theory

Enumeration is the unglamorous, decisive phase of any assessment: the more accurately you map what's running where, the more your exploitation phase becomes "selecting the right key" instead of "guessing." Metasploit's auxiliary scanner modules don't replace Nmap — they complement it, because results land directly in the Metasploit database (`hosts`, `services`, `creds` tables) where they're instantly available to every other module afterward.

## Deep Technical Explanation

### Host discovery

```bash
msf6 > db_nmap -sn 192.168.56.0/24      # ping sweep, populates `hosts`
msf6 > hosts                             # review what was found
```

`db_nmap` runs a real Nmap scan but pipes the XML output straight into the Metasploit database, which is strictly better than running Nmap standalone if you're going to work inside msfconsole anyway.

### Port scanning

```bash
msf6 > use auxiliary/scanner/portscan/tcp
msf6 auxiliary(scanner/portscan/tcp) > set RHOSTS 192.168.56.101
msf6 auxiliary(scanner/portscan/tcp) > set PORTS 1-1000
msf6 auxiliary(scanner/portscan/tcp) > set THREADS 20
msf6 auxiliary(scanner/portscan/tcp) > run
```

### Service detection and banner grabbing

Once ports are known, version-specific auxiliary modules grab banners and identify exact software versions, which is what eventually lets you match a target to an exploit module.

```bash
msf6 > use auxiliary/scanner/ftp/ftp_version
msf6 > use auxiliary/scanner/ssh/ssh_version
msf6 > use auxiliary/scanner/http/http_version
msf6 > use auxiliary/scanner/smb/smb_version
```

Each follows the same pattern: `set RHOSTS`, `run`, read the banner, check `services` afterward.

### SMB enumeration

```bash
msf6 > use auxiliary/scanner/smb/smb_version
msf6 > use auxiliary/scanner/smb/smb_enumshares
msf6 > use auxiliary/scanner/smb/smb_enumusers
```

`smb_enumshares` reveals accessible shares (and whether they allow anonymous/guest access — extremely common on misconfigured legacy targets like Metasploitable2). `smb_enumusers` enumerates valid local usernames via SAM/RID-cycling style techniques, which directly feeds later credential-brute-force or password-spray steps.

### FTP enumeration

```bash
msf6 > use auxiliary/scanner/ftp/ftp_version
msf6 > use auxiliary/scanner/ftp/anonymous
```

`anonymous` specifically checks whether the FTP service allows unauthenticated anonymous login — a classic, still-common misconfiguration.

### SSH enumeration

```bash
msf6 > use auxiliary/scanner/ssh/ssh_version
```

Beyond banner grabbing, `auxiliary/scanner/ssh/ssh_login` (with a wordlist) tests credential pairs — useful in labs where weak/default credentials are part of the intended path (Metasploitable2's `msfadmin:msfadmin` is the canonical teaching example).

### HTTP enumeration

```bash
msf6 > use auxiliary/scanner/http/http_version
msf6 > use auxiliary/scanner/http/dir_scanner
msf6 > use auxiliary/scanner/http/robots_txt
```

These complement (not replace) dedicated web tools like gobuster/ffuf/Burp — Metasploit's HTTP scanners are convenient when you're already living inside msfconsole and want quick coverage without context-switching tools.

### SNMP enumeration

```bash
msf6 > use auxiliary/scanner/snmp/snmp_enum
```

SNMP, especially with default community strings (`public`/`private`), can leak an enormous amount: running processes, installed software, routing tables, sometimes even credentials embedded in device configs.

## Practical Examples — Full Sweep Against Metasploitable2

```bash
msf6 > workspace -a m2-lab
msf6 > db_nmap -sV -p- 192.168.56.101
msf6 > services
msf6 > use auxiliary/scanner/smb/smb_version
msf6 auxiliary(scanner/smb/smb_version) > set RHOSTS 192.168.56.101
msf6 auxiliary(scanner/smb/smb_version) > run
```

**Expected output:** `services` after the Nmap scan should list dozens of open ports typical of Metasploitable2 (FTP 21, SSH 22, Telnet 23, SMTP 25, DNS 53, HTTP 80, RPC 111, NetBIOS/SMB 139/445, MySQL 3306, and more). The SMB version scan should report a Samba version string consistent with the deliberately outdated build shipped on Metasploitable2.

## Common Mistakes

- Running exploitation modules before enumeration is actually complete — "spray and pray" instead of matching exploit to confirmed version.
- Forgetting `set RHOSTS` (plural) on scanner modules versus `RHOST` (singular) expected by some single-target exploit modules — mismatched variable names are one of the most common "why isn't this working" moments for beginners.
- Not checking `services` and `hosts` after scans, and instead relying purely on raw console scroll-back, which gets lost as the session continues.
- Scanning too aggressively (`THREADS` very high) against fragile lab VMs, causing the target itself to crash before you even get to exploitation.

## Real-World Usage

On real engagements, you'll typically run Nmap (or `db_nmap`) first for broad coverage, then selectively bring in Metasploit's protocol-specific scanners only for services that look interesting or where Nmap's default scripts didn't give enough detail (SMB share enumeration in particular). The database becomes your single source of truth that you reference for the rest of the engagement and use to generate evidence for the report.

## Guided Lab — Full Enumeration Pass

**Objective:** Build a complete service map of Metasploitable2 inside the Metasploit database.

**Setup:** Kali and Metasploitable2 on the same host-only network; confirm connectivity with `ping`.

**Steps:**
1. `workspace -a m2-lab`
2. `db_nmap -sV -O 192.168.56.101` (replace with your actual target IP)
3. `services -c name,port,proto,info`
4. `use auxiliary/scanner/ftp/anonymous` → set RHOSTS → run
5. `use auxiliary/scanner/smb/smb_enumshares` → set RHOSTS → run

**Expected output:** Anonymous FTP should report as allowed on Metasploitable2 by design. SMB share enumeration should reveal at least one world-readable share.

**Verification:** `creds` and `loot` commands should reflect anything captured; `services` should now have an "info" column populated for FTP and SMB rows.

**Troubleshooting:** If `db_nmap` hangs, confirm the target IP is actually reachable with a plain `ping` first — a misconfigured host-only adapter is the most common cause.

## Challenge Lab

Without referring back to the command list above, fully enumerate DVWA's underlying web stack (server banner, any exposed `robots.txt`, directory scan for at least three hidden paths) using only Metasploit auxiliary modules, and record the exact module path and options you used for each step.

## Review Questions

1. What's the practical advantage of `db_nmap` over running Nmap directly in a separate terminal?
2. Which SMB auxiliary module specifically checks for accessible shares?
3. Why is anonymous FTP considered a misconfiguration worth flagging in a report?
4. What's the difference between `RHOST` and `RHOSTS` and why does it matter?
5. Name two pieces of information SNMP enumeration with default community strings can leak.

## Answers

1. Results are written directly into the Metasploit database, so every later module and the final report can reference the same structured data instead of you re-parsing raw scan output.
2. `auxiliary/scanner/smb/smb_enumshares`.
3. It allows unauthenticated users to read (and sometimes write) files on the server, which is a common foothold for further compromise and a textbook reportable finding.
4. `RHOST` (singular) is used by modules designed for exactly one target; `RHOSTS` (plural) is used by modules that accept a range/list of targets. Setting the wrong one silently does nothing in many modules.
5. Running processes/installed software lists and routing table information (also commonly: system descriptions, sometimes embedded credentials in device configs).
