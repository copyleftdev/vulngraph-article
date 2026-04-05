# LinkedIn Article: VulnGraph

## Publishing Notes

- **Cover image:** Upload `cover-linkedin.png` (1920x1080)
- **Title:** I Asked My AI Agent About axios. It Knew Everything in 0.03ms.
- LinkedIn has NO markdown support — copy the body text below and paste into the LinkedIn article editor
- Apply **bold** using Ctrl+B on the phrases marked with ** below
- Apply H2 headers using the editor toolbar on lines marked [H2]
- Bullet points paste natively
- For the data tables, screenshot them from the dev.to article or create images

---

## Article Body (copy from here)

I pointed an AI agent at a single npm package — **axios**, the HTTP client installed 55 million times per week — and asked one question: how risky is this?

In **under a millisecond**, it came back with 13 known vulnerabilities correlated across CVE databases, EPSS exploitation scores, CISA's Known Exploited Vulnerabilities catalog, public exploit repositories, and weakness classifications.

No API keys. No network calls. No rate limits.

One local graph. Sub-millisecond. Here's what happened.


[H2] The Problem

Vulnerability data is scattered across a dozen sources. CVE databases. EPSS scores. CISA KEV. ExploitDB. OSV advisories. ATT&CK mappings. Each has its own API, its own auth, its own rate limits.

To properly assess one CVE, you need **4-6 API calls** across different services. To do that for every dependency in a project? Minutes of latency. Quota limits. Fragile integrations.

AI agents make this worse. They need structured answers in milliseconds, not HTML pages in seconds. They can't authenticate to six different APIs. They can't wait 500ms per lookup when scanning hundreds of packages.

So I built something different.


[H2] VulnGraph: 9 Sources, One Graph, Sub-Millisecond

VulnGraph pre-joins data from **9 authoritative sources** into a single memory-mapped file:

- **CVE List V5** — 342,360 published vulnerabilities
- **EPSS** — 324,894 exploitation probability scores
- **CISA KEV** — 1,557 confirmed actively-exploited vulnerabilities
- **OSV** — 43,606 ecosystem-specific advisories with version ranges
- **ExploitDB** — 30,409 published exploits
- **PoC-in-GitHub** — 14,826 proof-of-concept repositories
- **MITRE ATT&CK** — 18,224 techniques, threat actors, and malware entries
- **Nuclei Templates** — 3,999 automated scanning templates
- **CWE** — 745 weakness classifications

Total: **467,939 nodes** and **602,467 edges** in a single binary file.

The graph opens in ~100 microseconds. Point lookups take less than 1ms. No network. No cold starts.


[H2] The axios Investigation

I connected VulnGraph to an AI agent using the Model Context Protocol (MCP) and started asking questions about axios.

**First question: What vulnerabilities exist?**

The agent called the lookup tool and got back **13 CVEs** in 0.14ms. Seven rated HIGH severity. The most concerning:

- **CVE-2025-27152** — SSRF vulnerability, CVSS 7.7, 3 public proof-of-concept exploits
- **CVE-2021-3749** — ReDoS, CVSS 7.5, EPSS 8.26% (92nd percentile for exploitation probability)
- **CVE-2026-25639** — HIGH severity, CVSS 7.5, recently disclosed
- **CVE-2024-39338** — MEDIUM severity, EPSS 2.88%

Every result included the CVSS base score, the EPSS exploitation probability, and PoC counts — all pre-joined, all instant.


**Second question: Is axios version 1.6.0 safe?**

Answer in **0.019ms**: Vulnerable. 13 known CVEs. Highest severity: HIGH.

The agent now knows not to suggest this version in any code it writes.


**Third question: What should we fix first?**

This is where it gets interesting. VulnGraph doesn't just list CVEs — it **triages** them.

The risk assessment tool scored the top axios CVEs using a weighted model that factors in CVSS severity, EPSS probability, exploit maturity, and exposure context:

**Priority 1: CVE-2021-3749** — Risk score 5.62, remediate within 7 days
**Priority 2: CVE-2025-27152** — Risk score 5.60, remediate within 7 days
**Priority 3: CVE-2026-25639** — Risk score 3.75, remediate within 30 days
**Priority 4: CVE-2024-39338** — Risk score 2.04, remediate within 30 days
**Priority 5: CVE-2023-45857** — Risk score 1.75, remediate within 30 days

Notice something? **CVE-2021-3749 outranks CVE-2025-27152 despite a lower CVSS score.** Why? Its EPSS is 8.26% — it's far more likely to actually be exploited in the wild.

CVSS alone would have gotten this wrong. Most vulnerability scanners would have gotten this wrong.


[H2] Following the Thread: SSRF Across the Graph

I asked the agent to go deeper on CVE-2025-27152. In **0.034ms** it mapped the full threat context:

- **Severity:** HIGH (CVSS 7.7)
- **Exploit Maturity:** POC — 3 public proof-of-concept exploits on GitHub
- **Weakness:** CWE-918 (Server-Side Request Forgery)
- **KEV Listed:** Not yet observed in the wild

Following the CWE-918 classification, VulnGraph revealed that SSRF vulnerabilities span **1,401 CVEs** across the entire graph. The most dangerous? CVE-2021-40438 — Apache mod_proxy SSRF, EPSS 94.4%, CISA KEV listed.

One CVE in axios pulled a thread that connected to **the entire SSRF weakness class** across hundreds of thousands of nodes.

That's the graph advantage. It's not six separate API calls stitched together. It's a single traversal across pre-joined data.


[H2] This Was Just One Package

The axios demo used 7 of VulnGraph's **16 MCP tools**. The full set includes:

- **Lookup tools** — instant queries on any CVE, package, or weakness
- **Search tools** — full-text search across 342K CVEs and 75K packages
- **Analysis tools** — multi-factor risk scoring, version-aware vulnerability checks, dependency analysis
- **Exploit intelligence** — exploit maturity, PoC availability, KEV status, trending threats
- **Graph traversal** — map attack surfaces, trace relationships from CVE to package to exploit to technique to threat actor
- **Batch scanning** — scan SBOMs and lockfiles in bulk
- **Data freshness** — every response includes source timestamps so agents know how current the data is


[H2] Why MCP Matters

The Model Context Protocol is the emerging standard for giving AI agents access to tools. VulnGraph implements it natively — 16 tools over JSON-RPC.

Any MCP-compatible agent can discover these tools automatically. The agent calls a single endpoint, sees 16 vulnerability intelligence tools with full schemas, and starts querying. No documentation needed. No integration code.

This enables:

- **Code review agents** that flag vulnerable dependencies before merge
- **Security copilots** that triage by real exploit intelligence, not just CVSS theater
- **Incident response agents** that map from a CVE to affected packages to ATT&CK techniques in seconds
- **CI/CD gates** that block deployments with actively-exploited vulnerabilities


[H2] Try It Live

The full 467,939-node graph is running in the browser via WebAssembly at **vulngraph.tools** — search any CVE, explore relationships, see the data freshness in real time.

What's the first package or CVE you'd investigate? I'd love to hear what you'd point this at.

---

## Files for LinkedIn

- `cover-linkedin.png` — 1920x1080 static cover image (upload as cover)
- `axios-hero.gif` — 1.8MB animated GIF (can embed inline in article body — will animate)
- `axios.mp4` — Full 5.5min demo (upload to YouTube, link from article)
