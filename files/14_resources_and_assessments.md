# Resources and Final Assessments

## Best Books

- **"Metasploit: The Penetration Tester's Guide"** by David Kennedy, Jim O'Gorman, Devon Kearns, and Mati Aharoni — the original, still-relevant deep dive into the framework from people directly tied to its ecosystem.
- **"Penetration Testing: A Hands-On Introduction to Hacking"** by Georgia Weidman — broader than just Metasploit, but covers it thoroughly inside a complete methodology, and is widely used as OSCP-adjacent prep reading.
- **"The Hacker Playbook" series** by Peter Kim — practical, scenario-driven, good for building exam-style intuition.
- **"RTFM: Red Team Field Manual"** by Ben Clark — not Metasploit-specific, but an essential pocket reference for the surrounding command-line skills this course assumes.

## Best YouTube Channels

- **IppSec** — long-running, extremely detailed Hack The Box walkthroughs; excellent for seeing professional-level methodology in real time.
- **The Cyber Mentor (TCM Security)** — strong beginner-to-intermediate pentesting content, including dedicated Metasploit material.
- **HackerSploit** — broad offensive security coverage with a Kali Linux focus.
- **John Hammond** — CTF-style walkthroughs that reinforce the "think like an attacker" mindset this course builds toward.

## Best GitHub Repositories

- **rapid7/metasploit-framework** — the framework's own source repository; reading real module code here is the single best way to deepen the Module 9/Deep Technical Internals material.
- **danielmiessler/SecLists** — the standard wordlist collection used constantly alongside Metasploit's brute-force/scanner modules.
- **swisskyrepo/PayloadsAllTheThings** — broad reference covering payload techniques across many vulnerability classes, useful context alongside MSFVenom work.
- **carlospolop/PEASS-ng** — privilege escalation enumeration scripts that pair well with `local_exploit_suggester`-style workflows.

## Best Blogs

- **Rapid7 Blog** — official updates, module additions, and technical write-ups directly from the maintainers.
- **Offensive Security Blog** — useful for certification-adjacent methodology content.
- **SANS Internet Storm Center** — broader security context that helps you understand *why* certain misconfigurations (anonymous FTP, weak SNMP strings) keep showing up in the real world.

## Best Practice Labs

- **Metasploitable2 / Metasploitable3** — the canonical intentionally-vulnerable targets this entire course is built around.
- **DVWA (Damn Vulnerable Web Application)** — web-focused vulnerability practice, ideal for the MSFVenom file-upload labs in Module 6.
- **OWASP Broken Web Apps (BWA)** — a broader collection of intentionally vulnerable web applications beyond DVWA.

## Best TryHackMe Rooms

- **"Metasploit: Introduction"** and the follow-on Metasploit room series — structured, guided practice directly aligned with this course's modules.
- **"Vulnversity"** — good general enumeration-to-exploitation practice room for reinforcing Module 3/4.
- **Any "Offensive Security Pathway" introductory rooms** — useful for cross-checking your fundamentals against a structured curriculum outside this course.

## Best Hack The Box Machines

- **"Lame"** — directly mirrors the Samba `usermap_script` exploitation path practiced in Module 4.
- **"Legacy"** and **"Blue"** — classic, beginner-friendly SMB-based exploitation machines that reinforce enumeration-to-exploitation discipline.
- **"Devel"** — good for reinforcing file-upload-to-execution thinking similar to the Module 6 DVWA lab, in a different context.

---

## Quizzes

**Quiz A — Architecture (Modules 1–2):**
1. What are the five core Metasploit module types?
2. What does `db_status` tell you?
3. What is the difference between a stager and a stage?

**Quiz B — Enumeration & Exploitation (Modules 3–4):**
1. What's the difference between `RHOST` and `RHOSTS`?
2. Why run `check` before `exploit`?
3. Name two protocol-specific enumeration categories covered in Module 3.

**Quiz C — Post-Exploitation & Pivoting (Modules 5, 7, 8):**
1. What does `migrate` actually do?
2. What's the difference between what autoroute enables and what a SOCKS proxy enables?
3. Why must harvested credentials be handled carefully during an engagement?

(Answers for all three quizzes are derivable directly from the corresponding module's Review Questions/Answers sections — re-attempt them from memory before checking.)

## Practical Challenges

1. **Full Recon-to-Root, Untimed:** Starting from a blank workspace, fully enumerate and exploit Metasploitable2 using only modules you select yourself via `search` (no referring back to this course's exact command lists).
2. **Timed Recon-to-Root (90 minutes):** Repeat the above against a fresh Metasploitable2 snapshot, with a hard 90-minute clock, to build exam-pace intuition.
3. **Pivot Chain:** Build a three-host lab (Kali → Pivot Host A → Pivot Host B → Internal Target) and successfully reach the deepest host using nested autoroute configuration.
4. **Custom Module Challenge:** Write a custom auxiliary module (not a banner grabber this time) that checks for a specific HTTP response header on a target and reports whether it's present.

## OSCP-Style Exercises

- Treat every guided lab in Modules 3, 4, 7, and 10 as a "no-notes" exercise on a second attempt: re-do them from a fresh VM snapshot without referring to this course at all, then compare against the course only afterward to self-grade.
- Practice writing a complete finding (per the Module 10 report skeleton) for every successful exploitation in this course, not just the capstone — building report-writing speed is as important as building exploitation speed for exam-style assessments.
- Deliberately practice the same Metasploitable2 vulnerabilities *manually* (hand-crafted commands instead of the Metasploit module) at least once each, since many practical exams restrict or ban Metasploit usage on a subset of targets.

## Capstone Project

Build a complete, three-host lab network (a DMZ-style host directly reachable from Kali, a pivot host bridging two networks, and an internal-only target), perform a full Module 10-style assessment across all three hosts including at least one pivot hop, and produce a complete written report following the Module 10 skeleton with no fewer than four findings, each with a risk rating, evidence reference, and remediation recommendation. Treat this as the final proof that every module in this course has actually become working skill rather than just reading material.
