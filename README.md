# Pentest Lab: Footprinting & Network Scanning (Phases 1-2)

Week 2 practical for the Networkwalks Cybersecurity Internship (Batch B082). Phase 1 is footprinting and reconnaissance of `networkwalks.com` with six Kali Linux tools. Phase 2 is a Zenmap ping scan of my own local network. The full write-up is in the report (see Files).

## Scope and authorization

- **Targets:** `networkwalks.com` (public website and its DNS records) and my own local LAN. Nothing else was tested.
- **Authorization:** Written Letter of Authorization from Networkwalks, the domain owner (Ref. NW-LOA-B082-017, issued 17 August 2026)
- **Authorization period:** 17 to 24 August 2026. [add renewal reference once confirmed, since testing was performed after this period]
- **Dates performed:** 10-11 November 2026
- **Allowed and done:** passive and light information gathering, and a host-discovery ping scan of my own subnet
- **Not done:** exploitation, brute forcing, denial of service, or testing of any system outside the scope
- **Impact:** low-volume, read-only queries. Nothing on the target was modified.

Only run these tools against systems you own or have written permission to test.

## Tools

| Tool | Command | Purpose |
|------|---------|---------|
| WHOIS | `whois networkwalks.com` | Domain registration details |
| WhatWeb | `whatweb networkwalks.com` | Fingerprint web technologies |
| nslookup | `nslookup networkwalks.com` | Resolve the domain to an IP |
| curl | `curl -I https://networkwalks.com` | Read HTTP response headers |
| wafw00f | `wafw00f networkwalks.com` | Detect a web application firewall |
| dnsrecon | `dnsrecon -d networkwalks.com` | Enumerate DNS records |
| Zenmap (Nmap) | `nmap -sn 192.168.100.0/24` | Discover live hosts on my LAN |

## Phase 1: Footprinting `networkwalks.com`

| Tool | Finding |
|------|---------|
| WHOIS | Registrar GoDaddy.com, LLC. Created 2019-11-06, expires 2027-11-06. Name servers on HostGator. DNSSEC unsigned. Registrar transfer/update/delete locks set. |
| WhatWeb | Apache server running WordPress 7.1.4 with WordPress Download Manager 3.3.76. Site title "Networkwalks Academy". |
| nslookup | Resolves to `192.232.216.135` (non-authoritative answer). |
| curl -I | `HTTP/2 200` from Apache. Exposes the WordPress REST API link `/wp-json/` and a `__wpdm_client` cookie marked Secure and HttpOnly. |
| wafw00f | Site is behind ModSecurity (SpiderLabs) WAF. |
| dnsrecon | 8 records: SOA and NS (HostGator), MX `mail.networkwalks.com`, A (`192.232.216.135`), and SRV `_autodiscover._tcp` records for cPanel email discovery. DNSSEC query returned no answer. No SPF/TXT records were returned. |

## Phase 2: Zenmap network scan (own LAN)

- **Target:** `192.168.100.0/24`, Ping Scan (`nmap -sn`)
- **Result:** 256 addresses scanned in 2.46 seconds, 2 hosts up: the gateway at `192.168.100.1` and my own machine at `192.168.100.130`
- A network topology graphic was generated and saved as a PDF.

## Risk summary

These are observations, not confirmed vulnerabilities. No exploitation or vulnerability validation was performed.

| Finding | Level |
|---------|-------|
| Web technology and plugin versions visible (WhatWeb) | Medium |
| DNS infrastructure details exposed (dnsrecon) | Medium |
| Live hosts visible on local network (Zenmap) | Medium |
| Server IP identifiable (nslookup) | Low |
| HTTP headers and `/wp-json/` exposed (curl) | Low |
| WAF identifiable (wafw00f) | Low |

## Recommendations

1. Review what technology and version information is publicly visible.
2. Keep the CMS and plugins updated against current security advisories.
3. Review HTTP headers for unnecessary technical detail.
4. Review DNS records regularly and consider enabling DNSSEC.
5. Keep ModSecurity enabled and tuned.
6. Scan internal networks periodically and investigate any unknown device.
7. Document network topology and keep it current.

## Files

- `W2-PM-FINAL - Pentest Report.docx`: the full report
- `WHOIS.png`, `WHOIS RESULTS.txt`
- `WHATWEB.png`, `WHATWEB RESULT.txt`
- `NSLOOKUP.png`, `NSLOOKUP RESULT.txt`
- `curl -I RESULTS.png`, `curl I https networkwalks RESULT.txt`
- `wafw00f_Result.png`, `wafw00f Result.txt`
- `dnsrecon -d RESULT.com.png`
- `Zenmap_subnet_scan.png`, `Zenmap_subnet_scan_screenshot.png`, `zenmap_output topology.pdf`

## What I learned

Public records alone (WHOIS, DNS, HTTP headers, page fingerprints) reveal a site's hosting provider, CMS, plugins, mail setup, and firewall. Each tool shows a different layer, and they cross-check each other. Reconnaissance must always stay inside an authorized scope.
