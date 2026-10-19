# 🔍 Room 2 — Content Discovery

> **Language Navigation:** [🇸🇦 النسخة العربية](./README.ar.md) | [💼 Real-World Impact & Career Guide](./real-world-impact.md) | [⬅️ Return to Root](../README.md)

---

## 🎯 Executive Summary & Mental Model

Content Discovery is the systematic practice of unearthing web assets, files, directories, background endpoints, and host routing that are **not publicly linked** within an application's visible user interface.

In real-world security assessments, developers rarely leave critical vulnerabilities on the homepage. High-severity flaws almost invariably reside in **forgotten staging portals, development branches, diagnostic files, legacy API versions, and exposed backups**.

### The Tri-Layer Reconnaissance Architecture
A professional tester does not rely on a single approach. Content discovery requires three mutually reinforcing layers:

```
┌─────────────────────────────────────────────────────────────┐
│ 1. MANUAL DISCOVERY                                         │
│ Fast, zero-noise, highly accurate.                           │
│ • robots.txt (Crawler boundaries)                           │
│ • sitemap.xml (Indexed structures & staging paths)           │
│ • HTTP Headers (Server banners, X-Powered-By, custom flags)  │
└──────────────────────────────┬──────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────┐
│ 2. OSINT (OPEN SOURCE INTELLIGENCE)                         │
│ Passive, completely undetectable by target monitoring.      │
│ • Google Dorking (site:, inurl:, filetype:, cache:)         │
│ • Wayback Machine (Historical snapshots, deleted endpoints) │
│ • GitHub / Code Repos (Secrets in commit history)           │
│ • Cloud Storage (S3 bucket permutation scans)               │
└──────────────────────────────┬──────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────┐
│ 3. AUTOMATED FUZZING (GOBUSTER)                             │
│ High-volume, algorithmic coverage across dictionary lists.  │
│ • Directory & file brute-forcing (`dir`)                    │
│ • Real DNS subdomain brute-forcing (`dns`)                  │
│ • Same-server Virtual Host brute-forcing (`vhost`)          │
└─────────────────────────────────────────────────────────────┘
```

---

## 📋 Task Breakdown & Investigative Methodology

### Task 1 — Introduction (Environment Verification)
* **Concept:** Boot the lab instance and confirm connectivity to the target web application.
* **Method:** Run a preliminary curl check to verify HTTP responsiveness before firing wordlist scanners:
  ```bash
  curl -sI http://MACHINE_IP/
  ```

---

### Task 2 — Manual Discovery: Common Files (`robots.txt` & `sitemap.xml`)

#### 1. The `robots.txt` Protocol
* **Core Mental Model:** `robots.txt` is an advisory guideline for well-behaved search engine web crawlers (like Googlebot); **it is never an access control mechanism**.
* When a developer puts a directory under `Disallow:`, they are creating an attacker's roadmap: *"Here is a sensitive directory that I don't want the general public to discover via Google."*
* **Investigative Steps:**
  1. Request `http://MACHINE_IP/robots.txt` directly via browser or terminal:
     ```bash
     curl -s http://MACHINE_IP/robots.txt
     ```
  2. Inspect the lines beginning with `Disallow:`.
  3. Extract the disallowed directory path and navigate to it in your browser.

#### 2. The `sitemap.xml` Map
* **Core Mental Model:** `sitemap.xml` provides search engines with a structured index of site URLs. Developers frequently auto-generate this file, inadvertently indexing development routes, beta feature testing pages, or staging portals.
* **Investigative Steps:**
  1. Request `http://MACHINE_IP/sitemap.xml`:
     ```bash
     curl -s http://MACHINE_IP/sitemap.xml
     ```
  2. Scan through the listed `<loc>` URLs.
  3. Look for anomalies: unconventional URL paths, leetspeak naming conventions (e.g., using numbers in place of letters), or references to internal tools.

---

### Task 3 — Manual Discovery: Framework Stack & Headers

#### 1. Inspecting HTTP Response Headers
* **Concept:** Web servers and application frameworks communicate metadata using HTTP headers. Custom debugging headers left by developers during testing can expose internal state.
* **Investigative Steps:**
  1. Send an HTTP request and inspect all returned response headers:
     ```bash
     curl -v http://MACHINE_IP/
     # OR
     curl -I http://MACHINE_IP/
     ```
  2. Scrutinize headers beyond standard ones like `Content-Type`. Look for non-standard or custom headers (e.g., starting with `X-`, such as `X-FLAG` or `X-Debug-Token`).

#### 2. Framework Documentation & Default Credentials
* **Concept:** If response headers or source comments disclose the application framework (e.g., `THM Framework v1.0`), consult the framework's official documentation or open-source repository.
* **Investigative Steps:**
  1. Identify the framework name from the source code or response headers.
  2. Research standard installation conventions for that framework:
     * What is the default administrative login path? (e.g., `/admin`, `/controlpanel`, `/thm-admin`).
     * What are the factory-default credentials? (e.g., `admin:admin`, `admin:password`, `root:root`).
  3. Navigate to the discovered administrative path and attempt authentication using the factory credentials.
* **Evidence Location:** Upon successful authentication, the administrative dashboard surfaces the administrative flag.

---

### Task 4 — OSINT: Search Engines & Web Tools

#### 1. Advanced Google Dorking
* **Concept:** Google dorks leverage specialized search operators to query Google's indexed cache for misconfigured assets, sensitive files, or forgotten portals.
* **Essential Dorking Operators:**
  | Operator | Function | Example Target |
  |---|---|---|
  | `site:` | Restricts results strictly to the designated domain | `site:example.com` |
  | `inurl:` | Matches strings occurring within the URL path | `site:example.com inurl:admin` |
  | `filetype:` | Filters by specific file extension | `site:example.com filetype:pdf` or `filetype:env` |
  | `intitle:` | Searches within the HTML `<title>` tag | `site:example.com intitle:"Index of /"` |
  | `intext:` | Searches within visible body content | `site:example.com intext:"internal use only"` |
  | `cache:` | Displays Google's cached snapshot of a page | `cache:example.com` |

#### 2. Wappalyzer
* **Concept:** Wappalyzer is a passive browser extension and CLI utility that inspects JavaScript variables, HTML meta tags, cookie formats, and response headers to map the complete technology stack (CMS, programming language, server, analytics, UI frameworks) along with version numbers.

---

### Task 5 — OSINT: Archives & Repositories

#### 1. The Wayback Machine (Internet Archive)
* **URL:** `https://archive.org/web/`
* **Concept:** The Wayback Machine stores historic snapshots of web pages dating back to 1996.
* **Pentester Application:** If a company updates or removes a legacy login page, API endpoint, or employee portal, the live site may remove the link, but the backend endpoint might still be active on the server. Historical snapshots expose these legacy paths.

#### 2. GitHub Code Repositories
* **Concept:** Developers frequently push private project repositories to public accounts or accidentally commit `.env` files, SSH keys, or API tokens.
* **CRITICAL LESSON:**
  > **Deleting a secret in a subsequent commit does NOT remove it from Git.**
  > Unless the entire commit history is rewritten using tools like `git-filter-repo` or BFG Repo-Cleaner, every previous commit remains retrievable in the repo's permanent history tree via `git log -p` or commit diffs.

#### 3. AWS S3 Buckets
* **Standard URL Scheme:** `https://{bucket-name}.s3.amazonaws.com`
* **Concept:** Cloud storage buckets configured with public read access allow unauthenticated users to download backups, source archives, customer documents, and media assets.
* **Naming Enumeration Patterns:**
  * `{company}-assets`
  * `{company}-backup`
  * `{company}-dev` / `{company}-staging`

---

### Task 6 — Automated Discovery: Gobuster Fundamentals

Gobuster is a high-performance content brute-forcer written in Go.

```
       [ Wordlist (SecLists) ]
       ├── admin
       ├── backup        HTTP GET http://TARGET/FUZZ
       ├── api       ─────────────────────────────────▶ [ Web Server ]
       └── secret                                             │
                                                              ▼
                      [ HTTP Response Codes ] ◀───────────────┘
                       • 200 OK (Direct hit)
                       • 301/302 (Redirect)
                       • 403 (Forbidden - Exists!)
```

#### Core CLI Flags:
* `-u`: Target base URL.
* `-w`: Path to dictionary wordlist.
* `-x`: File extensions to append (e.g., `-x php,txt,html,bak`).
* `-t`: Concurrent threads (default is 10; increase for speed, decrease to avoid rate limits).
* `-o`: Output file for logging results.
* `--delay`: Pause duration between requests (bypasses basic rate limiting).

#### Practical Execution:
```bash
# Directory brute-forcing using SecLists common wordlist
gobuster dir -u http://MACHINE_IP/ \
  -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt \
  -t 20 -x txt,php,html,log
```
* **Investigation:** Review output for:
  * Directories beginning with `mo/` (identifying matching subdirectories).
  * System or diagnostic log files with the `.log` extension.

---

### Task 7 — Automated Discovery: Subdomains vs. Virtual Hosts

This is the most critical conceptual distinction in the entire module.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ 1. SUBDOMAIN (DNS LAYER RESOLUTION)                                         │
│ Request: blog.example.com                                                   │
│                                                                             │
│ [ Client ] ──▶ [ DNS Server ] ──▶ Returns IP: 10.10.20.5                    │
│ [ Client ] ─────────────────────▶ Connects to 10.10.20.5                    │
│                                                                             │
│ • Tool Mode: `gobuster dns -d example.com -w wordlist.txt`                  │
│ • Fails if: The subdomain has no public DNS A record registered.            │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ 2. VIRTUAL HOST (VHOST - HTTP LAYER RESOLUTION)                             │
│ Request sent to fixed IP: 10.10.20.5 with dynamic HTTP Header:              │
│ Host: internal-dev.example.com                                              │
│                                                                             │
│ [ Client ] ──▶ Sends HTTP to 10.10.20.5 with `Host: internal-dev.thm` ───┐  │
│                                                                          ▼  │
│ [ Apache / Nginx Server ] ◀──────────────────────────────────────────────┘  │
│ Server reads `Host:` header and routes internally to private vhost folder!  │
│                                                                             │
│ • Tool Mode: `gobuster vhost -u http://IP --domain example.thm -w list.txt` │
│ • Succeeds even with ZERO DNS records configured for that hostname!         │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### Environment Preparation (Local DNS Setup):
Before running DNS/vhost enumeration in the lab environment:
1. Update `/etc/resolv-dnsmasq` or `/etc/hosts` to point the target domain to the lab machine IP:
   ```bash
   echo "MACHINE_IP nameserver" >> /etc/resolv-dnsmasq
   systemctl restart dnsmasq
   # OR add directly to hosts:
   echo "MACHINE_IP acmeitsupport.thm" >> /etc/hosts
   ```

#### Gobuster DNS Mode (`dns`):
```bash
gobuster dns -d acmeitsupport.thm \
  -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt \
  --wildcard
```
* `--wildcard`: Forces Gobuster to continue even if a wildcard DNS record exists (which otherwise produces false positives on every queried word).

#### Gobuster Virtual Host Mode (`vhost`):
```bash
gobuster vhost -u http://MACHINE_IP/ \
  --domain acmeitsupport.thm \
  -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt \
  --append-domain \
  --exclude-length 250-320
```
* `--append-domain`: Appends the base domain to each word in the list (e.g., `word` becomes `word.acmeitsupport.thm`).
* `--exclude-length`: Filters out false positive responses that return the default fallback page size.
* **Investigative Step:** Count how many discovered virtual hosts return an HTTP `200 OK` status code.

---

### Task 8 — Room Conclusion
* Reconnaissance is not a single scan; it is an iterative pipeline.
* Always begin with manual checks (`robots.txt`, `sitemap.xml`, headers) because they take 30 seconds and produce pinpoint findings.
* Leverage passive OSINT before active scanning to build your target list without alerting defense systems.
* Use automated tools (`Gobuster`) to exhaustively explore the remaining search space.
