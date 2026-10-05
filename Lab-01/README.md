# Lab-01 — Footprinting and Reconnaissance

## CEH v13 Practical Cybersecurity Lab

**Difficulty:** Beginner → Intermediate  
**Category:** Reconnaissance / OSINT / DNS  
**Platform:** Kali Linux  
**Primary Target:** `example.com`  
**Tools:** WHOIS, dig, nslookup, host, theHarvester

---

## 1. Problem Statement

Before a penetration tester attempts vulnerability discovery or exploitation, the tester must first understand the target's publicly exposed information.

Organizations expose information through domain registration records, DNS infrastructure, IP addresses, mail configuration, certificate records, and other public sources.

The objective of this laboratory is to perform a structured **external reconnaissance assessment** against an authorized target.

The learner will collect information without attempting exploitation and will transform the collected information into a basic external attack-surface profile.

### Questions this lab should answer

- What information is publicly available about the target?
- What IP addresses are associated with the domain?
- Which DNS servers are authoritative?
- Is email infrastructure configured?
- What DNS security policies are visible?
- What information can passive OSINT sources provide?
- What information could be useful during a later penetration test?
- How should reconnaissance results be interpreted rather than blindly trusted?

---

# 2. Learning Objectives

After completing this lab, you should be able to:

- Explain footprinting and reconnaissance.
- Understand passive reconnaissance.
- Understand the difference between passive and active reconnaissance.
- Perform WHOIS enumeration.
- Enumerate IPv4 and IPv6 information.
- Identify authoritative DNS servers.
- Enumerate MX records.
- Analyze SOA records.
- Analyze TXT and SPF records.
- Understand DNS resolution using `dig +trace`.
- Perform basic passive OSINT using theHarvester.
- Interpret reconnaissance results.
- Correlate information from multiple tools.
- Identify security-relevant observations.
- Document reconnaissance findings professionally.

---

# 3. Important Lab Safety

Perform reconnaissance only against:

- Systems you own.
- Systems belonging to your organization with permission.
- Training environments.
- Targets explicitly authorized for security testing.

For this demonstration, `example.com` is used as a reserved example domain.

Do not assume that a publicly accessible system is automatically authorized for penetration testing.

**Passive does not mean invisible.** Passive tools may query third-party providers, and those providers can receive or log the target string. Some theHarvester features also perform DNS resolution or direct target interaction, so those features require appropriate authorization.

---

# 4. Reconnaissance Methodology

The workflow used in this lab is:

```text
Target Identification
        ↓
WHOIS Enumeration
        ↓
DNS Enumeration
        ↓
IPv4 / IPv6 Identification
        ↓
NS / MX Analysis
        ↓
SOA Analysis
        ↓
TXT / SPF Analysis
        ↓
DNS Trace
        ↓
Passive OSINT
        ↓
Correlation
        ↓
Attack-Surface Analysis
```

The important principle is:

```text
Command
   ↓
Evidence
   ↓
Interpretation
   ↓
Security Relevance
```

Running commands is not enough. A penetration tester must understand what the results mean.

---

# 5. Lab Environment

## Attacker / Analyst

Kali Linux

## Target

```text
example.com
```

## Tools

```text
whois
dig
nslookup
host
theHarvester
```

Check that the tools are available:

```bash
whois --help
```

```bash
dig -v
```

```bash
nslookup -version
```

```bash
host -V
```

```bash
theHarvester --version
```

---

# 6. Task 1 — WHOIS Enumeration

## Objective

Collect domain registration and administrative information.

## Command

```bash
whois example.com
```

## What to examine

Look for:

- Domain name
- Registrar
- Creation date
- Expiry date
- Domain status
- Name servers
- DNSSEC information

## Result observed

```text
Domain Name: EXAMPLE.COM

Registrar:
RESERVED-Internet Assigned Numbers Authority

Creation Date:
1995-08-14T04:00:00Z

Registry Expiry Date:
2027-08-13T04:00:00Z

Name Servers:
ELLIOTT.NS.CLOUDFLARE.COM
HERA.NS.CLOUDFLARE.COM

DNSSEC:
signedDelegation
```

## Analysis

The WHOIS information identifies the domain as an IANA-reserved example domain.

Two Cloudflare name servers are listed, and DNSSEC is enabled.

## Security relevance

WHOIS information can help establish:

- Domain ownership/administrative context.
- Registration history.
- DNS infrastructure.
- Potential organizational relationships.

During a real authorized assessment, this information can help the tester understand the target's external footprint.

---

# 7. Task 2 — DNS A Record Enumeration

## Objective

Identify IPv4 addresses associated with the target.

## Command

```bash
dig example.com
```

## Result observed

```text
example.com. 5 IN A 104.20.23.154
example.com. 5 IN A 172.66.147.243
```

## Findings

```text
IPv4 Address 1:
104.20.23.154

IPv4 Address 2:
172.66.147.243
```

## Explanation

An **A record** maps a hostname to an IPv4 address.

Multiple addresses may be returned for reasons such as distributed infrastructure, load balancing, redundancy, or CDN deployment.

## Security relevance

IP addresses provide an initial view of the infrastructure associated with a domain.

However, an IP address alone does not prove ownership of a particular server or application. Infrastructure may be shared or protected by a CDN.

---

# 8. Task 3 — Name Server Enumeration

## Objective

Identify the authoritative DNS servers.

## Command

```bash
dig example.com NS
```

## Result observed

```text
example.com. 5 IN NS elliott.ns.cloudflare.com.
example.com. 5 IN NS hera.ns.cloudflare.com.
```

## Findings

```text
Authoritative DNS Server 1:
elliott.ns.cloudflare.com

Authoritative DNS Server 2:
hera.ns.cloudflare.com
```

## Explanation

An **NS record** identifies the name servers responsible for DNS information for the domain.

## Security relevance

DNS infrastructure is an important part of external attack-surface mapping.

---

# 9. Task 4 — MX Record Enumeration

## Objective

Determine whether the domain publishes mail-exchange information.

## Command

```bash
dig example.com MX
```

## Result observed

```text
example.com. 5 IN MX 0 .
```

## Analysis

The domain publishes a **null MX** record.

The `.` value indicates that the domain does not advertise a normal mail server for receiving email.

## Security relevance

Mail infrastructure is an important part of reconnaissance because email services can become relevant during authorized security assessments.

A null MX record also provides useful defensive information: the domain is explicitly indicating that it does not accept normal inbound email.

---

# 10. Task 5 — DNS Enumeration Using nslookup

## Objective

Use a second DNS utility and compare the results.

## Command

```bash
nslookup example.com
```

## Result observed

```text
Name:
example.com

IPv4:
172.66.147.243
104.20.23.154

IPv6:
2606:4700:10::ac42:93f3
2606:4700:10::6814:179a
```

## Findings

### IPv4

```text
104.20.23.154
172.66.147.243
```

### IPv6

```text
2606:4700:10::ac42:93f3
2606:4700:10::6814:179a
```

## Security relevance

IPv6 should not be ignored during reconnaissance.

A tester who checks only IPv4 may miss part of an organization's externally visible infrastructure.

---

# 11. Task 6 — DNS Enumeration Using host

## Objective

Use another DNS utility to obtain a concise summary.

## Command

```bash
host example.com
```

## Result observed

```text
example.com has address 104.20.23.154
example.com has address 172.66.147.243

example.com has IPv6 address
2606:4700:10::6814:179a

example.com has IPv6 address
2606:4700:10::ac42:93f3

example.com mail is handled by 0 .

example.com has HTTP service bindings
alpn="h3,h2"
```

## Analysis

The result confirms the IPv4 and IPv6 information collected using `dig` and `nslookup`.

The HTTP service binding advertises:

```text
h3
h2
```

which indicates HTTP/3 and HTTP/2 protocol support being advertised through DNS service information.

## Security relevance

Using several DNS tools is useful because it allows a tester to validate and correlate findings.

---

# 12. Task 7 — SOA Enumeration

## Objective

Understand the administrative information contained in the Start of Authority record.

## Command

```bash
dig example.com SOA
```

## Result observed

```text
example.com. 5 IN SOA
elliott.ns.cloudflare.com.
dns.cloudflare.com.
2416374680
10000
2400
604800
1800
```

## Extracted information

```text
Primary Name Server:
elliott.ns.cloudflare.com

Responsible Party:
dns.cloudflare.com

Serial:
2416374680

Refresh:
10000

Retry:
2400

Expire:
604800

Minimum:
1800
```

## Explanation

SOA stands for **Start of Authority**.

The SOA record contains administrative information about a DNS zone.

Important fields include:

- Primary name server
- Responsible party
- Serial number
- Refresh interval
- Retry interval
- Expiration interval
- Minimum/negative caching TTL

## Security relevance

SOA information helps a tester understand the DNS architecture and administrative configuration of the target.

---

# 13. Task 8 — TXT and SPF Enumeration

## Objective

Identify publicly available TXT records and examine email security configuration.

## Command

```bash
dig example.com TXT
```

## Result observed

```text
example.com. 5 IN TXT "_k2n1y4vw3qtb4skdx9e7dxt97qrmmq9"

example.com. 5 IN TXT "v=spf1 -all"
```

## Important finding

```text
v=spf1 -all
```

## Explanation

SPF stands for **Sender Policy Framework**.

SPF is used to publish information about which systems are authorized to send email for a domain.

The `-all` mechanism represents a strict SPF failure policy for senders not authorized by the policy.

## Security relevance

Email configuration is important during reconnaissance because email infrastructure may be relevant to later authorized security testing and defensive assessment.

---

# 14. Task 9 — DNS Trace

## Objective

Understand DNS resolution and the DNS hierarchy.

## Command

```bash
dig +trace example.com
```

## Result observed

The trace returned root DNS infrastructure and ultimately returned:

```text
example.com. 287 IN A 104.20.23.154
example.com. 287 IN A 172.66.147.243
```

It also identified:

```text
elliott.ns.cloudflare.com
hera.ns.cloudflare.com
```

## Explanation

`dig +trace` performs iterative DNS resolution and helps demonstrate how DNS information is obtained through the hierarchy.

## Important observation

Do not claim that every trace output will visibly show every DNS hierarchy stage. The exact output depends on the resolver and response behavior.

## Security relevance

Understanding DNS resolution helps security professionals understand how domain names map to infrastructure.

---

# 15. Task 10 — Passive Reconnaissance with theHarvester

## Objective

Use an OSINT tool to collect publicly available information from supported sources.

Check the installed version:

```bash
theHarvester --version
```

Result:

```text
theHarvester 4.10.1
```

Check available options:

```bash
theHarvester --help
```

Run a certificate-transparency source:

```bash
theHarvester -d example.com -b crtsh
```

## Result observed

```text
[*] Target: example.com

[*] Searching CRTsh.

[*] No IPs found.

[*] No emails found.

[*] No people found.

[*] Hosts found: 0
```

## Analysis

The selected CRT.sh source returned no findings for this target.

This is **not automatically a tool failure**.

A reconnaissance source can legitimately return zero findings.

Current theHarvester documentation specifically distinguishes empty results from failed or incomplete execution.

## Security lesson

Never write:

> "No vulnerabilities exist because the tool found nothing."

Instead write:

> "The selected reconnaissance source returned no findings during this collection."

This distinction is extremely important in professional security assessments.

---

# 16. Reconnaissance Findings Summary

| Category | Finding |
|---|---|
| Target | example.com |
| Registrar | IANA |
| DNS Provider | Cloudflare infrastructure |
| IPv4 | 104.20.23.154 |
| IPv4 | 172.66.147.243 |
| IPv6 | 2606:4700:10::6814:179a |
| IPv6 | 2606:4700:10::ac42:93f3 |
| NS | elliott.ns.cloudflare.com |
| NS | hera.ns.cloudflare.com |
| MX | Null MX |
| SOA | elliott.ns.cloudflare.com |
| TXT | `_k2n1y4vw3qtb4skdx9e7dxt97qrmmq9` |
| SPF | `v=spf1 -all` |
| DNSSEC | Enabled |
| HTTP ALPN | h3, h2 |
| CRT.sh | No hosts/IPs/emails/people returned |

---

# 17. What a Penetration Tester Learns From This

The purpose of reconnaissance is not simply collecting information.

A professional tester asks:

### What infrastructure exists?

IP addresses and DNS records provide an initial infrastructure picture.

### How is DNS managed?

NS and SOA records provide information about DNS architecture.

### Is email infrastructure exposed?

MX and SPF records provide useful information about email configuration.

### Is IPv6 present?

Yes. IPv6 addresses were identified.

### Is the infrastructure behind a service provider?

The DNS information indicates Cloudflare infrastructure.

### Did every OSINT source produce results?

No.

This demonstrates an important professional principle:

> **Absence of evidence is not automatically evidence of absence.**

---

# 18. Reconnaissance vs Vulnerability Assessment

Do not confuse these stages.

```text
Reconnaissance
      ↓
Identify infrastructure
      ↓
Identify hosts
      ↓
Identify services
      ↓
Vulnerability Assessment
      ↓
Identify vulnerabilities
      ↓
Validation
      ↓
Authorized Exploitation
```

This lab stops at reconnaissance.

Nmap, Nessus, vulnerability validation, exploitation, privilege escalation, etc. belong to later labs.

---

# 19. Real-World Scenario

You are an external penetration tester.

Your client gives you:

```text
Target:
AUTHORIZED-DOMAIN
```

You are not initially given:

- IP addresses
- DNS information
- Mail infrastructure
- Subdomains
- Technology information

Your first task is to build an external reconnaissance profile.

Your report should identify:

1. Domain information
2. DNS infrastructure
3. IPv4 addresses
4. IPv6 addresses
5. Name servers
6. Mail configuration
7. DNS security configuration
8. Public OSINT information
9. Interesting observations
10. Potential areas for further authorized investigation

Do not exploit the target during this exercise.

---

# 20. Learner Questions

Answer these after completing the lab.

### Basic

1. What is footprinting?
2. What is reconnaissance?
3. What is passive reconnaissance?
4. What is active reconnaissance?
5. Why is reconnaissance performed before vulnerability assessment?

### DNS

6. What is an A record?
7. What is an AAAA record?
8. What is an NS record?
9. What is an MX record?
10. What is an SOA record?
11. What is a TXT record?
12. What is SPF?
13. What does `-all` mean in an SPF policy?
14. What is DNSSEC?
15. What does `dig +trace` demonstrate?

### Professional analysis

16. Why should IPv6 be included during reconnaissance?
17. Why might multiple IP addresses be returned?
18. Why can an OSINT tool return zero results?
19. Why should findings from multiple tools be correlated?
20. Why does publicly available information still require careful handling?
21. Why should a tester distinguish between "no result" and "failed collection"?
22. What information from this lab could be useful during a later authorized security assessment?

---

# 21. Common Troubleshooting

## Problem: WHOIS command unavailable

Check:

```bash
which whois
```

Install if required:

```bash
sudo apt update
sudo apt install whois
```

---

## Problem: DNS queries fail

Check connectivity:

```bash
ip addr
```

Check routing:

```bash
ip route
```

Check DNS configuration:

```bash
cat /etc/resolv.conf
```

Test DNS:

```bash
dig example.com
```

---

## Problem: theHarvester returns zero results

Check the installed version:

```bash
theHarvester --version
```

Check supported sources:

```bash
theHarvester --help
```

A zero-result source is not automatically a failed scan. Check the source outcome and, where appropriate, validate the question using another authorized source.

---

# 22. Recommended Evidence

This repository intentionally does not require screenshots.

The important evidence is documented directly below each command in this README.

The actual outputs obtained during this lab have been recorded in the relevant sections.

This keeps the repository:

- Clean
- Easy to navigate
- Lightweight
- Beginner-friendly
- Recruiter-friendly

---

# 23. Key Commands Learned

```bash
whois example.com

dig example.com

dig example.com NS

dig example.com MX

nslookup example.com

host example.com

dig example.com SOA

dig example.com TXT

dig +trace example.com

theHarvester --version

theHarvester --help

theHarvester -d example.com -b crtsh
```

---

# 24. Important Professional Tips

### Tip 1 — Never trust one tool

Validate important findings with multiple sources.

Example:

```text
dig
 ↓
nslookup
 ↓
host
 ↓
OSINT source
```

---

### Tip 2 — Understand before using a command

Do not memorize:

```bash
dig example.com MX
```

Understand:

> "I am asking DNS for the domain's mail-exchange records."

---

### Tip 3 — Record negative results

A result such as:

```text
Hosts found: 0
```

is still evidence.

Document what the tool actually observed.

---

### Tip 4 — Don't confuse reconnaissance with exploitation

Reconnaissance answers:

> "What exists?"

Vulnerability assessment asks:

> "What might be vulnerable?"

Exploitation asks:

> "Can the vulnerability actually be validated?"

These are different stages.

---

### Tip 5 — Learn DNS properly

DNS is fundamental to:

- Web security
- Email security
- Cloud security
- Network reconnaissance
- Subdomain discovery
- Infrastructure mapping

Do not skip DNS fundamentals.

---

### Tip 6 — Think like a defender

For every finding, ask:

> "If I were defending this organization, why would this information matter?"

This develops security-analysis skills rather than command memorization.

---

# 25. Useful Tools

## DNS / Network

- `dig`
- `nslookup`
- `host`
- `whois`

## OSINT

- theHarvester
- Certificate Transparency
- Wayback Machine
- URLScan
- DNS reconnaissance platforms

## Later Labs

As this repository progresses, additional tools will be introduced for:

- Network discovery
- Port scanning
- Service enumeration
- Vulnerability assessment
- Web security
- Wireless security
- Password security
- Traffic analysis
- Exploitation
- Privilege escalation
- Cloud security

Do not use later-stage tools simply because they are available. Learn them when their role in the methodology becomes clear.

---

# 26. Learning Resources

### Kali Linux Documentation

Official Kali documentation:

https://www.kali.org/docs/

Useful for understanding Kali tools, installation, configuration, and troubleshooting.

### theHarvester

Official project repository and documentation:

https://github.com/laramies/theHarvester

theHarvester is designed for OSINT/reconnaissance and supports multiple public information sources.

### theHarvester Quick Start

https://github.com/laramies/theHarvester/blob/master/docs/wiki/Quick-Start.md

Useful for learning source selection, reporting, and result interpretation.

### theHarvester Responsible Use

https://github.com/laramies/theHarvester/blob/master/docs/wiki/Responsible-Use-and-Scope.md

Read this before using active/enrichment features.

### DNS — Cloudflare Learning Center

https://www.cloudflare.com/learning/dns/

Recommended for understanding DNS concepts from fundamentals.

### ICANN

https://www.icann.org/

Useful for understanding domain names, DNS, registries, and Internet naming infrastructure.

### Certificate Transparency

https://certificate.transparency.dev/

Useful for understanding publicly logged TLS certificates and their role in security research.

---

# 27. Recommended Learning Path After Lab-01

Do not immediately jump into exploitation.

Build your skills progressively:

```text
Lab-01
Footprinting & Reconnaissance
        ↓
Lab-02
DNS / OSINT Reconnaissance
        ↓
Lab-03
Active Reconnaissance
        ↓
Lab-04
Host Discovery
        ↓
Lab-05
Port Scanning
        ↓
Lab-06
Service Enumeration
        ↓
Lab-07
Windows Enumeration
        ↓
Vulnerability Assessment
        ↓
System Hacking
        ↓
Privilege Escalation
        ↓
Web Security
        ↓
Network Security
        ↓
Advanced Security Testing
```

The exact later lab sequence will be developed progressively rather than adding unrelated commands to this lab.

---

# 28. Final Takeaways

After completing Lab-01, you should understand that professional penetration testing begins with information gathering.

You should now be comfortable with:

```text
WHOIS
DNS
A / AAAA
NS
MX
SOA
TXT
SPF
DNSSEC
DNS Trace
OSINT
theHarvester
```

More importantly, you should understand the methodology:

```text
Discover
   ↓
Verify
   ↓
Correlate
   ↓
Analyze
   ↓
Document
```

The objective is not to become someone who can execute commands.

The objective is to become a security professional who can **interpret what those commands reveal and explain why the information matters**.

---

# 29. Lab Completion Checklist

- [x] WHOIS enumeration
- [x] A record enumeration
- [x] IPv6 identification
- [x] NS enumeration
- [x] MX enumeration
- [x] SOA enumeration
- [x] TXT enumeration
- [x] SPF analysis
- [x] DNS trace
- [x] theHarvester reconnaissance
- [x] Result interpretation
- [x] Security analysis
- [x] Troubleshooting guidance
- [x] Real-world scenario
- [x] Learner questions
- [x] Additional learning resources

## Lab Status

**COMPLETED**

---

### Next Lab

**Lab-02 — DNS & OSINT Reconnaissance**

The next lab will build on the foundation established here and introduce additional reconnaissance techniques rather than repeating the same DNS commands.
