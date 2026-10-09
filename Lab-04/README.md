# Lab-04 — WHOIS Footprinting and DNS Verification

## 1. Problem Statement

During security reconnaissance, a penetration tester needs to collect publicly available domain registration and DNS information to understand a target's external footprint. This information can reveal the domain registrar, registration dates, nameservers, DNS records, hosting-related hostnames, and domain configuration details.

This lab demonstrates how to perform WHOIS footprinting and verify the collected information using DNS queries and publicly available lookup services.

## 2. Objective

- Identify domain registration information using WHOIS.
- Determine the registrar, creation date, expiry date, and domain status.
- Identify authoritative nameservers.
- Resolve a domain to its IPv4 address.
- Perform reverse DNS lookup on the resolved IP.
- Compare information from command-line tools and online reconnaissance services.
- Document findings accurately without assuming that public information proves ownership or vulnerability.

## 3. Lab Environment

| Component | Details |
|---|---|
| Operating system | Kali Linux |
| Domain used | `certifiedhacker.com` |
| WHOIS tool | `whois` |
| DNS utility | `dig` |
| DNS resolver | Cloudflare `1.1.1.1` |
| Online lookup service | DomainTools WHOIS |
| Activity type | Passive reconnaissance |

**Scope:** This lab uses publicly accessible domain-registration and DNS information. It does not attempt to exploit the domain or access private systems.

## 4. Theory

### 4.1 What is WHOIS?

WHOIS is a protocol and lookup mechanism used to retrieve domain-registration information from registration databases. Depending on the registry and registrar, the returned information may include:

- Domain creation, update, and expiry dates.
- Registrar name and identifier.
- Domain status codes.
- Nameserver details.
- Registrant information, which may be redacted or replaced by a privacy service.
- DNSSEC status.

WHOIS data is useful during reconnaissance because it provides administrative information about a domain before active security testing begins.

### 4.2 What is DNS?

The Domain Name System (DNS) translates domain names into records that applications and computers can use.

Common record types include:

- **A:** Maps a domain name to an IPv4 address.
- **AAAA:** Maps a domain name to an IPv6 address.
- **NS:** Identifies authoritative nameservers.
- **MX:** Identifies mail servers for a domain.
- **PTR:** Maps an IP address to a hostname through reverse DNS.
- **TXT:** Stores text information, often used for email policies and domain verification.

### 4.3 What is reverse DNS?

Reverse DNS uses a PTR record to look up a hostname associated with an IP address. It can provide a useful hosting clue, but it does not prove that the hostname and IP are exclusively associated with the target domain.

### 4.4 Why compare multiple sources?

WHOIS websites, registrar records, and DNS resolvers can return different fields or timestamps. Comparing sources helps identify consistent information and distinguish registry updates from registrar updates.

## 5. Practical Implementation

### Step 1: Collect WHOIS information

**Command:**

```bash
whois certifiedhacker.com
```

**Purpose:** Retrieves publicly available domain-registration information.

**Important fields to inspect:**

- `Registrar`
- `Creation Date`
- `Registry Expiry Date`
- `Updated Date`
- `Domain Status`
- `Name Server`
- `DNSSEC`

**Observed results:**

| Field | Actual result |
|---|---|
| Registrar | Network Solutions, LLC |
| IANA registrar ID | 2 |
| Creation date | 2002-07-30T00:32:00Z |
| Registry expiry date | 2027-07-30T00:32:00Z |
| Registry update date | 2026-05-30T06:36:42Z |
| Registrar-record update date | 2026-08-22T06:43:08Z |
| Domain status | `clientTransferProhibited` |
| Nameserver 1 | `ns1.bluehost.com` |
| Nameserver 2 | `ns2.bluehost.com` |
| DNSSEC | Unsigned |

**Analysis:**

The output identifies Network Solutions, LLC as the registrar and provides the domain's registration timeline. The registrant information was associated with Perfect Privacy LLC, indicating that a privacy service was used in the returned record.

The two update dates are not necessarily contradictory: the registry record and registrar record can have separate update timestamps.

The status `clientTransferProhibited` is a transfer-lock status. By itself, it is not evidence of a security vulnerability.

### Step 2: Cross-check WHOIS using DomainTools

**Website:** https://whois.domaintools.com/certifiedhacker.com

**Procedure:**

1. Open the DomainTools WHOIS lookup page.
2. Search for `certifiedhacker.com`.
3. Review the registrar, dates, nameservers, registrant, IP address, and hosting-related information.
4. Compare the findings with the Kali WHOIS output.

**Observed results:**

| Field | DomainTools result |
|---|---|
| Registrar | Network Solutions, LLC |
| Creation date | 2002-07-30 |
| Expiry date | 2027-07-30 |
| Nameservers | `NS1.BLUEHOST.COM`, `NS2.BLUEHOST.COM` |
| IP address | `162.241.216.11` |
| ASN reported | AS31898 |
| Registrant | Perfect Privacy LLC |
| DNSSEC | Unsigned |
| Other sites reported on IP | 706 |

**Analysis:**

The registrar, registration dates, nameservers, IP address, and privacy-related registrant information provide useful cross-reference points.

DomainTools reported 706 other sites on the same server. This should be treated as a service-reported observation, not independent proof that all listed sites were hosted there at the exact time of this lab. Shared hosting also means that an IP address may serve multiple unrelated domains.

### Step 3: Verify authoritative nameservers

**Command:**

```bash
dig @1.1.1.1 certifiedhacker.com NS +noall +answer
```

**Command explanation:**

- `dig`: Performs DNS queries.
- `@1.1.1.1`: Sends the query to Cloudflare's public DNS resolver.
- `certifiedhacker.com`: Specifies the target domain.
- `NS`: Requests nameserver records.
- `+noall +answer`: Displays only the answer section.

**Actual output:**

```text
certifiedhacker.com. 21600 IN NS ns1.bluehost.com.
certifiedhacker.com. 21600 IN NS ns2.bluehost.com.
```

**Analysis:**

The DNS response returned `ns1.bluehost.com` and `ns2.bluehost.com`, matching the nameservers identified in WHOIS and DomainTools.

The TTL is `21600` seconds, equivalent to six hours.

**Troubleshooting note:** A query using the default resolver initially timed out against `192.168.127.2` and `192.168.181.1`. Specifying `@1.1.1.1` succeeded. This confirms that the explicit resolver worked for these queries; it does not establish the precise cause of the default-resolver failure.

### Step 4: Resolve the domain to an IPv4 address

**Command:**

```bash
dig @1.1.1.1 certifiedhacker.com A +noall +answer
```

**Actual output:**

```text
certifiedhacker.com. 14400 IN A 162.241.216.11
```

**Analysis:**

The A record returned the IPv4 address `162.241.216.11`, matching the address reported by DomainTools.

The TTL is `14400` seconds, equivalent to four hours. DNS answers can change, so this result describes the response observed during the lab rather than a permanent mapping.

### Step 5: Perform reverse DNS lookup

**Command:**

```bash
dig @1.1.1.1 -x 162.241.216.11 +noall +answer
```

**Actual output:**

```text
11.216.241.162.in-addr.arpa. 21600 IN PTR box5331.bluehost.com.
```

**Analysis:**

The PTR record maps `162.241.216.11` to `box5331.bluehost.com`.

This is consistent with the hosting-related hostname reported by DomainTools and DNSDumpster. Reverse DNS is a supporting clue; it does not establish exclusive ownership of the IP or hostname.

## 6. Consolidated Findings

| Item | Finding | Verification |
|---|---|---|
| Domain | `certifiedhacker.com` | All lookup methods |
| Registrar | Network Solutions, LLC | WHOIS and DomainTools |
| Creation date | July 30, 2002 | WHOIS and DomainTools |
| Expiry date | July 30, 2027 | WHOIS and DomainTools |
| Nameservers | `ns1.bluehost.com`, `ns2.bluehost.com` | WHOIS, DomainTools, DNS |
| IPv4 address | `162.241.216.11` | DomainTools and A record |
| Reverse DNS | `box5331.bluehost.com` | PTR record |
| Domain status | `clientTransferProhibited` | WHOIS |
| DNSSEC | Unsigned | WHOIS and DomainTools |
| Registrant privacy | Perfect Privacy LLC | WHOIS and DomainTools |

## 7. Security Relevance

WHOIS and DNS footprinting can help a penetration tester:

1. Establish an initial picture of a target's public domain infrastructure.
2. Identify authoritative nameservers and current DNS mappings.
3. Discover hosting-related information that may support further authorized reconnaissance.
4. Cross-check administrative information across independent sources.
5. Record registration and DNS details as a baseline for later assessment.

**Important limitation:** These findings do not demonstrate an exploitable vulnerability. Any subsequent scanning or testing must be authorized and restricted to the approved scope.

## 8. Troubleshooting

### Issue 1: WHOIS command is unavailable

Check whether the tool is installed:

```bash
which whois
```

If it is missing, install it on Kali Linux:

```bash
sudo apt update
sudo apt install whois
```

### Issue 2: DNS queries time out

Test basic connectivity:

```bash
ping -c 4 1.1.1.1
```

Then try an explicit DNS resolver:

```bash
dig @1.1.1.1 certifiedhacker.com NS +noall +answer
```

If the explicit query works but the default query fails, investigate the configured DNS resolvers and network settings rather than assuming that the domain is unavailable.

### Issue 3: DNS output is empty

An empty answer section does not always mean that the command failed. Check the requested record type, DNS response status, and whether that record exists.

### Issue 4: WHOIS websites and command-line output differ

Compare the field names and timestamps carefully. Registry-level and registrar-level records may have different update times, and lookup services may cache data or display additional enrichment.

## 9. Real-World Scenario

A penetration tester is assigned an authorized external assessment of an organization's domain. Before active testing, the tester collects WHOIS data, identifies authoritative nameservers, resolves the domain to an IP address, and checks reverse DNS.

The tester then records which findings agree across sources and flags discrepancies for verification. This provides an evidence-based starting point for the approved assessment without assuming that registration or hosting information alone reveals a vulnerability.

## 10. Assessment Questions

1. What information can WHOIS reveal about a domain?
2. What is the difference between an A record and an NS record?
3. What is the purpose of a PTR record?
4. Why might registry and registrar update timestamps differ?
5. What does `clientTransferProhibited` indicate?
6. Why should a penetration tester compare information from multiple sources?
7. Does a shared hosting IP prove that every associated domain belongs to the same organization?
8. Why is an unsigned DNSSEC field not, by itself, proof that a domain is exploitable?
9. What should you investigate when a DNS query works through `1.1.1.1` but fails through the default resolver?
10. Why must reconnaissance findings be verified before drawing security conclusions?

## 11. Completion Criteria

- [x] Collected WHOIS information using Kali Linux.
- [x] Cross-checked domain details using DomainTools.
- [x] Retrieved nameserver records using `dig`.
- [x] Retrieved the IPv4 A record.
- [x] Retrieved the reverse DNS PTR record.
- [x] Compared key fields across sources.
- [x] Documented actual outputs and their limitations.
- [x] Identified a default DNS resolver issue without changing the lab network configuration.

## 12. Tools and References

- Kali Linux: https://www.kali.org/
- WHOIS lookup — DomainTools: https://whois.domaintools.com/
- DNS lookup — Google Admin Toolbox: https://toolbox.googleapps.com/apps/dig/
- Cloudflare public DNS: https://1.1.1.1/
- ICANN Lookup: https://lookup.icann.org/
- `dig` manual: https://bind9.readthedocs.io/
- OWASP Web Security Testing Guide — Information Gathering: https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/01-Information_Gathering/00-Information_Gathering_Overview/

## 13. Conclusion

This lab demonstrated WHOIS footprinting and DNS verification using Kali Linux and publicly accessible lookup services. The collected evidence identified the registrar, registration dates, domain status, nameservers, IPv4 address, and reverse DNS hostname.

Comparing WHOIS, DomainTools, and DNS responses helped validate several findings and highlighted the importance of distinguishing observed results from assumptions. These techniques form a practical foundation for passive reconnaissance during an authorized penetration test.
