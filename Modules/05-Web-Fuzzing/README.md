# Module 05 — Web Fuzzing

![Status](https://img.shields.io/badge/Status-In%20Progress-yellow)
![Platform](https://img.shields.io/badge/Platform-HackTheBox-red)
![Category](https://img.shields.io/badge/Category-Offensive-orange)

## Sections Progress
- [x] Introduction
- [ ] Section 2 (skipped)
- [x] Directory and File Fuzzing
- [x] Recursive Fuzzing
- [x] Parameter and Value Fuzzing
- [ ] (sections 6-12 to be added as covered)

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

## Key Takeaways
- Web fuzzing automates testing with unexpected/malformed input to surface vulnerabilities manual testing misses
- Fuzzing (wide net, unexpected input) and brute-forcing (targeted guessing) are related but distinct techniques
- Response Analysis — reading status codes, errors, and anomalies — is what turns fuzzer noise into actual findings
- Every fuzzing result needs manual verification — false positives and false negatives are both expected, not exceptions
- Defining a tight Fuzzing Scope keeps testing efficient and reduces unnecessary detection risk
- Directory fuzzing before file fuzzing is the right sequence — find the folder, then dig inside it; `.bak`/leftover-extension files are consistently high-value finds since they bypass normal app access controls
- Recursive fuzzing automates nested directory discovery — always cap it with `-recursion-depth` and throttle with `-rate` to avoid overwhelming the target
- Parameter fuzzing (GET via URL, POST via request body) targets the app's actual input validation logic — product IDs, hidden params, and search fields are the highest-value spots to test