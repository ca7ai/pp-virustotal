# VirusTotal CLI - Transcendence Features

Novel features that transcend direct API mirroring and transform the CLI into an intelligence platform.

## Local Intelligence Store

**Feature ID**: `vtstore`  
**Type**: Data Layer  
**Novelty**: Persistent SQLite cache with FTS5 full-text search across all IOCs (files, domains, IPs, URLs) and relationship tracking for graph workflows.

**Why it matters**: VirusTotal API rate limits (4 req/min free tier) make repeated lookups painful. Local store enables instant retrieval, offline search, and historical analysis without burning API quota.

**Implementation**:
- Schema: `files`, `domains`, `ip_addresses`, `urls`, `relationships`, `iocs_fts` (FTS5), `schema_meta`
- Location: `~/.virustotal/cache.db`
- API: `vtstore.Open()`, `StoreFile()`, `GetFile()`, `SearchIOCs()`, `Stats()`
- Auto-migration on first open

**Files**: `internal/vtstore/vtstore.go`

## Pivot Workflows (Graph Traversal)

**Feature ID**: `pivot`  
**Type**: Workflow  
**Novelty**: BFS graph traversal across IOC relationships (file→domain→IP→file chains) with cycle detection and Mermaid diagram output.

**Why it matters**: Threat actors rarely operate in isolation. A malicious file contacts domains, those domains resolve to IPs, those IPs serve other files. Pivoting reveals infrastructure patterns and connected campaigns.

**Supported pivots**:
- `file → domains → ips → files` (infrastructure mapping)
- `domain → ips → domains` (shared hosting detection)
- `ip → files` (malware distribution from IP)

**Output formats**: table, JSON, Mermaid flowchart

**Example**:
```bash
virustotal-pp-cli pivot file <hash> --through domains --to ips --depth 2 --format mermaid
```

**Files**: `internal/cli/pivot.go`

## Batch IOC Enrichment Pipeline

**Feature ID**: `enrich`  
**Type**: Workflow  
**Novelty**: Concurrent batch processor with auto-type detection (SHA256/MD5/SHA1/IP/domain), worker pool, progress tracking, and structured summary report.

**Why it matters**: Security teams handle hundreds of IOCs daily (SIEM alerts, threat intel feeds). Manual one-by-one lookup wastes time. Batch enrichment with auto-detection and concurrency turns 2 hours of work into 2 minutes.

**Features**:
- Auto-detects IOC type via regex
- Worker pool (configurable concurrency, default 5)
- Progress bar with ETA
- Summary report (successful lookups, API errors, invalid IOCs)
- Respects API rate limits

**Example**:
```bash
cat iocs.txt | virustotal-pp-cli enrich -o report.json
```

**Files**: `internal/cli/enrich.go`

## LLM-Native Output

**Feature ID**: `llm-format`  
**Type**: Output  
**Novelty**: `--llm` flag restructures JSON responses with natural language summaries, threat context, and tool-calling suggestions optimized for LLM consumption.

**Why it matters**: Raw VirusTotal API responses are verbose JSON (100+ lines per file). LLMs struggle to extract key signals (malicious/clean verdict, detection ratios, relationships). LLM-native output surfaces critical context in 10 lines.

**Output structure**:
```json
{
  "summary": "SHA256 abc123: 45/72 vendors flagged as malicious. Contacted 3 domains, 2 IPs.",
  "verdict": "malicious",
  "detection_ratio": "45/72",
  "key_relationships": ["domain:evil.com", "ip:1.2.3.4"],
  "suggested_actions": ["pivot to contacted domains", "check IP reputation"],
  "full_report": { ... }
}
```

**Files**: `internal/cli/llm_format.go`, `internal/cli/helpers.go`

## Threat Detection Diff

**Feature ID**: `diff`  
**Type**: Workflow  
**Novelty**: Compares detection results between two scans (same IOC over time or two different IOCs) to surface vendor disagreements, new detections, and classification changes.

**Why it matters**: Malware evolves. A file flagged by 10 vendors yesterday may be flagged by 50 today. Diff surfaces detection velocity (how fast vendors catch up) and vendor disagreements (false positive signals).

**Comparison modes**:
- Time-based: same hash, different scan timestamps (requires local store history)
- Cross-IOC: two different hashes (variant comparison)

**Output**:
```json
{
  "added_detections": ["Kaspersky", "Sophos"],
  "removed_detections": ["Avast"],
  "changed_labels": {"Microsoft": "Trojan:Win32/Wacatac.B!ml" → "Trojan:Win32/Wacatac.C!ml"},
  "detection_delta": "+2",
  "verdict_change": "suspicious → malicious"
}
```

**Files**: `internal/cli/diff.go`

---

## Metadata

**Total transcendence features**: 5  
**Coverage**: Data layer (1), Workflows (3), Output (1)  
**Novel capabilities not in official VirusTotal CLI**: All 5 (official CLI is read-only API proxy)  
**Novel capabilities not in competing CLIs** (vt-cli, virustotalapi3): 4 (pivot, enrich, llm-format, diff)  
**Implementation files**: 5 (`vtstore.go`, `pivot.go`, `enrich.go`, `llm_format.go`, `diff.go`)
