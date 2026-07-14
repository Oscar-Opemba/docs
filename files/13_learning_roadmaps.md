# Learning Roadmaps, Interview Prep, and Certification Guidance

## 7-Day Intensive Plan

Use this if you have a deadline (an exam, an internship task, a CTF) and need working competence fast. Assume 4-6 focused hours/day.

- **Day 1:** Module 1 + Module 2 fully. Get the lab network (Kali + Metasploitable2) running end to end. Goal: comfortable navigating msfconsole, workspaces, and database basics.
- **Day 2:** Module 3 in full. Run a complete enumeration pass against Metasploitable2 and DVWA, every result logged to the database.
- **Day 3:** Module 4 in full. Get at least two different working exploits/sessions on Metasploitable2 (vsftpd backdoor + samba usermap_script).
- **Day 4:** Module 5 in full. Practice Meterpreter command fluency until it's automatic, not looked-up.
- **Day 5:** Module 6 in full. Generate at least three different payload formats and successfully catch sessions for each.
- **Day 6:** Module 7 + Module 8. Do post-exploitation enumeration, then build and use a two-hop pivot lab.
- **Day 7:** Module 9 + Module 10. Write your own resource script, then run the full Module 10 capstone assessment end-to-end and produce a short written report.

By the end of Day 7 you should be able to go from "blank Kali VM" to "documented finding with a session and a report paragraph" without referring back to the course files for syntax.

## 30-Day Mastery Roadmap

- **Week 1 (Days 1–7):** Follow the 7-Day Intensive Plan above as your foundation pass.
- **Week 2 (Days 8–14):** Repeat every guided lab from Modules 1–10, but this time *only* using the Challenge Lab prompts (no copy-pasting commands) — force recall instead of recognition. Start a TryHackMe or Hack The Box machine in parallel (see resources file).
- **Week 3 (Days 15–21):** Go deeper on Module 9 and the Deep Technical Internals file — read at least three real exploit modules from `/usr/share/metasploit-framework/modules/exploits/` end to end and explain each one's `check`/`exploit` logic in your own words. Write one small custom auxiliary module from scratch.
- **Week 4 (Days 22–30):** Run two or three full Module 10-style capstone assessments against different lab combinations (Metasploitable2 + DVWA, then a Windows lab VM if you have one set up, then a multi-hop pivot scenario). Write a complete, polished report for at least one of them as if it were a real deliverable. Use the remaining days for whichever Hack The Box/TryHackMe rooms from the resources file felt hardest the first time.

## Interview Questions and Answers

**Q: Walk me through your methodology when given a fresh target IP with no other information.**
A: Start with host discovery and a full port/service scan, log everything to the Metasploit database, run protocol-specific enumeration on anything interesting, cross-reference exact versions against known vulnerabilities before attempting exploitation, validate with `check` where supported, then move into post-exploitation only after confirming session privilege and stability.

**Q: What's the difference between a staged and non-staged payload, and when would you choose one over the other?**
A: Staged payloads split delivery into a small stager (which calls back and fetches the larger stage) and the stage itself, useful when exploit space for shellcode is limited. Non-staged payloads deliver everything in one block — simpler and more reliable when space isn't a constraint, but larger.

**Q: How does Meterpreter avoid leaving forensic artifacts on disk?**
A: It's loaded reflectively, directly into process memory, rather than being written to disk as a standalone executable, so it leaves a much smaller footprint than a traditional dropped binary.

**Q: What would you do if an exploit reports success but no session opens?**
A: Check that `LHOST` matches an interface actually reachable by the target, check for firewall rules (both attacker and target side) potentially dropping the callback, and confirm payload architecture matches the target process.

**Q: Explain pivoting and why it matters on internal engagements.**
A: Pivoting routes traffic through an already-compromised host into network segments not otherwise reachable from the attacker's position; it matters because initial access is often on a lower-value host while higher-value systems sit deeper in the network, only reachable via pivoting/lateral movement.

**Q: What's the actual purpose of an MSFVenom encoder, and what's a common misconception about it?**
A: Its real purpose is byte-compatibility — avoiding characters that would break the specific delivery context. The common misconception is that encoding meaningfully defeats modern antivirus/EDR; it generally does not on its own.

**Q: How would you explain the business value of a penetration test to a non-technical stakeholder?**
A: Frame it around demonstrated, real-world impact and risk reduction: not "we found a vulnerability" but "here's exactly what an attacker could access, what that would cost the business, and exactly what to fix first to reduce that risk."

**Q: What's the difference between auxiliary and post modules?**
A: Auxiliary modules don't require an existing session (scanning, enumeration, brute force); post modules run on a target you've already compromised, after a session exists.

**Q: How do you decide whether a finding is Critical, High, Medium, or Low?**
A: By assessing actual exploitability and business impact together — often anchored to a standard like CVSS — and applying that rating consistently across all findings in the same report so severity comparisons are meaningful.

**Q: Why is documentation considered as important as the technical exploitation itself in professional pentesting?**
A: Because a finding nobody can reproduce, verify, or act on (with a clear remediation step) doesn't actually reduce the client's risk — the technical work only creates value once it's translated into something the client can use.

## Certification Preparation Guidance

**OSCP (Offensive Security Certified Professional):** Heavy emphasis on manual methodology and "no-Metasploit" or limited-Metasploit exploitation in parts of the exam historically, alongside full methodology documentation. This course's discipline around verification-before-exploitation and continuous documentation maps directly onto OSCP's grading rubric. Treat Metasploitable2 and similar boxes as your baseline, then deliberately practice doing the *same* exploitation steps manually (without Metasploit) to build the underlying skill the exam expects.

**eJPT (eLearnSecurity/INE Junior Penetration Tester):** More directly Metasploit-friendly than OSCP; this course's Modules 1–7 map closely onto eJPT's practical skill expectations.

**PNPT (Practical Network Penetration Tester):** Strong emphasis on the full workflow including reporting (Module 10 here is directly relevant) and on Active Directory environments specifically — supplement this course with AD-focused lab practice (a small AD lab with a domain controller) before attempting PNPT.

**General exam advice regardless of certification:** Practice the *documentation habit* as much as the technical skill — most candidates who fail practical exams fail on either time management or incomplete/poorly evidenced reporting, not on total technical inability.
