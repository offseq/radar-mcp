# OffSeq Threat Radar MCP server

Connect coding agents to [OffSeq Threat Radar](https://radar.offseq.com): CVE records with CVSS, CISA KEV, EPSS history, public exploits and ENISA EUVD; SBOM and lockfile scans; patch prioritisation; IOC checks; and your own Radar hosts and decisions.

Radar is a hosted service. This repository holds the client configuration, a Gemini CLI extension and a Claude Code plugin; the server itself runs at `https://radar.offseq.com/mcp` (Streamable HTTP). It is listed in the official MCP Registry as [`com.offseq/radar`](https://registry.modelcontextprotocol.io/v0.1/servers?search=com.offseq/radar).

## API key

Every call needs a Radar API key (`tr_…`), sent in the `X-API-Key` header. Sign up at [radar.offseq.com](https://radar.offseq.com) and copy the key from Console → Settings. The free plan allows 30 requests a day; paid plans raise the quota and unlock the decision tools. Details: [API docs](https://radar.offseq.com/api-docs?tab=mcp).

## Install

### Claude Code

As a plugin (reads `RADAR_API_KEY` from your environment):

```sh
export RADAR_API_KEY=tr_your_key
claude plugin marketplace add offseq/radar-mcp
claude plugin install offseq-radar@offseq
```

Or add the server directly:

```sh
claude mcp add --transport http radar https://radar.offseq.com/mcp --header "X-API-Key: tr_your_key"
```

### Gemini CLI

```sh
gemini extensions install https://github.com/offseq/radar-mcp
```

The CLI asks for the key and keeps it in the system keychain.

### Cursor

`~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "radar": {
      "url": "https://radar.offseq.com/mcp",
      "headers": { "X-API-Key": "${env:RADAR_API_KEY}" }
    }
  }
}
```

### VS Code

`.vscode/mcp.json`:

```json
{
  "inputs": [
    { "type": "promptString", "id": "radar-key", "description": "Radar API key", "password": true }
  ],
  "servers": {
    "radar": {
      "type": "http",
      "url": "https://radar.offseq.com/mcp",
      "headers": { "X-API-Key": "${input:radar-key}" }
    }
  }
}
```

### Cline

`cline_mcp_settings.json`:

```json
{
  "mcpServers": {
    "radar": {
      "type": "streamableHttp",
      "url": "https://radar.offseq.com/mcp",
      "headers": { "X-API-Key": "tr_your_key" }
    }
  }
}
```

Any other client that supports Streamable HTTP and custom headers works the same way.

## Tools

| Tool | What it does |
|---|---|
| `get_threat` | One CVE or threat record: CVSS, priority and its factors, CISA KEV, EPSS with 30-day history, exploit maturity, ENISA EUVD, OffSeq Mirage sightings, affected versions and remediation |
| `search_threats` | Search the corpus, e.g. "exploited Fortinet CVEs this month" or "critical CVEs with rising EPSS" |
| `get_landscape` | What changed in a window: KEV additions, exploitation not in KEV, new public exploits, EPSS risers, new critical CVEs |
| `prioritize_cves` | Rank 2 to 50 CVEs into a patch order, with the reason for each position |
| `match_packages` | Check purls, name + ecosystem + version, or CPEs against known vulnerabilities, with the smallest fixing version |
| `scan_sbom` | Scan a CycloneDX/SPDX SBOM or a lockfile (npm, pnpm, yarn, Cargo, Go, Poetry, uv, Composer, Bundler, pip) with an optional CI-style gate |
| `check_iocs` | Look up IPs, domains, URLs and hashes in indicators extracted from threat reports |
| `list_hosts` | Hosts in your Radar inventory, registered by [threat-finder](https://github.com/offseq/threat-finder) |
| `list_host_findings` | Findings for one host, riskiest first, with exposure, KEV, EPSS and fix versions |
| `list_changes` | New signals on CVEs you starred or that affect your hosts |
| `list_decisions` | Your decision ledger per CVE and whether each decision still holds |
| `record_decision` | Record a decision (patch now, defer, accept, not affected, mitigated); writes to your account |

All tools except `record_decision` are read-only.

## Example prompts

- "Is CVE-2024-3400 exploited, and is there a public exploit?"
- "Scan this package-lock.json and tell me what to upgrade first."
- "What changed in the vulnerability landscape this week?"

## Privacy and support

The key and your requests go only to radar.offseq.com, operated by SEQ SIA in Latvia. [Privacy policy](https://radar.offseq.com/privacy) · [Terms](https://radar.offseq.com/terms) · support@offseq.com · Security reports: security@offseq.com

## License

The configuration in this repository is MIT licensed. Radar itself is a commercial hosted service.
