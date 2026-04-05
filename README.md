# VulnGraph: AI Agent Vulnerability Intelligence via MCP

Article and demo assets for the VulnGraph announcement post.

## Article

- [`article.md`](article.md) — Dev.to formatted post with frontmatter
- [`axios-hero.gif`](axios-hero.gif) — Animated cover image (1.8 MB)
- [`axios.mp4`](axios.mp4) — Full 5.5-minute demo recording

## What is VulnGraph?

VulnGraph is a vulnerability intelligence graph that pre-joins 9 authoritative sources (CVE List, EPSS, CISA KEV, OSV, ExploitDB, PoC-in-GitHub, ATT&CK, Nuclei, CWE) into a single memory-mapped binary file. It exposes 16 tools via the Model Context Protocol (MCP) for AI agent integration.

- **467,939 nodes** | **602,467 edges** | **9 sources**
- Sub-millisecond lookups via mmap
- No API keys, no rate limits, no network

**Live demo:** [vulngraph.tools](https://vulngraph.tools)

## Publishing

The article is formatted for [dev.to](https://dev.to). To publish:

1. Create a new post on dev.to
2. Upload `axios-hero.gif` as the cover image
3. Paste the contents of `article.md` (everything below the frontmatter `---`)
4. Set tags: `security`, `mcp`, `ai`, `vulnerabilities`
