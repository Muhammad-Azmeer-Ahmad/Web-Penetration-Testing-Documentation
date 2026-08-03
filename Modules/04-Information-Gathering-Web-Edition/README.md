# \# Module 04 — Information Gathering - Web Edition

# 

# !\[Status](https://img.shields.io/badge/Status-In%20Progress-yellow)

# !\[Platform](https://img.shields.io/badge/Platform-HackTheBox-red)

# !\[Category](https://img.shields.io/badge/Category-Offensive-orange)

# 

# \## Sections Progress

# \- \[x] Introduction

# \- \[ ] WHOIS

# \- \[ ] (sections 3–19 to be added as covered)

# 

# \---

# 

# \## What This Module Is About

# Web reconnaissance — systematically collecting information about a target website/application before deeper analysis or exploitation. Sits inside the Information Gathering phase of the penetration testing process (Pre-Engagement → Information Gathering → Vulnerability Assessment → Exploitation → Post-Exploitation → Lateral Movement → Proof-of-Concept → Post-Engagement).

# 

# \---

# 

# \## Introduction

# 

# \### Goals of Web Reconnaissance

# 

# | Goal | What It Covers |

# |------|-----------------|

# | Identifying Assets | Web pages, subdomains, IPs, technologies — full picture of the online footprint |

# | Discovering Hidden Information | Backup files, configs, internal docs left exposed |

# | Analysing the Attack Surface | Technologies, configurations, possible entry points |

# | Gathering Intelligence | Key personnel, emails, behavior patterns for social engineering |

# 

# Same information cuts both ways — attackers use it to target weaknesses, defenders use it to find and patch them first.

# 

# \### Active vs Passive Reconnaissance

# 

# \*\*Active\*\* — directly interacts with the target. More comprehensive, higher detection risk.

# 

# | Technique | What It Does | Tools | Detection Risk |

# |-----------|---------------|-------|-----------------|

# | Port Scanning | Finds open ports/services | Nmap, Masscan, Unicornscan | High |

# | Vulnerability Scanning | Probes for known vulns (outdated software, misconfig) | Nessus, OpenVAS, Nikto | High |

# | Network Mapping | Maps topology/connected devices | Traceroute, Nmap | Medium–High |

# | Banner Grabbing | Reads service banners (e.g. web server + version) | Netcat, curl | Low |

# | OS Fingerprinting | Identifies target OS | Nmap (`-O`), Xprobe2 | Low |

# | Service Enumeration | Identifies specific service versions | Nmap (`-sV`) | Low |

# | Web Spidering | Crawls site for pages/directories/files | Burp Suite Spider, ZAP Spider, Scrapy | Low–Medium |

# 

# \*\*Passive\*\* — gathers info without touching the target directly. Stealthier, less comprehensive.

# 

# | Technique | What It Does | Tools | Detection Risk |

# |-----------|---------------|-------|-----------------|

# | Search Engine Queries | Finds public info via search engines | Google, DuckDuckGo, Bing, Shodan | Very Low |

# | WHOIS Lookups | Domain registration details | `whois`, online WHOIS services | Very Low |

# | DNS Analysis | Subdomains, mail servers, infra via DNS records | dig, nslookup, host, dnsenum, fierce, dnsrecon | Very Low |

# | Web Archive Analysis | Historical snapshots of the target site | Wayback Machine | Very Low |

# | Social Media Analysis | Employee/org info from public profiles | LinkedIn, Twitter, Facebook, OSINT tools | Very Low |

# | Code Repositories | Exposed credentials/code vulns in public repos | GitHub, GitLab | Very Low |

# 

# \### Pentesting Relevance

# \- Passive recon should always come first — zero detection risk, and often surfaces enough to plan active recon precisely instead of scanning blindly

# \- Detection risk table matters for engagement rules of scope — some clients explicitly restrict active scanning until a defined phase

# \- Web spidering straddles the line — configure crawler behavior carefully or it can look like abnormal traffic

# \- Code repository exposure (GitHub/GitLab) is one of the highest-value passive techniques — leaked credentials or internal endpoints show up here constantly

# \- This module starts with WHOIS as the entry point into passive recon before moving into more advanced techniques

# 

# \---

# 

# \## Key Takeaways

# \- Web recon is the foundation of the Information Gathering phase — everything downstream depends on what's found here

# \- Active recon = direct interaction, higher yield, higher detection risk

# \- Passive recon = indirect/public sources only, lower yield, near-zero detection risk

# \- Real engagements typically start passive and escalate to active only as scope/rules allow

# \- WHOIS is the starting point for the deeper techniques covered later in this module

