# 🛡️ Web Application Security Fundamentals — Module Write-Up

> A learning-oriented write-up covering all four rooms of TryHackMe's **Web Application Security Fundamentals** module: manual browser-based testing, content discovery, modern web stack fingerprinting & exploitation, and Linux web server misconfigurations.

**Platform:** TryHackMe · **Module:** Web Application Security Fundamentals
**Rooms covered:** Walking An Application → Content Discovery → Modern Web Stacks → Web Server Attacks - I

> ⚠️ This is a **methodology guide, not an answer key**. Every question is broken down into the concept it tests, where the evidence lives, and how to reason toward the answer — final flag values are left for you to extract from your own TryHackMe lab instance, both because they differ per deployment and because deriving them yourself is the actual skill being built.
>
> 📄 A full **Arabic-language study guide (PDF)** covering the same four rooms in greater depth is included in this repo: [`Web-Application-Security-Fundamentals-Arabic-Guide.pdf`](./Web-Application-Security-Fundamentals-Arabic-Guide.pdf).

---

## 📑 Table of Contents

- [Module Overview](#-module-overview)
- [Room 1 — Walking An Application](#-room-1--walking-an-application)
- [Room 2 — Content Discovery](#-room-2--content-discovery)
- [Room 3 — Modern Web Stacks](#-room-3--modern-web-stacks)
- [Room 4 — Web Server Attacks - I](#-room-4--web-server-attacks---i)
- [Core Concepts Reference](#-core-concepts-reference)
- [Command Reference](#-command-reference)
- [CVE Summary Table](#-cve-summary-table)
- [Lessons Learned](#-lessons-learned)
- [Skills Practiced](#-skills-practiced)

---

## 🔎 Module Overview

This module builds the foundation for web application penetration testing, moving through four distinct layers of the discipline:

1. **Walking An Application** — manual reconnaissance using nothing but browser DevTools. No scanners, no automation. The point: automated tools routinely miss what a trained eye catches in page source, hidden DOM elements, JavaScript logic, network traffic, and client-side storage.
2. **Content Discovery** — finding content that isn't linked anywhere obvious, via three complementary approaches: manual (`robots.txt`, `sitemap.xml`, headers), OSINT (Google dorking, Wappalyzer, Wayback Machine, GitHub, S3 buckets), and automated (Gobuster across `dir`, `dns`, and `vhost` modes).
3. **Modern Web Stacks** — stack fingerprinting as a direct multiplier on exploitation speed, followed by four real, numbered CVEs across four stacks: MERN/Express (Prototype Pollution), Next.js (middleware auth bypass), Django (SQL injection via `order_by()`), and Apache 2.4.49 (path traversal → RCE).
4. **Web Server Attacks - I** — misconfiguration identification (not exploitation) across four Linux-hosted web servers: Apache2, Python's built-in HTTP server, Node.js/Express, and Nginx.

The throughline across all four rooms: **reconnaissance quality determines exploitation speed and accuracy.** Every room reinforces the same discipline — read the signals carefully before reaching for a tool or an exploit.

---

## 🧭 Room 1 — Walking An Application

**Focus:** Browser DevTools as a penetration testing toolkit — no automated scanners.

### What it covers
- **Page Source vs. live DOM**: Page Source is what the server sent; the Inspector shows the live, post-JavaScript state of the page — and lets you edit it client-side only, useful for understanding how (weak) client-side content gating like paywalls actually works.
- **HTML comments and hidden links**: developer comments and unlinked `<a href>` elements in the raw source are free reconnaissance — they're never shown to users but are sitting right there in the response.
- **Directory listing on static asset paths**: static file directories (CSS/JS/images) sometimes have directory browsing enabled, exposing every file inside, including ones never meant to be public.
- **Framework fingerprinting from source comments**: a comment naming the framework and version, leading to the framework's own site/changelog for update and vulnerability context.
- **The Debugger/Sources tab**: setting a breakpoint to pause JavaScript execution *before* a line runs — e.g., before a script removes or hides something from the page.
- **The Network tab**: capturing AJAX/XHR requests a page makes in the background, and reading their full response bodies even when nothing is rendered to the user.
- **The Storage/Application tab**: reviewing cookies and their security flags — `HttpOnly` (blocks JS access, mitigates XSS-based theft), `Secure` (HTTPS-only transmission), `SameSite` (CSRF mitigation).

### Key takeaway
> Before any automated tool, start manual. Comments, hidden links, JavaScript behavior, network requests, and local storage are all free information sources that require no active exploitation — and are exactly what pattern-matching scanners tend to miss, because they require human context and judgment.

---

## 🔍 Room 2 — Content Discovery

**Focus:** Finding content that isn't linked anywhere obvious, via manual, OSINT, and automated methods.

### Manual discovery
- **`robots.txt`**: tells crawlers what *not* to index — a guideline for bots, **not an access control**. A `Disallow:` entry is a map of "interesting" paths the site owner didn't want search-indexed, but it's directly reachable by anyone who requests it.
- **`sitemap.xml`**: the inverse — lists pages the owner *wants* indexed, sometimes including staging pages or hard-to-reach URLs with parameters worth testing.
- **HTTP headers & framework stack**: `Server`, `X-Powered-By`, and any custom headers reveal the tech stack; once identified, the framework's own documentation often reveals default admin panel paths and (sometimes still-unchanged) default credentials.

### OSINT
- **Google dorking**: `site:`, `inurl:`, `filetype:`, `intitle:`, `intext:`, `cache:` — combinable operators that surface indexed content the owner never intended to be easily found.
- **Wappalyzer**: browser extension/tool that fingerprints the full tech stack (CMS, framework, CDN, analytics, payment processors) often including version numbers.
- **Wayback Machine**: historical snapshots of a domain — can surface removed pages, old login forms, or forgotten endpoints that may still be live.
- **GitHub**: developers sometimes commit secrets before realizing a repo is public — and critically, **secrets removed in a later commit still exist in commit history** unless that history is explicitly rewritten.
- **S3 buckets**: `https://{name}.s3.amazonaws.com` — misconfigured public buckets are common; try naming patterns like `{company}-assets`, `{company}-backup`, `{company}-dev`.

### Automated discovery — Gobuster
Three modes, each solving a different problem:

| Mode | Resolves via | Finds |
|---|---|---|
| `dir` | HTTP requests against a wordlist of paths | Files and directories on a web server |
| `dns` | Actual DNS lookups | Real subdomains with DNS records |
| `vhost` | The `Host:` header sent to one fixed IP | Virtual hosts the web server itself routes, independent of DNS |

**The subdomain vs. virtual host distinction is the conceptual core of this room**: a subdomain is resolved *before* the request reaches the server (at the DNS layer); a virtual host is resolved *by the server itself*, reading the `Host:` header to decide which site to serve. This is why `vhost` scanning sends raw HTTP requests to one IP with a changing `Host` header rather than performing DNS lookups at all — and why a vulnerability patched on the main domain might not be patched on a same-server vhost that was never DNS-registered.

### Key takeaway
> A complete content discovery workflow runs all three layers — manual, OSINT, automated — before moving to exploitation. They're complementary, not redundant: manual is fast and precise, OSINT exploits information outside the target's control, automation covers breadth no human could match.

---

## ⚙️ Room 3 — Modern Web Stacks

**Focus:** Fast, evidence-based fingerprinting followed by exploitation of four real CVEs across four stacks.

> "During a time-boxed engagement, the first tester to identify `Apache/2.4.49` in a `Server:` header already knows the exact CVE before their teammate has finished running a port scan." — the room's framing, and the whole point of this section.

The workflow is identical for every stack: **fingerprint from passive HTTP signals (no payloads) → confirm version → understand the root cause → execute the exploit chain.**

### MERN / Express — Prototype Pollution
- **Fingerprint**: `X-Powered-By: Express`, `connect.sid` session cookie, plain-text `Cannot GET /path` on unhandled routes.
- **Root cause**: in JavaScript, every object inherits from a shared `Object.prototype`. A merge/update function that copies keys from user-supplied JSON into a target object **without filtering `__proto__`** can be tricked into writing directly onto `Object.prototype` itself — and that pollution then affects *every object in the process* that looks up the same property via the prototype chain, even objects that never explicitly set it.
- **Why it matters**: an admin check like `if (currentUser.isAdmin)` on a plain session object with no own `isAdmin` property can resolve to `true` once `Object.prototype.isAdmin` has been polluted — a full authentication bypass via a JSON body, no credentials needed.

### Next.js — Middleware Authentication Bypass (CVE-2025-29927, CVSS 9.1)
- **Fingerprint**: `X-Powered-By: Next.js`, `window.__next_f` in page source (definitive App Router indicator — RSC hydration payload), `x-nextjs-*` headers.
- **Root cause**: Next.js uses an internal header, `x-middleware-subrequest`, to prevent infinite middleware recursion loops. **It never validated whether that header originated from an internal process or an external client** — so an attacker simply including it in their own request causes the framework to skip middleware entirely, including any authentication check implemented there.
- **Why it matters**: middleware is commonly the *single, central* place developers implement auth checks in Next.js — bypassing it bypasses everything built on top of it, with one HTTP header and zero credentials.

### Django — SQL Injection via `order_by()` (CVE-2021-35042, CVSS 9.8)
- **Fingerprint**: `WSGIServer/0.2 CPython/X.X.X` server header, `csrftoken` cookie, and — the near-certain tell — a `csrfmiddlewaretoken` hidden field auto-injected into every POST form by `CsrfViewMiddleware`.
- **Root cause**: Django's ORM protects against SQL injection *when used as intended*. The vulnerability appears when a developer bypasses the ORM and builds raw SQL via string concatenation of user input — here, an `order` query parameter dropped directly into an `ORDER BY (CASE WHEN (1=1) THEN {order} ...)` clause.
- **Extraction technique**: MySQL's `updatexml()` raises an XPath error when given invalid XPath; wrapping a `SELECT` inside that XPath via `concat(0x7e, (select ...))` leaks the query result inside the resulting error message — visible only when Django's `DEBUG = True` surfaces full database errors in the HTTP 500 response.

### Apache 2.4.49 (LAMP) — Path Traversal → RCE (CVE-2021-41773, CVSS 9.8)
- **Fingerprint**: `Server: Apache/2.4.49 (Unix)` — an exact, unambiguous CVE match by itself. A `403` (not `404`) on `/cgi-bin/` confirms the directory exists with `mod_cgi` enabled.
- **Root cause**: a change to `ap_normalize_path()` in 2.4.49 broke traversal filtering due to **decode-order**: the filter runs *before* full URL decoding. A path like `.%2e/` isn't recognized as `../` by the filter at check time, but the OS resolves it as `../` once the filesystem call actually happens — the filter never sees what it's actually being asked to do.
- **Why it's critical, not just a file-read bug**: combined with `mod_cgi` enabled on `/cgi-bin/`, traversing to an executable like `/bin/sh` causes Apache to run it *as a CGI script*, piping the POST body to its stdin — full unauthenticated remote command execution.
- **`curl --path-as-is` is required**: `curl` normalizes `.%2e/`-style sequences before sending by default; this flag sends the URL exactly as typed.

### Key takeaway
> Every stack leaks its identity. Once you can read those signals fluently, you stop guessing and start targeting — the difference between a noisy, generic scanner run and a precise, evidence-driven attack chain.

---

## 🖥️ Room 4 — Web Server Attacks - I

**Focus:** Misconfiguration *identification* (not exploitation/RCE) across four Linux web server types.

### Fingerprinting baseline

| Port | Server | Default `Server` header |
|---|---|---|
| 80 | Apache2 | `Apache/2.4.x (Ubuntu)` |
| 8000 | Python HTTP Server | `SimpleHTTP/0.6 Python/3.xx.x` |
| 3000 | Node.js Express | *(none — uses `X-Powered-By: Express` instead)* |
| 8080 | Nginx | `nginx/1.xx.x` |

Default 404 error pages are a secondary fingerprint even when the `Server` header is suppressed — each server's error page has a distinct shape (plain text for Python, version-stamped HTML footer for Nginx, named HTML body for Apache).

### Python's built-in HTTP server
No access control whatsoever — not even for dotfiles. `python3 -m http.server` serves the entire working directory to anyone who can reach the port, including `.env` files and any backup archives left in the same directory. This server is dangerous purely by *existing* in the wrong place; there's no vulnerability to trigger, it's functioning exactly as designed.

### Apache2
- **`mod_status` (`/server-status`)**: real-time server stats. Correctly restricted to `localhost` by default via `Require local` — but a `Require all granted` directive *anywhere* in a vhost config silently overrides that, even on an otherwise-default setup.
- **Directory listing** (`Options +Indexes`): exposes everything in a directory with no index file.
- **Unlinked files via Gobuster**: `.bak` files and `.htpasswd` files are high-value Gobuster findings — backups often contain config/credential fragments, and an accessible `.htpasswd` yields crackable Basic Auth credential hashes.

### Node.js (Express)
Development-mode habits that survive into production:
- **Custom error handlers** that leak full stack traces regardless of `NODE_ENV` — Express's own default error handler respects `NODE_ENV=production`, but a developer-written handler often doesn't.
- **Debug endpoints enumerating all registered routes** (reading Express's internal `app._router.stack`) — a massive recon shortcut when present.
- **Debug endpoints dumping `process.env`** — direct credential/secret exposure, plus `NODE_ENV=development` itself as a production deployment red flag.
- **Static files containing embedded config** (internal hostnames, API base URLs, debug flags) — technically "meant to be public" since the browser needs them, but often containing more than intended.
- Note: `express.static()` returns 404 for dotfiles by default — the *opposite* of Python's HTTP server, which serves them like anything else.

### Nginx
- **`server_tokens`**: controls version disclosure in both the `Server` header *and* default error page footers simultaneously.
- **`autoindex on;`**: Nginx's equivalent of Apache's directory listing — a documented, legitimate directive misused on the wrong path.
- **`stub_status` / `/nginx_status`**: Nginx's equivalent of `mod_status` — correctly restricted via `allow 127.0.0.1; deny all;`, commonly misconfigured to `allow all;`.

### Cross-cutting patterns
- **Missing security headers are universal** across all four server types by default: `X-Frame-Options`, `X-Content-Type-Options`, `Content-Security-Policy`, `Referrer-Policy`. None of these are ever present without active configuration.
- **Nikto** automates the header/misconfiguration sweep quickly but is loud and easily detected — appropriate for authorized testing only, and limited to pattern-based findings (it won't catch logic-level issues like the Prototype Pollution or ORM injection from Room 3).

### Key takeaway
> Default configurations prioritize ease of deployment over security everywhere. Version disclosure, directory listings, and status pages are default-on for diagnostic convenience — disabling them takes deliberate action, and finding them in a real engagement usually means defaults were simply never reviewed, not that anyone was careless.

---

## 📚 Core Concepts Reference

**Page Source vs. DOM** — Source is static, server-delivered; DOM is live and JS-mutated. Client-side-only content hiding (paywalls via CSS `display`) is not real security — the data is already in the browser.

**Subdomain vs. Virtual Host** — Subdomain resolves via DNS (before the request reaches the server); Virtual Host resolves via the `Host:` HTTP header (by the server itself, after the request arrives).

**Prototype Pollution** — Unfiltered `__proto__` (or `constructor.prototype` as a bypass route) in a recursive merge function writes onto the shared `Object.prototype`, affecting every object in the process via the prototype chain.

**CVE-2025-29927** — Next.js trusted an internal-only header (`x-middleware-subrequest`) without validating its origin, letting any client skip middleware-based auth entirely.

**Django ORM SQL injection** — Only occurs when a developer bypasses the ORM's parameterized-query protection via raw string concatenation; `updatexml()` error-based extraction requires `DEBUG = True`.

**CVE-2021-41773** — Apache 2.4.49's traversal filter ran before full URL decoding, so `.%2e/` bypassed it at filter time but still resolved to `../` at filesystem-call time; combined with `mod_cgi`, this became full RCE.

**Default configurations vs. security** — Nearly every finding across all four rooms traces back to the same root cause: defaults favor diagnostics and ease of deployment, and removing that exposure requires deliberate, active configuration.

---

## 💻 Command Reference

```bash
# --- Room 1: Browser DevTools has no CLI equivalent — see the guide for step-by-step UI instructions ---

# --- Room 2: Content Discovery ---
curl http://MACHINE_IP/robots.txt
curl http://MACHINE_IP/sitemap.xml
curl -v http://MACHINE_IP                      # headers + framework fingerprint

gobuster dir -u http://MACHINE_IP -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt -x bak,txt
gobuster dns -d example.thm -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt --wildcard
gobuster vhost -u http://MACHINE_IP --domain example.thm -w /path/to/wordlist --append-domain --exclude-length 250-320

# --- Room 3: Modern Web Stacks ---
curl -I MACHINE_IP:3000/                        # MERN/Express fingerprint
curl -b cookies.txt -X POST http://MACHINE_IP:3000/api/user/update \
  -H "Content-Type: application/json" -d '{"__proto__": {"isAdmin": true}}'

curl -H "x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware" \
  http://MACHINE_IP:3001/dashboard                # Next.js middleware bypass

curl -s "http://MACHINE_IP:8000/products/?order=updatexml(1,concat(0x7e,(select%20database())),1)" \
  | grep -o '~[0-9a-zA-Z_][^&]*'                  # Django error-based SQLi

curl -s --path-as-is "http://MACHINE_IP:8080/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh" \
  --data 'echo Content-Type: text/plain; echo; id'  # Apache 2.4.49 traversal RCE

# --- Room 4: Web Server Attacks - I ---
curl -sI http://MACHINE_IP:PORT                 # fingerprint any of the 4 servers
curl -s http://MACHINE_IP:PORT/nonexistent-page-xyz   # default error page fingerprint
curl -s http://MACHINE_IP:8000/.env             # Python HTTP server — dotfile exposure
curl -s http://MACHINE_IP:80/server-status       # Apache mod_status
curl -s http://MACHINE_IP:3000/api/debug/env     # Express debug endpoint
curl -s http://MACHINE_IP:8080/nginx_status      # Nginx stub_status

for port in 80 8000 3000 8080; do
  curl -sI http://MACHINE_IP:$port/ | grep -iE "x-frame-options|x-content-type|content-security-policy|strict-transport|referrer-policy"
done

nikto -h http://MACHINE_IP:80 -nointeractive
```

---

## 🧬 CVE Summary Table

| Stack | CVE | Impact | CVSS |
|---|---|---|---|
| MERN / Express | CVE-2020-8203 | Prototype pollution → auth bypass | 7.4 High |
| Next.js Middleware | CVE-2025-29927 | Single header → full middleware bypass | 9.1 Critical |
| Django ORM | CVE-2021-35042 | SQL injection via unparameterized `ORDER BY` | 9.8 Critical |
| Apache LAMP | CVE-2021-41773 | Path traversal + `mod_cgi` → RCE | 9.8 Critical |

---

## 📖 Lessons Learned

- Manual inspection always comes before automation — comments, hidden links, JS behavior, network traffic, and storage require human judgment that pattern-matching scanners don't have.
- `robots.txt` is a crawler *courtesy*, never an access control — treat every `Disallow:` entry as a map of interesting paths worth visiting directly.
- The subdomain/vhost distinction isn't academic: a vulnerability patched on the main domain may not be patched on a same-server vhost with no DNS record pointing to it.
- Framework fingerprints (`csrfmiddlewaretoken`, `connect.sid`, `window.__next_f`, `X-Powered-By`) are often more reliable than version banners, which are easier to suppress.
- The most severe modern vulnerabilities often come from a *trust boundary* failure, not memory corruption: Next.js trusting an unvalidated internal header, or a merge function trusting unfiltered object keys.
- A server requires **no vulnerability at all** to be a critical finding — Python's HTTP server is dangerous purely through its total absence of access control, wherever it happens to be running.
- Defaults are rarely malicious, almost always just unreviewed. Document findings accordingly — the fix is usually a one-line configuration change, not a patch.

---

## ✅ Skills Practiced

- [x] Browser DevTools-based manual reconnaissance (Inspector, Debugger, Network, Storage)
- [x] Page source and comment-based information leakage analysis
- [x] `robots.txt` / `sitemap.xml` reconnaissance
- [x] HTTP header and framework fingerprinting
- [x] Google dorking and OSINT tooling (Wappalyzer, Wayback Machine, GitHub, S3)
- [x] Gobuster `dir`, `dns`, and `vhost` enumeration
- [x] Prototype pollution exploitation (Node.js/Express)
- [x] Next.js middleware authentication bypass (CVE-2025-29927)
- [x] Error-based SQL injection via Django ORM bypass (CVE-2021-35042)
- [x] Path traversal → RCE via Apache `mod_cgi` (CVE-2021-41773)
- [x] Cross-server misconfiguration identification (Apache, Python, Node.js, Nginx)
- [x] Security header auditing

*(Practiced — not claimed as mastered. Revisit periodically.)*

---

*Full guided write-up with per-question breakdowns in Arabic: [`Web-Application-Security-Fundamentals-Arabic-Guide.pdf`](./Web-Application-Security-Fundamentals-Arabic-Guide.pdf)*
