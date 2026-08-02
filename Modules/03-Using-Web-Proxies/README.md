# Module 03 — Using Web Proxies

![Status](https://img.shields.io/badge/Status-Complete-brightgreen)
![Platform](https://img.shields.io/badge/Platform-HackTheBox-red)
![Category](https://img.shields.io/badge/Category-Offensive-orange)

## Sections Progress
- [x] Intro to Web Proxies
- [x] Setting Up
- [x] Proxy Setup
- [x] Intercepting Web Requests
- [x] Intercepting Responses
- [x] Automatic Modification
- [x] Repeating Requests
- [x] Encoding/Decoding
- [x] Proxying Tools
- [x] Burp Intruder
- [x] ZAP Fuzzer
- [x] Burp Scanner
- [x] ZAP Scanner
- [x] Extensions

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

## Encoding/Decoding

Custom requests need correct encoding or the server throws errors — this section covers the built-in encoders both tools provide.

### URL Encoding

Key characters that must be encoded in request data:

| Character | Why It Must Be Encoded |
|-----------|--------------------------|
| Space | May be read as end of request data |
| `&` | Interpreted as a parameter delimiter |
| `#` | Interpreted as a fragment identifier |

| Tool | How |
|------|-----|
| Burp | Select text > right-click > Convert Selection > URL > URL-encode key characters, or `Ctrl+U`. Can also auto-encode as you type (right-click toggle) |
| ZAP | Auto URL-encodes request data in the background before sending — no manual step needed |

Other variants exist too — Full URL-encoding, Unicode URL-encoding — useful when a request has heavy special-character content.

### Decoding

Beyond URL encoding, both tools support quick conversion between common formats:

| Encoding Type | Common Use |
|----------------|-------------|
| HTML | Escaped markup in responses |
| Unicode | Non-ASCII characters |
| Base64 | Tokens, cookies, serialized data |
| ASCII hex | Raw byte-level data |

| Tool | Access |
|------|--------|
| Burp | Decoder tab, or Burp Inspector (available inline in Proxy/Repeater) |
| ZAP | Encoder/Decoder/Hash tool, `Ctrl+E` |

### Real Example

Found a cookie value: `eyJ1c2VybmFtZSI6Imd1ZXN0IiwgImlzX2FkbWluIjpmYWxzZX0=`

Decoded via Burp Decoder (Decode as > Base64) revealed:
```json
{"username":"guest", "is_admin":false}
```

Changed `guest` → `admin` and `false` → `true`, re-encoded the modified JSON back to Base64, and swapped it into the request cookie via Repeater — a direct test for privilege escalation through client-controlled, base64-encoded state.

Tip: Burp Decoder output can be chained — reapply a different encoder/decoder directly to the output pane without re-copying the text. ZAP requires copy-pasting the output back into the input field for chaining.

### Pentesting Relevance
- Any base64/hex-looking cookie or token is worth decoding immediately — client-controlled state like `is_admin` embedded in a cookie is a common and severe privilege escalation vector
- Auto URL-encoding in ZAP vs manual in Burp matters when crafting exact payloads — know which tool is silently transforming your input
- Chained encoding/decoding (Burp Decoder output pane) speeds up working with double-encoded or nested-format data (e.g. base64-encoded JSON, URL-encoded base64)
- Being able to encode/decode inline in the proxy tool removes dependency on external tools (CyberChef, python one-liners) mid-engagement

---

## Proxying Tools

Interception isn't limited to browsers — CLI tools and thick clients can be routed through the same proxy, giving full visibility into their requests too. Setup is the same as browser proxying: point the tool at `http://127.0.0.1:8080`. Method varies per tool.

Note: proxying slows tools down noticeably — only enable it when actively investigating requests, not for routine use.

### Proxychains

Routes traffic from any CLI tool through a specified proxy. Simplest way to proxy command-line tools without per-tool configuration.

**Config** — edit `/etc/proxychains.conf`, comment the last line, add:
```
#socks4         127.0.0.1 9050
http 127.0.0.1 8080
```

**Usage** — the `-q` flag suppresses connection noise in the terminal:
```bash
proxychains -q curl http://SERVER_IP:PORT
```

The request shows up in the proxy's history exactly like browser traffic.

### Metasploit

Set the `PROXIES` flag before running any module:

```
msfconsole

use auxiliary/scanner/http/robots_txt
set PROXIES HTTP:127.0.0.1:8080
set RHOST SERVER_IP
set RPORT PORT
run
```

Request shows up in proxy history same as any other traffic — in this case a GET to `/robots.txt` returning 404. Same method applies to any Metasploit module: scanners, exploits, everything.

### Real Example

Ran `auxiliary/scanner/http/http_put` through Burp with `PROXIES` set — proxy history showed the full PUT request, including its body ending in `msf test file`, confirming exactly what the module sends without needing to read Metasploit's source to find out.

### Pentesting Relevance
- Proxying CLI tools/scripts/thick clients turns a black-box tool into a fully inspectable one — essential when debugging why an exploit or scanner isn't behaving as expected
- Proxychains is the fastest way to get arbitrary CLI tools (curl, custom scripts, etc.) through Burp/ZAP without per-tool proxy config
- Setting `PROXIES` in Metasploit is a direct way to verify exactly what a module sends before trusting its output — useful for confirming false positives/negatives
- Slows tools down, so this is a diagnostic step, not a permanent workflow — turn it off once the investigation is done

---

## Burp Intruder

Burp and ZAP both include built-in web fuzzers/scanners as alternatives to CLI tools like ffuf, dirbuster, gobuster, wfuzz. Burp's is called **Intruder** — fuzzes pages, directories, sub-domains, parameters, and parameter values.

Community edition throttles Intruder to **1 request/second** (CLI fuzzers can do 10k+/sec), so it's only practical for short queries. Pro removes the throttle entirely, making it competitive with dedicated fuzzing tools while keeping Intruder's extra features.

### Sending a Request to Intruder

From Proxy History: right-click the target request > "Send to Intruder", or `Ctrl+I`. Jump to the Intruder tab via `Ctrl+Shift+I`. The **Target** box auto-populates from the request sent over.

### Positions

Marks where wordlist entries get inserted — wrap the target token in `§` markers, or select it and click "Add §". For directory fuzzing (`GET /DIRECTORY/`), existing paths return `200 OK`, missing ones `404`.

Note: keep the two trailing blank lines at the end of the request or the server may return an error.

### Payloads — Four Configuration Areas

**1. Payload Position & Payload Type**

| Payload Type | Behavior |
|---------------|----------|
| Simple List | Basic — iterates line by line over a provided wordlist |
| Runtime File | Same as Simple List, but streams the file instead of loading it all into memory — better for huge wordlists |
| Character Substitution | Tests all permutations from a defined character replacement list |

Attack type (Sniper, Cluster Bomb, etc.) determines how many payload positions/sets are available.

**2. Payload Configuration**

Build the wordlist by adding entries manually, or Load a file — e.g. `/opt/useful/seclists/Discovery/Web-Content/common.txt`. Multiple wordlists/manual entries can be combined into one list. Burp Pro also offers built-in wordlists via "Add from list".

**3. Payload Processing**

Applies rules to the loaded wordlist before the attack runs — e.g. skip lines matching a regex. Example: skip all lines starting with `.`:
```
Rule: Skip if matches regex
Pattern: ^\..*$
```

**4. Payload Encoding**

Toggle URL-encoding for special characters (`./^=<>&+?*:;'{}|^`) — left enabled by default.

### Settings

| Option | Purpose |
|--------|---------|
| Retries on network failure / Pause before retry | Set to 0 to skip unnecessary delays |
| Grep - Match | Flags responses matching a string/regex (e.g. `200 OK`) — enable, clear defaults, add own pattern |
| Exclude HTTP Headers | Disable if the match string appears in headers (e.g. status line) |
| Grep - Extract | Pulls out just a relevant part of long responses — skip if only status matters |
| Resource Pool | Controls network resource usage for large attacks — default is fine for most cases |

### Real Example

Fuzzed `GET /§DIRECTORY§/` against a target using `common.txt`, with a Grep - Match rule for `200 OK` and a Payload Processing rule skipping lines starting with `.`. Results table sorted by the 200 OK column showed nearly everything as `404` (length 458) except one hit: `/admin/` returning `200` (length 244). Manually confirmed the path existed by visiting it directly.

### Pentesting Relevance
- Directory fuzzing via Intruder mirrors exactly what ffuf/gobuster do — useful when a CLI tool isn't available or when staying inside the same tool as the rest of the engagement is preferred
- Grep - Match is the fast way to filter noise out of large wordlist runs — sort by match instead of manually scanning hundreds of results
- Payload Processing rules (skip-if-regex) keep irrelevant wordlist entries (dotfiles, comments, headers in seclists files) from wasting requests
- Intruder also handles password spraying against AD-backed auth (OWA, SSL VPN, RDS, Citrix, custom AD-integrated apps) — same payload-position mechanism applied to a login form instead of a URL path
- Community's 1 req/sec throttle makes it impractical for large wordlists — know when to switch to ZAP's fuzzer or a CLI tool instead

---

## ZAP Fuzzer

ZAP's built-in fuzzer, unlike Community Intruder, has **no throttling** — a real advantage even though it has fewer advanced features than Intruder.

### Starting a Fuzz

Locate the target request in History, right-click > Attack > Fuzz, which opens the Fuzzer window. Four things to configure: Fuzz Location, Payloads, Processors, Options.

### Locations

Same concept as Intruder's Payload Position — select the target word/token in the request and click Add. A green marker appears on the selected location.

### Payloads

| Payload Type | Purpose |
|----------------|---------|
| File | Provide a custom wordlist file |
| File Fuzzers | Use ZAP's built-in wordlist databases (e.g. dirbuster lists) — no need to supply your own |
| Numberzz | Generates numeric sequences with custom increments |

Advantage over Intruder: built-in wordlists out of the box, expandable via the ZAP Marketplace.

### Processors

Applied to each payload before sending — options include Base64 Encode/Decode, MD5/SHA-1/256/512 hashing, Prefix/Postfix String, URL Encode/Decode, or a custom Script. URL Encode is the standard choice to avoid server errors from special characters — use "Generate Preview" to check the final payload before running.

### Options

| Setting | Purpose |
|---------|---------|
| Concurrent Scanning Threads | Set higher (e.g. 20) for faster scans, limited by CPU/server connection limits |
| Depth First | Exhausts all payloads on one position before moving to the next (e.g. all passwords for one user) |
| Breadth First | Runs one payload across all positions before moving to the next payload (e.g. one password across all users) |

### Real Example

Fuzzed `/test/` for directories using a dirbuster wordlist (`directory-list-1.0.txt`) with URL Encode processing applied. Sorted results by response code — got a single `200 OK` hit on the `skills` payload, confirming `/skills/` existed and was accessible. Verified by viewing the request/response details directly in the results pane.

### Pentesting Relevance
- No throttling makes ZAP Fuzzer the better default for large wordlists when Burp Pro isn't available
- Built-in wordlists (dirbuster, FuzzDB via Marketplace) save setup time versus manually sourcing lists
- Depth First vs Breadth First matters directly for password spraying — Breadth First avoids triggering account lockouts by not hammering one account with all passwords in a row
- Response size (Size Resp. Body) and RTT are useful secondary signals beyond status code — e.g. spotting time-based SQLi via response delay

---

## Burp Scanner

Burp's built-in vulnerability scanner — **Pro-only**, not available in Community. Combines a Crawler (site mapping) with Passive and Active scanning.

### Target Scope

Scans can start from: a specific request in Proxy History, a custom target set, or items already in scope. Scope is defined via Target > Site map (right-click > "Add to scope") or Target > Scope for advanced regex-based include/exclude rules — useful for excluding dangerous endpoints like logout functions from active scanning.

### Crawler

Two scan modes: **Crawl** (maps the site by following links/forms) or **Crawl and Audit** (crawl, then scan). Crawl only follows referenced links — it doesn't fuzz for unlinked pages (that's Intruder/Content Discovery's job).

Scan configs can be built from scratch or picked from presets (e.g. "Crawl strategy - fastest"). Login credentials or a recorded login flow can be added so the crawler covers authenticated areas.

### Passive Scanner

Analyzes already-captured traffic without sending new requests — flags things like missing security headers or potential DOM-based XSS. Fast, but can only suggest — not confirm — vulnerabilities. Each finding includes a Confidence rating (Certain/Firm/Tentative) alongside severity.

### Active Scanner

The most thorough option — runs a crawl + fuzzer, passive scan, then actively verifies findings by sending test payloads (XSS, SQLi, command injection, etc.) and performing JS analysis. Configurable audit presets exist (e.g. "Audit checks - critical issues only" for high-value findings only).

### Real Example

Ran a Crawl and Audit scan with the "critical issues only" audit preset. Filtered Issue Activity results to High severity / Certain confidence and found an **OS command injection** finding on the `ip` parameter, rated High severity / Firm confidence — matching the same class of vulnerability manually found earlier in the module via interception, but discovered automatically here.

### Reporting

Target > Site map > right-click target > Issue > Report issues for this host. Exportable in multiple formats, includes PoC details and remediation guidance. Scanner reports are supplementary appendix material for client deliverables — never the final report on their own.

### Pentesting Relevance
- Passive scanning is a free way to catch low-hanging issues (headers, cookie flags) just from traffic already generated during normal testing
- Active scanning automates what was done manually earlier in this module (command injection via the `ip` param) — useful for coverage, but manual testing remains necessary for anything scanner logic doesn't anticipate
- Scope control (Add to scope / Remove from scope) is essential before running an active scan — prevents accidentally scanning out-of-scope hosts or hitting destructive endpoints like logout
- Pro-only status makes this a budget/tooling decision point on client engagements

---

## ZAP Scanner

ZAP's equivalent scanning suite — free, using **Spider** for site mapping and both passive and active scanning.

### Spider

Right-click a request in History > Attack > Spider, or use the HUD's Spider Start button. Follows and validates links similarly to Burp's Crawler. Results appear in the Sites Tree (tree view of discovered URLs/files).

**Ajax Spider** — a separate spider that also identifies links loaded via JavaScript/AJAX after page load. Slower, but catches things the standard Spider misses; worth running after the normal Spider finishes.

### Passive Scanner

Runs automatically as the Spider crawls — flags issues like missing security headers or DOM-based XSS directly from response source, no extra requests needed. Alerts populate incrementally as pages are visited; visible per-page (left pane) or app-wide (right pane / Alerts tab).

### Active Scanner

Click Active Scan to run comprehensive attacks against all discovered pages/params. Auto-triggers a Spider run first if one hasn't been done. Takes longer than passive scanning since it's actively sending test payloads.

### Real Example

Active scan on a target surfaced a **High** alert: Remote OS Command Injection, with an example attack string of `127.0.0.1&cat /etc/passwd&` and evidence showing actual `/etc/passwd` contents in the response — Medium confidence but High risk, confirming exploitability directly in the alert details. Request could be replayed via ZAP HUD or Request Editor straight from the alert.

### Reporting

Report > Generate HTML Report (also supports XML, Markdown). Summarizes all findings by severity for later reference or client appendix material.

### Pentesting Relevance
- Ajax Spider matters increasingly given how many modern apps are SPA/JS-heavy — the standard Spider alone will miss AJAX-loaded routes
- Passive alerts building up during normal Spider/browsing means useful findings show up "for free" before any active scanning starts
- Being able to replay a scanner-found vulnerability directly (HUD/Request Editor) turns an automated finding into a manually-verified one in seconds
- Fully free with no Pro tier — makes ZAP Scanner the default choice for engagements without a Burp Pro license

---

## Extensions

Both tools support community-built extensions/add-ons — Burp via the **BApp Store**, ZAP via the **ZAP Marketplace**.

### BApp Store (Burp)

Extensions Tab > BApp Store. Sortable by popularity. Some extensions are Pro-only; most are free. Some require external dependencies (e.g. Jython) not installed by default.

Notable extensions: Active Scan++, Decoder Improved, Autorize, Retire.JS, CSRF Scanner, JS Link Finder, Backslash Powered Scanner, PHP Object Injection Check, Java Deserialization Scanner, Wsdler, AWS Security Checks, CMS Scanner, and others covering specific tech stacks or vuln classes.

### ZAP Marketplace

Manage Add-ons > Marketplace tab. Add-ons are marked Release (stable) or Beta/Alpha (less stable). Example: installing **FuzzDB Files** and **FuzzDB Offensive** adds new wordlists to ZAP's fuzzer, including an OS Command Injection wordlist (`fuzzdb > attack > os-cmd-execution`) — directly useful for command injection fuzzing against WAF-protected targets.

### Real Example

Installed FuzzDB add-ons, then ran the ZAP Fuzzer against the `/ping` command injection target from earlier sections using the `command_execution-unix.txt` wordlist. Multiple payloads (`;id`, `` `id` ``, etc.) returned `200 OK`, confirming several viable injection syntaxes worked simultaneously — useful for finding a bypass when one payload syntax is filtered by a WAF.

### Pentesting Relevance
- Extensions turn Burp/ZAP from proxies into broader vulnerability-testing platforms without leaving the tool
- FuzzDB-style wordlists targeting specific vuln classes (command injection, path traversal, etc.) are more effective than generic content-discovery wordlists for exploitation-focused fuzzing
- Worth periodically reviewing the BApp Store/Marketplace for new releases — extension ecosystems update independently of the core tool

---

## Closing Thoughts

Burp Suite and ZAP are core tools for web application penetration testing — essential not just for dedicated web testers but for offensive security practitioners generally, and useful for blue team/defensive work too. They belong in the same toolbox tier as Nmap, Hashcat, Wireshark, tcpdump, sqlmap, ffuf, and Gobuster. Practical fluency with both comes from working through real exercises and boxes, not just reading documentation.

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
- Request repeating (Burp Repeater / ZAP Request Editor) is the daily workflow for fast payload iteration — intercept once, repeat freely
- Client-controlled state (e.g. `is_admin` in a base64 cookie) is a direct privilege escalation vector — always decode and inspect tokens/cookies
- CLI tools and thick clients can be proxied too (proxychains, Metasploit's PROXIES flag) — not just browsers
- Burp Intruder and ZAP Fuzzer both replicate CLI fuzzing tools inside the proxy — Community Intruder is throttled, ZAP Fuzzer isn't
- Burp Scanner (Pro-only) and ZAP Scanner (free) both combine crawling with passive/active vulnerability detection — automating what was done manually earlier in the module
- Extensions (BApp Store / ZAP Marketplace) extend both tools well beyond core proxy functionality — wordlists, scanners, decoders, and vuln-specific checks
- Everything else in this module depends on proxy setup working correctly first