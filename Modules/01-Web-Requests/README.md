# Module 01 — Web Requests

![Status](https://img.shields.io/badge/Status-Completed-yellow)
![Platform](https://img.shields.io/badge/Platform-HackTheBox-red)
![Category](https://img.shields.io/badge/Category-General-blue)

## What This Module Is About
HTTP is the protocol that powers all web communication. A client
sends a request to a server, the server processes it and returns
a response. Understanding this flow is the foundation of every
web pentest — if you don't know how requests work, you can't
manipulate them.

---

## HTTP Basics

Default port for HTTP is **80**, for HTTPS is **443**.

To reach a website we use a Fully Qualified Domain Name (FQDN)
written as a URL (Uniform Resource Locator).

### URL Structure

| Component | Example | Description |
|-----------|---------|-------------|
| Scheme | `http://` `https://` | Protocol being used, ends with `://` |
| User Info | `admin:password@` | Optional credentials before the host |
| Host | `inlanefreight.com` | Hostname or IP address of the server |
| Port | `:80` | Defaults: 80 (http), 443 (https) |
| Path | `/dashboard.php` | File or folder being requested |
| Query String | `?login=true` | Parameters passed to the server |
| Fragments | `#status` | Processed client-side to jump to a section |

---

## cURL

cURL (client URL) is a command-line tool for sending HTTP requests.
Essential for web pentesting — lets you craft, send, and inspect
requests without a browser.

### Commands I Used

```bash
# Basic GET request
curl http://info.cern.ch/

# Download a file silently
curl -s -O http://info.cern.ch/index.html

# View response headers
curl -i http://<SERVER_IP>:<PORT>/

# Verbose output — see full request and response
curl -v http://admin:admin@<SERVER_IP>:<PORT>/
```

---

## HTTP Basic Auth

Some pages use Basic HTTP Auth — handled by the webserver directly,
not a login form. Browser prompts for credentials before the page loads.

### How to Handle With cURL

```bash
# Method 1 — -u flag
curl -u admin:admin http://<SERVER_IP>:<PORT>/

# Method 2 — credentials in URL
curl http://admin:admin@<SERVER_IP>:<PORT>/

# Method 3 — manually set Authorization header
# admin:admin in base64 = YWRtaW46YWRtaW4=
curl -H 'Authorization: Basic YWRtaW46YWRtaW4=' http://<SERVER_IP>:<PORT>/
```

Key observation: Basic Auth encodes credentials in Base64 — not
encrypted. Anyone intercepting the request can decode it instantly.
This is why HTTPS matters.

---

## GET Requests and Parameters

GET requests pass parameters directly in the URL after `?`.

```bash
# Search request with GET parameter
curl 'http://<SERVER_IP>:<PORT>/search.php?search=le' \
  -H 'Authorization: Basic YWRtaW46YWRtaW4='

# Response
Leeds (UK)
Leicester (UK)
```

Useful trick — in browser DevTools Network tab, right-click any
request → Copy → Copy as cURL. Paste directly into terminal.
Saves time and gets the exact headers the browser sent.

---

## HTTP POST Requests

### Why POST Instead of GET
POST sends data in the request body instead of the URL:
- Avoids logging sensitive data in URL and server logs
- Accepts raw and binary data without URL encoding
- No character length limit like GET has

### Logging In With POST

```bash
curl -X POST -d 'username=admin&password=admin' http://<SERVER_IP>:<PORT>/
```

- `-X POST` sets the method
- `-d` attaches body data
- Add `-L` to follow redirects after successful login

### Capturing the Session Cookie

```bash
curl -X POST -d 'username=admin&password=admin' http://<SERVER_IP>:<PORT>/ -i
```

- `-i` prints response headers so you can see `Set-Cookie: PHPSESSID=...`
- In browser — DevTools → Storage tab → Cookies

### Reusing Cookie to Stay Authenticated

```bash
# Manual
curl -b 'PHPSESSID=<value>' http://<SERVER_IP>:<PORT>/

# Cleaner — let curl handle automatically
curl -X POST -d 'username=admin&password=admin' \
  http://<SERVER_IP>:<PORT>/ -c cookies.txt
curl -b cookies.txt http://<SERVER_IP>:<PORT>/
```

- `-c cookies.txt` saves cookies the server sets
- `-b cookies.txt` replays them on next request
- Key insight — a valid cookie alone authenticates you as that user.
  This is exactly what session hijacking exploits.

### Sending JSON POST Data

Some endpoints expect JSON instead of form data. Check
DevTools → request headers → Content-Type to confirm.

```bash
curl -X POST -d '{"search":"london"}' \
  -b cookies.txt \
  -H 'Content-Type: application/json' \
  http://<SERVER_IP>:<PORT>/search.php
```

Without `-H 'Content-Type: application/json'` the server rejects
the request even with a valid cookie.

---

## APIs and CRUD Operations

Web applications expose functionality through APIs that map directly
to database operations following the CRUD model.

| Operation | HTTP Method | What It Does |
|-----------|-------------|--------------|
| Create | POST | Adds a new row |
| Read | GET | Returns matching entries as JSON |
| Update | PUT | Replaces an entire entry |
| Delete | DELETE | Removes an entry |

A typical API URL pattern: `api.php/city/london` — changing the
HTTP method on the same URL controls what operation runs.

### Read

```bash
# Search for a city
curl -s http://<SERVER_IP>:<PORT>/api.php/city/london | jq

# Return all entries
curl -s http://<SERVER_IP>:<PORT>/api.php/city/ | jq
```

`jq` formats raw JSON into readable indented output.
`-s` silences curl progress so only response body shows.

### Create

```bash
curl -X POST http://<SERVER_IP>:<PORT>/api.php/city/ \
  -d '{"city_name":"HTB_City","country_name":"HTB"}' \
  -H 'Content-Type: application/json'
```

### Update

```bash
curl -X PUT http://<SERVER_IP>:<PORT>/api.php/city/london \
  -d '{"city_name":"New_HTB_City","country_name":"HTB"}' \
  -H 'Content-Type: application/json'
```

PUT requires the target in the URL — server needs to know which
row to modify. Some APIs treat PUT as "create if not exists" —
worth testing against a nonexistent entity.

### Delete

```bash
curl -X DELETE http://<SERVER_IP>:<PORT>/api.php/city/New_HTB_City
```

No body needed — identifier in the URL is enough.

---

## Lab Work

No lab environment was available for this module. The following
is based on running the curl commands against the HTB Academy
provided instances during the exercises.

### What I Tested

- Sent a basic GET request with curl and confirmed the response body
- Tested Basic Auth using all three methods — `-u` flag, credentials
  in URL, and manual Authorization header — all returned the same page
- Captured the PHPSESSID cookie after POST login using `-i` flag,
  then reused it with `-b` to access the authenticated page without
  logging in again
- Sent a JSON POST to the search endpoint — confirmed that without
  the correct Content-Type header the server returned an error even
  with a valid session cookie
- Tested all four CRUD operations on the API endpoint — created a
  city, read it back, updated it with PUT, confirmed the change, then
  deleted it and verified it was gone

### What I Got Wrong First

Forgot `-L` on the POST login — kept getting the redirect response
instead of the dashboard. Added `-L` and it followed through correctly.

Also tried sending JSON data without setting Content-Type — got a
server error. Realized the server was reading it as form data and
couldn't parse it.

### Key Observation

A session cookie is the only thing standing between an anonymous
user and an authenticated session. Once you have a valid PHPSESSID
you don't need the password at all. That is the core of session
hijacking — steal the cookie, become the user.

---

## Security Perspective

- Basic Auth = Base64, not encryption. Trivial to decode.
- GET parameters are in the URL — always logged, never secret
- Session cookies alone = full authentication, no password needed
- JSON endpoints are often less tested than form endpoints —
  always check Content-Type and test both formats
- CRUD endpoints with missing auth checks = IDOR and broken
  access control. Test every method on every endpoint.

---

## Key Takeaways

- HTTP default port 80, HTTPS 443
- URL has 7 components — each one is a potential attack surface
- cURL is the primary tool for manual HTTP request testing
- Basic Auth encodes in Base64 — never secure over plain HTTP
- GET parameters live in the URL — always logged, never secret
- POST body data is not in the URL but still readable in transit
  without HTTPS
- A valid session cookie = authenticated access, no password needed
- API endpoints follow CRUD — test all four methods, not just GET