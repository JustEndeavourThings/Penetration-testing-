# Penetration Testing Notes

Personal penetration-testing notes — methodology, vulnerability classes, tooling
and lab write-ups. Mirrored from the Notion database
**“Nothing Much, Just Endeavour’s things”** so everything lives next to the code
and can be searched, diffed and versioned.

> Everything here is for **authorised testing only** — labs, CTFs and engagements
> with written permission. Never point these techniques at systems you don't own
> or have explicit approval to test.

---

## Map of contents

| Area | Notes |
|---|---|
| **Web applications** | [01-web-application/01-web-vulnerabilities.md](01-web-application/01-web-vulnerabilities.md) |

---

## Web application topics

`01-web-application/01-web-vulnerabilities.md`

**Exported so far**

| # | Topic |
|---|---|
| 1 | File upload vulnerability |
| 2 | Server-side request forgery (SSRF) |
| 3 | Authentication vulnerabilities |
| 4 | Business logic vulnerabilities |
| 5 | Prototype pollution |
| 6 | Information disclosure |
| 7 | Race conditions |
| 8 | Host header injection |
| 9 | HTTP request smuggling |
| 10 | Web socket vulnerabilities |

**Still to export from Notion**

| Topic | Notion location |
|---|---|
| Server-side template injection | Web application → Nothing Much |
| JWT attacks | Web application → Nothing Much |
| Insecure deserialization | Web application → Nothing Much |
| Web cache poisoning | Web application → Nothing Much |
| Web cache deception | Web application → Nothing Much |
| SQLMAP and SQLi | Web application → Nothing Much |
| NoSQL injection | Web application → Nothing Much |
| CRLF injection | Web application → Nothing Much |
| Directory listing | Web application → Nothing Much |
| Local file inclusion (LFI) | Web application → Nothing Much |
| DDL hijacking | Web application → Nothing Much |
| API testing | Web application → Nothing Much |
| Gobuster | Web application → Nothing Much |
| OWASP | Web application → OWASP |
| Black box testing | Web application → Black box testing |
| Active Directory and Linux | top-level page |
| Mobile | top-level page |
| Thick client | top-level page |
| Tools | top-level page |
| Checklist | top-level page |
| Courses | top-level page |
| Sources | top-level page |
| need to do | top-level page |
| kct / kct dynamic / kbnu | top-level lab write-ups |
| Docker Install | top-level page |
| BloodHound CE | top-level page |

---

## How these notes are written

Each topic follows the same shape so it is quick to scan during an engagement:

1. **What it is** — one paragraph definition.
2. **Impact** — what an attacker actually gets.
3. **How to test** — numbered steps, Burp workflow, payloads and commands.
4. **Labs** — the PortSwigger / TryHackMe lab that exercises the technique.
5. **Prevention** — where the notes capture the fix.

Conventions:

- Payloads, commands and code are always in fenced code blocks so they can be
  copy-pasted.
- Lab-specific values (`YOUR-LAB-ID`, `192.168.0.1`, `carlos`, `wiener`) are kept
  verbatim from the original notes.
- Where the original note explicitly flags something as unfinished, that is
  preserved (e.g. *“need to study this”*, *“requires revision”*) rather than
  silently filled in.

---

## Repository layout

```
.
├── README.md                     # this index
└── 01-web-application/
    └── 01-web-vulnerabilities.md # web vulnerability classes (in progress)
```

Folders are numbered so the reading order matches the Notion structure:

- `01-web-application` — web app vulnerability classes and lab walkthroughs
  _(exists)_
- `02-active-directory-linux` — AD attack paths, Linux privesc _(planned)_
- `03-mobile` — Android / iOS testing _(planned)_
- `04-thick-client` — desktop client testing _(planned)_
- `05-tools` — tool notes and one-liners _(planned)_
- `06-labs` — kct, kct dynamic, kbnu write-ups _(planned)_
- `07-setup` — Docker, BloodHound CE and other lab setup _(planned)_
- `08-reference` — courses, sources, to-do list _(planned)_
