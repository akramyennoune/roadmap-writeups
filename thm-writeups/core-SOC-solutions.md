# TryHackMe: Elastic Stack: The Basics — Write-Up

**Room:** [Elastic Stack: The Basics](https://tryhackme.com/room/investigatingwithelk101)
**Status:** Completed (100%)
**Category:** SOC Level 1 — SIEM / Log Analysis
**Tools:** Kibana (Discover), Elasticsearch, KQL (Kibana Query Language)

## Overview

This room covers how SOC analysts use the Elastic Stack (ELK) — Elasticsearch, Logstash, and Kibana — to search, filter, and investigate log data. The practical focus was Kibana's **Discover** view, using an index of VPN connection logs (`vpn_connections`) to practice narrowing large datasets down to specific indicators using filters, KQL queries, and field statistics.

## Objectives

- Navigate the Kibana Discover interface and understand its layout (index pattern, search bar, time range, histogram, document table).
- Use the time range picker to scope an investigation to a relevant window.
- Apply filters and KQL queries to isolate events of interest.
- Use field "Top 5 values" statistics to spot patterns (frequent IPs, usernames, locations).
- Pivot from a single suspicious value into a focused investigation.

## Walkthrough

### 1. Exploring the `vpn_connections` index

The dataset used throughout the room is a `vpn_connections` index containing VPN log fields such as `action`, `Company`, `EventTime`, `port`, `protocol`, `Source_Country`, `Source_ip`, `source_state`, and `UserName`.

<img width="1355" height="640" alt="elk1" src="https://github.com/user-attachments/assets/f0da1ea6-8d76-4a8b-a2ca-b9c9e95f8f93" />


With the default view, the index returned **1,280 hits** over the selected window (Jan 17 – Feb 1, 2022), with each document showing a VPN action (e.g. `teardown`), the source IP/country/state, the connecting user, and the port/protocol used (TCP/443).

### 2. Adjusting the time range

Widening the time range picker to **Dec 31, 2021 → Feb 2, 2022** expands the scope of the search to capture the full month of log activity relevant to the investigation.

<img width="443" height="29" alt="elk2" src="https://github.com/user-attachments/assets/85f71cd2-a4e6-4841-9a6b-21eb68442da4" />


With this range applied, the index returned **2,861 hits** — the full pool of VPN connection events available for analysis.

### 3. Identifying frequent values with field statistics

Kibana's field statistics panel makes it easy to spot which values dominate a field without manually scrolling through records. Checking **Source_ip** across a 500-record sample surfaced the top offenders:

| Source IP | % of records |
|---|---|
| 238.163.231.224 | 3.2% |
| 69.208.133.98 | 2.8% |
| 66.125.69.78 | 2.8% |
| 64.171.101.56 | 2.6% |
| 107.14.4.82 | 2.6% |


![Top 5 Source_ip values](images/elk4.PNG)<img width="236" height="196" alt="elk4" src="https://github.com/user-attachments/assets/ec88e221-6cb0-41fd-8c65-2a4c8d98b805" />


The same panel was used on **UserName**, showing which accounts generated the most VPN activity:


| Username | % of records |
|---|---|
| James | 4.0% |
| Paul King | 2.8% |
| Katie Green | 2.8% |
| Kate Wistle | 2.8% |
| Emanda | 2.6% |


![Top 5 UserName values](images/elk5.PNG)<img width="249" height="223" alt="elk5" src="https://github.com/user-attachments/assets/d74102f9-a20c-43f6-9c10-afb63353e3cd" />


These "Top 5 values" breakdowns are a quick way to triage a large dataset — an IP or user that appears disproportionately often is a natural starting point for deeper investigation.

### 4. Narrowing to a single suspicious IP

Filtering down further on a specific `Source_ip` (107.14.x range) showed the address splitting activity almost evenly between two related IPs:

<img width="237" height="189" alt="elk6" src="https://github.com/user-attachments/assets/ca46ab4f-4301-4586-9632-b8d4499f62f7" />


| Source IP | % of records |
|---|---|
| 107.14.1.247 | 53.6% |
| 107.14.4.82 | 46.4% |

### 5. Investigating a failed-login pattern

Applying a filter for `Source_ip: 172.201.60.191` and narrowing the time range to a single event window (Jan 11, 2022, 02:29:27) revealed **52 hits**, all logged with `action: failed`, over TCP/443, originating from **Alberta, Canada**, under the username **Simon**.

<img width="1081" height="518" alt="elk7" src="https://github.com/user-attachments/assets/91c5938b-3d55-4073-8b02-2cedc6bf1ff1" />


52 failed connection attempts from a single IP within the same second is consistent with an automated/brute-force login attempt rather than normal user behavior — a strong indicator worth escalating in a real investigation.

### 6. Spotting a geographic anomaly

Filtering for `Source_ip: 238.163.231.224` while excluding `source_state: New York` returned **48 hits**, all associated with a completely different state — **Michigan (100%)**.

<img width="1355" height="603" alt="elk8" src="https://github.com/user-attachments/assets/cdea18ec-4f5a-45cd-8d80-6f554e2e8ffb" />


This kind of query is useful for catching **impossible travel** or **geo-anomaly** scenarios: if a user or IP is expected to be tied to one location (e.g. New York) but the logs show it consistently originating from another (Michigan), that mismatch is worth flagging.

### 7. Reviewing the broader picture

Removing the narrow filters and viewing the fuller dataset (**2,856 hits**) in tabular form — Time, Source_ip, UserName, Source_Country — gave a consolidated view of VPN connection activity across the monitored period, mostly originating from the United States.

<img width="1067" height="454" alt="elk9" src="https://github.com/user-attachments/assets/d624d519-a7a0-43ab-b754-d19ac1399831" />


## Key Takeaways

- **Discover is the SOC analyst's primary triage screen** in Kibana — the histogram + document table combination makes it fast to see volume changes and drill into raw events.
- **Time range scoping matters**: the same index returned very different hit counts (1,280 vs. 2,861) depending on the window selected, so getting the time range right is a key first step in any investigation.
- **Field statistics ("Top 5 values") are a fast triage tool** for spotting outliers — a single IP or user making up a disproportionate share of traffic is a natural pivot point.
- **KQL filters (including negation, e.g. `NOT source_state:X`) let you isolate anomalies quickly**, such as an IP that should map to one state but doesn't.
- **Repetition + timing = brute force signal**: 52 failed logins from one IP in the same second is a textbook automated attack pattern, not a user mistyping a password.

## Skills Demonstrated

- Kibana Discover navigation and time-range filtering
- KQL query construction (field filters, negation)
- Log pivoting: value → field stats → filtered investigation
- Pattern recognition for brute-force and geo-anomaly indicators in VPN/authentication logs
