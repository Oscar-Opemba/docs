# Module 8 — Pivoting and Routing

## Objectives

Understand and practice pivoting: route traffic through a compromised host into a network segment you can't directly reach, set up port forwarding, and configure a SOCKS proxy to point external tools (like Nmap or a browser) through your foothold.

## Theory

Real networks are segmented. The host you compromise first is rarely the host that matters most — it's the bridge to an internal segment that wasn't reachable from your original attack position. Pivoting is how you extend your reach through that bridge without needing to physically move your attacking machine.

## Deep Technical Explanation

### The concept

```
[Kali] --- reachable ---> [Host A, compromised, dual-homed] --- NOT directly reachable from Kali ---> [Host B, internal segment]
```

Once Host A is compromised and you have a Meterpreter session on it, Metasploit can route traffic destined for Host B's subnet *through* that session, as if Kali itself had a network interface on the internal segment.

### Autoroute

```bash
meterpreter > run autoroute -s 192.168.57.0/24
# or, post-module style:
msf6 > use post/multi/manage/autoroute
msf6 post(multi/manage/autoroute) > set SESSION 1
msf6 post(multi/manage/autoroute) > set SUBNET 192.168.57.0
msf6 post(multi/manage/autoroute) > run
msf6 > route print
```

Once a route exists, **other Metasploit modules** (scanners, exploits, even another `multi/handler` for a second-stage payload) can now target hosts on `192.168.57.0/24` directly — msfconsole transparently routes that traffic through the session.

### Port forwarding

```bash
meterpreter > portfwd add -l 3389 -p 3389 -r 192.168.57.20
```

This makes `localhost:3389` on Kali forward straight to `192.168.57.20:3389` through the compromised host — letting you point a normal local RDP client at `127.0.0.1:3389` as if it were directly reachable.

### SOCKS proxies

Autoroute only helps *Metasploit's own* modules. To point external tools (Nmap, a browser, curl) through your pivot, you need a SOCKS proxy:

```bash
msf6 > use auxiliary/server/socks_proxy
msf6 auxiliary(server/socks_proxy) > set SRVPORT 1080
msf6 auxiliary(server/socks_proxy) > set VERSION 5
msf6 auxiliary(server/socks_proxy) > run -j
```

Then configure `proxychains` (on Kali, edit `/etc/proxychains4.conf` to point at `127.0.0.1 1080`) and run any external tool prefixed with `proxychains`:

```bash
proxychains nmap -sT -Pn 192.168.57.20
```

(`-sT` and `-Pn` matter here — SOCKS proxies generally only support full TCP connect scans, not raw SYN scans or ICMP-based host discovery.)

### Multi-network environments

A realistic lab setup mirrors common corporate segmentation: a DMZ-style segment Kali can reach directly, and a fully internal segment it cannot, with the only path between them being a dual-homed host you must first compromise.

```
[Kali] -- vboxnet0 (DMZ) -- [Pivot Host: 2 NICs] -- vboxnet1 (Internal) -- [Internal Target]
```

## Practical Examples — Two-Hop Lab

1. Compromise the dual-homed pivot host on the DMZ segment (any Module 4 technique).
2. `run autoroute -s <internal-subnet>/24`
3. `route print` — confirm the route now exists.
4. `use auxiliary/scanner/portscan/tcp` → `set RHOSTS <internal-target-ip>` → `run` — this now works *through* the pivot, even though Kali has no direct route to that subnet at the OS level.

**Expected output:** The portscan returns results for the internal target, proving the route is functioning.

## Common Mistakes

- Forgetting that autoroute only benefits Metasploit's own modules — trying to `nmap` (the standalone CLI tool) an internal host directly and being confused why it times out.
- Using `-sS`/`-sn` through a SOCKS proxy (these require raw socket access SOCKS can't provide) instead of `-sT -Pn`.
- Losing the route silently when the underlying session dies, then spending time debugging "why did my internal scan stop working" instead of just re-checking `sessions -l`.
- Not documenting the pivot topology, which makes the eventual report's network diagram much harder to reconstruct accurately.

## Real-World Usage

Pivoting is frequently the single most valuable skill that separates a junior tester from a senior one on internal/segmented engagements — initial access is often on a low-value DMZ host, and the real demonstrable risk (domain controllers, finance systems, internal admin panels) sits one or two pivots deeper.

## Guided Lab — Building and Using a Pivot

**Objective:** Establish a route through a compromised dual-homed host and successfully scan a target on the otherwise-unreachable segment.

**Setup:** Two internal/host-only networks in your hypervisor; a pivot VM with one NIC on each, plus a target VM only on the second network.

**Steps:**
1. Confirm from Kali that you can reach the pivot host but *not* the deeper internal target (`ping` should fail to the internal target directly).
2. Compromise the pivot host, get a session.
3. `run autoroute -s <internal-subnet>/24`
4. `use auxiliary/scanner/portscan/tcp`, target the internal host, `run`.

**Expected output:** Open ports on the internal target are reported despite Kali having no direct OS-level route to that subnet.

**Verification:** `route print` inside msfconsole lists the active route through the correct session ID.

**Troubleshooting:** If the scan hangs indefinitely, confirm the pivot session is still alive (`sessions -l`) — a dead session silently breaks the route.

## Challenge Lab

Set up a SOCKS proxy through your existing pivot session, configure proxychains, and successfully run an external tool (`proxychains curl` or `proxychains nmap -sT -Pn`) against the internal target — demonstrating that non-Metasploit tools can now also reach the deeper segment.

## Review Questions

1. What's the core difference between what autoroute enables versus what a SOCKS proxy enables?
2. Why must SOCKS-proxied Nmap scans use `-sT -Pn`?
3. What happens to an active route if the underlying session dies?
4. In the two-hop lab topology, why does the pivot host need two network interfaces?
5. Why is pivoting often described as more valuable on internal engagements than initial exploitation itself?

## Answers

1. Autoroute only routes traffic for Metasploit's own internal modules; a SOCKS proxy exposes that same routed access to any external tool capable of using a SOCKS proxy.
2. Because SOCKS proxies only support full TCP connections, not raw SYN packets or ICMP, so SYN scans (`-sS`) and ping-based host discovery don't function through them.
3. The route becomes unusable/breaks silently — any module relying on it will fail or hang until a new session is established and a new route configured.
4. One interface keeps it reachable from your existing position (Kali); the second interface is what actually connects it to the otherwise-unreachable internal segment, making it the bridge.
5. Because initial access is frequently on a lower-value perimeter/DMZ host, while the systems that demonstrate real business risk are typically one or more network segments deeper, only reachable via pivoting.
