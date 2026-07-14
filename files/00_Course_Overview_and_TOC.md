# Metasploit Mastery
### From Zero to Professional Penetration Tester — A Complete Ethical Hacking Course

**Author / Compiled for:** Oscar Opemba (oxghostx)
**Platform assumed:** Kali Linux
**Scope:** Authorized lab environments, CTFs, OSCP-style practice, and legally authorized engagements only

---

## Legal and Ethical Notice

Everything in this course is designed to be practiced exclusively against systems you own, systems explicitly built for practice (Metasploitable2, DVWA, OWASP Broken Web Apps, Hack The Box, TryHackMe), or systems you have **written authorization** to test. Running any of these techniques against systems without explicit permission is illegal in essentially every jurisdiction, including Kenya's Computer Misuse and Cybercrimes Act. Treat every command in this course as a lab exercise, not a template for unauthorized activity.

The labs deliberately rely on intentionally vulnerable training platforms. None of the modules, exploits, or payloads referenced here are novel — everything is a standard, publicly documented feature of the Metasploit Framework that ships on Kali by default.

---

## Prerequisites

- Comfortable with basic Linux command line (cd, ls, grep, ssh, file permissions)
- Basic networking (IP addressing, TCP/UDP, ports, NAT vs bridged networking)
- A hypervisor: VirtualBox or VMware Workstation/Player
- Kali Linux VM (attacker)
- Metasploitable2 VM (primary intentionally-vulnerable target)
- DVWA (Damn Vulnerable Web Application), deployed either as a Docker container or on a small Linux VM
- Optional: a Windows 7/Server 2008 VM (legally owned, used purely for legacy SMB/Meterpreter labs), OWASP Broken Web Apps VM

### Recommended Lab Network Topology

```
[Kali Linux - Attacker]  --- Host-Only/Internal Network ---  [Metasploitable2]
        |                                                       |
        |---------------------------- Internal Network ---------|--- [DVWA / OWASP BWA]
        |
   (later: Module 8) ---- pivot ----> [Internal-only segment, no direct route from Kali]
```

Keep this network **host-only/internal**, never bridged to your home network or the internet. This single decision prevents 90% of "accidental scope creep" mistakes.

---

## How This Course Is Organized

Each numbered file below is a self-contained module. Work through them in order — later modules assume the muscle memory built in earlier ones. Every module follows the same internal structure: Objectives, Theory, Deep Technical Explanation, Practical Examples, Common Mistakes, Real-World Usage, Guided Lab, Challenge Lab, Review Questions, and Answers.

| File | Module |
|---|---|
| 01_module1_foundations.md | Module 1 — Foundations |
| 02_module2_architecture.md | Module 2 — Understanding Metasploit Architecture |
| 03_module3_enumeration.md | Module 3 — Enumeration Using Metasploit |
| 04_module4_exploitation.md | Module 4 — Exploitation |
| 05_module5_meterpreter.md | Module 5 — Meterpreter Mastery |
| 06_module6_msfvenom.md | Module 6 — MSFVenom Mastery |
| 07_module7_post_exploitation.md | Module 7 — Post-Exploitation |
| 08_module8_pivoting.md | Module 8 — Pivoting and Routing |
| 09_module9_advanced.md | Module 9 — Advanced Metasploit |
| 10_module10_professional_workflow.md | Module 10 — Professional Workflow (full simulated assessment) |
| 11_deep_technical_internals.md | Deep Technical Internals (Meterpreter, payload comms, handlers, module dev, troubleshooting) |
| 12_cheatsheets.md | Metasploit, Meterpreter & MSFVenom cheat sheets + workflow checklist |
| 13_learning_roadmaps.md | 7-day intensive plan, 30-day mastery roadmap, interview Q&A, cert prep |
| 14_resources_and_assessments.md | Best books/channels/repos/rooms/machines + quizzes, challenges, capstone |

---

## A Note on Pacing

This is dense, professional-grade material. A realistic full pass — actually typing every command, not just reading — takes 30–50 hours. The 7-day and 30-day plans in file 13 give you two different paces depending on how much time you have before you need to be "competent."
