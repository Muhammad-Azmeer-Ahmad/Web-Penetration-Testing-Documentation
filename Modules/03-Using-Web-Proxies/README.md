# Module 03 — Using Web Proxies

![Status](https://img.shields.io/badge/Status-In%20Progress-yellow)
![Platform](https://img.shields.io/badge/Platform-HackTheBox-red)
![Category](https://img.shields.io/badge/Category-Offensive-orange)

## Sections Progress
- [x] Intro to Web Proxies
- [x] Setting Up
- [x] Proxy Setup
- [x] Intercepting Web Requests
- [x] Intercepting Responses
- [x] Automatic Modification
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

## Intercepting Web Requests

With the proxy set up, the next step is actually intercepting and manipulating requests before they reach the server.

### Turning Interception On

| Tool | Default State | Toggle |
|------|---------------|--------|
| Burp | On by default | Proxy > Intercept > "Intercept is on/off" button |
| ZAP | Off by default (green = passing) | Click the toggle button, or `Ctrl+B` |

ZAP also has a **Heads Up Display (HUD)** — controls most ZAP features directly inside the pre-configured browser, including a dedicated interception toggle. First launch prompts a HUD tutorial; worth doing once basics are covered.

Burp: `Forward` sends the request through. ZAP: `Step` sends and pauses on the next request, `Continue` forwards everything remaining uninterrupted — useful once you've isolated the one request you care about.

### Manipulating Intercepted Requests

Once intercepted, a request hangs until forwarded — giving a window to edit any part of it (headers, body, parameters) before it hits the server. This is the mechanism behind testing for:

| Vulnerability Class | What Interception Enables |
|---------------------|---------------------------|
| SQL injection | Inject payloads into params bypassing front-end validation |
| Command injection | Send OS metacharacters the UI blocks |
| Upload bypass | Alter Content-Type / filename mid-request |
| Auth bypass | Modify session tokens, roles, or auth headers |
| XSS / XXE | Inject raw markup/XML the client would normally escape |
| Error handling | Send malformed data to trigger verbose errors |
| Deserialization | Tamper with serialized objects in the request body |

### Real Example

Target had an IP field restricted client-side to numbers only via front-end JS — a classic case of relying on the browser to enforce validation. Intercepted the POST request:

```http
POST /ping HTTP/1.1
Host: 94.237.62.138:32306
Content-Type: application/x-www-form-urlencoded

ip=1
```

Changed the body from `ip=1` to `ip=;ls;` and forwarded it. Response returned a directory listing (`flag.txt`, `index.html`, `node_modules`, `package-lock.json`, `public`, `server.js`) instead of the expected ping output — confirming the back end passed the parameter straight into a shell command with zero server-side validation.

### Pentesting Relevance
- Front-end validation (JS restricting input in the browser) is not security — it's UX. Always test what the back end actually accepts, not what the form allows
- This ip=;ls; example is a textbook OS command injection — proxy interception is what makes it testable at all
- ZAP's HUD is worth learning early since later sections build on it for fuzzing and scanning workflows
- Step vs Continue in ZAP mirrors a real workflow: step through unfamiliar app flows, continue once you know exactly which request matters

---

## Intercepting Responses

Instead of editing the request, you can intercept the server's **response** before it reaches the browser — useful for enabling disabled fields, unhiding hidden fields, or changing how a page renders client-side restrictions.

### Enabling Response Interception

| Tool | How |
|------|-----|
| Burp | Proxy > Proxy settings > enable "Intercept Response" under Response interception rules |
| ZAP | Step on an intercepted request auto-intercepts the matching response |

### Example: Bypassing Client-Side Input Restrictions

The IP field from the previous section only accepted numeric input due to `type="number"` and `maxlength="3"` in the HTML. Intercepting the response and editing it directly changes what the browser renders:

```html
<input type="text" id="ip" name="ip" min="1" max="255" maxlength="100"
    oninput="javascript: if (this.value.length > this.maxLength) this.value = this.value.slice(0, this.maxLength);"
    required>
```

Forwarding this modified response makes the field accept any input — including the command injection payload from Section 4 — directly through the browser instead of intercepting every request.

### ZAP HUD Shortcuts

| Feature | What It Does |
|---------|---------------|
| Show/Enable (light bulb icon) | Instantly enables disabled fields / shows hidden fields without intercepting |
| Comments button | Flags positions in the page with HTML comments, shows content on hover |

Burp has an equivalent under Proxy > Proxy settings > Response modification rules (e.g. "Unhide hidden form fields").

### Pentesting Relevance
- Response interception reveals what the app *would* let you do if the front end weren't restricting it — critical for finding features gated only by client-side JS
- HTML comments are a common source of leftover debug info, credentials, or dead endpoints — the Comments HUD feature surfaces these fast without manually viewing source
- Editing responses is a manual, one-off fix — doesn't persist across page reloads, which is the motivation for automatic modification (next section)

---

## Automatic Modification

Manually intercepting every request/response gets tedious fast. Both tools support rule-based automatic modification so changes apply persistently without manual intervention each time.

### Automatic Request Modification

**Burp (Match and Replace)** — Proxy > Proxy settings > HTTP match and replace rules > Add:

| Field | Value | Why |
|-------|-------|-----|
| Type | Request header | Change targets the header, not the body |
| Match | `^User-Agent.*$` | Regex to catch the whole User-Agent line regardless of value |
| Replace | `User-Agent: HackTheBox Agent 1.0` | New header value |
| Regex match | True | Exact original value unknown, so pattern-match instead |

**ZAP (Replacer)** — `Ctrl+R` or Options > Replacer > Add:

| Field | Value |
|-------|-------|
| Description | HTB User-Agent |
| Match Type | Request Header (adds if not present) |
| Match String | User-Agent |
| Replacement String | HackTheBox Agent 1.0 |
| Initiators | Apply to all HTTP(S) messages (default) |

### Automatic Response Modification

Same concept applied to responses — makes the manual edit from the previous section persistent across refreshes instead of needing to be reapplied every time.

Burp: Proxy > Options > Match and Replace, type **Response body**:

| Match | Replace |
|-------|---------|
| `type="number"` | `type="text"` |
| `maxlength="3"` | `maxlength="100"` |

No regex needed here since the exact string is known. After adding both rules and refreshing, the field permanently accepts any input — command injection payloads work directly through the UI without touching the proxy again.

### Pentesting Relevance
- User-Agent spoofing via Match and Replace is a fast way around basic UA-based filtering/blocking
- Persistent response rules turn a one-off bypass into a standing condition for the rest of the engagement — no repeated manual interception needed
- This is the same principle behind bypassing client-side validation at scale: fix it once in a rule, then test freely
- Useful pattern for engagements involving repetitive testing across many pages sharing the same restricted field

---

## Repeating Requests

Manually intercepting, editing, and forwarding every single request to test different payloads doesn't scale — enumerating a system this way would take 5-6 steps per command. Request repeating fixes this by resending any previously captured request, editing it freely, and viewing the response without re-intercepting each time.

### Proxy History

| Tool | Location |
|------|----------|
| Burp | Proxy > HTTP History |
| ZAP | Bottom History pane (HUD) or main UI's History tab |

Both tools support filtering/sorting for large request volumes. Both also log **WebSocket history** separately — async updates and data fetching from the page after load — useful for advanced testing but out of scope here.

Burp shows both the *original* and *edited* request if one was modified (toggle in the pane header). ZAP only shows the final request sent.

### Repeating a Request

| Tool | Workflow |
|------|----------|
| Burp | `Ctrl+R` sends selected request to Repeater; `Ctrl+Shift+R` jumps to the Repeater tab; click Send |
| ZAP | Right-click request > "Open/Resend with Request Editor" > Send |
| ZAP HUD | Click request in History pane > Request Editor opens > "Replay in Console" (response in HUD) or "Replay in Browser" (rendered response) |

All three let you freely edit the request body/headers before resending, and quick-switch HTTP method (GET/POST/etc.) via a dropdown or right-click menu instead of rewriting the whole request.

### Real Example

Using Burp Repeater on the `/ping` endpoint from earlier sections, swapped the injected command in the body and hit Send — got the modified output back instantly, without re-intercepting. Same request reused repeatedly to iterate through different command injection payloads in seconds instead of minutes.

Lab flag recovered from this exercise: `HTB{qu1ckly_r3p3471n6_r3qu3575}`

### Pentesting Relevance
- Repeater/Request Editor is the actual daily-driver workflow for iterating on injection payloads — intercept once, then repeat endlessly
- Comparing original vs. edited requests in Burp is useful for sanity-checking exactly what was sent vs. what you intended
- Quick HTTP method switching (GET ↔ POST) surfaces endpoints that behave differently or unexpectedly accept methods they shouldn't
- WebSocket history matters on SPAs/modern apps where most real functionality happens over async connections, not classic page loads
- URL-encoded request bodies are the norm — encoding/decoding payloads correctly (next section) is essential for anything beyond plain text

---

## Key Takeaways
- Web proxies are MITM tools focused on HTTP/HTTPS traffic — not full packet sniffers
- Burp is the industry standard; ZAP is the free, unthrottled alternative
- Temporary projects are fine for labs; persistent projects matter for long/large engagements
- Two ways to proxy traffic: pre-configured browser (fast, lab-only) or manual Firefox setup (realistic, client-facing)
- CA certificate installation is mandatory for clean HTTPS interception — skipping it is the most common setup mistake
- Interception lets you bypass client-side-only validation entirely — never trust what the browser allows
- A single unsanitized parameter passed into a system call is enough for full command injection
- Response interception reveals hidden/disabled functionality gated only by the front end
- Match and Replace / Replacer rules turn manual bypasses into persistent, automatic ones
- Everything else in this module depends on proxy setup working correctly first