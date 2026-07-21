# Module 02 — Introduction to Web Applications


## What This Module Is About
Understanding how web applications are built, how they differ from
static websites, and why they are such a massive attack surface.
Before you attack something you need to understand what it is.

## Sections
- [x] Introduction
- [ ] More sections coming as I progress...

## Web App vs Static Website
- Static = same for everyone, manually updated, Web 1.0
- Web App = dynamic, changes per user, fully functional, Web 2.0

## Why Web Apps Are a Massive Attack Surface
- Accessible by anyone with a browser worldwide
- Constantly changing — a single code change can introduce a critical vuln
- Linked to databases with sensitive data
- Automated tools make scanning and attacking easier than ever

## Real World Attack Examples

| Vulnerability | Real World Impact |
|---------------|-------------------|
| SQL Injection | Extract AD usernames → password spray VPN/email portal |
| File Inclusion | Read source code → find hidden pages → RCE |
| File Upload | Upload malicious file as profile picture → full server control |
| IDOR | Change `/user/701/edit` to `/user/702/edit` → access another user |
| Broken Access Control | Register with `roleid=0` in POST body → instant admin account |

---

## Web Application Layout

Three main categories every web app is built around:

| Category | Description |
|----------|-------------|
| Infrastructure | Database, servers, hosting setup |
| Components | UI/UX, Client, Server parts |
| Architecture | Relationships between all components |

---

## Infrastructure Models

### Client-Server
Most common. Server hosts the app, browser is the client.
Front end runs in browser, back end runs on server.

### One Server
Everything on one machine — app, database, all of it.
Riskiest model. One breach = everything compromised.
Classic "all eggs in one basket."

### Many Servers — One Database
Multiple web servers, single shared database.
If one server is compromised, others still run.
Better segmentation than one server model.

### Many Servers — Many Databases
Each app gets its own database, sometimes its own DB server.
Best security — proper segmentation and access control.
Used for redundancy too — backup runs if primary goes down.

---

## Three Tier Architecture

| Layer | What It Does |
|-------|-------------|
| Presentation | What user sees — HTML, CSS, JS in browser |
| Application | Processes requests, checks auth and privileges |
| Data | Stores and retrieves data for the application |

---

## Microservices and Serverless

**Microservices** — app broken into independent components,
each doing one job (payments, search, ratings etc).
Stateless communication between them.
Can be written in different languages and still talk to each other.

**Serverless** — runs in cloud containers like Docker on AWS/GCP/Azure.
No server management needed — cloud handles it all.

---

## Security Perspective
- One server model = single point of failure and single point of compromise
- Architecture flaws are just as dangerous as code flaws
- Missing RBAC = users accessing admin features without a single line
  of vulnerable code — it is a design error not a coding error
- If you breach a server and can not find the database — it is
  probably on a separate server. Keep enumerating.

<<<<<<< HEAD
---

## Front End vs Back End

### Front End
Runs in the browser — everything the user sees and interacts with.
Built with three technologies:
- **HTML** — structure and content
- **CSS** — design and styling
- **JavaScript** — functionality and interaction

### Back End
Runs on the server — everything the user never sees.

| Component | Examples |
|-----------|---------|
| Back End Servers | Linux, Windows, Docker containers |
| Web Servers | Apache, NGINX, IIS |
| Databases | MySQL, PostgreSQL, MongoDB |
| Frameworks | Laravel, Django, Spring, ASP.NET |

### Security Angle
- **Front end** = Whitebox — we can read the source code directly
- **Back end** = Blackbox — source is hidden on the server by default
- LFI vulnerabilities can leak back end source code — turning
  blackbox into whitebox mid-engagement

---

## Top Developer Mistakes (Pentester's Checklist)

| # | Mistake |
|---|---------|
| 1 | Permitting invalid data into the database |
| 5 | Storing plain text passwords |
| 7 | Storing unencrypted data in database |
| 8 | Depending excessively on client side validation |
| 10 | Permitting variables via URL path |
| 13 | Unverified SQL injections |
| 14 | Remote file inclusions |

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

## HTML Basics

HTML is the skeleton of every web page. Browser reads it and
renders it visually. Structure is a tree called the DOM.

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

Key elements:
- `<head>` — not visible, holds title, CSS, JS links
- `<body>` — everything the user sees
- `<style>` — CSS goes here
- `<script>` — JavaScript goes here

### URL Encoding
Browsers only support ASCII in URLs. Everything else gets
percent-encoded — special characters replaced with `%XX`.

| Character | Encoded |
|-----------|---------|
| space | %20 |
| `'` | %27 |
| `"` | %22 |
| `#` | %23 |

Why this matters for pentesting — SQL injection payloads and
XSS often need URL encoding to pass through filters or get
correctly interpreted by the server.

### DOM — Document Object Model
Every HTML element is a DOM node. JavaScript can access and
manipulate any element via the DOM.
Useful for XSS — we can inject JS that reads, modifies, or
creates DOM elements to steal data or hijack sessions.

=======
>>>>>>> 7be55426794e3a3164b786ebabb29ceeaab9a089
## Status
🔄 In progress — updating as I complete each section