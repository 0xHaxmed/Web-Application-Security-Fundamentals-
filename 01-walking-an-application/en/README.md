# 🧭 Room 1 — Walking An Application

> **Language Navigation:** [🇸🇦 النسخة العربية](./README.ar.md) | [💼 Real-World Impact & Career Guide](./real-world-impact.md) | [⬅️ Return to Root](../README.md)

---

## 🎯 Executive Summary & Mental Model

"Walking an application" is the discipline of conducting comprehensive manual reconnaissance on a web application using nothing more than a standard browser and its built-in Developer Tools (DevTools). In modern penetration testing, automated scanners (e.g., Nessus, Burp Scanner, Zap) are ubiquitous, but they operate predominantly on signature matching and generic crawler heuristics. 

A human tester with an investigative mindset observes anomalies, business logic implementations, client-side assumptions, developer comments, and timing artifacts that scanners blind-fire past.

```
       [ Client Browser ]
        │             ▲
        │ Page Source │ Inspector (Live DOM)
        ▼             │
┌───────────────┐  ┌──────────────────┐
│ Initial HTML  │─▶│ JS Engine / CSSOM│──▶ Live Rendered UI
│ (Server sent) │  │  Execution State │    (Paywalls/Hiding)
└───────────────┘  └──────────────────┘
        │                     ▲
        ▼                     │
 [ Network Tab ] ───────▶ [ Debugger ]
 (AJAX/Fetch API)         (Breakpoints & State Freezing)
```

### The Core Pentester Principle
> **Never trust client-side enforcement.** Anything sent down to the client's device—whether hidden via CSS, stripped by a client-side JavaScript timer, or stored in cookies without proper defensive flags—is fully under the tester's control.

---

## 📋 Task Breakdown & Investigative Methodology

### Task 1 — Walking An Application (Environment Setup)
* **Objective:** Establish network reachability to the target lab instance.
* **Target:** Acme IT Support Portal (`http://MACHINE_IP/`).
* **Concept:** Before launching any analysis, verify that DNS, routing, and machine services are up. If a web server fails to respond, checking basic network connectivity (`ping`, `curl -I`) prevents wasting time debugging what turns out to be an unbooted machine.

---

### Task 2 — Exploring The Website (Application Mapping)
* **Concept:** Application Mapping is building a mental graph of every accessible route, user role, functional boundary, parameter, and interaction point on the target.
* **The Pentester's Checklist:**
  1. *Form Discovery:* Which pages accept input (`<form>`, search inputs, ticket creation)?
  2. *Parameter Tracking:* Which links pass URL queries (e.g., `?id=1`, `?page=contact`)? These represent immediate candidates for SQLi, IDOR, or SSRF.
  3. *Privilege Gating:* Which pages require session authentication versus unauthenticated public availability?
* **Investigative Step-by-Step:**
  1. Log in with user credentials into the portal.
  2. Navigate to the **Customers** / **Dashboard** section.
  3. Locate the user action button: **Create Ticket**.
  4. Inspect the destination URL path or form submission endpoint.
* **Evidence Location:** The browser URL bar upon clicking the ticket creation interface or viewing the `href` attribute on the button element in the DOM.

---

### Task 3 — Viewing The Page Source (Static Code Reconnaissance)
* **Concept:** The raw Page Source (`view-source:http://...` or `Ctrl + U`) represents the exact HTTP response body sent across the wire by the web server *before* the browser's JavaScript engine parses, executes, or mutates the document.

#### 1. Developer Comments (`<!-- ... -->`)
* **Why it matters:** Developers frequently leave internal notes, roadmap details, developer contact info, or testing routes directly in HTML comments, forgetting that comments are delivered to every visitor.
* **Method:** Open Page Source (`Ctrl + U`), press `Ctrl + F`, and search for `<!--`.
* **Investigation:** Locate the top comment referencing developmental or pending pages (e.g., an unlinked alternative page under construction).

#### 2. Hidden Links & Assets (`<a href="..." hidden>`)
* **Why it matters:** Links may be intentionally hidden via CSS or omitted from navigation menus, but the anchor tag remains in the HTML.
* **Method:** Search for `<a ` tags in the page source. Specifically look for links whose anchor or path begins with suspicious strings (e.g., `secr`, `private`, `admin`, `staging`).
* **Evidence:** Open the discovered relative path directly in the browser address bar to view the unlinked asset.

#### 3. Static Asset Directory Browsing (`Directory Listing`)
* **Why it matters:** Static folders (`/assets/`, `/images/`, `/static/`, `/uploads/`) are often configured without an `index.html` file. If web server options permit directory listing (`Options +Indexes` in Apache or `autoindex on;` in Nginx), the server serves a file index.
* **Method:** Identify asset paths referenced in `<link>` or `<script>` tags (e.g., `http://MACHINE_IP/assets/`). Strip the filename and request the base folder: `http://MACHINE_IP/assets/`.
* **Evidence:** If directory browsing is active, review the indexed files. Look for `.txt`, `.bak`, `.log`, or target flags (e.g., `flag.txt`).

#### 4. Framework Fingerprinting from Source Comments
* **Why it matters:** Comments at the bottom of the source often state the framework and version (e.g., `Powered by XYZ Framework v1.2`).
* **Method:** Scroll to the footer of the HTML source. Note the framework name and exact version number.
* **Exploitation Link:** Check public vulnerability databases (NVD, Exploit-DB, GitHub advisories) or changelogs for known issues and default admin paths (`/admin`, `/login`).

---

### Task 4 — Developer Tools: Inspector (DOM Manipulation & Client-Side Gating)
* **Concept:** The **Elements / Inspector** tab reveals the **live DOM (Document Object Model)**. Unlike static page source, the DOM reflects client-side modifications made by CSS and JavaScript.
* **The Vulnerability Pattern — Client-Side Paywalls & Overlays:**
  Many low-cost paywalls or modal gates do not withhold content on the backend; they send the full premium content to the client browser and simply position a visual overlay (mask) over it, or set the container's CSS property to `display: none;` or `visibility: hidden;`.
* **Investigative Step-by-Step:**
  1. Navigate to the **News** or premium article section.
  2. Notice the article is blocked by an overlay prompting for a subscription (`Paywall / Overlay`).
  3. Right-click the blocking overlay and choose **Inspect**.
  4. In the DOM tree, locate the overlay container (e.g., `<div class="premium-customer-blocker">` or similar gating wrapper).
  5. **Bypass Method A:** In the Styles panel, change `display: block;` to `display: none;` (or uncheck the property).
  6. **Bypass Method B:** Select the blocking node in the DOM tree and press `Delete` on your keyboard.
* **Evidence:** The obscured text beneath becomes immediately legible. The sensitive value/flag was delivered to the browser all along.

---

### Task 5 — Developer Tools: Debugger / Sources (JavaScript State Control)
* **Concept:** The **Debugger** (called **Sources** in Chrome DevTools) allows you to pause the execution of JavaScript in real time using **Breakpoints**.

```
[ Page Load Event ] ──▶ [ Script Runs ] ──▶ [ Element Hidden/Removed ]
                                │
                    [ Breakpoint Hits Here ] ⏸️ Execution Paused
                                │
                     DOM Remains Fully Visible!
```

* **Key Terms:**
  * **Minification:** Whitespace and formatting removal to reduce file transfer size (not security).
  * **Obfuscation:** Deliberately scrambling variable and function names to deter human reading.
  * **Pretty Print (`{ }`):** Automatically re-indents and formats minified code into readable lines.
  * **Breakpoint:** An execution pause flag set on a specific line of code.

* **Investigative Step-by-Step:**
  1. Open the `/contact` page. Observe that a red alert box flashes for a fraction of a second and disappears.
  2. Open DevTools -> **Debugger / Sources** tab.
  3. Expand the project tree and locate client scripts under `/assets/`.
  4. Click the **Pretty Print** icon (`{ }`) at the bottom of the script window to make the code readable.
  5. Search for DOM removal methods: `.remove()`, `.style.display = 'none'`, `delete`, or element IDs matching the contact container.
  6. Click the line number containing the removal function to set a **Breakpoint** (it turns blue).
  7. Refresh the page (`F5`).
  8. Execution pauses *before* the removal line runs! The red box stays frozen on the screen.
* **Evidence:** Read the sensitive text or flag displayed inside the frozen red box directly on the viewport.

---

### Task 6 — Developer Tools: Network (API & Background Request Inspection)
* **Concept:** Modern web applications rely heavily on asynchronous background communication (AJAX, Fetch API, XHR). When an action occurs, the page does not reload; instead, the browser fires an HTTP request in the background and processes the response.
* **Security Risk:** Developers often return complete database objects, internal tokens, or diagnostic messages in the JSON response, displaying only a small generic success string in the frontend UI.
* **Investigative Step-by-Step:**
  1. Navigate to `/contact`.
  2. Open DevTools -> **Network** tab. Ensure recording is enabled (red circle icon).
  3. Fill out the contact form with sample inputs and click **Submit / Send**.
  4. Watch the Network log for a new outbound request (e.g., `POST` to `/contact-msg` or `/api/contact`).
  5. Click on the request entry and select the **Response** (or **Preview**) tab.
* **Evidence:** Compare what was shown on the webpage ("Message received") against the raw server response body. The JSON response exposes internal confirmation details and the extracted task string.

---

### Task 7 — Developer Tools: Storage / Application (Client-Side State & Cookie Flags)
* **Concept:** Web applications store state data on the client device across several storage types:
  * **Local Storage:** Persistent key-value storage with no expiration unless explicitly cleared.
  * **Session Storage:** Temporary key-value storage restricted to the active browser tab.
  * **Cookies:** Key-value pairs transmitted with HTTP requests, essential for session management.

#### Critical Cookie Security Attributes:
| Attribute | Security Guarantee | Attack Mitigated |
|---|---|---|
| `HttpOnly` | Prohibits JavaScript (`document.cookie`) from reading the cookie | Mitigates session hijacking via Cross-Site Scripting (XSS) |
| `Secure` | Cookie is transmitted strictly over encrypted HTTPS connections | Mitigates cleartext interception (Man-in-the-Middle / sniffing) |
| `SameSite` | Controls cross-site cookie transmission (`Strict`, `Lax`, `None`) | Mitigates Cross-Site Request Forgery (CSRF) |

* **Investigative Step-by-Step:**
  1. Create a user account or log in at `/customers/signup` / `/customers/login`.
  2. Open DevTools -> **Storage** (Firefox) or **Application** (Chrome).
  3. In the left sidebar, expand **Cookies** and select the target host domain.
  4. Identify the active session cookie (e.g., `session`, `PHPSESSID`, `connect.sid`).
  5. Inspect the `HttpOnly` column.
* **Evidence:** Note whether `HttpOnly` is marked `true` or `false`. If `false`, any injected JavaScript can immediately read and exfiltrate this session token.

---

### Task 8 — Room Conclusion
* **Key Mental Synthesis:**
  * Automated tools do not understand human intention or visual context.
  * Client-side controls exist for user experience (UX), never for authorization or security.
  * Every DevTools tab provides a different vantage point on the application:
    * **Page Source:** Raw server baseline.
    * **Inspector:** Live mutated client state.
    * **Debugger:** Temporal execution control.
    * **Network:** API communication channel.
    * **Storage:** State and credential tokens.

---

## 🛠️ Rapid Command & Action Reference

```bash
# Verify base reachability and server headers
curl -I http://MACHINE_IP/

# Inspect raw HTML for comments and hidden links
curl -s http://MACHINE_IP/ | grep -E "<!--|hidden|secr"

# Check if directory listing is enabled on common asset folders
curl -s -o /dev/null -w "%{http_code}\n" http://MACHINE_IP/assets/

# Inspect cookie attributes via CLI headers
curl -i -s http://MACHINE_IP/customers/login | grep -i "set-cookie"
```
