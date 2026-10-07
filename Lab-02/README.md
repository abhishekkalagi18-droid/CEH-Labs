# Lab-02 — Internet Research & DNS Reconnaissance

## 📌 Overview

This lab demonstrates passive reconnaissance using publicly available Internet research services.

The objective was to identify publicly exposed information about a target domain, including hosting infrastructure, IP addresses, autonomous systems, DNS records, subdomains/hosts, mail infrastructure, TLS configuration, and technology information.

Two research services were used:

- **Netcraft Site Report**
- **DNSDumpster**

The lab was performed for cybersecurity learning and reconnaissance practice. No exploitation or unauthorized access was attempted.

---

## 🎯 Objectives

- Understand Internet-based passive reconnaissance.
- Identify publicly visible hosting and network information.
- Identify IP addresses and autonomous systems.
- Discover DNS name servers and MX records.
- Discover subdomains and associated hosts.
- Examine publicly reported services and technologies.
- Analyze SSL/TLS configuration.
- Understand how reconnaissance information contributes to attack-surface mapping.

---

## 🧠 What is Internet Reconnaissance?

Internet reconnaissance is the process of collecting information about an organization's publicly exposed infrastructure before performing security testing.

Typical information includes:

```text
Domain
 ├── Subdomains
 ├── DNS Servers
 ├── Mail Servers
 ├── IP Addresses
 ├── ASN / Network
 ├── Hosting Provider
 ├── Technologies
 └── Publicly visible Services
```

This information helps penetration testers understand the external attack surface before moving to vulnerability assessment.

DNSDumpster describes domain-based reconnaissance as a way to correlate DNS records with Internet-scale datasets to discover networks, IP addresses, and services.

---

# 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| Netcraft Site Report | Infrastructure and technology reconnaissance |
| DNSDumpster | DNS, host and subdomain reconnaissance |
| Web Browser | Accessing public research services |

Netcraft's current research tools provide Site Report and DNS-related research capabilities for examining Internet infrastructure and technologies.

---

# 🌐 Target

```text
Target Domain: certifiedhacker.com
```

The domain was used for passive reconnaissance through publicly available research services.

---

# 🔎 Part 1 — Netcraft Site Report

## Objective

Use Netcraft to collect publicly available information about the target's network infrastructure, DNS configuration, hosting environment, TLS configuration, and technology stack.

Netcraft's Site Report is designed to provide information about hosting providers, registrars, IP addresses, SSL certificates, historical observations, and related infrastructure.

---

## Step 1 — Open Netcraft

Open the Netcraft Site Report:

https://sitereport.netcraft.com/

Enter:

```text
certifiedhacker.com
```

Run the report.

---

# 📊 Netcraft Findings

## Background

The Site Report identified:

```text
Site title: Not Acceptable!
Site rank: 15052
Date first seen: December 2002
Primary language: English
```

---

## Network Information

| Property | Result |
|---|---|
| Site | `http://certifiedhacker.com` |
| Netblock Owner | Unified Layer |
| Hosting Company | Newfold Digital |
| Hosting Country | United States |
| IPv4 | `162.241.216.11` |
| IPv4 ASN | `AS31898` |
| IPv6 | Not Present |
| Reverse DNS | `box5331.bluehost.com` |
| Nameserver | `ns1.bluehost.com` |
| Registrar | `networksolutions.com` |
| Nameserver Organisation | `whois.domain.com` |
| DNSSEC | Enabled |

### Analysis

The reported IPv4 address belongs to the Unified Layer network and is associated with AS31898.

Reverse DNS resolves to:

```text
box5331.bluehost.com
```

This provides an additional infrastructure identifier that can be useful during external attack-surface mapping.

DNSSEC was reported as enabled.

---

# 🌐 IP Delegation

The IPv4 address:

```text
162.241.216.11
```

was shown within:

```text
162.240.0.0 – 162.241.255.255
```

with the network identified as:

```text
UNIFIEDLAYER-NETWORK-16
```

### Security relevance

IP ownership and network allocation information can help a penetration tester determine:

- Which network provider hosts the asset.
- Which IP ranges may belong to the organization or provider.
- Whether multiple discovered assets may share infrastructure.

---

# 🔐 SSL/TLS Analysis

The HTTP Netcraft report indicated that the tested URL was not HTTPS.

Therefore, the report itself stated that HTTPS information should be obtained through the HTTPS Site Report.

This is an important reconnaissance observation because HTTP and HTTPS should not be treated as equivalent endpoints.

---

# 📧 Sender Policy Framework

Netcraft identified the following SPF mechanisms:

```text
+ a
+ mx
+ ptr
+ include:bluehost.com
? all
```

### Analysis

The record indicates that several mechanisms are permitted to contribute to the SPF evaluation, while `?all` represents a neutral result for other senders.

This information is useful during email-security reconnaissance.

---

# 📩 DMARC

Netcraft reported:

```text
DMARC record: Not present
```

### Security relevance

The absence of a DMARC record may be relevant when assessing an organization's email-security posture.

However, the absence of a DMARC record alone does not prove that email spoofing is exploitable.

---

# 🔍 Part 2 — DNSDumpster

## Objective

Use DNSDumpster to discover:

- Hostnames
- Subdomains
- IP addresses
- DNS servers
- Mail servers
- TXT/SPF records
- Publicly reported services

DNSDumpster is specifically designed for domain-based DNS reconnaissance and discovering hosts related to a domain.

---

## Step 1 — Open DNSDumpster

Open:

https://dnsdumpster.com/

Enter:

```text
certifiedhacker.com
```

Start the test.

---

# 📋 DNSDumpster Host Discovery

DNSDumpster returned numerous hostnames associated with:

```text
certifiedhacker.com
```

Examples include:

```text
autodiscover.certifiedhacker.com
blog.certifiedhacker.com
ciphershield.certifiedhacker.com
cpanel.certifiedhacker.com
demo.certifiedhacker.com
events.certifiedhacker.com
fleet.certifiedhacker.com
ftp.certifiedhacker.com
iam.certifiedhacker.com
itf.certifiedhacker.com
mail.certifiedhacker.com
news.certifiedhacker.com
notifications.certifiedhacker.com
pstn.certifiedhacker.com
sftp.certifiedhacker.com
soc.certifiedhacker.com
trustcenter.certifiedhacker.com
webdisk.certifiedhacker.com
webmail.certifiedhacker.com
www.certifiedhacker.com
```

The supplied DNSDumpster result contains additional hostnames beyond those listed above.

---

# 🌐 Host-to-IP Mapping

A large number of discovered hosts were associated with:

```text
IP Address:
162.241.216.11

Reverse DNS:
box5331.bluehost.com

ASN:
AS31898

Network:
162.241.216.0/22
```

The result identifies the ASN/network as associated with Oracle Corporation in its database output.

### Security relevance

Multiple hostnames resolving to the same IP can indicate shared hosting or a centralized infrastructure environment.

A penetration tester would normally investigate the ownership and authorization boundaries before testing any discovered host.

---

# ⚠️ Database-Reported Services

DNSDumpster's database reported services for several hosts mapped to `162.241.216.11`, including:

```text
SSH
OpenSSH 9.9

FTP
Pure-FTPd

HTTP
Apache HTTP Server

HTTPS
Apache HTTP Server
```

For example, these observations appeared for `autodiscover.certifiedhacker.com` and other hosts.

### Important distinction

These are **DNSDumpster database observations**, not results from an Nmap scan performed during this lab.

Therefore:

```text
DNSDumpster result ≠ confirmed current service availability
```

A separate authorized active scan would be required to verify current service exposure.

---

# 📧 MX Record

DNSDumpster identified:

```text
Priority: 0

Mail Server:
mail.certifiedhacker.com

IP:
162.241.216.11
```



### Analysis

The domain's mail exchanger points to:

```text
mail.certifiedhacker.com
```

This provides useful information about the organization's externally visible email infrastructure.

---

# 🌐 NS Records

DNSDumpster identified:

```text
ns1.bluehost.com
    ↓
162.159.24.80

ns2.bluehost.com
    ↓
162.159.25.175
```

The database associates both addresses with:

```text
AS13335
CLOUDFLARENET
```



### Security relevance

Name servers are important reconnaissance targets because they provide information about the DNS infrastructure responsible for the domain.

---

# 📝 TXT / SPF Record

DNSDumpster returned:

```text
v=spf1 a mx ptr include:bluehost.com ?all
```



This provides additional confirmation of the SPF configuration discovered during reconnaissance.

---

# 🧩 Attack Surface Mapping

The reconnaissance results can be represented conceptually as:

```text
                    certifiedhacker.com
                           |
             +-------------+-------------+
             |             |             |
          DNS/NS          MX           Hosts
             |             |             |
     ns1.bluehost.com   mail.*       many subdomains
     ns2.bluehost.com      |             |
             |              |             |
             +--------------+-------------+
                            |
                     162.241.216.11
                            |
              +-------------+-------------+
              |             |             |
             SSH           FTP        HTTP/HTTPS
```

This demonstrates how passive reconnaissance can progressively build an external attack-surface map.

---

# 🔬 Findings Summary

| Category | Finding |
|---|---|
| Target | `certifiedhacker.com` |
| IPv4 | `162.241.216.11` |
| Reverse DNS | `box5331.bluehost.com` |
| ASN | `AS31898` |
| Network | `162.241.216.0/22` |
| Name Server 1 | `ns1.bluehost.com` |
| Name Server 2 | `ns2.bluehost.com` |
| Mail Server | `mail.certifiedhacker.com` |
| DNSSEC | Enabled according to Netcraft |
| SPF | `v=spf1 a mx ptr include:bluehost.com ?all` |
| DMARC | Not reported by Netcraft |
| Discovered Hosts | Multiple subdomains |
| Database-reported services | SSH, FTP, HTTP, HTTPS |

---

# 🧠 Security Analysis

The reconnaissance exercise demonstrates how much information can be obtained before performing an active penetration test.

The most significant observations were:

### 1. Large hostname footprint

Multiple subdomains were discovered, including administrative, mail, development/demo, FTP, SFTP, webmail, and other functional names.

These names can help a security professional understand the organization's external attack surface.

### 2. Shared infrastructure

Many discovered hosts resolve to:

```text
162.241.216.11
```

This suggests that multiple externally visible services may share the same infrastructure.

### 3. Mail infrastructure

The MX record identifies:

```text
mail.certifiedhacker.com
```

as the mail server.

### 4. Service information

DNSDumpster's database associates several hosts with SSH, FTP, HTTP, and HTTPS services.

These should be treated as **leads for authorized validation**, not automatically as confirmed vulnerabilities.

### 5. DNS infrastructure

The discovered nameservers provide information about the DNS architecture.

---

# 🛡️ Defensive Recommendations

From a defensive perspective, organizations should:

- Maintain an accurate inventory of public subdomains.
- Remove unused or forgotten DNS records.
- Review exposed administrative hostnames.
- Avoid exposing unnecessary services to the Internet.
- Regularly review externally reachable infrastructure.
- Maintain appropriate SPF, DKIM and DMARC policies.
- Monitor certificate and DNS changes.
- Ensure old development/demo environments are removed or protected.
- Periodically perform external attack-surface assessments.

---

# ⚠️ Legal and Ethical Considerations

This exercise was performed as a reconnaissance-learning activity using publicly available information.

Passive reconnaissance does not automatically authorize active testing.

Before performing:

```text
Port scanning
Service enumeration
Vulnerability scanning
Authentication testing
Exploitation
Brute-force attacks
```

obtain explicit authorization and define the target scope.

DNSDumpster itself describes domain-based reconnaissance as useful for both penetration testers and defenders when mapping Internet-facing infrastructure.

---

# 🧪 Troubleshooting

### Problem: Netcraft does not show SSL/TLS information

Check whether the report was generated for:

```text
http://target
```

instead of:

```text
https://target
```

Netcraft provides separate HTTPS reporting for TLS information.

### Problem: DNSDumpster returns fewer hosts

DNS datasets change over time. Results can vary based on the current state of DNS and the research provider's data.

### Problem: DNSDumpster reports a service

Do not immediately assume the service is currently open.

Record it as:

```text
Database-reported service
```

and verify it separately only within an authorized scope.

### Problem: Different tools show different IP addresses

DNS records and hosting infrastructure can change, and CDN/proxy infrastructure may return different addresses.

Always record:

```text
Tool
Date
Target
Result
```

when performing reconnaissance.

---

# 📝 Assessment Questions

### Q1. What is passive reconnaissance?

Passive reconnaissance is information gathering performed without directly interacting with the target's systems in a way that constitutes active probing.

### Q2. What is DNS enumeration?

DNS enumeration is the process of identifying DNS records, nameservers, hosts, mail servers, and related infrastructure associated with a domain.

### Q3. What is an ASN?

An Autonomous System Number identifies an autonomous system that participates in Internet routing.

### Q4. What is an MX record?

An MX record identifies the mail server responsible for receiving email for a domain.

### Q5. Why are subdomains important during penetration testing?

Subdomains can expose additional applications, administrative interfaces, development environments, APIs, and other Internet-facing assets.

### Q6. Does finding an open service mean the system is vulnerable?

No. An exposed service is an attack-surface observation. Vulnerability requires further authorized validation.

### Q7. Why should DNSDumpster results not automatically be treated as current?

Because DNSDumpster can report information from its own datasets, and infrastructure can change over time.

---

# 🎯 Completion Criteria

This lab is considered complete when you can:

- [x] Explain passive reconnaissance.
- [x] Use Netcraft Site Report.
- [x] Identify hosting information.
- [x] Identify IP and ASN information.
- [x] Identify DNS infrastructure.
- [x] Analyze TLS information where available.
- [x] Use DNSDumpster.
- [x] Discover subdomains/hosts.
- [x] Identify MX records.
- [x] Identify NS records.
- [x] Identify TXT/SPF records.
- [x] Interpret database-reported services correctly.
- [x] Explain how reconnaissance contributes to attack-surface mapping.
- [x] Document findings without performing unauthorized exploitation.

---

# 📚 References

- Netcraft Research Tools: https://www.netcraft.com/resources/research-tools
- Netcraft Site Report: https://sitereport.netcraft.com/
- DNSDumpster: https://dnsdumpster.com/
- DNSDumpster Footprinting & Reconnaissance: https://dnsdumpster.com/footprinting-and-reconnaissance/

Netcraft describes its Site Report as a tool for researching Internet infrastructure and technologies.

DNSDumpster describes its service as a domain research tool for discovering hosts and DNS information associated with a domain.

---

# ✅ Lab Status

**Status:** Completed

**Lab:** 02 — Internet Research & DNS Reconnaissance

**Target:** `certifiedhacker.com`

**Techniques:** Passive Reconnaissance, DNS Enumeration, Host Discovery, Infrastructure Identification

**Tools:** Netcraft, DNSDumpster

**Result:** Successfully mapped publicly visible DNS, host, network, mail, and infrastructure information.
