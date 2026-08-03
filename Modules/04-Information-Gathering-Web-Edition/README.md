# Module 04 — Information Gathering - Web Edition

![Status](https://img.shields.io/badge/Status-In%20Progress-yellow)
![Platform](https://img.shields.io/badge/Platform-HackTheBox-red)
![Category](https://img.shields.io/badge/Category-Offensive-orange)

## Sections Progress
- [x] Introduction
- [x] WHOIS
- [ ] (sections 3–19 to be added as covered)

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

## Key Takeaways
- Web recon is the foundation of the Information Gathering phase — everything downstream depends on what's found here
- Active recon = direct interaction, higher yield, higher detection risk
- Passive recon = indirect/public sources only, lower yield, near-zero detection risk
- Real engagements typically start passive and escalate to active only as scope/rules allow
- WHOIS is the starting point for the deeper techniques covered later in this module
- WHOIS records can leak personnel contacts and infra details, but GDPR-era privacy masking limits how much is exposed by default