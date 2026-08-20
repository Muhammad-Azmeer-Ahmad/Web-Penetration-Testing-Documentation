# Module 05 — Web Fuzzing

![Status](https://img.shields.io/badge/Status-Complete-brightgreen)
![Platform](https://img.shields.io/badge/Platform-HackTheBox-red)
![Category](https://img.shields.io/badge/Category-Offensive-orange)

## Sections Progress
- [x] Introduction
- [ ] Section 2 (skipped)
- [x] Directory and File Fuzzing
- [x] Recursive Fuzzing
- [x] Parameter and Value Fuzzing
- [x] Virtual Host and Subdomain Fuzzing
- [x] Filtering Fuzzing Output
- [x] Validating Findings
- [x] Web APIs
- [x] Identifying Endpoints
- [x] API Fuzzing
- [~] Skills Assessment (Section 12 — practical exercise, not documented)

---

## What This Module Is About
Automated testing of web applications with unexpected/malformed input to uncover vulnerabilities traditional manual testing misses. Direct follow-on from Module 04's recon — fuzzing is how the endpoints and parameters found there actually get probed for weaknesses.

---

## Introduction

Web fuzzing = automated testing with unexpected or random data to detect flaws attackers could exploit.

### Fuzzing vs. Brute-forcing

Used interchangeably by beginners, but subtly different:

| | Fuzzing | Brute-forcing |
|-|---------|----------------|
| Approach | Wide net — malformed data, invalid chars, nonsensical combos | Targeted — systematically tries all possibilities for one specific value |
| Goal | See how the app reacts to strange/unexpected input | Guess the correct value (password, ID) through trial and error |
| Input source | Wordlists of patterns, mutations, random sequences | Predefined dictionaries (e.g. password lists) |

Analogy: fuzzing is throwing everything at a locked door — keys, screwdrivers, a rubber duck — to see what happens. Brute-forcing is trying every key on a specific ring until one opens it.

### Why Fuzz Web Applications

| Benefit | Why It Matters |
|---------|------------------|
| Uncovering Hidden Vulnerabilities | Unexpected/invalid input triggers behavior manual testing wouldn't think to try |
| Automating Security Testing | Generates and sends test inputs automatically — frees time for analyzing results |
| Simulating Real-World Attacks | Mimics actual attacker techniques proactively, before exploitation happens for real |
| Strengthening Input Validation | Directly surfaces weak validation — the root cause behind SQLi, XSS, etc. |
| Improving Code Quality | Bugs/errors found via fuzzing feed back into more robust development |
| Continuous Security | Fits into CI/CD pipelines for regular, ongoing testing rather than one-off checks |

### Essential Concepts

| Concept | Description | Example |
|---------|--------------|---------|
| Wordlist | Dictionary of words/phrases/filenames/params used as fuzzing input | `admin`, `login`, `backup`; app-specific: `productID`, `checkout` |
| Payload | Actual data sent during fuzzing | `' OR 1=1 --` (SQLi) |
| Response Analysis | Examining responses (codes, errors) for anomalies indicating vulnerabilities | `200 OK` normal vs `500` with a DB error message = potential SQLi |
| Fuzzer | Tool automating payload generation, sending, and response analysis | ffuf, wfuzz, Burp Suite Intruder |
| False Positive | Fuzzer flags something as a vuln that isn't one | `404` on a non-existent directory misread as significant |
| False Negative | Real vulnerability the fuzzer misses | A subtle logic flaw in payment processing |
| Fuzzing Scope | The specific part(s) of the app being targeted | Only the login page, or one API endpoint |

### Pentesting Relevance
- Fuzzing is the natural next step after Module 04's recon — discovered endpoints/parameters are exactly what gets fed into a fuzzer
- Understanding the fuzzing/brute-forcing distinction matters for tool selection — a directory fuzzer and a password brute-forcer solve different problems even though both "guess" input
- Response Analysis is the actual skill here — a fuzzer only generates noise; knowing what response pattern signals a real finding vs a false positive is where the value is
- False positives/negatives are inherent to fuzzing — results always need manual verification, never trust raw fuzzer output as confirmed findings
- Defining Fuzzing Scope tightly avoids wasting time/requests on irrelevant parts of the app and reduces detection risk from unnecessary noise

---

## Directory and File Fuzzing

Many web apps have directories/files not linked anywhere in the visible UI — backups, configs, old app versions, hidden endpoints. Directory and file fuzzing systematically probes for these by testing candidate names and analyzing server responses.

### What Hidden Assets Can Contain

| Category | Risk |
|----------|------|
| Sensitive data | Backup files, config settings, logs with credentials |
| Outdated content | Old file/script versions vulnerable to known exploits |
| Development resources | Test/staging environments, admin panels |
| Hidden functionality | Undocumented features/endpoints with unexpected vulns |

Hidden areas typically lack the security hardening of public-facing components, making them disproportionately valuable targets. Even a "boring" find (no immediate vuln) can still inform later stages — tech stack details, naming conventions, etc.

### Wordlists

The core input for this technique — quality directly determines yield. Compiled from sources like scraped common names, breach data analysis, and known-vulnerability directory patterns, then curated to cut duplicates/noise.

Fuzzers (ffuf, wfuzz) don't ship with built-in wordlists — they're designed to plug in external wordlist files, giving full flexibility to use existing lists or build custom ones.

**SecLists** (github.com/danielmiessler/SecLists) is the standard go-to collection — covers common names, backups, configs, vulnerable scripts, and more. On PwnBox: `/usr/share/seclists/` (lowercase); other distros may use `SecLists` — check the path if a command fails.

| Wordlist | Best For |
|----------|----------|
| `Discovery/Web-Content/common.txt` | General-purpose starting point, broad common names |
| `Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt` | Deeper directory-focused sweep |
| `Discovery/Web-Content/raft-large-directories.txt` | Large, multi-source directory list for thorough campaigns |
| `Discovery/Web-Content/big.txt` | Massive combined dir + file list, wide-net approach |

### How ffuf Works

1. Provide a **wordlist**
2. Build a URL with the **FUZZ** keyword as a placeholder
3. ffuf iterates the wordlist, substituting each entry into FUZZ and sending requests
4. **Response analysis** — filters results by status code, content length, etc.

Example target pattern: `http://localhost/FUZZ` → tries `/admin`, `/backup`, `/uploads`, etc.

### Directory Fuzzing

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -u http://IP:PORT/FUZZ
```

| Flag | Purpose |
|------|---------|
| `-w` | Wordlist path |
| `-u` | Target URL with `FUZZ` placeholder |

Result: discovered `w2ksvrus` — `Status: 301` (Moved Permanently), a strong signal of a real, protected directory worth further investigation.

### File Fuzzing

Once a directory is found, dig deeper for specific files inside it. Common extensions worth targeting:

| Extension | What It Usually Means |
|-----------|--------------------------|
| `.php` | Server-side PHP code |
| `.html` | Static page structure/content |
| `.txt` | Plain text — logs, notes |
| `.bak` | Backup files — often leftover, unprotected copies |
| `.js` | Client-side JavaScript |

A found `config.php.bak` could leak DB credentials/API keys; a forgotten `test.php` could expose a vulnerable script never meant to stay live.

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://IP:PORT/w2ksvrus/FUZZ -e .php,.html,.txt,.bak,.js -v
```

| Flag | Purpose |
|------|---------|
| `-e` | Extensions to append to each wordlist entry |
| `-v` | Verbose — shows full matched URLs |

Result: two files found inside `/w2ksvrus/`:
- `dblclk.html` (111 bytes, `Status: 200`) — purpose unclear, worth manual investigation
- `index.html` (112 bytes, `Status: 200`) — likely the directory's default page

### Pentesting Relevance
- Directory fuzzing before file fuzzing is the right order — find the container first, then dig inside it for specific files, rather than fuzzing files blindly across the whole app
- `.bak` and similar leftover-file extensions are consistently high-value — they bypass normal access controls since they're not "real" application files, just forgotten copies
- Wordlist choice trades speed for coverage — `common.txt` for a fast first pass, `raft-large-directories.txt`/`big.txt` when the target warrants a deeper sweep
- A 301 on a directory (like `w2ksvrus`) is worth flagging immediately — redirect behavior often reveals real, functioning paths versus dead ends
- Unclear/oddly-named files (like `dblclk.html`) shouldn't be dismissed just because their purpose isn't obvious — manual follow-up is often where fuzzing pays off

---

## Recursive Fuzzing

Manually re-running a fuzzer against every newly discovered directory doesn't scale on deeply nested apps. Recursive fuzzing automates that expansion.

### How It Works

1. **Initial Fuzzing** — starts at the web root, sends requests from the wordlist, looks for successful responses (e.g. `200`) indicating a real directory
2. **Directory Discovery and Expansion** — a found directory (e.g. `admin`) gets appended to the base URL, becoming a new fuzzing target (`http://localhost/admin/FUZZ`)
3. **Iterative Depth** — repeats for every newly discovered directory, branching deeper until a depth limit is hit or no more valid directories are found

Think of it as a tree — web root is the trunk, each discovered directory a branch, recursion walks every branch outward until it hits leaves (files) or a stopping point.

### Why Use It

| Benefit | Why |
|---------|-----|
| Efficiency | Automates nested discovery — no manual re-running per directory |
| Thoroughness | Systematically covers every branch, less risk of missing hidden assets |
| Reduced Manual Effort | Tool handles the whole expansion, not the operator |
| Scalability | Essential on large apps where manual exploration isn't practical |

### Recursive Fuzzing with ffuf

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -ic -v -u http://IP:PORT/FUZZ -e .html -recursion
```

| Flag | Purpose |
|------|---------|
| `-recursion` | Automatically re-fuzzes any newly discovered directory |
| `-ic` | Ignore comment lines (`#`) in the wordlist so they aren't treated as valid inputs |
| `-v` | Verbose output |
| `-e` | Extensions to test |

### Real Example

Starting at web root, ffuf found `level1` (`301`), auto-queued `http://IP:PORT/level1/FUZZ`, which turned up both `level2` and `level3` (also `301`s) plus an `index.html`. Both sub-branches got queued and fuzzed in turn, each yielding their own `index.html`. The `level3/index.html` stood out by file size — inspecting it revealed the flag `HTB{r3curs1v3_fuzz1ng_w1ns}`, confirming the nested structure was worth the full recursive walk.

### Being Responsible

Recursive fuzzing multiplies request volume fast — can overwhelm a target or trigger detection if left unbounded.

| Flag | Purpose |
|------|---------|
| `-recursion-depth` | Caps how many levels deep to go (e.g. `2` = starting dir + immediate subdirs only) |
| `-rate` | Limits requests per second |
| `-timeout` | Per-request timeout, avoids hanging on unresponsive targets |

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -ic -u http://IP:PORT/FUZZ -e .html -recursion -recursion-depth 2 -rate 500
```

### Pentesting Relevance
- Recursive fuzzing is the right call the moment a target shows nested structure — trying to manually chase each new directory wastes time recursion handles automatically
- File size as a signal (the oversized `level3/index.html` in the example) is worth watching even inside otherwise-identical filenames across directories — content, not just presence, matters
- Uncapped recursion on a large/complex app can spiral into an enormous request count fast — always set `-recursion-depth` deliberately rather than letting it run unbounded
- `-rate` and `-timeout` are the responsible-use levers — especially relevant on production-adjacent or client-owned infrastructure where server load matters
- `-ic` prevents wasted requests against wordlist comment lines — small detail, but avoids polluting results with junk fuzzing "hits"

---

## Parameter and Value Fuzzing

Beyond directories/files — manipulating request parameters and their values to see how the app processes input. Parameters are the variables carrying data between browser and server.

### GET Parameters

Visible directly in the URL after `?`, chained with `&`:

```
https://example.com/search?query=fuzzing&category=security
```

`query=fuzzing`, `category=security` — both openly visible, like a postcard. Typically used for actions that don't change server state (search, filter).

### POST Parameters

Carried in the request body, not the URL — used for sensitive data (credentials, personal/financial info).

**Encoding formats:**

| Format | Use Case |
|--------|----------|
| `application/x-www-form-urlencoded` | Key-value pairs joined by `&`, same shape as GET params but in the body |
| `multipart/form-data` | Used when submitting files alongside other data |

Example login POST request:

```http
POST /login HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded

username=your_username&password=your_password
```

### Why Parameters Matter for Fuzzing

| Attack | Example |
|--------|---------|
| Altered product ID | Pricing errors, unauthorized access to other users' orders |
| Modified hidden parameter | Unlocks hidden features/admin functions |
| Injected malicious code in a param | XSS, SQL injection |

### Fuzzing GET Parameters with wenum

Install:
```bash
pipx install git+https://github.com/WebFuzzForge/wenum
pipx runpip wenum install setuptools
```

**Manual probing first** — always worth doing before automating:

```bash
curl http://IP:PORT/get.php
# Invalid parameter value / x:

curl http://IP:PORT/get.php?x=1
# Invalid parameter value / x: 1
```

Confirms the app validates `x` and returns different behavior based on its value — a fuzzing target.

**Automated fuzzing:**

```bash
wenum -w /usr/share/seclists/Discovery/Web-Content/common.txt --hc 404 -u "http://IP:PORT/get.php?x=FUZZ"
```

| Flag | Purpose |
|------|---------|
| `-w` | Wordlist path |
| `--hc 404` | Hide 404 responses — wenum logs every request by default |
| `?x=FUZZ` | Insertion point for wordlist values |

Result: one line stood out with `200` instead of the usual invalid-value response — that value was the correct one, confirmed by requesting it directly and retrieving the flag.

### Fuzzing POST Parameters with ffuf

Different mechanism — payload goes in the request body, not the URL.

**Manual probe:**
```bash
curl -d "" http://IP:PORT/post.php
# Invalid parameter value / y:
```

**Automated fuzzing:**
```bash
ffuf -u http://IP:PORT/post.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "y=FUZZ" -w /usr/share/seclists/Discovery/Web-Content/common.txt -mc 200 -v
```

| Flag | Purpose |
|------|---------|
| `-X POST` | Set HTTP method |
| `-H` | Set request header (content type) |
| `-d "y=FUZZ"` | POST body with the fuzz insertion point |
| `-mc 200` | Only match responses with status 200 |

Result: one value returned `200` where all others returned the invalid-parameter message — confirmed with a direct `curl -d "y=<value>"` request, returning the flag.

### Pentesting Relevance
- Manual probing before fuzzing (checking how the app responds to missing/wrong values) confirms there's actually a validation mechanism worth automating against — don't fuzz blind
- GET vs POST fuzzing require different tooling mechanics (`-u` URL substitution vs `-d` body substitution) — know which applies before building the command
- `--hc`/`-mc` (hide/match status codes) is the essential filtering step — without it, results are buried in thousands of "invalid" responses
- Product IDs, hidden params, and search/query fields are the highest-value targets for this technique — direct paths to IDOR, auth bypass, and injection vulnerabilities
- In real engagements (unlike this flag-based exercise), correct values won't be obviously flagged — response size, timing, and subtle content differences become the actual signal to watch for

---

## Virtual Host and Subdomain Fuzzing

Both vhosts and subdomains organize web content, but work through completely different mechanisms — worth fuzzing both.

| | Virtual Hosts | Subdomains |
|--|----------------|------------|
| Identified by | `Host` header in HTTP requests | DNS records pointing to an IP |
| Purpose | Multiple sites on one server | Organizing sections/services of a site |
| Security Risk | Misconfigured vhosts can expose internal apps | Subdomain takeover if DNS records are mismanaged |

### gobuster

Multi-purpose brute-forcer: directories, files, subdomains, and vhosts all in one tool.

**VHost fuzzing:**

```bash
echo "IP inlanefreight.htb" | sudo tee -a /etc/hosts
gobuster vhost -u http://inlanefreight.htb:81 -w /usr/share/seclists/Discovery/Web-Content/common.txt --append-domain
```

`--append-domain` appends the base domain to each wordlist word, ensuring the `Host` header sent is a complete domain (e.g. `admin.inlanefreight.htb`) — essential for the fuzzing to actually work correctly.

Real example result: `Found: admin.inlanefreight.htb:81 Status: 200 [Size: 100]` — a live, previously undocumented vhost.

**Subdomain (DNS) fuzzing:**

```bash
gobuster dns -d inlanefreight.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

Generates candidate subdomains from the wordlist, appends to the target domain, and attempts DNS resolution — a resolved subdomain is valid and gets reported.

Note: in newer Gobuster releases, `-d` sets request delay, not domain — use `--do`/`--domain` instead for the target domain.

### Pentesting Relevance
- VHost fuzzing catches internal/undocumented sites sharing an IP that DNS enumeration alone would never surface — the two techniques cover different blind spots
- `--append-domain` is easy to forget and silently breaks results — always double-check the Host header being sent matches expectations
- Command syntax drift between Gobuster versions (`-d` meaning changes) is a real trap — verify flag meaning against the installed version before relying on old command references

---

## Filtering Fuzzing Output

Fuzzers generate huge volumes of output — filtering is what turns raw noise into actionable findings.

### Gobuster

| Flag | Purpose |
|------|---------|
| `-s` (include) | Only show specified status codes (dir mode only) |
| `-b` (exclude) | Hide specified status codes (dir mode only) |
| `--exclude-length` | Hide responses of specific content lengths |

```bash
gobuster dir -u http://example.com/ -w wordlist.txt -s 200,301 --exclude-length 0
```

### ffuf

| Flag | Purpose |
|------|---------|
| `-mc` / `-fc` | Match / filter status code(s) — default match: `200-299,301,302,307,401,403,405,500` |
| `-ms` / `-fs` | Match / filter response size |
| `-mw` / `-fw` | Match / filter word count |
| `-ml` / `-fl` | Match / filter line count |
| `-mt` | Match by time-to-first-byte (e.g. `>500` for slow responses) |

```bash
# Status 200, specific word count, size over 500 bytes
ffuf -u http://example.com/FUZZ -w wordlist.txt -mc 200 -fw 427 -ms >500

# Exclude common noise codes
ffuf -u http://example.com/FUZZ -w wordlist.txt -fc 404,401,302
```

### wenum

| Flag | Purpose |
|------|---------|
| `--hc` / `--sc` | Hide / show status code(s) |
| `--hl` / `--sl` | Hide / show by line count |
| `--hw` / `--sw` | Hide / show by word count |
| `--hs` / `--ss` | Hide / show by response size |
| `--hr` / `--sr` | Hide / show by regex match on response body |
| `--filter` / `--hard-filter` | General show/hide by regex; hard-filter also blocks post-processing |

```bash
wenum -w wordlist.txt --sr "admin\|password" -u https://example.com/FUZZ
```

### Feroxbuster

| Flag | Purpose |
|------|---------|
| `--dont-scan` | Exclude specific paths from scanning (even via recursion) |
| `-S` / `--filter-size` | Exclude by response size |
| `-X` / `--filter-regex` | Exclude by regex match |
| `-W` / `--filter-words` | Exclude by word count |
| `-N` / `--filter-lines` | Exclude by line count |
| `-C` / `--filter-status` | Denylist status codes |
| `-s` / `--status-codes` | Allowlist status codes |
| `--filter-similar-to` | Exclude pages similar to a reference page |

```bash
feroxbuster --url http://example.com -w wordlist.txt -s 200 -S 10240 -X "error"
```

### Real Example — Why Filtering Matters

Removing ffuf's default match filter (`-mc all` instead of the implicit default) on the same POST fuzzing command from earlier flooded the output with hundreds of `404` results, burying anything meaningful. The default matcher (`200-299,301,302,307,401,403,405,500`) exists specifically to suppress that noise automatically.

### Pentesting Relevance
- Every fuzzer defaults to *some* filtering already — know what's implicit before assuming raw output is unfiltered
- Combine filters (status + size + word count) rather than relying on just one — a single filter dimension is easy to accidentally match/exclude the wrong thing
- Regex-based filtering (`-sr`/`-hr`, `-X`) is the most powerful option when status codes alone don't distinguish real findings from noise (e.g. a `200` error page vs a `200` real page)
- Filtering choice should be revisited per-target — defaults tuned for one app's error page pattern won't necessarily fit another

---

## Validating Findings

Fuzzing generates leads, not confirmed vulnerabilities. False positives are expected — validation is a required step, not optional polish.

### Why Validate

| Purpose | Value |
|---------|-------|
| Confirming Vulnerabilities | Separates real issues from fuzzer noise |
| Understanding Impact | Establishes severity |
| Reproducing the Issue | Enables consistent replication for fix/mitigation work |
| Gathering Evidence | Builds proof to present to developers/stakeholders |

### Manual Verification Process

1. **Reproduce the request** — curl or browser, same request that triggered the anomaly
2. **Analyze the response** — look for errors, unexpected content, deviation from normal behavior
3. **Exploitation (cautious, authorized only)** — build a harmless proof-of-concept, not a full exploit; e.g. a SQLi PoC that returns the DB version string rather than exfiltrating data

Goal: enough evidence to convince stakeholders, without causing harm or exceeding authorization.

### Real Example — Validating a Backup Directory

Fuzzer found `/backup/` returning `200`. Rather than immediately browsing it (which could expose sensitive file contents unnecessarily), validate responsibly:

```bash
curl http://IP:PORT/backup/
```

Directory listing confirms the directory is browsable — response includes an actual file listing (`backup.sql`, etc.).

**Safer alternative — headers only:**

```bash
curl -I http://IP:PORT/backup/password.txt
```
```
HTTP/1.1 200 OK
Content-Type: text/plain;charset=utf-8
Content-Length: 171
```

`Content-Type` confirms file type without reading contents; `Content-Length: 171` confirms the file has real data (not empty) — enough evidence of a real exposure without touching the actual sensitive content.

### Pentesting Relevance
- Headers-only validation (`curl -I`) is the responsible default for anything that might contain sensitive data — confirms existence/non-empty status without exfiltrating content
- A non-zero `Content-Length` on a suspiciously-named file (`password.txt`, `dump.sql`) is strong evidence on its own — full content isn't always necessary to prove the finding
- Building a harmless PoC (DB version string instead of data dump) is the difference between responsible disclosure and actual data exposure — always minimize impact while proving impact
- Every fuzzer hit needs this validation step before being written up — an unvalidated finding in a report undermines credibility if it turns out to be a false positive

---

## Web APIs

APIs let different software communicate over the web — a bridge between server (data/functionality) and client (browser, app, another server).

### Three Common API Styles

| Style | Model | Data Format | Example |
|-------|-------|--------------|---------|
| REST | Stateless, resource-based URLs, standard HTTP methods (GET/POST/PUT/DELETE) | JSON/XML | `GET /users/123` |
| SOAP | Formal XML-based protocol, built-in security/reliability/transactions | XML (SOAP envelope) | `<soapenv:Envelope>...<GetStockPrice>...` |
| GraphQL | Single endpoint, client specifies exactly what data it wants | JSON | `query { user(id: 123) { name email } }` |

### Why APIs Matter

Enable code reuse, third-party integrations (payments, social logins, maps), and underpin microservices architecture — breaking monoliths into independently communicating services.

### Web Server vs API

| Feature | Web Server | API |
|---------|-----------|-----|
| Purpose | Serves static/dynamic pages | Enables app-to-app communication |
| Protocol | HTTP | HTTP, HTTPS, SOAP, others |
| Data Format | HTML/CSS/JS | JSON, XML, others |
| User Interaction | Direct (browser) | Indirect — apps consume it on the user's behalf |
| Access | Usually public | Public, private, or partner-restricted |

### Pentesting Relevance
- Knowing which API style is in play (REST/SOAP/GraphQL) determines fuzzing approach entirely — REST needs endpoint/param discovery, GraphQL needs schema introspection, SOAP needs WSDL analysis
- APIs are frequently less hardened than the public-facing web app since they're assumed to be "internal" or consumed only by trusted clients — a flawed but common assumption
- Data format matters for payload construction — JSON injection differs from XML injection differs from GraphQL query manipulation
- Microservices architecture means one API compromise can be a pivot point into other internal services communicating with it

---

## Identifying Endpoints

Before fuzzing an API, you need to know what to fuzz — endpoint discovery comes first.

### REST

Resources identified by URLs (endpoints), often hierarchical:

```
/users        → collection
/users/123    → specific resource
/products/456 → specific resource
```

| Parameter Type | Location | Example |
|-----------------|----------|---------|
| Query | After `?` in URL | `/users?limit=10&sort=name` |
| Path | Embedded in the URL | `/products/{id}` |
| Request Body | POST/PUT/PATCH body | `{"name": "New Product", "price": 99.99}` |

**Discovery methods:**
- **API documentation** — Swagger/OpenAPI, RAML — most reliable source when available
- **Network traffic analysis** — Burp/browser devtools intercepting real requests
- **Parameter name fuzzing** — same tools/technique as directory fuzzing, applied to param names instead

### SOAP

Single endpoint for all operations — the SOAP message body (XML) determines the specific action.

Structure defined by a **WSDL** file (operations, parameters, data types, response formats, endpoint location).

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:lib="http://example.com/library">
   <soapenv:Body>
      <lib:SearchBooks>
         <lib:keywords>cybersecurity</lib:keywords>
         <lib:author>Dan Kaminsky</lib:author>
      </lib:SearchBooks>
   </soapenv:Body>
</soapenv:Envelope>
```

**Discovery methods:** WSDL analysis (primary source), network traffic analysis (Wireshark/tcpdump), fuzzing for undocumented operations/parameters.

### GraphQL

Single endpoint (typically `/graphql`) for everything — queries (read) and mutations (write).

| Component | Purpose | Example |
|-----------|---------|---------|
| Field | Specific data point | `name`, `email` |
| Relationship | Connection to related data | `posts` |
| Nested Object | Traverse deeper into the graph | `posts { title, body }` |
| Argument | Refine query behavior | `posts(limit: 5)` |

```graphql
query {
  user(id: 123) {
    name
    posts(limit: 5) { title body }
  }
}
```

Mutations follow the same pattern for create/update/delete:

```graphql
mutation {
  createPost(title: "New Post", body: "content") {
    id
    title
  }
}
```

**Discovery methods:**
- **Introspection** — a special query retrieves the entire schema (types, fields, queries, mutations, arguments) directly from the API
- **Documentation/tooling** — GraphiQL, GraphQL Playground for interactive schema exploration
- **Network traffic analysis** — same principle as REST/SOAP

### Pentesting Relevance
- Introspection is the single most valuable GraphQL recon technique — a misconfigured API leaving introspection enabled in production hands over the entire schema for free
- WSDL files are effectively SOAP's documentation — always check for one before manually fuzzing blind
- Parameter name fuzzing matters most when documentation is absent/incomplete — the same directory-fuzzing mindset applies directly to undocumented API parameters
- Network traffic analysis is the universal fallback across all three API styles when docs don't exist — intercepting real traffic reveals real usage patterns

---

## API Fuzzing

Applying fuzzing principles specifically to API structure and protocols — altering param values, headers, param order, and data types to trigger errors or unexpected behavior.

### Why Fuzz APIs

| Reason | Value |
|--------|-------|
| Uncovering Hidden Vulnerabilities | Undocumented endpoints/params are common and under-tested |
| Testing Robustness | Confirms the API handles malformed input gracefully instead of crashing/leaking data |
| Automating Security Testing | Manual testing of every input combination isn't feasible |
| Simulating Real-World Attacks | Mimics actual attacker behavior proactively |

### Three Types of API Fuzzing

| Type | Target | Reveals |
|------|--------|---------|
| Parameter Fuzzing | Query params, headers, request bodies | SQLi, command injection, XSS, parameter tampering |
| Data Format Fuzzing | JSON/XML structure, encoding | Parsing errors, buffer overflows, special-character handling flaws |
| Sequence Fuzzing | Order/timing of multi-endpoint request chains | Race conditions, IDOR, authorization bypasses |

### Real Example — Discovering an Undocumented Endpoint

Target API documented 5 endpoints via Swagger (`/docs`): root, get/update/delete/create item. Ran a dedicated API fuzzer against it:

```bash
git clone https://github.com/PandaSt0rm/webfuzz_api.git
cd webfuzz_api
pip3 install -r requirements.txt
python3 api_fuzzer.py http://IP:PORT
```

Result: 4727 `404`s (expected noise), but 2 valid hits — the documented `/docs`, and an **undocumented** endpoint. A `405` on `/items` also flagged, indicating a request using the wrong HTTP method (worth retrying with the correct one).

```bash
curl http://localhost:8000/cz...
# {"flag":"<snip>"}
```

The undocumented endpoint returned the flag directly — confirming hidden endpoints found via fuzzing can carry real functionality/data that public documentation never revealed.

### Beyond Endpoint Discovery — Parameter-Level Fuzzing

Once endpoints are known, fuzzing their parameters can surface:

| Vulnerability | How Fuzzing Reveals It |
|----------------|---------------------------|
| Broken Object-Level Authorization (BOLA) | Manipulated param values grant access to objects that shouldn't be accessible |
| Broken Function-Level Authorization | Manipulated params trigger functions the caller shouldn't be authorized for |
| SSRF | Malicious param values trick the server into making unintended internal/external requests |

Full exploitation of these is covered in the API Attacks module.

### Pentesting Relevance
- Undocumented endpoints are one of the highest-value findings in API testing — they're untested by definition, often missing the validation/auth checks applied to documented ones
- A `405 Method Not Allowed` is a signal, not a dead end — it confirms the endpoint exists and expects a specific method worth identifying (try the other CRUD verbs)
- Sequence fuzzing matters especially for stateful APIs (checkout flows, multi-step auth) — single-request fuzzing alone won't catch race conditions or ordering-dependent bugs
- BOLA/SSRF risks mean parameter fuzzing on APIs isn't just about crashing the app — it's a direct path to unauthorized data access or internal network pivoting

---

## Closing Thoughts

Fuzzing is where Module 04's recon becomes actionable — every subdomain, endpoint, and parameter discovered there is exactly what gets fed into ffuf, gobuster, or an API fuzzer here. The technique scales from simple directory guessing to full API schema exploitation, but the discipline stays constant throughout: filter aggressively to cut noise, validate every hit before trusting it, and match the fuzzing approach to the target's actual structure (web app vs REST vs SOAP vs GraphQL) rather than treating everything as a generic directory brute-force. Response Analysis — not the fuzzer itself — is the real skill this module builds.

---

## Key Takeaways
- Web fuzzing automates testing with unexpected/malformed input to surface vulnerabilities manual testing misses
- Fuzzing (wide net, unexpected input) and brute-forcing (targeted guessing) are related but distinct techniques
- Response Analysis — reading status codes, errors, and anomalies — is what turns fuzzer noise into actual findings
- Every fuzzing result needs manual verification — false positives and false negatives are both expected, not exceptions
- Defining a tight Fuzzing Scope keeps testing efficient and reduces unnecessary detection risk
- Directory fuzzing before file fuzzing is the right sequence — find the folder, then dig inside it; `.bak`/leftover-extension files are consistently high-value finds since they bypass normal app access controls
- Recursive fuzzing automates nested directory discovery — always cap it with `-recursion-depth` and throttle with `-rate` to avoid overwhelming the target
- Parameter fuzzing (GET via URL, POST via request body) targets the app's actual input validation logic — product IDs, hidden params, and search fields are the highest-value spots to test
- VHost fuzzing and subdomain/DNS fuzzing cover different blind spots — always run both, since VHosts can exist with zero DNS footprint
- Every fuzzer applies some default filtering already — know the implicit matcher before assuming raw output is unfiltered, and combine filter dimensions (status + size + word count) for precision
- Fuzzer hits are leads, not confirmed vulnerabilities — always validate manually (headers-first, minimal-impact PoC) before reporting a finding
- API style (REST/SOAP/GraphQL) determines the fuzzing approach entirely — endpoint discovery must come before parameter fuzzing, and GraphQL introspection left enabled is a direct schema leak
- Undocumented API endpoints are consistently high-value — untested by definition, often missing the auth/validation checks applied to documented ones
- API-specific fuzzing (parameter, data format, sequence) targets vulnerability classes standard web fuzzing misses entirely — BOLA, SSRF, and race conditions require this specialized approach