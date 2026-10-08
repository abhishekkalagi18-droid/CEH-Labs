# Lab-03 — Footprinting Through Social Networking Sites

## 📌 Overview

This lab demonstrates how **Open-Source Intelligence (OSINT)** techniques can be used to identify publicly accessible social-media and online profiles associated with a username.

The lab uses **Sherlock**, an open-source username OSINT tool, to search for a controlled username across multiple online services. A detected Reddit result is then manually verified using an HTTP request.

> **Lab safety:** This exercise uses the synthetic username `cyberlab_test_2026`. When performing authorized security assessments, only investigate usernames and targets that are explicitly within scope.

---

## 🎯 Objectives

By completing this lab, you will learn how to:

- Understand username-based OSINT.
- Perform social-network footprinting.
- Use Sherlock from Kali Linux.
- Search a username across multiple platforms.
- Save Sherlock results to a text file.
- Manually verify a reported result.
- Interpret HTTP response codes during verification.
- Understand false positives in username enumeration.
- Document OSINT findings professionally.

---

## 🧠 What Is Social-Network Footprinting?

Social-network footprinting is the process of collecting publicly available information about an online identity from social-media platforms, forums, developer communities, gaming platforms, and other public services.

A common technique is **username enumeration**.

For example:

```text
Username
   │
   ▼
Sherlock
   │
   ├── Reddit
   ├── GitHub
   ├── Spotify
   ├── Hashnode
   ├── HackerEarth
   └── Other platforms
           │
           ▼
    Potential profiles
           │
           ▼
     Manual verification
```

The important point is that a username appearing on multiple websites **does not automatically prove that all accounts belong to the same person**.

---

# 🧪 Lab Environment

| Component | Details |
|---|---|
| Operating System | Kali Linux |
| Tool | Sherlock |
| Sherlock Version | 0.16.0 |
| Test Username | `cyberlab_test_2026` |
| Target Type | Synthetic username |
| Network | Internet |
| Verification Tool | `curl` |

---

# 🔧 Tool Used

## Sherlock

Sherlock is an open-source username OSINT tool designed to search for usernames across many online services.

Official project:

[Sherlock Project — GitHub](https://github.com/sherlock-project/sherlock?utm_source=chatgpt.com)

The official project documents searching one or multiple usernames and saving discovered accounts to output files.

---

# 1️⃣ Check Sherlock Version

### Command

```bash
sherlock --version
```

### Actual Result

```text
Sherlock v0.16.0
```

An update to a newer version was also reported by the tool.

### What this tells us

Checking the version before performing the assessment helps document:

- Tool version
- Reproducibility
- Environment used during testing
- Potential differences between tool versions

---

# 2️⃣ Perform Username Enumeration

For this lab, a synthetic username was used:

```text
cyberlab_test_2026
```

### Command

```bash
sherlock cyberlab_test_2026
```

Sherlock searched the username across multiple supported websites.

### Actual Result

Sherlock reported:

```text
[*] Search completed with 44 results
```

The reported platforms included:

```text
7Cups
Airliners
AniWorld
Apple Developer
Apple Discussions
Archive.org
CGTrader
Championat
F3.cool
Freelance.habr
GaiaOnline
GeeksforGeeks
HackerEarth
Hashnode
Hubski
kaskus
LessWrong
Open Collective
OurDJTalk
PCGamer
PepperIT
Reddit
ReverbNation
Roblox
SlideShare
Splice
Spotify
Trakt
Velomania
Weblate
authorSTREAM
dailykos
fixya
hunting
igromania
interpals
mastodon.cloud
mercadolivre
omg.lol
opennet
phpRU
social.tchncs.de
svidbook
threads
```

### Important observation

The result count was:

```text
44 results
```

This should be interpreted as **potential username matches**, not as proof that one individual controls all 44 accounts.

---

# 3️⃣ Save the Results

Saving the output makes the investigation reproducible and provides evidence for later analysis.

### Command

```bash
sherlock cyberlab_test_2026 --output sherlock_result.txt
```

### Verify the file

```bash
ls -lh sherlock_result.txt
```

### Actual Result

```text
-rw-r--r-- ... 2.3K ... sherlock_result.txt
```

The result file was successfully created.

---

# 4️⃣ Review the Saved Results

### Command

```bash
cat sherlock_result.txt
```

### Important Result

The file contained the detected profile URLs and ended with:

```text
Total Websites Username Detected On : 44
```

This confirms that the Sherlock output was successfully saved.

---

# 5️⃣ Select a Result for Manual Verification

One reported result was Reddit.

The profile URL was:

```text
https://www.reddit.com/user/cyberlab_test_2026/
```

Instead of assuming that Sherlock's detection is automatically correct, we manually tested the URL.

This is an important OSINT principle:

```text
Automated Detection
       ↓
Potential Match
       ↓
Manual Verification
       ↓
Finding
```

---

# 6️⃣ Verify the Reddit Profile Using HTTP Headers

### Command

```bash
curl -I -L "https://www.reddit.com/user/cyberlab_test_2026"
```

The `-I` option requests HTTP headers, while `-L` follows redirects.

---

## Actual Result

The first response was:

```text
HTTP/2 301
location: https://www.reddit.com/user/cyberlab_test_2026/
```

The server redirected the request to the canonical URL.

The next response was:

```text
HTTP/2 200
content-type: text/html
content-length: 8421
server: snooserv
```

### Interpretation

```text
Request
   ↓
Reddit
   ↓
301 Redirect
   ↓
Canonical profile URL
   ↓
200 OK
```

### Meaning of `301`

`301 Moved Permanently` indicates that the requested URL redirects to another URL.

In this case:

```text
/user/cyberlab_test_2026
          ↓
/user/cyberlab_test_2026/
```

The trailing slash is the canonical URL.

### Meaning of `200`

`200 OK` indicates that the redirected request successfully returned a webpage.

Therefore, the Reddit endpoint was reachable and returned content.

---

# 7️⃣ Verify Username in the Returned HTML

The page was further checked for the username.

### Command

```bash
curl -L -s "https://www.reddit.com/user/cyberlab_test_2026/" | grep -i -E "cyberlab_test_2026|not found|page not found" | head
```

### Actual Result

```html
<form hidden method="GET" action="/user/cyberlab_test_2026/">
```

### Analysis

The returned HTML contained:

```text
cyberlab_test_2026
```

This provides an additional verification signal that the requested Reddit profile path was represented in the returned page.

---

# 🔎 Findings

| Test | Result |
|---|---|
| Sherlock installed | ✅ |
| Sherlock version | `0.16.0` |
| Username searched | `cyberlab_test_2026` |
| Potential platforms detected | `44` |
| Results saved | ✅ |
| Reddit result detected | ✅ |
| Reddit HTTP response | `200 OK` after redirect |
| Username present in returned HTML | ✅ |
| Real-world identity established | ❌ |

---

# ⚠️ Understanding False Positives

Username OSINT tools can produce false positives.

For example:

```text
Username: cyberlab_test_2026

          ↓

Reddit       → Potential match
GitHub       → Potential match
Spotify      → Potential match
Hashnode     → Potential match
```

This does **not** prove:

```text
One person = all accounts
```

A username can be:

- Commonly reused.
- Registered by different people.
- Reserved but unused.
- Detected because of website response behavior.
- Associated with an old or inactive profile.
- A false positive caused by the site's response.

Therefore:

> **Automated OSINT results must be manually validated before being treated as evidence.**

---

# 🛡️ Security Relevance

Username enumeration is useful during the reconnaissance phase of an authorized penetration test or security assessment.

A tester may use publicly available usernames to understand an organization's external digital footprint.

Potential information sources include:

- Social networks
- Developer platforms
- Forums
- Technical communities
- Gaming platforms
- Public repositories
- Professional platforms

This information can help security professionals understand what information is publicly exposed.

OWASP describes information gathering as a foundational part of security testing and emphasizes identifying the assets and technologies that make up the attack surface.

---

# 🌐 Passive vs Active Reconnaissance

### Passive Reconnaissance

Information is collected without directly interacting with the target infrastructure.

Examples:

```text
Search engines
Public profiles
Public documents
Certificate databases
Public DNS information
```

### Active Reconnaissance

The tester directly interacts with a target system.

Examples:

```text
HTTP requests
Port scanning
Service enumeration
Banner grabbing
```

In this lab:

```text
Sherlock username search
        ↓
Mostly public-information discovery

curl verification
        ↓
Direct HTTP interaction
```

The distinction is important when defining scope for an authorized engagement.

---

# 🧪 Practical Workflow

The complete workflow performed in this lab was:

```text
1. Check Sherlock version
          ↓
2. Select controlled username
          ↓
3. Search username
          ↓
4. Identify potential profiles
          ↓
5. Save results
          ↓
6. Select one result
          ↓
7. Send HTTP request
          ↓
8. Follow redirect
          ↓
9. Inspect returned content
          ↓
10. Analyze confidence and limitations
```

---

# 📋 Evidence Collected

The following evidence was generated during the lab:

### Sherlock result file

```text
sherlock_result.txt
```

### Username

```text
cyberlab_test_2026
```

### Detected profiles

```text
44
```

### Verified platform

```text
Reddit
```

### HTTP behavior

```text
301 → 200
```

### HTML verification

```html
<form hidden method="GET" action="/user/cyberlab_test_2026/">
```
---
# 📌 Evidence Summary

| Evidence | Result |
|---|---|
| Test username | `cyberlab_test_2026` |
| Sherlock version | `0.16.0` |
| Potential profiles detected | `44` |
| Output file | `sherlock_result.txt` |
| Verified platform | Reddit |
| Initial HTTP response | `301 Moved Permanently` |
| Final HTTP response | `200 OK` |
| Username found in HTML | `cyberlab_test_2026` |
| Identity/ownership confirmed | ❌ No |

### Key Evidence

- Sherlock reported **44 potential username matches**.
- Results were successfully saved to `sherlock_result.txt`.
- The Reddit URL returned a **301 redirect** to its canonical URL.
- The canonical Reddit URL returned **HTTP 200 OK**.
- The returned HTML contained the tested username.
- These results verify the availability of the tested Reddit URL, but **do not prove real-world identity or ownership**.

> **Evidence conclusion:** The automated OSINT finding was successfully verified at the URL/content level, while identity attribution remains unconfirmed.
---

# 🧠 Key Learning Points

After completing this lab, you should understand:

1. What username OSINT is.
2. How Sherlock performs username enumeration.
3. How to save OSINT results.
4. Why automated findings require verification.
5. What HTTP `301` means.
6. What HTTP `200` means.
7. How `curl` can be used for lightweight verification.
8. Why username reuse can expose an organization's digital footprint.
9. Why a username match does not prove account ownership.
10. Why OSINT investigations must remain within authorized scope.

---

# 📝 Assessment Questions

### Basic

**1. What is OSINT?**

Open-Source Intelligence is the collection and analysis of information available from publicly accessible sources.

**2. What is Sherlock used for?**

Sherlock is used to search for a username across supported online services.

**3. What does HTTP 200 mean?**

The request was successfully processed and the server returned a successful response.

**4. What does HTTP 301 mean?**

The requested resource redirects to another URL.

### Intermediate

**5. Why should Sherlock results be manually verified?**

Because automated username detection can produce false positives.

**6. Does finding the same username on 10 websites prove that all accounts belong to one person?**

No.

**7. Why was a synthetic username used in this lab?**

To practice the technique without investigating a real person's identity.

### Practical

**8. How do you save Sherlock output?**

```bash
sherlock username --output result.txt
```

**9. How can you inspect HTTP headers?**

```bash
curl -I URL
```

**10. How can you follow redirects with curl?**

```bash
curl -L URL
```

---

# 🧩 Troubleshooting

## Sherlock command not found

Check:

```bash
sherlock --version
```

If Sherlock is not installed, consult the official Sherlock installation documentation rather than using an untrusted package source.

Official project:

[Sherlock GitHub Repository](https://github.com/sherlock-project/sherlock?utm_source=chatgpt.com)

---

## No results found

Possible reasons include:

- Username does not exist on supported services.
- Website changed its profile URL.
- Website is unavailable.
- Detection rule is outdated.
- Network connectivity problem.
- The username produces a false negative.

A zero-result search does not prove that the username has no online presence.

---

## curl does not return expected content

Test basic connectivity:

```bash
curl -I https://www.reddit.com/
```

Then test the specific profile:

```bash
curl -I -L "https://www.reddit.com/user/cyberlab_test_2026/"
```

---

# 🎯 Completion Criteria

The lab is considered complete when you can:

- [x] Run Sherlock.
- [x] Check the Sherlock version.
- [x] Search a username.
- [x] Identify potential profiles.
- [x] Save results to a file.
- [x] Read the saved results.
- [x] Select a potential profile.
- [x] Verify the profile using HTTP.
- [x] Understand `301` and `200` responses.
- [x] Inspect returned HTML.
- [x] Explain false positives.
- [x] Document findings professionally.

---

# 🌍 Real-World Scenario

During an authorized penetration test, an organization may provide a list of known employee usernames or public identities.

A penetration tester can use username OSINT to determine whether the same identifiers appear on publicly accessible platforms.

The workflow could be:

```text
Authorized Username
        ↓
OSINT Search
        ↓
Potential Public Profiles
        ↓
Manual Verification
        ↓
Correlate Only Relevant Public Information
        ↓
Document Exposure
        ↓
Recommend Risk Reduction
```

The objective is **not** to invade accounts or obtain private information.

The objective is to understand the organization's publicly exposed digital footprint.

---

# 🔐 Ethical and Legal Considerations

Only perform OSINT investigations against:

- Your own accounts.
- Synthetic test identities.
- Systems explicitly authorized for testing.
- Targets covered by a penetration-testing scope.

Do not:

- Attempt to log into discovered accounts.
- Guess passwords.
- Bypass authentication.
- Collect private information.
- Harass or track individuals.
- Attempt to deanonymize private individuals without authorization.

Publicly accessible does not automatically mean unrestricted for every security purpose.

---

# 📚 External Learning Resources

## Sherlock

Official Sherlock repository and documentation:

[Sherlock Project — GitHub](https://github.com/sherlock-project/sherlock?utm_source=chatgpt.com)

Sherlock's official repository contains installation instructions, usage examples, command-line options, and project documentation.

---

## OWASP Web Security Testing Guide

[OWASP Web Security Testing Guide — Information Gathering](https://owasp.org/www-project-web-security-testing-guide/?utm_source=chatgpt.com)

Useful for understanding reconnaissance and information-gathering methodology in professional security testing. OWASP identifies information gathering as a foundation for understanding an application's attack surface.

---

## OWASP — Search Engine Reconnaissance

[OWASP WSTG — Search Engine Reconnaissance](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/01-Information_Gathering/01-Conduct_Search_Engine_Reconnaissance_for_Information_Leakage/?utm_source=chatgpt.com)

This covers reconnaissance through publicly indexed information and information leakage.

---

## Reddit Help Center

[Reddit Help Center](https://support.reddithelp.com/?utm_source=chatgpt.com)

Useful for understanding Reddit profiles, account features, privacy, and security.

---

## OWASP Amass

[OWASP Amass](https://owasp.org/www-project-amass/?utm_source=chatgpt.com)

Amass is useful for progressing from basic OSINT toward broader attack-surface discovery and network mapping.

---

# 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| Sherlock | Username OSINT |
| curl | HTTP verification |
| Kali Linux | Security testing environment |
| Browser | Manual investigation |

---

# 📌 Final Result

```text
Lab: Footprinting Through Social Networking Sites

Username:
cyberlab_test_2026

Sherlock:
44 potential results

Saved Output:
sherlock_result.txt

Manual Verification:
Reddit

HTTP:
301 → 200 OK

HTML:
Username detected

Identity Confirmation:
Not established
```

---

## ✅ Lab Status

**COMPLETED**

This lab demonstrated the complete workflow from **username enumeration → result collection → manual HTTP verification → evidence analysis → professional documentation**.
