---
name: "bounty-recon-pro"
description: "End-to-end bug bounty recon with real tool execution — passive OSINT first, then active scanning through a VPN proxy, auto-generates platform-ready reports. Requires osint-mcp-server + pentest-mcp."
triggers:
  - "bounty recon"
  - "recon on"
  - "scan target"
  - "bug bounty"
  - "passive recon"
  - "enumerate subdomains"
  - "nuclei scan"
  - "osint"
price: 19
---

# Bug Bounty Recon Pro

End-to-end recon workflow for authorized bug bounty hunting on HackerOne and Bugcrowd.
Unlike skills that just describe methodology, this one **executes tools** — passive OSINT
first via osint-mcp-server, then active scanning via pentest-mcp, with hard VPN egress
enforcement before any packet leaves your machine.

> ⚠️ **Authorized targets only.** Always read the program scope before running anything.
> This skill will refuse to scan out-of-scope assets and verifies VPN egress before
> any active scanning. You are responsible for staying within program rules.

---

## Prerequisites

Install once, then forget:

**1. osint-mcp-server** (passive recon — 37 tools, zero API keys needed for core features)
```bash
npm install -g osint-mcp-server
```
Add to your MCP client config:
```json
{
  "mcpServers": {
    "osint-mcp": {
      "command": "npx",
      "args": ["osint-mcp-server"]
    }
  }
}
```

**2. pentest-mcp** (active scanning — nmap, nuclei, sqlmap, gobuster, ZAP, etc.)
```bash
docker pull ramgameer/pentest-mcp:latest
```
Add to your MCP client config:
```json
{
  "mcpServers": {
    "pentest-mcp": {
      "command": "docker",
      "args": ["run", "-i", "--rm", "--network", "host", "ramgameer/pentest-mcp:latest"]
    }
  }
}
```

**3. A VPN with HTTP proxy support**  
Any VPN that exposes an HTTP CONNECT proxy works — PIA, Mullvad, ProtonVPN.
Note your proxy address (e.g., `http://127.0.0.1:8888`).

---

## Workflow

This skill runs in four phases, always in order. It will not skip phases.

### Phase 1 — Scope Review
Before touching anything:
- Parse the program's scope (domains, IPs, out-of-scope assets)
- Identify wildcard vs explicit targets
- Flag any ambiguous scope items and ask for clarification
- Record the confirmed scope in `bounty/<program>/scope.md`

### Phase 2 — Passive Recon (zero noise, zero risk)
Uses **osint-mcp-server** tools — no packets sent to target:

| Tool | What it finds |
|------|--------------|
| `crtsh_search` | Subdomains via certificate transparency logs |
| `osint_domain_recon` | Full passive bundle (DNS, WHOIS, BGP, email security) |
| `wayback_urls` | Forgotten endpoints, old API paths, leaked params |
| `dns_email_security` | SPF/DMARC/DKIM misconfigs (often reportable) |
| `hackertarget_hostsearch` | Reverse IP neighbors |
| `hackertarget_reverseip` | Other domains on the same infrastructure |
| `m365_tenant` | Microsoft tenant info (if target uses M365) |
| `whois_domain` + `whois_ip` | Registrar, hosting, org details |

Output: `bounty/<program>/passive/` — subdomains, URLs, DNS records, email findings.

### Phase 3 — VPN Egress Verification + Active Scanning
**Hard stop before any active scanning:** verifies your public IP is NOT your real IP.

Uses **pentest-mcp** tools, all traffic routed through your VPN proxy:

| Tool | Purpose |
|------|---------|
| `nmap` | Port/service/version discovery on confirmed live hosts |
| `nuclei` | CVE + misconfiguration scanning (template-based) |
| `gobuster` / `ffuf` | Directory and endpoint discovery |
| `whatweb` | Technology fingerprinting |
| `sqlmap` | SQL injection testing on identified parameters |
| `nikto` | Web server misconfiguration audit |
| `sublist3r` | Active subdomain brute-force (supplements passive) |

Active scans are rate-limited by default (`--rate-limit 50` for nuclei) to avoid
triggering WAFs or getting banned from programs.

### Phase 4 — Report Generation
Produces two outputs:

**1. Internal findings log** (`bounty/<program>/findings.md`)  
Structured by severity (Critical → High → Medium → Low → Info) with:
- Finding title and description
- Affected endpoint/asset
- Steps to reproduce
- Evidence (command output, screenshots)
- CVSS score estimate

**2. Platform-ready submission** (`bounty/<program>/submissions/<id>.md`)  
Pre-formatted for HackerOne or Bugcrowd — fill in program handle and submit.

---

## Usage

Just tell the agent what you want:

```
bounty recon greenhouse.io
```
```
run passive recon on hackerone.com — stay out of scope on api.hackerone.com
```
```
enumerate subdomains for target.com then nuclei scan anything alive
```
```
check email security for company.com and generate a finding if there's no DMARC
```

The agent handles sequencing, tool selection, and report formatting automatically.

---

## VPN Configuration

Set your proxy once at the start of a session:

```
my VPN proxy is http://127.0.0.1:8888
```

The skill stores this for the session and prepends it to every active scan command.
If egress verification fails (real IP detected), active scanning is aborted.

---

## What Makes This Different

Most "bug bounty" skills on MCP Market are prompt guides — they tell you *what* to do
but you still have to go do it manually. This skill:

- **Actually runs the tools** via osint-mcp-server and pentest-mcp
- **Enforces passive-first discipline** — passive recon finds 80% of bugs with 0% detection risk
- **Hard VPN enforcement** — no active scan leaves without egress verification
- **Generates ready-to-submit reports** — not just raw tool output

---

## Responsible Use

- Only run against targets you are explicitly authorized to test
- Read program scope before every engagement — scope changes
- Respect rate limits and program rules
- If you find something critical, report it promptly
- Never test out-of-scope assets even if they look interesting
