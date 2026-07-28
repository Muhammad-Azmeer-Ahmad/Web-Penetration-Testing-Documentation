# \# Module 03 — Using Web Proxies

# 

# !\[Status](https://img.shields.io/badge/Status-In%20Progress-yellow)

# !\[Platform](https://img.shields.io/badge/Platform-HackTheBox-red)

# !\[Category](https://img.shields.io/badge/Category-Tools-blue)

# 

# \## Sections Progress

# \- \[x] Intro to Web Proxies

# \- \[x] Setting Up

# \- \[x] Proxy Setup

# \- \[ ] Site Map

# \- \[ ] Repeater

# \- \[ ] Intruder

# \- \[ ] Decoder / Comparer

# \- \[ ] Sequencer

# \- \[ ] Extensions

# \- \[ ] Fuzzing with Proxies

# \- \[ ] Automating with Macros

# \- \[ ] Skills Assessment

# 

# \---

# 

# \## Intro to Web Proxies

# 

# MITM tools that sit between browser/app and back-end server, capturing and letting you modify every HTTP/HTTPS request. Unlike Wireshark (all traffic), proxies focus on web ports (80/443).

# 

# \*\*Uses:\*\* scanning, fuzzing, crawling, app mapping, request analysis, config testing, code review.

# 

# | | Burp Suite | ZAP |

# |-|-----------|-----|

# | Cost | Free / Paid Pro | Fully free |

# | Active scanner | Pro only | Free |

# | Throttling | Community limited | None |

# 

# \*\*Pentesting Relevance\*\*

# \- Foundation tool — nearly every later technique (fuzzing, IDOR, auth bypass) routes through the proxy

# \- Burp Community is enough for most work; Pro's scanner/Intruder speed matters at scale

# \- Know both — client setups won't always have your preferred tool

# 

# \---

# 

# \## Setting Up

# 

# | Tool | Launch (installed) | Launch (JAR) |

# |------|--------------------|--------------|

# | Burp | `burpsuite` | `java -jar burpsuite.jar` |

# | ZAP | `zaproxy` | `java -jar zap.jar` |

# 

# \- Burp Community → temporary project only, no save/resume

# \- Burp Pro → temp or disk-based persistent project

# \- ZAP → persistent (named/timestamped) or non-persistent session

# 

# \*\*Pentesting Relevance\*\*

# \- Temp projects = default for quick engagements/labs

# \- Persistent projects needed for long active scans on large scopes

# \- JAR launch useful on stripped-down VMs without package managers

# 

# \---

# 

# \## Proxy Setup

# 

# \*\*Pre-configured browser (fastest):\*\* Burp → Proxy > Intercept > Open Browser. ZAP → Firefox icon in toolbar. Both come with cert + proxy already set.

# 

# \*\*Manual Firefox setup:\*\*

# \- Default port `8080` for both tools (configurable if in use)

# \- FoxyProxy extension: IP `127.0.0.1`, Port `8080` — quick toggle instead of editing settings

# 

# \*\*CA Certificate (required for clean HTTPS interception):\*\*

# 

# | Tool | Get Cert |

# |------|----------|

# | Burp | `http://burp` → CA Certificate |

# | ZAP | Tools > Options > Network > Server Certificates > Save |

# 

# Import in Firefox: `about:preferences#privacy` → View Certificates → Authorities → Import → trust both boxes.

# 

# \*\*Pentesting Relevance\*\*

# \- No proxy setup = no interception = nothing else in this module works

# \- Missing CA cert = #1 cause of broken/missing HTTPS traffic in labs

# \- Manual Firefox config reflects real client engagements; pre-configured browser is lab-only convenience

# 

# \---

# 

# \## Key Takeaways

# \- Proxy setup is the precondition for everything else in this module

# \- Burp vs ZAP choice depends on budget and throttling needs, not raw capability

# \- CA cert install is the step people forget and then debug for an hour

