# Module 04 — Information Gathering - Web Edition

![Status](https://img.shields.io/badge/Status-In%20Progress-yellow)
![Platform](https://img.shields.io/badge/Platform-HackTheBox-red)
![Category](https://img.shields.io/badge/Category-Offensive-orange)

## Sections Progress
- [x] Introduction
- [x] WHOIS
- [x] Utilising WHOIS
- [x] DNS
- [x] Digging DNS
- [x] Subdomains
- [x] Subdomain Bruteforcing
- [x] DNS Zone Transfers
- [x] Virtual Hosts
- [x] Certificate Transparency Logs
- [ ] (sections 11–19 to be added as covered)

---

## What This Module Is About
Web reconnaissance — systematically collecting information about a target website/application before deeper analysis or exploitation. Sits inside the Information Gathering phase of the penetration testing process (Pre-Engagement → Information Gathering → Vulnerability Assessment → Exploitation → Post-Exploitation → Lateral Movement → Proof-of-Concept → Post-Engagement).

---

## Introduction

### Goals of Web Reconnaissance

| Goal | What It Covers |
|------|-----------------|
| Identifying Assets | Web pages, subdomains, IPs, technologies — full picture of the online footprint |
| Discovering Hidden Information | Backup files, configs, internal docs left exposed |
| Analysing the Attack Surface | Technologies, configurations, possible entry points |
| Gathering Intelligence | Key personnel, emails, behavior patterns for social engineering |

Same information cuts both ways — attackers use it to target weaknesses, defenders use it to find and patch them first.

### Active vs Passive Reconnaissance

**Active** — directly interacts with the target. More comprehensive, higher detection risk.

| Technique | What It Does | Tools | Detection Risk |
|-----------|---------------|-------|-----------------|
| Port Scanning | Finds open ports/services | Nmap, Masscan, Unicornscan | High |
| Vulnerability Scanning | Probes for known vulns (outdated software, misconfig) | Nessus, OpenVAS, Nikto | High |
| Network Mapping | Maps topology/connected devices | Traceroute, Nmap | Medium–High |
| Banner Grabbing | Reads service banners (e.g. web server + version) | Netcat, curl | Low |
| OS Fingerprinting | Identifies target OS | Nmap (`-O`), Xprobe2 | Low |
| Service Enumeration | Identifies specific service versions | Nmap (`-sV`) | Low |
| Web Spidering | Crawls site for pages/directories/files | Burp Suite Spider, ZAP Spider, Scrapy | Low–Medium |

**Passive** — gathers info without touching the target directly. Stealthier, less comprehensive.

| Technique | What It Does | Tools | Detection Risk |
|-----------|---------------|-------|-----------------|
| Search Engine Queries | Finds public info via search engines | Google, DuckDuckGo, Bing, Shodan | Very Low |
| WHOIS Lookups | Domain registration details | `whois`, online WHOIS services | Very Low |
| DNS Analysis | Subdomains, mail servers, infra via DNS records | dig, nslookup, host, dnsenum, fierce, dnsrecon | Very Low |
| Web Archive Analysis | Historical snapshots of the target site | Wayback Machine | Very Low |
| Social Media Analysis | Employee/org info from public profiles | LinkedIn, Twitter, Facebook, OSINT tools | Very Low |
| Code Repositories | Exposed credentials/code vulns in public repos | GitHub, GitLab | Very Low |

### Pentesting Relevance
- Passive recon should always come first — zero detection risk, and often surfaces enough to plan active recon precisely instead of scanning blindly
- Detection risk table matters for engagement rules of scope — some clients explicitly restrict active scanning until a defined phase
- Web spidering straddles the line — configure crawler behavior carefully or it can look like abnormal traffic
- Code repository exposure (GitHub/GitLab) is one of the highest-value passive techniques — leaked credentials or internal endpoints show up here constantly
- This module starts with WHOIS as the entry point into passive recon before moving into more advanced techniques

---

## WHOIS

Query/response protocol for looking up registered internet resources — domains primarily, but also IP blocks and autonomous systems. Effectively a public phonebook for internet infrastructure.

```bash
whois inlanefreight.com
```

```
Domain Name: inlanefreight.com
Registry Domain ID: 2420436757_DOMAIN_COM-VRSN
Registrar WHOIS Server: whois.registrar.amazon
Registrar URL: https://registrar.amazon.com
Updated Date: 2023-07-03T01:11:15Z
Creation Date: 2019-08-05T22:43:09Z
```

### Typical Record Contents

| Field | What It Shows |
|-------|-----------------|
| Domain Name | The domain itself |
| Registrar | Where it was registered (GoDaddy, Namecheap, etc.) |
| Registrant Contact | Who registered it |
| Administrative Contact | Who manages it |
| Technical Contact | Who handles technical issues |
| Creation/Expiration Dates | Registration and expiry timeline |
| Name Servers | Servers resolving the domain to an IP |

### Brief History

Originated in the 1970s at Stanford Research Institute's NIC under Elizabeth Feinler, built to track ARPANET resources. Formalized in RFC 812 (1982). Moved to a distributed model with Regional Internet Registries (RIRs) in the 1990s as centralized WHOIS couldn't scale. ICANN (1998) took over global DNS/WHOIS policy, standardizing formats and handling domain disputes via the UDRP. GDPR (2018) forced privacy-conscious changes — much registrant data is now masked by default, with RDAP emerging as a more granular, privacy-aware successor protocol.

### Pentesting Relevance
- Registrant/Admin/Technical contacts are direct leads for social engineering or phishing targeting — names, emails, phone numbers often exposed
- Name servers and associated IPs are a starting point for mapping network infrastructure and identifying hosting providers
- Historical WHOIS data (via services like WhoisFreaks) can reveal past ownership changes, old contact info, or infrastructure shifts — useful for building a timeline of the target's digital footprint
- GDPR-era privacy masking means modern WHOIS records are often less revealing than pre-2018 — don't expect the same yield as older engagements or training material assumes
- WHOIS is entirely passive — zero detection risk, always safe to run early regardless of engagement rules of engagement

---

## Utilising WHOIS

Three scenarios showing how WHOIS data gets applied in practice.

### Scenario 1: Phishing Investigation

Suspicious bank-impersonation email flagged. WHOIS on the linked domain showed:
- Registered days ago
- Registrant hidden behind a privacy service
- Name servers tied to a known bulletproof hosting provider

Combination of recency + privacy masking + suspicious hosting = strong phishing indicator. Domain blocked, employees warned. Investigating the hosting provider/IPs further could surface related phishing infrastructure.

### Scenario 2: Malware Analysis (C2 Infrastructure)

Malware C2 domain investigated via WHOIS:
- Registrant using an anonymous free email service
- Registrant address in a high-cybercrime-prevalence country
- Registered through a registrar known for lax abuse policies

Pointed to likely bulletproof/compromised hosting. Hosting provider identified and notified.

### Scenario 3: Threat Intelligence Report

Multiple domains tied to a known threat actor group analyzed in aggregate:
- Registration dates clustered shortly before major attacks
- Registrants used varied aliases/fake identities
- Shared name servers across domains — common infrastructure
- Many domains later taken down (law enforcement/security action)

Pattern analysis across many WHOIS records built a TTP profile and produced usable IOCs for other orgs.

### Using the `whois` Command

Install if missing:
```bash
sudo apt update
sudo apt install whois -y
```

Query:
```bash
whois facebook.com
```

```
Domain Name: FACEBOOK.COM
Registrar: RegistrarSafe, LLC
Creation Date: 1997-03-29T05:00:00Z
Registry Expiry Date: 2033-03-30T04:00:00Z
Registrant Organization: Meta Platforms, Inc.
Domain Status: clientDeleteProhibited, clientTransferProhibited, ...
Name Server: A.NS.FACEBOOK.COM
Name Server: B.NS.FACEBOOK.COM
```

### Reading the Output

| Field | What It Confirms |
|-------|--------------------|
| Long creation date, distant expiry | Established, legitimate domain |
| Registrant Organization | Confirms actual domain owner |
| Domain Status flags (client/server *Prohibited) | Domain protected against unauthorized transfer/deletion — indicates good security hygiene |
| Name servers within the same domain | Owner manages their own DNS infrastructure — typical for large orgs |

WHOIS alone rarely identifies individual employees or specific vulnerabilities — it's a starting signal, not a complete picture. Needs to be combined with other recon techniques.

### Pentesting Relevance
- The three-scenario pattern (recent registration + privacy masking + suspicious hosting) is a reusable heuristic for triaging any suspicious domain fast, not just during incident response
- Domain age and registrar reputation are quick legitimacy signals before deeper investigation
- Shared name servers or registration clustering across multiple domains can reveal an actor's broader infrastructure footprint
- WHOIS is a first-pass filter — treat its findings as leads to pursue with DNS enumeration, not final answers

---

## DNS

Translates human-readable domain names (`www.example.com`) into IP addresses (`192.0.2.1`) that machines actually use to communicate. Functions as the internet's directory lookup system.

### Resolution Flow

1. **DNS Query** — computer checks local cache; if empty, queries a DNS resolver (usually ISP-provided)
2. **Recursive Lookup** — resolver checks its own cache, then queries a **root name server** if needed
3. **Root Name Server** — points to the correct **TLD name server** (e.g. `.com`, `.org`)
4. **TLD Name Server** — points to the **authoritative name server** for the specific domain
5. **Authoritative Name Server** — holds and returns the actual IP address
6. **Resolver Returns Result** — sends the IP back to the computer, caching it for future requests
7. **Connection** — computer connects directly to the resolved IP

### The Hosts File

Manual override that bypasses DNS entirely — local hostname-to-IP mappings.

| OS | Location |
|----|----------|
| Windows | `C:\Windows\System32\drivers\etc\hosts` |
| Linux / macOS | `/etc/hosts` |

Format: `<IP Address>    <Hostname> [<Alias> ...]`

```
127.0.0.1       localhost
192.168.1.10    devserver.local
```

Common uses: pointing a domain to a local dev server (`127.0.0.1 myapp.local`), testing connectivity to a specific IP, or blocking a domain by redirecting it to `0.0.0.0`. Edits take effect immediately, no restart needed — requires admin/root privileges to edit.

### Zones and Zone Files

A **zone** is a portion of the DNS namespace managed by a specific entity — e.g. `example.com` and its subdomains. The **zone file** defines the resource records within it:

```dns-zone
$TTL 3600
@       IN SOA   ns1.example.com. admin.example.com. (
                2024060401 ; Serial
                3600       ; Refresh
                900        ; Retry
                604800     ; Expire
                86400 )    ; Minimum TTL

@       IN NS    ns1.example.com.
@       IN NS    ns2.example.com.
@       IN MX 10 mail.example.com.
www     IN A     192.0.2.1
mail    IN A     198.51.100.1
ftp     IN CNAME www.example.com.
```

### Core Concepts

| Concept | Description | Example |
|---------|--------------|---------|
| Domain Name | Human-readable label | `www.example.com` |
| IP Address | Numerical device identifier | `192.0.2.1` |
| DNS Resolver | Translates names to IPs | ISP resolver, Google DNS `8.8.8.8` |
| Root Name Server | Top of the DNS hierarchy | 13 worldwide, `a.root-servers.net` etc. |
| TLD Name Server | Handles a specific TLD | Verisign (`.com`), PIR (`.org`) |
| Authoritative Name Server | Holds the actual record for a domain | Managed by host/registrar |

### DNS Record Types

| Type | Full Name | Purpose | Example |
|------|-----------|---------|---------|
| A | Address Record | Hostname → IPv4 | `www.example.com. IN A 192.0.2.1` |
| AAAA | IPv6 Address Record | Hostname → IPv6 | `www.example.com. IN AAAA 2001:db8::1` |
| CNAME | Canonical Name | Alias to another hostname | `blog.example.com. IN CNAME webserver.example.net.` |
| MX | Mail Exchange | Mail server for the domain | `example.com. IN MX 10 mail.example.com.` |
| NS | Name Server | Delegates zone to an authoritative server | `example.com. IN NS ns1.example.com.` |
| TXT | Text Record | Arbitrary text — verification, SPF, etc. | `example.com. IN TXT "v=spf1 mx -all"` |
| SOA | Start of Authority | Zone admin info | `example.com. IN SOA ns1... admin... (...)` |
| SRV | Service Record | Hostname/port for specific services | `_sip._udp.example.com. IN SRV 10 5 5060 sipserver.example.com.` |
| PTR | Pointer Record | Reverse lookup — IP → hostname | `1.2.0.192.in-addr.arpa. IN PTR www.example.com.` |

`IN` = "Internet" class — nearly always this value in modern DNS (other classes like `CH`/`HS` are legacy/rare).

### Pentesting Relevance
- CNAME records pointing to outdated/deprecated infrastructure (`dev.example.com CNAME oldserver.example.net`) are a direct lead toward a potentially vulnerable, forgotten system
- NS records reveal the hosting provider; A records for things like `loadbalancer.example.com` help map real network topology and traffic flow
- Monitoring DNS changes over time can catch new subdomains appearing (`vpn.example.com`) — often a new, untested entry point
- TXT records can leak the org's tooling stack (e.g. `_1password=...` confirms password manager usage) — direct input for social engineering/phishing pretexting
- The hosts file is useful defensively during testing too — pin a domain to a specific IP locally when DNS is unreliable or when testing against a staging server sharing a production hostname
- This is entirely passive when reading public DNS records — no different in risk profile from WHOIS

---

## Digging DNS

Practical tools and techniques for DNS reconnaissance.

### DNS Tools

| Tool | Key Features | Use Case |
|------|----------------|----------|
| `dig` | Versatile, supports all record types, detailed output | Manual queries, zone transfers, troubleshooting, in-depth analysis |
| `nslookup` | Simpler, mainly A/AAAA/MX | Basic checks, quick resolution/mail server lookups |
| `host` | Streamlined, concise output | Quick A/AAAA/MX checks |
| `dnsenum` | Automated enumeration, dictionary attacks, zone transfers | Subdomain discovery at scale |
| `fierce` | Recursive search, wildcard detection | User-friendly subdomain recon |
| `dnsrecon` | Combines multiple techniques, multiple output formats | Comprehensive DNS enumeration |
| `theHarvester` | OSINT — pulls emails/employee info from DNS + other sources | Email/personnel gathering tied to a domain |
| Online DNS Lookup Services | Web UI, no CLI needed | Quick lookups, domain availability checks |

### Core `dig` Commands

| Command | Description |
|---------|--------------|
| `dig domain.com` | Default A record lookup |
| `dig domain.com A` / `AAAA` / `MX` / `NS` / `TXT` / `CNAME` / `SOA` | Query specific record type |
| `dig @1.1.1.1 domain.com` | Query a specific name server |
| `dig +trace domain.com` | Shows full resolution path |
| `dig -x 192.168.1.1` | Reverse lookup (IP → hostname) |
| `dig +short domain.com` | Concise answer only |
| `dig +noall +answer domain.com` | Answer section only |
| `dig domain.com ANY` | All record types (often ignored by servers per RFC 8482) |

Caution: excessive DNS queries can be detected/rate-limited. Always have permission before extensive DNS recon on a live target.

### Real Example — Reading a `dig` Response

```bash
dig google.com
```

```
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 16449
;; flags: qr rd ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             0       IN      A       142.251.47.142

;; Query time: 0 msec
;; SERVER: 172.23.176.1#53(172.23.176.1) (UDP)
```

| Section | What It Shows |
|---------|-----------------|
| Header | Query type, status (`NOERROR` = success), flags (`qr`=response, `rd`=recursion desired, `ad`=data considered authentic) |
| Question | What was asked — here, the A record for `google.com` |
| Answer | The result — IP `142.251.47.142`, TTL `0` (cache duration) |
| Footer | Query time, server used, protocol (UDP), timestamp, message size |

An `OPT` pseudosection may appear too — related to EDNS (Extension Mechanisms for DNS), enabling larger messages and DNSSEC support.

### Pentesting Relevance
- `dig +trace` is useful for understanding the full resolution chain when diagnosing DNS-based misdirection or unexpected results
- `dig -x` (reverse lookup) helps confirm hostnames behind IPs discovered through other recon, useful for identifying shared hosting or related infrastructure
- Automated tools (`dnsenum`, `fierce`, `dnsrecon`) turn manual `dig` queries into repeatable enumeration workflows — faster subdomain/record discovery at scale
- `theHarvester` bridges DNS recon with OSINT — email addresses pulled this way feed directly into phishing/social engineering pretexting
- Respecting rate limits matters — aggressive DNS querying can trip detection even though DNS itself is a low-risk passive technique

---

## Subdomains

Beyond the main domain, most organizations run a network of subdomains (`blog.example.com`, `shop.example.com`, `mail.example.com`) — often separately secured and separately discoverable.

### Why Subdomains Matter for Recon

| Category | Risk |
|----------|------|
| Dev/Staging Environments | Relaxed security, may expose vulnerabilities or sensitive data not present on production |
| Hidden Login Portals | Admin panels not meant to be public — attractive unauthorized-access targets |
| Legacy Applications | Forgotten apps running outdated, vulnerable software |
| Sensitive Information | Exposed configs, internal docs, confidential files |

### Subdomain Enumeration

DNS-wise, subdomains show up as `A`/`AAAA` records (direct IP mapping) or `CNAME` records (aliases to other hosts).

**Active Enumeration** — directly queries the target's DNS servers:
- **Zone transfer attempt** — misconfigured servers can leak the full subdomain list; rare on modern, properly hardened DNS
- **Brute-force enumeration** — tests a wordlist of common subdomain names against the target using tools like `dnsenum`, `ffuf`, `gobuster`

**Passive Enumeration** — uses external sources, no direct target interaction:
- **Certificate Transparency (CT) logs** — public SSL/TLS certificate repositories; the Subject Alternative Name (SAN) field often lists every subdomain a cert covers
- **Search engines** — `site:` operator narrows results to a target domain, surfacing indexed subdomains
- **Aggregator databases/tools** — compile DNS data from multiple sources for subdomain search without touching the target directly

### Pentesting Relevance
- CT logs are one of the highest-value passive subdomain sources — a single wildcard cert can reveal dozens of subdomains never linked anywhere publicly
- Active brute-forcing finds subdomains CT logs and search engines miss (unlisted, no cert issued yet) but carries higher detection risk — sequence it after passive methods
- Dev/staging subdomains found this way are consistently one of the most productive vulnerability sources in real engagements — weaker auth, debug mode left on, outdated dependencies
- Combining active + passive gives the most complete picture — passive alone will always undercount what actually exists

---

## Subdomain Bruteforcing

Active discovery technique — systematically tests candidate subdomain names against the target domain using a wordlist.

### Process

1. **Wordlist Selection** — General-purpose (common names like `dev`, `staging`, `admin`), targeted (industry/tech-specific), or custom (built from gathered intel)
2. **Iteration and Querying** — each wordlist entry gets prepended to the domain (`dev.example.com`, `staging.example.com`, etc.)
3. **DNS Lookup** — A/AAAA query for each candidate to check resolution
4. **Filtering and Validation** — successful resolutions get added to the valid list; further checks (e.g. browser access) confirm actual functionality

### Tools

| Tool | Notes |
|------|-------|
| dnsenum | Comprehensive, supports dictionary/brute-force |
| fierce | Recursive discovery, wildcard detection |
| dnsrecon | Combines multiple techniques, flexible output |
| amass | Actively maintained, integrates with other tools, broad data sources |
| assetfinder | Lightweight, quick scans |
| puredns | Fast, flexible resolving and filtering |

### dnsenum in Detail

Beyond brute-forcing, `dnsenum` also handles: DNS record enumeration (A/AAAA/NS/MX/TXT), automatic zone transfer attempts against discovered name servers, Google scraping for subdomains not in DNS directly, reverse lookups (IP → other hosted domains), and WHOIS queries.

```bash
dnsenum --enum inlanefreight.com -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -r
```

| Flag | Purpose |
|------|---------|
| `--enum` | Shortcut bundle of tuning options |
| `-f <wordlist>` | Wordlist path (SecLists in this case) |
| `-r` | Recursive — enumerate subdomains of discovered subdomains too |

### Real Example

Ran against `inlanefreight.com` with the SecLists top-20k subdomains wordlist. Output confirmed the base A record, then brute-forced hits including `www.inlanefreight.com` and `support.inlanefreight.com`, both resolving to `134.209.24.248`.

### Pentesting Relevance
- Wordlist quality directly determines yield — general-purpose lists are a fast baseline, but targeted/custom lists based on the org's actual naming conventions (industry jargon, product names, team structure) find far more
- Recursive brute-forcing (`-r`) can uncover deeply nested subdomains (`internal.dev.example.com`) that a single-pass scan misses
- `amass` and `puredns` are worth learning beyond `dnsenum` for larger scopes — better performance and data source integration at scale
- Active brute-forcing is noisier than passive techniques — sequence after CT logs/search engines to avoid redundant queries

---

## DNS Zone Transfers

A DNS zone transfer replicates all DNS records in a zone from a primary to a secondary name server — legitimate for redundancy, but a serious information leak if misconfigured to allow transfers to unauthorized clients.

### How It Works (AXFR)

1. **Zone Transfer Request** — secondary server sends an AXFR (full zone transfer) request to the primary
2. **SOA Record Transfer** — primary responds with the SOA record (includes serial number for freshness checking)
3. **DNS Records Transmission** — all records (A, AAAA, MX, CNAME, NS, etc.) sent one by one
4. **Zone Transfer Complete** — primary signals the transfer is done
5. **Acknowledgement** — secondary confirms receipt

### The Vulnerability

Historically, many DNS servers allowed zone transfer requests from *any* client, not just trusted secondaries. An unauthorized AXFR request against a misconfigured server hands over:

| Data Leaked | Value to an Attacker |
|-------------|------------------------|
| Full subdomain list | Includes unlinked dev/staging/admin subdomains |
| IP addresses per subdomain | Direct targets for further recon |
| Name server records | Reveals hosting provider, possible misconfig |

Modern servers are largely hardened to restrict AXFR to trusted secondaries only — but misconfiguration still happens, making a zone transfer attempt a low-cost, high-value check on every engagement (even failed attempts reveal server posture).

### Attempting a Zone Transfer

```bash
dig axfr @nsztm1.digi.ninja zonetransfer.me
```

Against a properly configured server this returns nothing useful. Against `zonetransfer.me` (a domain intentionally left open to demonstrate the risk), it returns the full zone:

```
zonetransfer.me.    7200 IN SOA nsztm1.digi.ninja. robin.digi.ninja. 2019100801 ...
zonetransfer.me.    300  IN HINFO "Casio fx-700G" "Windows XP"
zonetransfer.me.    7200 IN MX 0 ASPMX.L.GOOGLE.COM.
zonetransfer.me.    7200 IN A 5.196.105.14
zonetransfer.me.    7200 IN NS nsztm1.digi.ninja.
_acme-challenge.zonetransfer.me. 301 IN TXT "6Oa05hbUJ9x..."
_sip._tcp.zonetransfer.me. 14000 IN SRV 0 0 5060 www.zonetransfer.me.
canberra-office.zonetransfer.me. 7200 IN A 202.14.81.230
...
;; XFR size: 50 records
```

### Pentesting Relevance
- A successful zone transfer is one of the single highest-value findings in DNS recon — complete infrastructure map in one request, zero brute-forcing needed
- Always attempt AXFR against every discovered name server for a target, even when success is unlikely — the check costs nothing and occasionally pays off massively
- Even a failed attempt is informative — confirms the server correctly restricts transfers, which is itself useful for a security posture assessment
- `zonetransfer.me` is a safe, legal practice target for this exact technique — good for building muscle memory before using it on real engagements

---

## Virtual Hosts

Once DNS resolves a domain to a server's IP, the web server itself decides how to handle the request — and can host multiple sites/apps on that single IP through **virtual hosting**.

### VHosts vs Subdomains

| | Subdomains | Virtual Hosts (VHosts) |
|--|-----------|--------------------------|
| Defined by | DNS records | Web server configuration |
| Scope | Extensions of a main domain | Can be top-level domains or subdomains |
| Discovery | DNS enumeration | Requires probing the server directly (often no DNS record at all) |

A VHost with **no DNS record** is still reachable by manually mapping it in the local hosts file — bypassing DNS resolution entirely. Non-public subdomains/VHosts that don't appear in DNS are found via **VHost fuzzing** — testing hostnames directly against a known IP.

### How the Server Picks the Right Site

Driven entirely by the **HTTP Host header**:

1. Browser requests a domain → sends an HTTP request to that domain's resolved IP
2. Request includes the domain name in the `Host` header
3. Server checks its virtual host config for a matching entry
4. Server serves the matching site's content from its configured document root

Example Apache config — three separate domains served from one IP:

```apacheconf
<VirtualHost *:80>
    ServerName www.example1.com
    DocumentRoot /var/www/example1
</VirtualHost>

<VirtualHost *:80>
    ServerName www.example2.org
    DocumentRoot /var/www/example2
</VirtualHost>

<VirtualHost *:80>
    ServerName www.another-example.net
    DocumentRoot /var/www/another-example
</VirtualHost>
```

### Types of Virtual Hosting

| Type | How It Works | Trade-off |
|------|----------------|-----------|
| Name-Based | Relies purely on the Host header | Cheapest, most common, but has some SSL/TLS limitations |
| IP-Based | Each site gets its own IP | Better isolation, protocol-agnostic, but costly/less scalable |
| Port-Based | Different sites on different ports (e.g. `:80` vs `:8080`) | Useful when IPs are scarce, but less user-friendly (users must specify the port) |

### Discovery Tools

| Tool | Notes |
|------|-------|
| gobuster | Multi-purpose brute-forcer, supports VHost mode, multiple HTTP methods |
| Feroxbuster | Rust-based, fast, supports recursion/wildcard detection/filters |
| ffuf | Fuzzes the Host header directly, highly customizable |

### gobuster VHost Brute-Forcing

Prep needed: target IP (via DNS/other recon) and a wordlist (SecLists or custom, built from industry/naming conventions).

```bash
gobuster vhost -u http://<target_IP_address> -w <wordlist_file> --append-domain
```

| Flag | Purpose |
|------|---------|
| `-u` | Target URL/IP |
| `-w` | Wordlist path |
| `--append-domain` | Appends the base domain to each wordlist entry (required in newer Gobuster versions — older versions handled this automatically) |
| `-t` | Increase threads for faster scanning |
| `-k` | Ignore SSL/TLS certificate errors |
| `-o` | Save output to a file |

### Real Example

```bash
gobuster vhost -u http://inlanefreight.htb:81 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt --append-domain
```

Result: `Found: forum.inlanefreight.htb:81 Status: 200 [Size: 100]` — a VHost with no public DNS record, only discoverable by brute-forcing the Host header directly against the known IP.

### Pentesting Relevance
- VHosts are frequently where non-public admin panels, internal tools, or forgotten apps live specifically *because* they have no DNS record — DNS enumeration alone will miss them entirely
- Once discovered, a VHost with no DNS record can still be accessed by adding a hosts file entry pointing the hostname to the known IP
- VHost brute-forcing is inherently noisier than DNS-based subdomain enumeration — it hits the live web server directly with every request, raising IDS/WAF detection risk
- `--append-domain` matters a lot for correctness — a Gobuster scan silently missing this flag on newer versions can produce broken/incomplete hostnames and waste the whole run
- Multiple domains sharing one IP (as in the Apache example) means a single discovered IP can be a pivot point to entirely unrelated sites/orgs hosted on the same infrastructure

---

## Certificate Transparency Logs

SSL/TLS certificates verify a website's identity and enable encrypted communication — but the issuance process isn't foolproof. Rogue or mis-issued certificates can be exploited to impersonate legitimate sites. **Certificate Transparency (CT) logs** exist to make certificate issuance publicly auditable.

### What CT Logs Are

Public, append-only ledgers recording every SSL/TLS certificate a Certificate Authority (CA) issues. CAs are required to submit new certificates to multiple independent CT logs, open for anyone to inspect.

| Purpose | What It Enables |
|---------|-------------------|
| Early rogue certificate detection | Researchers/owners spot suspicious/misissued certs quickly, before abuse |
| CA accountability | Rule-violating issuance is publicly visible, risking sanctions/loss of trust |
| Strengthening Web PKI | Public oversight reinforces the trust system behind secure web communication |

### Why CT Logs Matter for Recon

Unlike brute-forcing, which is limited by wordlist coverage, CT logs give a **definitive historical record** of every certificate issued for a domain and its subdomains — including ones no longer active or hard to guess. Old/expired certificates can point to outdated, potentially vulnerable subdomains still running the software/config from when the cert was issued.

### Search Tools

| Tool | Features | Pros | Cons |
|------|-----------|------|------|
| crt.sh | Simple web UI, domain search, shows SAN entries | Free, no registration, easy | Limited filtering/analysis |
| Censys | Powerful search engine for internet-connected devices/certs | Extensive filtering, API access | Requires registration (free tier available) |

### crt.sh via API

```bash
curl -s "https://crt.sh/?q=facebook.com&output=json" | jq -r '.[] | select(.name_value | contains("dev")) | .name_value' | sort -u
```

```
*.dev.facebook.com
*.newdev.facebook.com
*.secure.dev.facebook.com
dev.facebook.com
devvm1958.ftw3.facebook.com
facebook-amex-dev.facebook.com
facebook-amex-sign-enc-dev.facebook.com
newdev.facebook.com
secure.dev.facebook.com
```

| Part | Purpose |
|------|---------|
| `curl -s "https://crt.sh/?q=facebook.com&output=json"` | Pull JSON of all certs matching the domain |
| `jq -r '.[] \| select(.name_value \| contains("dev")) \| .name_value'` | Filter to entries containing "dev", output raw strings |
| `sort -u` | Alphabetize and dedupe results |

### Pentesting Relevance
- CT logs are the highest-signal passive subdomain source available — a comprehensive historical record, not a guess
- Filtering by keyword (e.g. `dev`, `staging`, `internal`) via `jq` turns a noisy full cert list into a focused target list in one command
- Expired/old certificates flag subdomains that may still be running outdated, unpatched software — worth checking even if the cert itself is dead
- Wildcard certs (`*.dev.facebook.com`) confirm entire subdomain patterns exist even without enumerating every individual host
- No wordlist dependency means CT logs catch naming conventions no wordlist would predict — internal codenames, employee names, project-specific subdomains

---

## Key Takeaways
- Web recon is the foundation of the Information Gathering phase — everything downstream depends on what's found here
- Active recon = direct interaction, higher yield, higher detection risk
- Passive recon = indirect/public sources only, lower yield, near-zero detection risk
- Real engagements typically start passive and escalate to active only as scope/rules allow
- WHOIS is the starting point for the deeper techniques covered later in this module
- WHOIS records can leak personnel contacts and infra details, but GDPR-era privacy masking limits how much is exposed by default
- DNS records (especially CNAME, TXT, NS) can reveal outdated infrastructure, hosting providers, and tooling stack — all passive, near-zero detection risk
- The recent-registration + privacy-masking + suspicious-hosting pattern in WHOIS is a fast triage heuristic for any suspicious domain
- `dig` is the core DNS recon tool — `+trace` and `-x` are especially useful for tracing resolution paths and confirming hostnames behind IPs
- Certificate Transparency logs are one of the best passive subdomain discovery sources — SAN fields on public certs often list subdomains never linked anywhere
- Dev/staging subdomains discovered through enumeration are consistently high-value targets — weaker security than production by default
- Wordlist quality (targeted/custom over generic) is the main lever for brute-force subdomain enumeration yield
- Always attempt a DNS zone transfer (AXFR) against every discovered name server — low cost, occasionally reveals the entire subdomain/infra map at once
- VHosts differ from subdomains in that they're defined by web server config, not DNS — many have no DNS record at all and require direct Host-header fuzzing to find
- A discovered VHost with no DNS record is still reachable via a local hosts file entry pointing to the known IP
- CT logs give a definitive historical subdomain record beyond wordlist coverage — `crt.sh` via `curl`/`jq` is a fast way to filter for specific naming patterns (dev, staging, etc.)