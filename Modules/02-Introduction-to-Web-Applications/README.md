# Module 02 — Introduction to Web Applications

![Status](https://img.shields.io/badge/Status-Complete-brightgreen)
![Platform](https://img.shields.io/badge/Platform-HackTheBox-red)
![Category](https://img.shields.io/badge/Category-General-blue)

## What This Module Is About
Understanding how web applications are built, how they differ from
static websites, and why they are such a massive attack surface.
Before you attack something you need to understand what it is.

## Sections Progress
- [x] Introduction
- [x] Web Application Layout
- [x] Front End vs Back End
- [x] HTML
- [x] CSS
- [x] JavaScript
- [ ] Sensitive Data Exposure
- [ ] HTML Injection
- [ ] Cross-Site Scripting XSS
- [ ] Cross-Site Request Forgery CSRF
- [ ] Back End Servers
- [ ] Web Servers
- [ ] Databases
- [ ] Development Frameworks and APIs
- [ ] Common Web Vulnerabilities
- [ ] Public Vulnerabilities

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
| One Server | ❌ Worst | Everything together — one breach = total loss |
| Client-Server | ✅ Standard | Front end in browser, back end on server |
| Many Servers — One DB | ✅ Better | Server breach doesn't take down DB |
| Many Servers — Many DBs | ✅ Best | Full segmentation, redundancy, proper access control |

### Three Tier Architecture

| Layer | What It Does |
|-------|-------------|
| Presentation | HTML, CSS, JS — what user sees in browser |
| Application | Processes requests, checks auth and privileges |
| Data | Stores and retrieves data for the application |

### Microservices
App broken into independent components each doing one job —
payments, search, ratings etc. Stateless communication between
them. Can be written in different languages and still interact.

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
- Front end = **Whitebox** — source code readable directly in browser
- Back end = **Blackbox** — hidden on server by default
- LFI vulnerabilities can leak back end source → turns blackbox
  into whitebox mid-engagement

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
Structure is a tree called the DOM.

```html
<!DOCTYPE html>
<html>
    <head>
        <title>Page Title</title>  <!-- not visible to user -->
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
Browsers only support ASCII in URLs. Everything else gets
percent-encoded — replaced with `%XX`.

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

XSS relevance — injected JS manipulates the DOM to steal
cookies, redirect users, or modify page content silently.

---

## CSS — Cascading Style Sheets

Defines how HTML elements look — colors, fonts, sizes, layout.

```css
body {
  background-color: black;
}

h1 {
  color: white;
  text-align: center;
}

p {
  font-family: helvetica;


---

## Sensitive Data Exposure

Front end code runs on the client side — meaning anyone can read
it. Developers often leave sensitive information in page source
by accident.

### What to Look For in Page Source
- Hardcoded credentials in HTML comments
- Hidden links and directories
- Debug parameters left behind
- API keys or tokens in JS files
- Test pages never removed from production

### How to View Page Source