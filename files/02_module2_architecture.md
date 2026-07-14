# Module 2 — Understanding Metasploit Architecture

## Objectives

Understand each module type Metasploit ships, how payloads are structured internally (staged vs non-staged), what encoders and NOP generators actually do, and the high-level mechanics of how Meterpreter gets from "exploit fires" to "interactive shell on your screen."

## Theory

Metasploit organizes everything into **module types**. Each type has a different job in the exploitation pipeline, and understanding the boundary between them is what separates someone who memorizes commands from someone who can reason about a new target.

## Deep Technical Explanation

### Exploit modules

An exploit module is Ruby code that triggers a specific vulnerability — a buffer overflow, a deserialization bug, an authentication bypass, a command injection — in order to get the target to execute attacker-supplied code or grant attacker-controlled access. Every exploit module declares a `check` method (safely tests whether the target is vulnerable, ideally without triggering the bug) and an `exploit` method (does the actual triggering). Exploits live under `modules/exploits/<platform>/<service>/`.

### Payload modules

A payload is the code that actually runs once the exploit succeeds — it's the "what happens next." Payloads are organized by:
- **Singles** — self-contained, do one thing and don't call back for more code (e.g., `windows/exec`, which just runs a command).
- **Stagers** — small, reliable pieces of shellcode whose only job is to open a connection back to the attacker and download a larger second-stage payload. Small size matters because exploit primitives (especially overflow-based ones) often have a strict byte budget.
- **Stages** — the larger payload the stager downloads and executes, e.g., the full Meterpreter DLL.

### Auxiliary modules

Auxiliary modules do everything that isn't "exploit a vulnerability to get code execution": port scanners, service version scanners, login brute-forcers, fuzzers, DoS modules, protocol-specific enumeration tools. They live under `modules/auxiliary/`.

### Post modules

Post modules run *after* you already have a session (a shell or Meterpreter session) and perform actions on the already-compromised host: gathering system info, dumping credentials, enumerating for privilege escalation, pivoting setup. They live under `modules/post/`.

### Encoders

Encoders transform the byte representation of shellcode so it avoids "bad characters" the vulnerable code path can't handle (e.g., null bytes that terminate a C string early) and, historically, to reduce the chance of trivial signature-based detection. The most well-known is `x86/shikata_ga_nai`, a polymorphic XOR additive-feedback encoder. It's important to understand that modern encoders do **not** reliably bypass modern antivirus/EDR — their real job is byte-compatibility with the exploit's constraints, not stealth.

### NOP generators

NOP ("no operation") generators produce padding instructions used to build a reliable landing zone in memory for shellcode in certain overflow-style exploits (a "NOP sled"), increasing the odds that an imprecise jump still lands somewhere that eventually executes your payload.

### Meterpreter architecture

Meterpreter is Metasploit's advanced, in-memory post-exploitation agent. Unlike a single-purpose payload, it's an extensible platform: a small core loads first, then loads "extensions" (stdapi for filesystem/process/network commands, priv for privilege-related actions, incognito for token manipulation, etc.) on demand over an encrypted TLV (Type-Length-Value) protocol channel back to msfconsole. Because it runs reflectively in memory rather than as a normal process written to disk, it leaves a much smaller forensic footprint than a traditional dropped executable.

### Staged vs non-staged payloads

A staged payload (e.g., `windows/meterpreter/reverse_tcp`) splits delivery into a tiny stager plus a larger stage downloaded on connect — useful when the exploit's available space for shellcode is small, but it requires the stager and `multi/handler` to successfully negotiate the stage transfer, which is one more thing that can fail (especially across flaky networks or aggressive endpoint defenses). A non-staged payload (e.g., `windows/meterpreter_reverse_tcp`, note the underscore) bundles everything into one block delivered at once — larger, but simpler and more reliable when you have the space for it.

### High-level exploitation workflow

```
1. Attacker selects exploit module that matches a known vulnerable service
2. Attacker selects a compatible payload (must match target OS/arch)
3. Attacker sets RHOSTS/RPORT (target) and LHOST/LPORT (attacker callback address)
4. exploit/run sends the crafted trigger to the vulnerable service
5. Vulnerable service's flawed code path executes attacker-supplied instructions
6. Stager (if staged) connects back to attacker's listener
7. Listener (multi/handler or the exploit's own handler) sends the stage
8. Stage (e.g., Meterpreter) loads in memory and opens a session
9. Session appears in msfconsole's session table, ready for interaction
```

## Practical Examples

```bash
msf6 > search type:payload platform:windows meterpreter reverse_tcp
msf6 > info payload/windows/meterpreter/reverse_tcp
msf6 > show encoders
msf6 > show nops
```

Compare a staged and non-staged payload side by side:

```bash
msf6 > info payload/windows/meterpreter/reverse_tcp        # staged
msf6 > info payload/windows/meterpreter_reverse_tcp         # non-staged
```

Look specifically at the "Total size" difference reported for each — this is the staged/non-staged trade-off made concrete.

## Common Mistakes

- Treating encoders as an "antivirus bypass button." In 2026 lab and exam environments, default `shikata_ga_nai` encoding gets flagged by any modern AV almost immediately — its purpose is byte sanitization, not evasion.
- Picking a payload architecture (x86 vs x64) that doesn't match the target process, causing a session to open and immediately die.
- Confusing a stager failing to fetch its stage (network/firewall issue) with the exploit itself failing — these produce similar-looking failures but need different troubleshooting.

## Real-World Usage

Professional testers constantly reason in these layers: "this web app has a deserialization bug (exploit), I need a payload small enough to fit in the gadget chain (payload size matters), and once I land I want full post-exploitation tooling (Meterpreter), but the corporate EDR is aggressive, so maybe a non-meterpreter single payload that does one quiet thing is safer for this step." That reasoning only works if you understand the architecture, not just the commands.

## Guided Lab — Comparing Module Types

**Objective:** Build a mental map of the five core module types by inspecting real ones.

**Steps:**
1. `search type:auxiliary smb` — note how many results, scan the names.
2. `search type:post windows gather` — note the naming pattern `post/windows/gather/...`.
3. `info auxiliary/scanner/portscan/tcp` — read the full description and options.
4. `info post/multi/gather/env` — compare its "Description" field to the auxiliary module's.

**Expected output:** You should be able to articulate, in your own words, why `portscan/tcp` is auxiliary (it doesn't exploit anything, it just gathers information) while `gather/env` is post (it requires an existing session).

**Verification:** Explain out loud (or write down) the difference between auxiliary and post in one sentence each, without looking back at this file.

**Troubleshooting:** If `search` feels slow, run `db_status` first — without a working database, Metasploit falls back to a much slower in-memory search index.

## Challenge Lab

Find one example each of: a staged Windows Meterpreter payload, a non-staged Linux Meterpreter payload, one encoder usable for x86 shellcode, and one NOP generator. Write down all four exact module paths from memory-checked `search`/`show` output, no copy-pasting from this file.

## Review Questions

1. What is the functional difference between a stager and a stage?
2. Why do exploit developers care about payload size at all?
3. What is the actual purpose of an encoder like shikata_ga_nai?
4. Why does Meterpreter load extensions on demand instead of bundling everything at once?
5. Name the five core Metasploit module types and one-sentence what each does.

## Answers

1. A stager is a small, reliable piece of code whose only job is to call back and fetch a larger payload; the stage is that larger payload (e.g., full Meterpreter) which the stager downloads and executes.
2. Many exploitation primitives (buffer overflows especially) only give you a small, fixed amount of space to inject shellcode, so a smaller initial payload is more likely to fit and succeed.
3. To transform shellcode bytes so they avoid characters the vulnerable code path can't tolerate (like null bytes), not to defeat modern antivirus/EDR.
4. To keep the initial in-memory footprint small and flexible — you only load the capabilities (filesystem, privilege escalation tooling, token manipulation) you actually need for a given target.
5. Exploit (triggers a vulnerability), Payload (the code that runs after triggering), Auxiliary (scanning/enumeration/brute-force, no code execution), Post (runs after a session already exists), Encoder (transforms shellcode bytes for compatibility).
