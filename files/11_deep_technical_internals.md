# Deep Technical Internals

This file consolidates the "how does this actually work under the hood" material referenced across earlier modules into one dedicated technical reference.

## How Meterpreter Works Internally

Meterpreter is delivered as a Reflective DLL (on Windows) or an equivalent in-memory loader (on other platforms) — meaning the agent's code is injected and executed directly in process memory without ever being written to disk as a standalone executable. This is the core reason it's both stealthier than a traditional dropped binary and entirely gone the moment the hosting process exits or the machine reboots (hence why persistence, covered conceptually in Module 5, is a separate, deliberate decision rather than automatic).

Internally, Meterpreter communicates over a custom **TLV (Type-Length-Value) protocol**: every command sent from msfconsole to the agent, and every response sent back, is packaged as a sequence of typed, length-prefixed fields. This is what lets a single consistent "session" abstraction work across wildly different underlying transports (plain TCP, HTTP, HTTPS) and across different target operating systems — the protocol doesn't care what's carrying it.

A minimal **core** loads first with just enough capability to receive further extensions. Functional command sets (`stdapi` for filesystem/process/networking basics, `priv` for privilege-related actions, `incognito` for token manipulation, `kiwi` for Mimikatz-style credential extraction) are loaded on demand over that same TLV channel, which is why you'll see `load <extension>` commands before certain command sets become available.

## How Payloads Communicate

For staged payloads, the stager's only job is establishing a connection (TCP, HTTP, or HTTPS depending on the payload variant) back to a listener and then receiving and executing the stage over that same connection. Modern Meterpreter variants (the HTTP/HTTPS-based ones especially) wrap this communication in TLS and add session-key-based encryption on top, both for reliability across proxies/firewalls that inspect plain TCP traffic less favorably than HTTPS, and to reduce trivial network-level signature detection in lab/exam-style "can you get past basic monitoring" exercises.

Non-staged payloads skip the two-step handshake entirely — the full payload is delivered and executed in one shot, trading a larger initial delivery size for one fewer point of failure.

## How Handlers Work

`exploit/multi/handler` (and the implicit handler many exploit modules spin up automatically) is, at its core, just a listener waiting for an incoming connection that matches the configured `PAYLOAD`/`LHOST`/`LPORT`. When a stager connects, the handler recognizes the payload type it's configured for and knows exactly what stage to send back (for staged payloads) or simply completes the session setup (for non-staged ones). Running a handler with `-j` backgrounds it as a job, which is essential when you need msfconsole free to do other things (like setting up a second handler for a different target) while still waiting on the first.

## Session Management

Every established session — Meterpreter or a plain command shell — gets a numeric ID and an entry in msfconsole's internal session table (`sessions -l`). Sessions are decoupled from the module that created them: backgrounding a session (`background` or `Ctrl+Z`) just steps you out of the interactive view; the underlying connection and remote agent keep running independently. This is also why a session can be upgraded in place — `post/multi/manage/shell_to_meterpreter` takes an existing plain shell session, uses it to deliver and execute a Meterpreter payload, and the *new* Meterpreter session appears as a new entry in the same table, with the old shell session typically still present until you choose to kill it.

## Module Development

Every Metasploit module is a Ruby class inheriting from a base type (`Msf::Exploit`, `Msf::Auxiliary`, `Msf::Post`) and including relevant **mixins** — reusable chunks of shared functionality (`Msf::Exploit::Remote::Tcp` for raw TCP work, HTTP-client mixins for web-facing modules, SMB mixins for SMB-specific protocol handling, and so on). The `initialize` method registers metadata (name, description, author, references, license) and any module-specific options via `register_options`. Exploit modules implement a `check` method (safe vulnerability verification) and an `exploit` method (the actual trigger); auxiliary scanner modules typically implement `run_host` (called once per target in a range). Reading existing modules under `/usr/share/metasploit-framework/modules/` with this mental model in hand turns them from intimidating Ruby files into recognizable, parseable patterns.

## Exploit Selection Methodology

A disciplined exploit-selection process looks like this: confirm the exact service name and version from enumeration data; search Metasploit (`search`) and, where Metasploit doesn't have a matching module, external resources like searchsploit/Exploit-DB for the same CVE/vulnerability; read the module's `info` output fully, including any version constraints in the description; check the module's **Rank** (Metasploit assigns reliability ratings — from low to high: Manual, Low, Average, Normal, Good, Great, Excellent — reflecting how reliably the module works without crashing the target or requiring manual tuning); run `check` if available before `exploit`; and only then fire the exploit, with a payload whose architecture matches the confirmed target.

## Common Troubleshooting Techniques

**No session opens despite "exploit completed":** Check `LHOST` matches an interface actually reachable by the target (common VirtualBox multi-adapter mistake); check Kali's own firewall/iptables isn't dropping the inbound callback; check the target's outbound firewall rules if it's a non-default lab network.

**Exploit reports failure outright:** Re-verify target architecture/OS/patch level matches the module's documented constraints; try `set TARGET` explicitly instead of relying on auto-detection; check `check` output for a more specific reason.

**Database not connected (`db_status` fails):** Re-run `sudo msfdb init`; confirm PostgreSQL service is actually running (`sudo systemctl status postgresql`); restart msfconsole entirely rather than retrying inside the same session.

**Module not found via `search`:** Run `reload_all` if you just added/edited a module; confirm package is up to date (`sudo apt update && sudo apt install metasploit-framework`) if you expect a recently-published module that might not be in your current build yet.

**Session opens then immediately dies:** Architecture mismatch between payload and target process is the most common cause; the second most common is an unstable landing process that crashes shortly after exploitation — migrate quickly once stable, or choose a more stable payload/process target next attempt.
