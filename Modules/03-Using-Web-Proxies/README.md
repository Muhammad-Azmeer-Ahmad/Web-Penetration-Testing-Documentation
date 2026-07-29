# Module 03 — Using Web Proxies

![Status](https://img.shields.io/badge/Status-In%20Progress-yellow)
![Platform](https://img.shields.io/badge/Platform-HackTheBox-red)
![Category](https://img.shields.io/badge/Category-Offensive-orange)

## Sections Progress
- [x] Intro to Web Proxies
- [x] Setting Up
- [x] Proxy Setup
- [ ] Site Map
- [ ] Repeater
- [ ] Intruder
- [ ] Decoder / Comparer
- [ ] Sequencer
- [ ] Extensions
- [ ] Fuzzing with Proxies
- [ ] Automating with Macros
- [ ] Skills Assessment

---

## What This Module Is About
How to capture, inspect, and manipulate the traffic between a browser/app and a back-end server using web proxies. Nearly every technique in the rest of the path — SQLi, XSS, fuzzing, auth bypass — depends on being able to see and modify requests first. This module is the tooling foundation everything else builds on.

---

## Intro to Web Proxies

Web proxies sit between the client (browser/mobile app) and the back-end server, acting as MITM tools that capture every request and response passing between them. Unlike full packet sniffers like Wireshark — which analyze *all* local traffic — proxies focus specifically on web ports (HTTP/80, HTTPS/443), which keeps the noise down and the workflow fast.

### Uses Beyond Intercept/Replay

| Use Case | Purpose |
|----------|---------|
| Vulnerability scanning | Automated detection of common web flaws |
| Fuzzing | Send malformed/edge-case input to find bugs |
| Crawling | Map out app structure and endpoints |
| App mapping | Build a picture of the attack surface |
| Request analysis | Inspect headers, params, auth tokens |
| Config testing | Check server/app misconfigurations |
| Code review support | Cross-reference traffic with source |

### Burp Suite vs ZAP

| | Burp Suite | ZAP |
|-|-----------|-----|
| Cost | Free (Community) / Paid (Pro/Enterprise) | Fully free, open-source |
| Active scanner | Pro only | Free |
| Intruder speed | Throttled in Community | No throttling |
| Extensions | Limited in Community | Community-driven, growing |
| Interface | Polished, built-in Chromium browser | Functional, less polished |

### Real Impact
- Burp is the industry-standard tool — most job listings and reports assume Burp familiarity
- ZAP's lack of throttling makes it the fallback when Intruder speed on Community edition becomes a bottleneck
- Neither tool alone is "the answer" — pick based on client budget, engagement scale, and what needs automating

### Pentesting Relevance
- Foundation tool — nearly every later technique (fuzzing, IDOR, auth bypass) routes through the proxy
- Burp Community is enough for most engagements; Pro's active scanner and Intruder speed matter at scale or corporate pentests
- Know both — client environments and CTF/box setups won't always have your preferred tool available

---

## Setting Up

Both Burp and ZAP come pre-installed on PwnBox, Parrot, and Kali. For a manual install on your own VM:

| Tool | Launch (installed) | Launch (JAR, cross-platform) |
|------|--------------------|-------------------------------|
| Burp | `burpsuite` | `java -jar </path/to/burpsuite.jar>` |
| ZAP | `zaproxy` | `java -jar </path/to/zap.jar>` |

Both rely on a Java Runtime Environment (JRE), normally bundled with the installer.

### Project Types

| Tool | Option | Notes |
|------|--------|-------|
| Burp Community | Temporary only | No save/resume — start fresh each session |
| Burp Pro | Temporary or disk-based | Persistent projects for long/large engagements |
| ZAP | Persistent (named/timestamped) or non-persistent | Choose "no" for quick temp sessions |

For one-off testing or labs, temporary/non-persistent is the default. Persistent projects matter for large scopes or long-running active scans you need to resume later.

### Real Example
Burp Community project setup only offers "temporary project" — no disk save option exists unless upgraded to Pro. Confirmed by walking through the setup wizard: Pro adds a second option ("New project on disk") that Community doesn't show at all.

### Pentesting Relevance
- Temporary projects are the default workflow for quick engagements/labs — no state to manage or clean up
- Persistent projects matter on large scopes where a scan runs for hours or days and needs to survive a restart
- JAR launch method is useful when working from a stripped-down VM without package manager access
- Dark theme toggle is cosmetic but worth setting once (Burp: Settings > User interface > Display; ZAP: Tools > Options > Display) since you'll live in this tool

---

## Proxy Setup

To intercept traffic, the browser has to be routed through the proxy. Two ways to do this:

### 1. Pre-configured Browser (fastest)
- Burp: Proxy tab > Intercept > Open Browser
- ZAP: click the Firefox icon in the top toolbar

Both launch a browser with proxy settings and the CA cert already installed — zero manual config needed. Good enough for lab work.

### 2. Manual Firefox Setup
- Default proxy port for both tools: `8080` (configurable if in use)
- Configurable in Burp under Proxy > Proxy settings > Proxy listeners, or ZAP under Tools > Options > Network > Local Servers/Proxies
- FoxyProxy extension makes switching proxies fast instead of manually editing Firefox settings every time

**FoxyProxy config:**

| Field | Value |
|-------|-------|
| IP | `127.0.0.1` |
| Port | `8080` |
| Name | Burp / ZAP |

### CA Certificate Installation

Skipping this causes broken HTTPS traffic or constant "accept risk" prompts in Firefox.

| Tool | Get Cert |
|------|----------|
| Burp | Visit `http://burp` while proxy active → click "CA Certificate" |
| ZAP | Tools > Options > Network > Server Certificates > Save |

Import in Firefox: `about:preferences#privacy` → scroll to bottom → View Certificates → Authorities tab → Import → select the downloaded cert → check **both** trust boxes ("identify websites" / "identify email users") → OK.

### Real Example
Without importing the CA cert, HTTPS sites either failed to load through the proxy or threw a certificate warning on every single request. After importing and trusting the cert under the Authorities tab, HTTPS traffic flowed cleanly through Burp with no interruptions.

### Pentesting Relevance
- This is the actual precondition for every technique later in the module — no proxy setup, no interception, no manipulation
- Missing the CA cert install is the #1 cause of "HTTPS traffic isn't showing up" issues in labs
- Manual Firefox proxy config reflects the realistic client-engagement setup; the pre-configured browser is fine for HTB labs but won't match real assessment conditions
- FoxyProxy toggle matters when quickly switching between proxied/non-proxied browsing without re-editing settings each time

---

## Key Takeaways
- Web proxies are MITM tools focused on HTTP/HTTPS traffic — not full packet sniffers
- Burp is the industry standard; ZAP is the free, unthrottled alternative
- Temporary projects are fine for labs; persistent projects matter for long/large engagements
- Two ways to proxy traffic: pre-configured browser (fast, lab-only) or manual Firefox setup (realistic, client-facing)
- CA certificate installation is mandatory for clean HTTPS interception — skipping it is the most common setup mistake
- Everything else in this module depends on proxy setup working correctly first
