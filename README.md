# OffSeq Threat Radar MCP server

Connect coding agents to [OffSeq Threat Radar](https://radar.offseq.com): CVE records with CVSS, CISA KEV, EPSS history, public exploits and ENISA EUVD; SBOM and lockfile scans; patch prioritisation; IOC and end-of-life checks; and your own Radar hosts and decisions.

Radar is a hosted service. This repository holds the client configuration, a Gemini CLI extension and a Claude Code plugin; the server itself runs at `https://radar.offseq.com/mcp` (Streamable HTTP). It is listed in the official MCP Registry as [`com.offseq/radar`](https://registry.modelcontextprotocol.io/v0.1/servers?search=com.offseq/radar).

## API key

Tool calls need a Radar API key (`tr_…`), sent as `X-API-Key: tr_…` or `Authorization: Bearer tr_…` (the `ApiKey` scheme also works). Sign up at [radar.offseq.com](https://radar.offseq.com) and copy the key from Console → Settings; a Free key works once your Console profile has your full name and use case. Radar uses API keys, not OAuth: clients that only accept a URL with OAuth, such as claude.ai custom connectors, are not supported yet.

The Free plan allows 30 requests a day, 20 an hour and 900 a month, with up to 10 results and 50 packages per call; paid plans raise the quota and unlock the decision tools. Details: [API docs](https://radar.offseq.com/api-docs?tab=mcp).

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
| `search_threats` | Search the corpus, e.g. "exploited Fortinet CVEs this month" or "critical CVEs with rising EPSS" |
| `get_threat` | One CVE or threat record: CVSS, priority and its factors, CISA KEV, EPSS with 30-day history, exploit maturity, ENISA EUVD, OffSeq Mirage sightings, source consensus, affected versions and remediation |
| `match_packages` | Check purls, name + ecosystem + version, or CPEs against known vulnerabilities, with the smallest fixing version |
| `scan_sbom` | Scan one or more SBOMs or lockfiles, optionally against their pull request base to report what a change introduces and fixes, with radar.yml, ignore entries and an optional CI-style gate |
| `prioritize_cves` | Rank 2 to 50 CVEs into a patch order, with the reason for each position |
| `check_iocs` | Look up IPs, domains, URLs and hashes in indicators extracted from threat reports |
| `check_eol` | End-of-life status of purls and CPEs with versions (from endoflife.date): release cycle, EOL date, days left |
| `get_landscape` | What changed in a window: KEV additions, exploitation not in KEV, new public exploits, EPSS risers, new critical CVEs |
| `list_changes` | New signals on CVEs you starred or that affect your hosts, and decisions that went stale or are due for review |
| `list_hosts` | Hosts in your Radar inventory, registered by [threat-finder](https://github.com/offseq/threat-finder) |
| `list_host_findings` | Findings for one host, riskiest first, with exposure, KEV, EPSS and fix versions |
| `list_decisions` | Your decision ledger per CVE and whether each decision still holds |
| `record_decision` | Record a decision (patch now, defer, accept, not affected, mitigated); writes to your account |

All tools except `record_decision` are read-only. The decision tools need a paid plan.

`scan_sbom` formats: CycloneDX (JSON, XML), SPDX (JSON, tag-value), package-lock.json, pnpm-lock.yaml, yarn.lock, Cargo.lock, go.mod, go.sum, poetry.lock, uv.lock, composer.lock, Gemfile.lock, requirements.txt and purl lists. Pass `filename` so the format is detected.

## Resources and prompts

Resources: `radar://kev/recent` (recent CISA KEV additions and due dates), `radar://threat/{cveId}` (one threat record) and `radar://openapi` (the REST API spec).

Prompts: `patch_decision` (weigh a CVE against your inventory and record a decision), `compare_and_prioritize` (patch order for a set of CVEs) and `dependency_triage` (turn an SBOM or lockfile scan into an upgrade plan).

## Protocol and quota

- Streamable HTTP with JSON responses, stateless. Protocol versions 2025-11-25, 2025-06-18, 2025-03-26 and 2024-11-05.
- `initialize`, `ping` and the tool, resource and prompt lists answer without a key, so a client can list the tools before you sign up. The server card is at [`/mcp/server-card`](https://radar.offseq.com/mcp/server-card).
- Each `tools/call`, and each read of a `radar://kev` or `radar://threat` resource, counts as one API request. Calls refused before they run (invalid arguments, over a plan cap) are not counted.
- Every JSON-RPC message counts toward 120 per minute per key (per IP address without one).
- A missing or invalid key returns HTTP 401 with JSON-RPC error -32001 and `WWW-Authenticate: ApiKey realm="radar", header="X-API-Key"`; a spent quota returns 429 with Retry-After.

## Example prompts

- "Is CVE-2024-3400 exploited, and is there a public exploit?"
- "Scan this package-lock.json and tell me what to upgrade first."
- "What changed in the vulnerability landscape this week?"

## Privacy and support

The key and your requests go only to radar.offseq.com, operated by SEQ SIA in Latvia. [Privacy policy](https://radar.offseq.com/privacy) · [Terms](https://radar.offseq.com/terms) · support@offseq.com · Security reports: security@offseq.com

## License

The configuration in this repository is MIT licensed. Radar itself is a commercial hosted service.
