# 🖥️ Room 4 — Web Server Attacks - I

> **Language Navigation:** [🇸🇦 النسخة العربية](./README.ar.md) | [💼 Real-World Impact & Career Guide](./real-world-impact.md) | [⬅️ Return to Root](../README.md)

---

## 🎯 Executive Summary & Mental Model

"Web Server Attacks - I" shifts the focus from application-level logic flaws to the underlying infrastructure hosting web services. This room emphasizes **misconfiguration identification (not exploitation/RCE)** across four foundational Linux web servers: **Apache2, Python's built-in HTTP server, Node.js (Express), and Nginx**.

### The Core Pentester Principle
> **"Default configurations prioritize ease of deployment and diagnostic convenience over security everywhere."**

When testers encounter exposed status pages, verbose error traces, or unauthenticated directory listings on real engagements, it rarely means a system administrator was malicious. It almost always means **defaults were never audited, or development conveniences survived into production**.

```
                           [ Web Server Tier ]
        ┌───────────────────┬───────────────────┬───────────────────┐
        ▼                   ▼                   ▼                   ▼
    [ Apache2 ]         [ Python ]          [ Node.js ]          [ Nginx ]
    (Port 80)           (Port 8000)         (Port 3000)         (Port 8080)
   • mod_status        • Zero Access       • NODE_ENV=dev      • stub_status
     (/server-status)    Control             Leakage             (/nginx_status)
   • Options +Indexes  • Exposes           • Debug Routes      • autoindex on;
   • Leftover .bak       Dotfiles (.env)     (/api/routes)     • server_tokens
   • .htpasswd leaks   • Backup Zips       • process.env dump    disclosure
```

---

## 📋 Task Breakdown & Investigative Methodology

### Task 1 — Introduction (Lab Infrastructure)
* **Target Services:**
  * **Port 80:** Apache2
  * **Port 8000:** Python built-in HTTP Server
  * **Port 3000:** Node.js / Express
  * **Port 8080:** Nginx

---

### Task 2 — Identifying Web Servers (Server Fingerprinting)

#### 1. Banner Fingerprinting via HTTP Response Headers
Different web servers expose their identities through standard response headers:
```bash
# Capture response headers across all 4 ports
for port in 80 8000 3000 8080; do
  echo "=== PORT $port ==="
  curl -sI http://MACHINE_IP:$port/ | grep -iE "server|x-powered-by"
done
```

| Port | Server Type | Primary Header Fingerprint | Default Value |
|---|---|---|---|
| **80** | Apache2 | `Server:` | `Apache/2.4.x (Ubuntu)` |
| **8000** | Python HTTP Server | `Server:` | `SimpleHTTP/0.6 Python/3.xx.x` |
| **3000** | Node.js Express | `X-Powered-By:` | `Express` *(Server header is omitted by default)* |
| **8080** | Nginx | `Server:` | `nginx/1.xx.x` |

#### 2. Secondary Fingerprinting: Error Page Signatures
Even when administrators suppress the `Server` header, default 404 error pages have distinct visual and structural shapes:
* **Python HTTP Server:** Minimalist plain-text response: `Error code: 404. Message: File not found.`
* **Nginx:** Centered HTML header with version-stamped footer: `<center>nginx/1.xx.x</center>`.
* **Apache2:** Structured HTML table with hostname and port footer.

---

### Task 3 — Python's Built-In HTTP Server (Zero Access Control)

#### 1. Concept & Threat Reality
Python's built-in server (`python3 -m http.server 8000`) is intended exclusively for rapid local testing.
* **The Fatal Flaw:** It has **zero authentication, zero authorization, and zero dotfile filtering**.
* Unlike production web servers that automatically hide or deny requests for files beginning with a dot (`.`), Python serves whatever exists in its working directory.

#### 2. Exploiting Leftover Artifacts
1. **Direct Dotfile Extraction (`.env`):**
   ```bash
   curl -s http://MACHINE_IP:8000/.env
   ```
   * *Evidence:* Look for database connection strings: `DATABASE_URL=mysql://user:password@localhost/dbname`.
2. **Backup Archive Extraction (`.zip` / `.tar.gz`):**
   * If directory listing is active or discovered via Gobuster, look for backup archives:
     ```bash
     curl -s http://MACHINE_IP:8000/backup.zip -o backup.zip
     unzip -l backup.zip
     unzip backup.zip -d backup-contents/
     ```
   * *Evidence:* Search extracted contents for source code, configuration files, and flags.

---

### Task 4 — Apache2: Listing, `mod_status`, & Leftover Files

#### 1. Server Version Disclosure (`ServerTokens`)
* By default on Ubuntu, Apache sets `ServerTokens OS`, broadcasting the exact Apache patch version and underlying OS distribution (`Apache/2.4.x (Ubuntu)`).

#### 2. Directory Listing (`Options +Indexes`)
* If an administrator omits an `index.html` file in a directory configured with `Options +Indexes`, Apache renders an index table exposing all files.
* Test common folders: `http://MACHINE_IP/files/`.

#### 3. Real-Time Diagnostics Leakage (`mod_status` / `/server-status`)
* **What it exposes:** Active client IP addresses, current worker process states, CPU usage, requested URL paths, and total request counts.
* **The Misconfiguration:** Apache's default configuration restricts `/server-status` to localhost via `Require local`. However, if an administrator adds `Require all granted` anywhere inside a virtual host configuration, it **silently overrides** the restriction server-wide!
* **Investigative Command:**
  ```bash
  curl -s http://MACHINE_IP:80/server-status
  ```

#### 4. Leftover & Backup File Enumeration
* Developers frequently rename files during manual editing: `config.php.bak`, `database.sql.old`, or `.htpasswd`.
* **Gobuster Execution:**
  ```bash
  gobuster dir -u http://MACHINE_IP:80/ \
    -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt \
    -x bak,txt,old -t 20
  ```
* **High-Value Finding (`.htpasswd`):** If an `.htpasswd` file is downloaded, it exposes Apache Basic Authentication username and password hashes (typically bcrypt or MD5-crypt), which can be cracked offline using John the Ripper or Hashcat.

---

### Task 5 — Node.js (Express): Development Mode Leaks

#### 1. Concept: Development vs. Production (`NODE_ENV`)
In Node.js applications, setting `NODE_ENV=production` optimizes performance and suppresses sensitive diagnostics. When developers run production deployments under `NODE_ENV=development`, severe information disclosure vulnerabilities emerge.

#### 2. Common Express Development Leak Patterns:
1. **Verbose Error Stack Traces:** Custom error handlers often dump full stack traces showing server-side file paths, library versions, and database query parameters upon triggering an HTTP 500 error.
2. **Debug Route Enumeration (`/api/routes`):** Developers sometimes build a temporary route that programmatically inspects Express's internal router stack (`app._router.stack`) to list all registered API endpoints.
3. **Environment Variable Dumps (`process.env`):** Debug routes like `/api/debug/env` return the entire `process.env` object in JSON format, exposing API keys, secret salts, and confirming `NODE_ENV: "development"`.
4. **Embedded Static Configuration:** Client-side JavaScript bundles (`/static/bundle.js`) frequently contain hardcoded staging API URLs, internal hostnames, and debug tokens.

#### 3. Dotfile Behavior Note:
* Unlike Python's HTTP server, `express.static()` returns `404 Not Found` for dotfiles (like `.env`) by default, preventing direct downloads unless an explicit custom route exists.

---

### Task 6 — Nginx: `autoindex` & `stub_status`

#### 1. Version Disclosure (`server_tokens`)
* Controlled by the `server_tokens` directive.
* If set to `on;` (default), Nginx reveals its exact version in both the `Server:` response header and the footer of default error pages (`nginx/1.18.0`).
* Hardening requires `server_tokens off;`.

#### 2. Directory Listing (`autoindex on;`)
* Unlike Apache, Nginx **disables directory listing by default**.
* Directory listing occurs only when an administrator explicitly enables it inside a location block:
  ```nginx
  location /files/ {
      autoindex on;
      root /var/www/nginx/;
  }
  ```

#### 3. Real-Time Status Module (`stub_status` / `/nginx_status`)
* Nginx provides the `ngx_http_stub_status_module` for basic metric tracking:
  ```
  Active connections: 1 
  server accepts handled requests
   15 15 15 
  Reading: 0 Writing: 1 Waiting: 0 
  ```
* **Secure Configuration:** Restricted via `allow 127.0.0.1; deny all;`.
* **Misconfiguration:** Replacing the rule with `allow all;` exposes live connection statistics to external attackers.

---

### Task 7 — Common Misconfigurations Across Servers (Missing Security Headers)

#### 1. The Missing Security Headers Audit
By default, **none of the four servers return defensive security headers**. Without explicit manual configuration, modern web applications remain vulnerable to client-side attacks.

```bash
# Automated security header audit script
for port in 80 8000 3000 8080; do
  echo "=== Auditing Security Headers on Port $port ==="
  curl -sI http://MACHINE_IP:$port/ | grep -iE "x-frame-options|x-content-type-options|content-security-policy|strict-transport-security|referrer-policy" || echo "[-] ZERO Security Headers Present!"
done
```

#### 2. Essential Security Headers Reference:
| Header | Recommended Value | Attack Mitigated |
|---|---|---|
| `X-Frame-Options` | `DENY` or `SAMEORIGIN` | Clickjacking (embedding application in malicious `<iframe>`) |
| `X-Content-Type-Options` | `nosniff` | MIME-type sniffing attacks |
| `Content-Security-Policy` | `default-src 'self';` | Cross-Site Scripting (XSS) and unauthorized data injection |
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains` | Man-in-the-Middle (forces HTTPS transmission) |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | Credential and token leakage via HTTP `Referer` headers |

#### 3. Automated Scanning with Nikto:
```bash
nikto -h http://MACHINE_IP:80 -nointeractive
```
* Nikto flags missing security headers and maps known OSVDB directory indexing vulnerabilities (`/files/`).

---

### Task 8 — Room Conclusion & Cross-Server Comparison Matrix

| Misconfiguration Vector | Apache2 (Port 80) | Python HTTP (Port 8000) | Node.js Express (Port 3000) | Nginx (Port 8080) |
|---|---|---|---|---|
| **Version Disclosure** | `ServerTokens OS` | Default in `Server:` header | `X-Powered-By: Express` | `server_tokens on;` |
| **Directory Listing** | `Options +Indexes` | Native on entire CWD | Not native in static | `autoindex on;` |
| **Diagnostic Status** | `/server-status` (`mod_status`)| N/A | `/api/debug/...` | `/nginx_status` (`stub_status`) |
| **Dotfile Exposure** | Blocked by default | **EXPOSED DIRECTLY** | Blocked by `express.static` | Blocked if configured |
| **Security Headers** | Missing by default | Missing by default | Missing by default | Missing by default |
