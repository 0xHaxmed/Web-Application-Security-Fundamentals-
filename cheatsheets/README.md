# ⚡ Master Cheat Sheet & Self-Assessment Checklist

> **Language Navigation:** [🇸🇦 النسخة العربية](./README.ar.md) | [⬅️ Return to Root](../README.md)

---

## 📌 Part 1: Quick Revision Cheat Sheet

### Room 1 — Walking An Application
* **Page Source vs. Live DOM:** Page Source is the immutable static HTML received from the server; the Inspector DOM reflects runtime JavaScript mutations and dynamic CSS states.
* **Debugger Breakpoints:** Pauses JavaScript execution before DOM removal functions (`.remove()`, `.style.display = 'none'`) execute, keeping transient elements visible.
* **Network Tab:** Inspects background AJAX/Fetch/XHR responses; often leaks internal JSON objects, debug messages, and sensitive fields omitted from the frontend UI.
* **Cookie Flags:** `HttpOnly` blocks JavaScript access (`document.cookie`), mitigating XSS cookie theft; `Secure` enforces HTTPS transmission; `SameSite` mitigates CSRF.

### Room 2 — Content Discovery
* **`robots.txt`:** Advisory crawler guidelines, **never access control**. Disallowed paths serve as an attacker roadmap of interesting endpoints.
* **`sitemap.xml`:** Structured list of URLs intended for indexing; frequently leaks beta, staging, and unlinked endpoints.
* **Subdomain vs. Virtual Host:**
  * **Subdomain (`gobuster dns`):** Resolved at the DNS layer *before* reaching the server.
  * **Virtual Host (`gobuster vhost`):** Resolved by the web server reading the HTTP `Host:` header; can route to unlinked portals with **zero public DNS records**.

### Room 3 — Modern Web Stacks
* **MERN / Express (`CVE-2020-8203`):** Unsanitized recursive object merging pollutes `Object.prototype`, propagating attacker-controlled attributes (`isAdmin: true`) across the entire runtime.
* **Next.js Middleware (`CVE-2025-29927`):** Unvalidated internal header `x-middleware-subrequest` instructs Next.js to bypass all centralized middleware authentication checks.
* **Django ORM (`CVE-2021-35042`):** Raw string concatenation inside `order_by()` bypasses parameterized query protection; leaks database data via MySQL `updatexml()` XPath errors when `DEBUG = True`.
* **Apache 2.4.49 (`CVE-2021-41773`):** Decode-order path normalization flaw allows `.%2e/` traversal; combined with `mod_cgi`, traversing to `/bin/sh` turns traversal into unauthenticated RCE (requires `curl --path-as-is`).

### Room 4 — Web Server Attacks - I
* **Python Built-in HTTP Server:** Has zero access control; serves working directory dotfiles (`.env`) and backup archives directly to unauthenticated clients.
* **Apache Diagnostics (`mod_status`):** `/server-status` discloses active visitor IPs and live request URLs; `Require local` is silently overridden by `Require all granted` in any vhost.
* **Express Development Leaks:** Running with `NODE_ENV=development` leaks verbose stack traces, route tables (`/api/routes`), and environment dumps (`/api/debug/env`).
* **Nginx Diagnostics & Listing:** `autoindex on;` enables directory browsing; `stub_status` exposes connection metrics; `server_tokens off;` suppresses version banners.
* **Security Headers:** Completely absent across all four servers by default (`X-Frame-Options`, `X-Content-Type-Options`, `CSP`, `HSTS`, `Referrer-Policy`).

---

## 🎯 Part 2: 15-Point Interview & Exam Self-Assessment

Before considering this module mastered, verify that you can explain each of the following 15 concepts from memory without looking at reference notes:

- [ ] **1. Page Source vs. Live DOM:** When is each used, and why is client-side content hiding ineffective?
- [ ] **2. Breakpoints:** How do you set a breakpoint to freeze dynamic DOM removal in JavaScript?
- [ ] **3. Network Tab Analysis:** How do background AJAX requests expose data omitted from the UI?
- [ ] **4. Cookie Security Flags:** What are the exact defensive guarantees of `HttpOnly`, `Secure`, and `SameSite`?
- [ ] **5. `robots.txt` vs. `sitemap.xml`:** What is the architectural purpose of each file from an attacker's perspective?
- [ ] **6. Google Dorking:** What are the primary operators (`site:`, `inurl:`, `filetype:`, `intitle:`) and how are they combined?
- [ ] **7. Git Commit History:** Why does deleting a secret in a subsequent commit fail to remediate exposure?
- [ ] **8. Subdomain vs. Virtual Host:** What is the technical routing difference between DNS resolution and the `Host:` header?
- [ ] **9. Prototype Pollution:** How does an attacker traverse from an object key to `Object.prototype` in JavaScript?
- [ ] **10. CVE-2025-29927 (Next.js):** Why did Next.js fail to validate `x-middleware-subrequest` and what was the impact?
- [ ] **11. Error-Based SQLi in Django:** How does MySQL's `updatexml()` leak subqueries, and why is `DEBUG = True` required?
- [ ] **12. CVE-2021-41773 (Apache):** How did decode order enable `.%2e/` traversal, and why did `mod_cgi` enable RCE?
- [ ] **13. Python HTTP Server Risks:** Why is `python3 -m http.server` dangerous in production environments?
- [ ] **14. Server Diagnostic Modules:** Compare Apache's `mod_status` with Nginx's `stub_status` in terms of exposure.
- [ ] **15. Security Headers:** List the five core defensive security headers and describe the attack each mitigates.
