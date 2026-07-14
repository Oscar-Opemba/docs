# Module 1 — Foundations

## Objectives

By the end of this module you will be able to explain what Metasploit is and where it came from, install and update it correctly on Kali, configure its PostgreSQL-backed database, navigate msfconsole fluently, and organize your work using workspaces.

## Theory

Metasploit is an open-source exploitation framework originally released in 2003 by HD Moore. The earliest versions were written in Perl; the project was rewritten in Ruby for version 3.0 in 2007, which is the codebase the modern framework descends from. Rapid7 acquired the project in 2009 and has maintained both the free, open-source **Metasploit Framework** (what ships on Kali) and a commercial product, **Metasploit Pro**, which adds a GUI, reporting, and team-collaboration features on top of the same underlying engine.

At its core, Metasploit is a structured library of exploit code, payloads, and supporting utilities, plus a console (msfconsole) that lets you search, configure, and fire that code against targets in a consistent way. The value isn't that Metasploit invents new vulnerabilities — it's that it standardizes how you weaponize a known vulnerability so you don't have to hand-write a new exploit script every time.

## Deep Technical Explanation

### Architecture at a glance

Metasploit Framework is built in layers:

- **Rex** (Ruby Extension Library) — low-level networking, sockets, protocols, and the SSL/encoding primitives almost everything else depends on.
- **Msf::Core** — the core API: module loading, the session manager, the event system, the database layer.
- **Msf::Base** — higher-level conveniences built on Core, including the plugin system and mixins that modules use (HTTP client mixin, SMB mixin, etc.).
- **Modules tree** — the actual content: exploits, payloads, auxiliary, post, encoders, and nops, organized in `/usr/share/metasploit-framework/modules/` on Kali.
- **msfconsole** — the primary interactive interface sitting on top of all of the above. msfrpcd, msfvenom, and msfdb are separate executables that talk to the same underlying libraries.

### Editions

- **Framework (free, open source)** — what you have on Kali. Command-line driven via msfconsole, fully scriptable.
- **Pro (commercial)** — adds a web GUI, automated workflows, social-engineering campaign tooling, and reporting aimed at consulting engagements. The module engine underneath is the same Framework code.

This course focuses entirely on Framework, since that's the version every certification (OSCP, eJPT, PNPT) and every CTF environment expects you to know cold.

## Practical Examples

### Checking and updating Metasploit on Kali

```bash
# Confirm version
msfconsole -v

# Update via apt (Kali ships Metasploit as a maintained package)
sudo apt update && sudo apt install metasploit-framework -y

# Inside msfconsole, you can also check for framework updates
msfconsole
msf6 > version
```

Expected output: a version string like `Framework: 6.x.x-dev` followed by a console version. If `msfconsole` doesn't exist at all, the apt install step above resolves it.

### Initializing and checking the database

```bash
sudo msfdb init
sudo msfdb status
```

Expected output from `msfdb status` should show the database service running and a database user/role created. Inside msfconsole, confirm the console is actually connected:

```bash
msf6 > db_status
```

Expected output: `[*] Connected to msf database: postgresql://...`. If you see `[-] No database support`, see Common Mistakes below.

### msfconsole basics

```bash
msf6 > help                  # list all console commands
msf6 > banner                # re-print the ASCII banner (harmless, good for screenshots)
msf6 > search type:exploit smb
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > show options
msf6 exploit(multi/handler) > back
msf6 > info exploit/multi/handler
msf6 > exit
```

### Workspaces

```bash
msf6 > workspace                  # list workspaces, * marks active one
msf6 > workspace -a internal-lab  # create and switch to a new workspace
msf6 > workspace internal-lab     # switch to an existing workspace
msf6 > workspace -d old-lab       # delete a workspace
```

Workspaces silo your hosts, services, credentials, and loot per engagement or per lab, so a Metasploitable2 lab and a DVWA lab don't pollute each other's data.

## Common Mistakes

- Forgetting to run `sudo msfdb init` after a fresh Kali install, then wondering why `hosts` and `services` return nothing.
- Running msfconsole with `sudo` inconsistently — sometimes as root, sometimes as your normal user — which causes it to connect to two different database configurations and "lose" data between sessions.
- Never creating workspaces, then six months later having one giant unsorted `default` workspace with unrelated labs mixed together.
- Assuming `apt upgrade` always pulls the very latest modules instantly; Kali's package sometimes lags behind the bleeding-edge GitHub `master` branch by days.

## Real-World Usage

On real engagements, the very first thing a competent tester does after opening a terminal is create a workspace named after the client/engagement (`workspace -a acme-corp-2026-q2`) and confirm `db_status` is healthy, because every host, service, vulnerability, and credential discovered for the rest of the engagement gets logged automatically to that workspace — this becomes the backbone of the final report.

## Guided Lab — Install, Configure, and Explore

**Objective:** Get a clean, database-backed Metasploit environment running and prove you can navigate it.

**Setup:** Kali Linux VM, internet access for `apt`.

**Steps:**
1. `sudo apt update && sudo apt install metasploit-framework -y`
2. `sudo msfdb init`
3. `msfconsole` — wait for the banner.
4. `db_status` — confirm connection.
5. `workspace -a m2-lab`
6. `search type:exploit platform:unix name:vsftpd`

**Expected output:** Step 6 should return at least one row referencing `exploit/unix/ftp/vsftpd_234_backdoor`, with a "Rank" column (e.g., Excellent).

**Verification:** Run `workspace` and confirm `m2-lab` is marked active with `*`.

**Troubleshooting:**
- `db_status` shows no database → re-run `sudo msfdb init`, then fully restart msfconsole.
- `search` returns nothing at all (not even unrelated results) → your modules cache may be stale; run `msf6 > reload_all` or restart msfconsole.

## Challenge Lab

Without referring back to the steps above: create a second workspace called `dvwa-lab`, switch between `m2-lab` and `dvwa-lab` at least twice using both forms of the `workspace` command shown earlier, then use `search` to find any three auxiliary scanner modules related to SMB. Document the exact module paths you find.

## Review Questions

1. What year was Metasploit originally released, and in what language was it first written?
2. What company currently maintains Metasploit, and since when?
3. What is the difference between Metasploit Framework and Metasploit Pro?
4. Which command initializes the PostgreSQL-backed database for Metasploit on Kali?
5. What is the purpose of a workspace?
6. Which msfconsole command shows whether the database connection is healthy?

## Answers

1. 2003, originally written in Perl.
2. Rapid7, which acquired the project in 2009.
3. Framework is the free, open-source, console-driven core engine; Pro is a commercial product built on top of the same engine that adds a GUI, automation workflows, and reporting for consulting use.
4. `sudo msfdb init`.
5. To isolate the hosts, services, credentials, vulnerabilities, and loot of one engagement/lab from another inside the shared database.
6. `db_status`.
