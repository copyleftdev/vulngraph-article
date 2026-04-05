---
title: I Asked My AI Agent About axios. It Knew Everything in 0.03ms.
published: false
description: How VulnGraph MCP gives AI agents instant vulnerability intelligence from 9 sources — no API keys, no rate limits, sub-millisecond.
tags: security, mcp, ai, vulnerabilities
cover_image: https://raw.githubusercontent.com/copyleftdev/vulngraph-article/main/axios-hero.gif
---

I pointed an AI agent at a single npm package — **axios**, the HTTP client installed 55 million times per week — and asked: *how risky is this?*

In **under a millisecond**, it came back with 13 known vulnerabilities correlated across CVE databases, EPSS exploitation scores, CISA KEV, public exploits, weakness classifications, and ATT&CK mappings.

No API keys. No network calls. No rate limits.

One local graph. Sub-millisecond.

Here's what happened.

---

## The Setup

[VulnGraph](https://vulngraph.tools) is a vulnerability intelligence graph that pre-joins 9 authoritative sources into a single memory-mapped file. It exposes 16 tools via the [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) — the standard for giving AI agents access to tools.

I connected it to an agent and started asking questions about `axios`.

---

## 13 CVEs. 7 High Severity. 0.14ms.

The first call — `lookup_package` — returned the full vulnerability profile:

| CVE | Severity | CVSS | EPSS | PoCs |
|-----|----------|------|------|------|
| CVE-2025-27152 | **HIGH** | 7.7 | 0.07% | 3 |
| CVE-2025-58754 | **HIGH** | 7.5 | 0.11% | — |
| CVE-2026-25639 | **HIGH** | 7.5 | 0.05% | — |
| CVE-2021-3749 | **HIGH** | 7.5 | 8.26% | 1 |
| CVE-2024-39338 | MEDIUM | 4.0 | 2.88% | — |
| CVE-2023-45857 | — | — | 0.13% | 3 |
| CVE-2019-10742 | — | — | 13.52% | — |

That's not just CVE IDs. Every row has a **CVSS base score**, an **EPSS exploitation probability** (likelihood of exploitation in the next 30 days), and **proof-of-concept counts** from GitHub and ExploitDB. All pre-joined. All instant.

---

## Is axios@1.6.0 Safe?

```json
{
  "package": "axios",
  "version": "1.6.0",
  "vulnerable": true,
  "cve_count": 13,
  "highest_severity": "HIGH"
}
```

**0.019ms.** The agent now knows not to suggest this version in any code it writes.

---

## What Should You Fix First?

This is where it gets interesting. VulnGraph doesn't just list CVEs — it **triages** them.

The `assess_risk` tool scored the top axios CVEs using a weighted model: CVSS severity, EPSS probability, exploit maturity, and exposure context.

| Priority | CVE | Risk Score | Level | Fix Within |
|----------|-----|------------|-------|------------|
| 1 | CVE-2021-3749 | 5.62 | MEDIUM | 7 days |
| 2 | CVE-2025-27152 | 5.60 | MEDIUM | 7 days |
| 3 | CVE-2026-25639 | 3.75 | LOW | 30 days |
| 4 | CVE-2024-39338 | 2.04 | LOW | 30 days |
| 5 | CVE-2023-45857 | 1.75 | LOW | 30 days |

**Notice something?** CVE-2021-3749 outranks CVE-2025-27152 despite a lower CVSS score. Why? Its EPSS is **8.26%** (92nd percentile) — it's far more likely to be exploited in the wild.

CVSS alone would have gotten this wrong. Most vulnerability scanners would have gotten this wrong.

---

## Deep Dive: SSRF in axios (CVE-2025-27152)

I asked the agent to go deeper on CVE-2025-27152. The `get_exploit_intel` tool mapped the full threat context in **0.034ms**:

- **Severity:** HIGH (CVSS 7.7)
- **Exploit Maturity:** POC — 3 public proof-of-concept exploits on GitHub
- **Weakness:** CWE-918 (Server-Side Request Forgery)
- **KEV Listed:** No (not yet seen in the wild)
- **EPSS:** 0.07% — low current probability

Following the CWE-918 thread, VulnGraph revealed that SSRF is classified across **1,401 CVEs** in the graph. The most dangerous? CVE-2021-40438 — Apache mod_proxy SSRF, EPSS 94.4%, CISA KEV listed, CVSS 9.0.

One CVE pulled a thread that unraveled an entire weakness class across the graph.

---

## The Graph Traversal

This is the core advantage. The `get_related` tool traced all connections from CVE-2025-27152 in a single traversal:

```
CVE-2025-27152
  |-- affects --> npm:axios
  |-- affects --> cpe:axios:axios
  |-- has_poc --> GitHub-PoC:CVE-2025-27152:0
  |-- has_poc --> GitHub-PoC:CVE-2025-27152:1
  |-- has_poc --> GitHub-PoC:CVE-2025-27152:2
  +-- classified_as --> CWE-918 (SSRF)
```

It's not 6 separate API calls stitched together. It's a single hop across pre-joined data. **0.05ms.**

---

## This Was Just axios

The demo exercised 7 of VulnGraph's **16 MCP tools**. The full toolset:

| Category | Tools |
|----------|-------|
| **Lookup** | `lookup_cve` `lookup_package` `lookup_weakness` |
| **Search** | `search_vulnerabilities` `search_packages` |
| **Analysis** | `analyze_dependencies` `assess_risk` `check_version` |
| **Exploit Intel** | `get_exploit_intel` `trending_threats` |
| **Graph** | `map_attack_surface` `get_related` |
| **Timeline** | `get_timeline` |
| **Batch** | `scan_sbom` `scan_lockfile` |
| **Meta** | `graph_stats` |

---

## 467,939 Nodes. 9 Sources. One File.

VulnGraph pre-joins data from 9 authoritative sources:

| Source | Records | What It Provides |
|--------|---------|-----------------|
| CVE List V5 | 342,360 | Every published vulnerability |
| EPSS | 324,894 | Exploitation probability — what's likely to be attacked |
| CISA KEV | 1,557 | Confirmed actively-exploited vulnerabilities |
| OSV | 43,606 | Ecosystem advisories with affected version ranges |
| ExploitDB | 30,409 | Published exploits |
| PoC-in-GitHub | 14,826 | Proof-of-concept code |
| MITRE ATT&CK | 18,224 | Techniques, threat actors, malware |
| Nuclei Templates | 3,999 | Automated scanning templates |
| CWE | 745 | Weakness taxonomy |

The graph opens in **~100 microseconds** via mmap. Point lookups run in **<1ms**. There are no network calls, no cold starts, no rate limits.

Every response includes a `data_freshness` envelope showing exactly when each source was last synced — because stale vulnerability data is worse than no data.

---

## Why MCP?

The [Model Context Protocol](https://modelcontextprotocol.io/) is the emerging standard for giving AI agents access to tools. VulnGraph implements it natively — 16 tools over JSON-RPC, available via HTTP or stdio.

Any MCP-compatible agent can discover these tools automatically. The agent calls `tools/list`, sees 16 vulnerability intelligence tools with full input schemas, and starts querying — no docs, no integration code.

```json
{
  "method": "tools/call",
  "params": {
    "name": "check_version",
    "arguments": {
      "ecosystem": "npm",
      "package": "axios",
      "version": "1.7.4"
    }
  }
}
```

0.019ms later, the agent knows whether the dependency it's about to recommend is safe.

**What this enables:**

- Code review agents that flag vulnerable dependencies before merge
- Security copilots that triage by real exploit intelligence, not just CVSS
- Incident response agents that map CVE to package to technique to threat actor
- CI/CD gates that block deploys with actively-exploited vulnerabilities

---

## Try It

**[vulngraph.tools](https://vulngraph.tools)** — the full 467K-node graph running in your browser via WebAssembly. Search any CVE, explore relationships, see the data freshness live.

---

*VulnGraph is a Rust graph engine backed by memory-mapped binary files. 467,939 nodes. 602,467 edges. 9 sources. Sub-millisecond. Built for agents.*
