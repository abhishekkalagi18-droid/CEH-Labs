# Lab-05: DNS Footprinting

## 1. Objective

To perform DNS footprinting using Kali Linux and identify DNS records associated with a domain. The results help understand domain resolution, mail configuration, name servers, and reverse DNS information.

## 2. Lab Environment

- **Operating System:** Kali Linux
- **Tool:** `nslookup`, `dig`
- **DNS resolver:** `1.1.1.1` (Cloudflare Public DNS)
- **Target domain:** `certifiedhacker.com`
- **Target IPv4 address:** `162.241.216.11`

> All results documented here are based on observations collected during this lab. DNS records may change over time.

## 3. Practical Tasks and Results

### Task 1: Find the IPv4 Address (A Record)

**Command:** Used interactive `nslookup` with `set type=a`.

**Observed result:**
```text
Name: certifiedhacker.com
Address: 162.241.216.11
```

**Analysis:** The A record maps the domain to an IPv4 address. The response was non-authoritative, meaning it was returned by the configured resolver rather than directly from the domain's authoritative name server.

### Task 2: Identify Name Servers (NS Record)

**Method:** Used `nslookup` with `set type=ns`.

**Observed result:** `ns1.bluehost.com` and `ns2.bluehost.com`.

**Analysis:** NS records identify the name servers responsible for hosting the domain's DNS zone.

### Task 3: Identify Mail Servers (MX Record)

**Method:** Used `nslookup` with `set type=mx`.

**Observed result:** Priority `0`, mail server `mail.certifiedhacker.com`.

**Analysis:** MX records identify the mail servers designated to receive email for a domain. The record alone does not prove that the mail server is reachable.

### Task 4: Inspect the TXT Record (SPF)

**Method:** Used `nslookup` with `set type=txt`.

**Observed result:**
```text
"v=spf1 a mx ptr include:bluehost.com ?all"
```

**Analysis:**
- `v=spf1`: identifies the SPF version.
- `a`: authorizes qualifying addresses from the domain's A record.
- `mx`: authorizes qualifying mail-server addresses.
- `ptr`: uses reverse-DNS-based matching; this mechanism is discouraged in modern SPF deployments.
- `include:bluehost.com`: incorporates the SPF policy of the included domain.
- `?all`: applies a neutral result to senders not matched by earlier mechanisms.

SPF is one part of email authentication. DKIM and DMARC should also be assessed.

### Task 5: Inspect the SOA Record

**Method:** Used `nslookup` with `set type=soa`.

**Observed result:**
```text
origin = ns1.bluehost.com
mail addr = dnsadmin.box5331.bluehost.com
serial = 2026090818
refresh = 86400
retry = 7200
expire = 3600000
minimum = 300
```

**Analysis:**
- **Primary name server:** `ns1.bluehost.com`
- **Responsible-party field:** `dnsadmin.box5331.bluehost.com` (SOA mailbox notation)
- **Serial:** `2026090818`, the zone version identifier
- **Refresh:** 86,400 seconds (24 hours)
- **Retry:** 7,200 seconds (2 hours)
- **Expire:** 3,600,000 seconds (approximately 41 days and 16 hours)
- **Minimum:** 300 seconds (5 minutes), commonly used as the negative-caching TTL in modern DNS

These values describe DNS zone administration; they do not independently establish a vulnerability.

### Task 6: Check the IPv6 Address (AAAA Record)

**Method:** Used `nslookup` with `set type=aaaa`.

**Observed result:**
```text
*** Can't find certifiedhacker.com: No answer
```

**Analysis:** The resolver returned no AAAA answer for the domain. This does not mean the domain is unavailable; its A record returned an IPv4 address.

### Task 7: Check for a CNAME Record

**Command:**
```bash
dig @1.1.1.1 certifiedhacker.com CNAME +noall +answer
```

**Observed result:** No output.

**Analysis:** No CNAME answer was returned for the root domain. This is consistent with the domain having an A record. It does not rule out CNAME records on individual subdomains.

### Task 8: Perform Reverse DNS Lookup (PTR)

**Command:**
```bash
dig @1.1.1.1 -x 162.241.216.11 +all +answer
```

**Observed result:**
```text
;; ->>HEADER<<- opcode: QUERY, status: NOERROR
;; flags: qr rd ra; QUERY: 1, ANSWER: 1

;; ANSWER SECTION:
11.216.241.162.in-addr.arpa. 21600 IN PTR box5331.bluehost.com.

;; AUTHORITY SECTION:
162.in-addr.arpa. 70914 IN NS y.arin.net.
162.in-addr.arpa. 70914 IN NS u.arin.net.
162.in-addr.arpa. 70914 IN NS z.arin.net.
162.in-addr.arpa. 70914 IN NS arin.authdns.ripe.net.
162.in-addr.arpa. 70914 IN NS x.arin.net.
162.in-addr.arpa. 70914 IN NS r.arin.net.
```

**Analysis:**
- `NOERROR` indicates the DNS query completed successfully.
- The PTR record maps the IP address to `box5331.bluehost.com`.
- The TTL is 21,600 seconds (6 hours).
- The reverse-DNS authority section lists name servers for the relevant reverse-DNS zone.

The PTR result is consistent with the hosting information observed in earlier footprinting tasks. It does not prove that every domain using this infrastructure has the same owner.

## 4. Security Relevance

DNS footprinting helps security professionals:
- Map domain-to-IP relationships.
- Identify authoritative name servers and mail infrastructure.
- Review publicly exposed email-authentication policies.
- Correlate reverse DNS with hosting information.
- Build an initial inventory for an authorized security assessment.

DNS findings should be validated before being used to make security conclusions.

## 5. Limitations

- Results reflect the resolver responses observed during this lab.
- No AAAA or CNAME answer was returned for the root domain in the tested queries.
- This lab did not establish whether the mail server accepts connections or whether the domain has exploitable vulnerabilities.
- The commands documented here were performed; no unexecuted commands are presented as completed.

## 6. Completion Criteria

- [x] A record queried
- [x] NS record identified
- [x] MX record identified
- [x] SPF TXT record analyzed
- [x] SOA record analyzed
- [x] AAAA lookup performed
- [x] CNAME lookup performed
- [x] PTR lookup performed

## 7. References

- [ISC `dig` documentation](https://bind9.readthedocs.io/)
- [RFC 1035 — Domain Names: Implementation and Specification](https://www.rfc-editor.org/rfc/rfc1035)
- [RFC 7208 — Sender Policy Framework (SPF)](https://www.rfc-editor.org/rfc/rfc7208)
- [RFC 2308 — Negative Caching of DNS Queries](https://www.rfc-editor.org/rfc/rfc2308)
