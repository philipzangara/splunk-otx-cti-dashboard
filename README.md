# Splunk OTX CTI Dashboard

A Splunk Classic Dashboard that visualizes Cyber Threat Intelligence (CTI) data from AlienVault Open Threat Exchange (OTX). Built as part of the ECA Cyber Range Splunk Security Dashboard Challenge to demonstrate real-time threat intelligence ingestion, environment correlation, and visualization using Splunk.

---

## Overview

This dashboard answers one core question a SOC analyst asks every shift: **is threat intelligence from the outside world actively hitting my environment right now?**

It does this by joining live Sysmon telemetry from your Windows endpoints against malicious indicators and adversary TTPs from AlienVault OTX — surfacing not just what threats exist globally, but which ones your machines are actually talking to.

---

## Features

- **15 dashboard panels** across 8 rows covering the full CTI lifecycle
- **Live threat data** powered by AlienVault OTX via the TA-otx Splunk Add-on
- **Dynamic filters** — time range picker, IOC type dropdown, and threat actor dropdown that affect all panels simultaneously
- **IOC type filter** — selecting IPv4, domain, or hash type filters all relevant IOC panels simultaneously; unrelated panels return no results
- **MITRE ATT&CK TTP tracking** — frequency and trend analysis of adversary techniques across all 14 tactics
- **IOC visibility** — malicious IPs, domains, and file hashes (SHA256) correlated against live Sysmon telemetry
- **Geographic intelligence** — bubble map of malicious IP hits with country and IP-level drill-down tables
- **Environment correlation** — every IOC panel joins OTX indicators against Sysmon events so results reflect actual activity in your environment, not just the global feed
- **Feed health monitoring** — tracks OTX ingestion status and data staleness

---

## Dashboard Panels

| # | Panel | Type | Row | Data Source |
|---|-------|------|-----|-------------|
| 1 | Unique Hosts with IOC Hits (Last 24h) | Single Value KPI | 1 | Sysmon + OTX |
| 2 | Total IOC Hits (Last 24h) | Single Value KPI | 1 | Sysmon + OTX |
| 3 | Critical TTPs in Environment | Single Value KPI | 1 | Sysmon + OTX |
| 4 | Malicious IPs Detected (Last 24h) | Single Value KPI | 1 | Sysmon + OTX |
| 5 | Unique Source Countries (Last 24h) | Single Value KPI | 1 | Sysmon + OTX |
| 6 | OTX Threat Intel Hits - TTPs Active in Your Environment | Table | 2 | Sysmon + OTX |
| 7 | Malicious IP Hits by Location | Bubble Map | 3 | Sysmon + OTX |
| 8 | Top Countries by Hit Count | Table | 4 | Sysmon + OTX |
| 9 | Top Malicious IPs | Table | 4 | Sysmon + OTX |
| 10 | OTX Malicious Domain Hits - Your Environment (Last 7 Days) | Table | 5 | Sysmon + OTX |
| 11 | OTX Malicious File Hits - Your Environment | Table | 5 | Sysmon + OTX |
| 12 | Top MITRE ATT&CK Techniques (TTPs) | Bar Chart | 6 | OTX |
| 13 | New TTPs Observed This Week | Table | 6 | OTX |
| 14 | MITRE ATT&CK Framework - Active TTPs (Last 7 Days) | Table | 7 | OTX |
| 15 | OTX Feed Health - Last Ingest Times | Table | 8 | OTX |

---

## Security Questions Answered

| Panel | Security Question |
|-------|------------------|
| Unique Hosts with IOC Hits | How many of my machines had confirmed malicious indicator contact in the last 24 hours? |
| Total IOC Hits | How many distinct malicious indicators were seen across my environment today? |
| Critical TTPs in Environment | How many adversary techniques seen in OTX pulses are firing at high volume in my Sysmon logs? |
| Malicious IPs Detected | How many OTX-flagged IPs did my environment connect to in the last 24 hours? |
| Unique Source Countries | How many countries are malicious connections originating from? |
| TTPs Active in Environment | Which specific MITRE techniques are both in OTX intelligence and actively observed in my Sysmon data? |
| Malicious IP Hits by Location | Where geographically are the malicious IPs hitting my environment located? |
| Top Countries by Hit Count | Which countries are responsible for the most malicious connection attempts? |
| Top Malicious IPs | Which specific IPs are hitting my environment most frequently and who do they belong to? |
| Malicious Domain Hits | Which OTX-flagged domains are my machines resolving via DNS? |
| Malicious File Hits | Which OTX-flagged file hashes have been executed, loaded, or created on my endpoints? |
| Top MITRE ATT&CK Techniques | Which adversary techniques appear most frequently across all OTX pulses? |
| New TTPs This Week | What new adversary techniques have appeared in OTX intelligence this week? |
| MITRE ATT&CK Framework | Which of the 14 ATT&CK tactic categories have active techniques in OTX right now? |
| Feed Health | Is OTX data flowing into Splunk and how fresh is it? |

---

## Requirements

- Splunk Enterprise 9.1 or higher
- [TA-otx — Add-on for Open Threat Exchange](https://splunkbase.splunk.com/app/4336) by Luke Monahan
- AlienVault OTX account and API key — sign up free at [otx.alienvault.com](https://otx.alienvault.com)
- An index named `otx` created in Splunk
- A Sysmon-instrumented Windows host forwarding to Splunk (index: `sysmon`)

---

## Installation

### 1. Create the OTX index in Splunk
```
Settings → Indexes → New Index
Index Name: otx
Index Type: Events
```

### 2. Install the TA-otx Add-on
- Download from [Splunkbase](https://splunkbase.splunk.com/app/4336)
- Install via Apps → Manage Apps → Install from file

### 3. Configure the OTX input
- Go to Apps → TA-OTX → Configuration
- Enter your OTX API key under the Account tab
- Save proxy settings (even if disabled) to avoid credential errors
- Go to Settings → Data Inputs → OTX → Enable OTX_Feed
- Set index to `otx` and interval to `3600` (1 hour)

### 4. Import the dashboard
- Go to Settings → User Interface → Views → Create New View
- Select Dashboard (Classic) and paste the contents of `cti_otx_dashboard.xml`
- Or navigate to any existing dashboard → Edit → Source and replace with the XML

### 5. Verify data is flowing
Run this search after enabling the input and waiting a few minutes:
```
index=otx | stats count by sourcetype
```
You should see `otx:indicator` and `otx:pulse` with non-zero counts.

---

## Dynamic Filters

| Filter | Description |
|--------|-------------|
| Time Range | Controls the time window for all panels |
| IOC Type | Filters all IOC panels by type (IPv4, domain, URL, hash) — selecting a type hides unrelated IOC panels |
| Threat Actor | Filters pulse and TTP panels by attributed adversary |

---

## Sysmon Event Codes Used

| Event Code | Description | IOC Type Matched |
|------------|-------------|------------------|
| 1 | Process Create | SHA256 file hash + TTP (T1059) |
| 3 | Network Connection | Destination IP + TTP (T1071) |
| 7 | Image Loaded | SHA256 file hash |
| 8 | CreateRemoteThread | TTP (T1055) |
| 11 | File Created | SHA256 file hash + TTP (T1027) |
| 13 | Registry Value Set | TTP (T1112) |
| 22 | DNS Query | Domain + TTP (T1071) |

---

## Known Issues

- Fields with `{}` notation (e.g. `attack_ids{}`) may show a yellow warning icon or a `[subsearch]: Field does not exist` error — this is cosmetic and does not affect results. Splunk's schema validator does not recognize the `{}` multi-value field notation used by TA-otx, even though the fields are correctly populated at search time
- The Threat Actor dropdown will only populate once OTX pulses with attributed adversaries have been ingested
- The Domain Hits panel uses a fixed 7-day window and the Feed Health panel uses a fixed 24-hour window, regardless of the time range picker setting
- The IOC Type filter does not affect the TTP panels — TTP correlation is based on Sysmon event codes and OTX pulse data, not OTX indicator types
- The map requires outbound HTTPS to `basemaps.cartocdn.com` for the dark tile layer. If your environment is air-gapped, remove the `mapping.tileLayer.url` and `mapping.tileLayer.attribution` options to fall back to Splunk's default tiles
- `iplocation` uses Splunk's bundled MaxMind GeoLite2 database — private/RFC1918 addresses and some hosting provider ranges may not resolve and will be excluded from the map and geo tables

---

## Technical Notes

- OTX data is stored in two sourcetypes: `otx:pulse` (pulse metadata) and `otx:indicator` (individual IOCs)
- Multi-value fields from the OTX API are stored with `{}` suffix notation (e.g. `attack_ids{}`)
- The TA-otx add-on requires OpenSSL 1.0 — on Ubuntu 24.04 install via:

```bash
wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.0.0_1.0.2n-1ubuntu5_amd64.deb
sudo dpkg -i libssl1.0.0_1.0.2n-1ubuntu5_amd64.deb
```

- If running Splunk on a VM, ensure the CPU type supports AVX2 (use `x86-64-v3` or higher in Proxmox)

---

## Production Considerations

This project uses the community-built **TA-otx** add-on which polls the OTX API on a scheduled interval. While functional for a lab or portfolio environment, a production CTI deployment would differ in several important ways.

### Real-time indicator ingestion

The TA-otx add-on uses a checkpoint-based polling mechanism — it tracks the last poll timestamp and requests pulses modified since that time. New indicators can take up to an hour to appear depending on the configured interval, and the checkpoint can become stale after restarts or clock drift.

In a production environment this would be replaced with:

- **Splunk Enterprise Security (ES)** — includes a native Threat Intelligence framework with a dedicated REST API that accepts indicators in real time, no polling required
- **TAXII 2.1** — the industry standard protocol for threat intel sharing. OTX supports TAXII and Splunk ES has a built-in TAXII client that receives indicator pushes the moment they are published
- **STIX 2.1** — the structured data format used with TAXII, providing richer context including relationships between indicators, threat actors, and campaigns

### What this would look like in production

```
OTX publishes new pulse
        ↓
TAXII 2.1 push (seconds)
        ↓
Splunk ES Threat Intelligence framework
        ↓
Automatic correlation against all indexes
        ↓
Real-time alerts and dashboard updates
```

### Other production improvements

- Automated indicator expiration and lifecycle management
- Multi-feed deduplication and confidence scoring
- Integration with SOAR platforms (Splunk SOAR, Palo Alto XSOAR) for automated response
- Role-based access control for sensitive threat intel

### MITRE ATT&CK tactic mapping

The MITRE ATT&CK Framework panel uses a hardcoded `case` statement to map technique IDs to their corresponding tactics across all 14 tactic categories. This is correct behavior — the mapping is defined by the MITRE ATT&CK framework itself and is not environment-specific. However it requires manual updates when MITRE releases new framework versions.

In a production deployment this would be replaced with a maintained lookup table:

```spl
| lookup mitre_attack_lookup technique_id OUTPUT tactic
```

Splunk Enterprise Security ships with a `mitre_attack_lookup` that is automatically updated with each ES release. The community add-on [MITRE ATT&CK App for Splunk](https://splunkbase.splunk.com/app/4617) also provides this lookup for non-ES deployments.

---

## Test Data

The Sysmon environment correlation panels were validated using [Atomic Red Team](https://github.com/redcanaryco/atomic-red-team) by Red Canary. Atomic tests were executed via the [Invoke-AtomicRedTeam](https://github.com/redcanaryco/invoke-atomicredteam) PowerShell framework to simulate adversary techniques mapped to MITRE ATT&CK — generating realistic Sysmon telemetry (process creation, network connections, DNS queries, file activity) that matches the IOC and TTP patterns ingested from OTX. This allowed the dashboard joins, correlation logic, and severity thresholds to be tested against representative malicious activity rather than a clean environment.

---

## Acknowledgements

- [AlienVault OTX](https://otx.alienvault.com) for the threat intelligence platform and API
- [Luke Monahan](https://splunkbase.splunk.com/app/4336) for the TA-otx Splunk Add-on
- [MITRE ATT&CK](https://attack.mitre.org) for the adversary technique framework
- [Red Canary](https://redcanary.com) for Atomic Red Team adversary simulation tests
- [ECA Cyber Range]([https://ellingtoncyber.com](https://www.skool.com/eca-cyber-range-4625/about?ref=176bfcb7a0794bdfbd3403e5ed04ac73)) for the Splunk Security Dashboard Challenge

---

MIT License — see [LICENSE](LICENSE) for details.
