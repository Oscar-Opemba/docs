# Module 9 — Advanced Metasploit

## Objectives

Automate repetitive workflows with resource scripts, understand the basic anatomy of a Metasploit module well enough to read and lightly modify one, integrate Nmap data into database-driven assessments, and use the database as the backbone of a structured engagement.

## Theory

Once the individual commands from earlier modules are muscle memory, the next level of competence is removing yourself from repetitive manual steps — both for speed and for consistency (a scripted lab setup runs the same way every time; a manually-typed one doesn't).

## Deep Technical Explanation

### Resource scripts

A resource script (`.rc` file) is just a plain text file of msfconsole commands, executed in order:

```text
# recon.rc
workspace -a auto-lab
db_nmap -sV 192.168.56.101
use auxiliary/scanner/smb/smb_version
set RHOSTS 192.168.56.101
run
```

```bash
msfconsole -r recon.rc
# or, from inside an already-running console:
msf6 > resource recon.rc
```

### Automation beyond resource scripts

```bash
msfconsole -q -x "workspace -a auto-lab; db_nmap -sV 192.168.56.101; exit"
```

`-x` runs a single semicolon-separated command string non-interactively, useful for cron-scheduled recurring scans of a long-running lab, or for chaining msfconsole into a larger shell script pipeline.

### Custom module basics

Every exploit/auxiliary module is a Ruby class. A minimal auxiliary scanner skeleton looks like this:

```ruby
class MetasploitModule < Msf::Auxiliary
  include Msf::Exploit::Remote::Tcp
  include Msf::Auxiliary::Scanner

  def initialize(info = {})
    super(update_info(info,
      'Name'        => 'Example Banner Grabber',
      'Description' => 'Connects to a TCP port and prints whatever banner is returned',
      'Author'      => ['your-handle'],
      'License'     => MSF_LICENSE
    ))
    register_options([ Opt::RPORT(80) ])
  end

  def run_host(ip)
    connect
    banner = sock.get_once
    print_status("#{ip}:#{datastore['RPORT']} -> #{banner.inspect}")
    disconnect
  end
end
```

Save this under `~/.msf4/modules/auxiliary/scanner/custom_banner_grab.rb`, then in msfconsole:

```bash
msf6 > reload_all
msf6 > use auxiliary/scanner/custom_banner_grab
```

This is intentionally a minimal, harmless teaching example (a banner grabber) — the point is to demystify the structure (`initialize`/options registration, a `run`/`run_host` method) so that reading *existing* modules in `/usr/share/metasploit-framework/modules/` stops feeling like reading a foreign language.

### Module modification

Editing an existing module (e.g., to add a missing target signature or fix a broken default) is usually as simple as copying it into your user module directory (`~/.msf4/modules/...`, mirroring the same path structure) and editing the copy — Metasploit prioritizes user-directory modules over the system ones, so your edits don't get overwritten by the next `apt upgrade`.

### Nmap integration

```bash
nmap -sV -oX scan.xml 192.168.56.0/24
msf6 > db_import scan.xml
msf6 > hosts
msf6 > services
```

`db_import` accepts Nmap XML (and several other scanner formats), letting you bring in results gathered outside msfconsole — useful when a teammate ran the initial recon with standalone Nmap.

### Database-driven assessments

```bash
msf6 > hosts -c address,os_name,os_flavor
msf6 > services -c port,proto,name,info
msf6 > vulns
msf6 > creds
msf6 > db_export -f xml my-engagement-export.xml
```

Treating the database as the engagement's single source of truth means your final report can be generated largely from structured queries against `hosts`/`services`/`vulns`/`creds` rather than reconstructed from scrollback and memory.

## Practical Examples

```bash
# Build and run a one-shot recon script
cat > recon.rc << 'EOF'
workspace -a adv-lab
db_nmap -sV -p 21,22,80,139,445 192.168.56.101
services
EOF
msfconsole -q -r recon.rc
```

**Expected output:** msfconsole launches, runs each line silently in order, and ends sitting at the `services` table output for the scanned ports.

## Common Mistakes

- Writing a custom module directly under the read-only system modules path instead of `~/.msf4/modules/`, then losing the edit on the next package update.
- Forgetting `reload_all` after adding/editing a module, then being confused why `use` can't find it.
- Treating `db_import` as a replacement for understanding what the data actually means, rather than just a convenience for ingestion.
- Building resource scripts with hardcoded IPs and never parameterizing them, making them brittle the moment the lab network changes.

## Real-World Usage

Consulting teams commonly maintain a library of resource scripts for "day one of every engagement" tasks (workspace creation, standard scan sweeps, standard reporting export commands), turning what used to be 20 minutes of manual setup into a single `msfconsole -r day1.rc`.

## Guided Lab — Build a Reusable Recon Script

**Objective:** Write, run, and verify your own resource script.

**Steps:**
1. Write a `.rc` file that creates a workspace, runs `db_nmap -sV` against your lab target, and prints `services`.
2. Run it with `msfconsole -q -r yourscript.rc`.
3. Confirm the workspace and scan results persist by opening msfconsole normally afterward and checking `workspace` / `services`.

**Expected output:** Identical results to manually typing the same commands, but reproducible with one command.

**Verification:** Re-run the same script against a second target IP by editing only the IP — confirm it still works cleanly.

**Troubleshooting:** If the script seems to "hang," it's often just `db_nmap` taking longer than expected on a full port range — add `-p` with a smaller port list while testing the script logic itself.

## Challenge Lab

Write a minimal custom auxiliary module (based on the skeleton above) that grabs a banner from a port of your choice, load it with `reload_all`, and successfully run it against Metasploitable2 — entirely typed from understanding, not copy-pasted verbatim.

## Review Questions

1. What is a resource script, mechanically?
2. Where should custom/modified modules live so they survive package upgrades?
3. What command must you run after adding a new module file before `use` can find it?
4. What format does `db_import` commonly accept from standalone Nmap scans?
5. Why treat the Metasploit database as the engagement's single source of truth?

## Answers

1. A plain text file containing a sequence of msfconsole commands, executed in order when loaded via `-r` or the `resource` command.
2. Under your user module directory, `~/.msf4/modules/...`, mirroring the same subdirectory structure as the system modules.
3. `reload_all`.
4. Nmap's XML output format (`-oX`).
5. Because it gives you one consistent, queryable record of every host, service, vulnerability, and credential discovered, which both downstream modules and final reporting can rely on instead of reconstructing data from memory or scrollback.
