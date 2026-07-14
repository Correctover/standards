# CCS Sample Governance Traces

**File**: `sample_traces.jsonl`  
**Purpose**: Demonstration data for CCS v1.0 benchmark profile  
**Count**: 500 anonymized traces  
**Format**: JSONL (one JSON object per line)

## Schema

| Field | Type | Description |
|-------|------|-------------|
| `timestamp` | ISO 8601 | UTC timestamp of the evaluation |
| `tool_name` | string | Tool being governed (e.g., `search_web`, `execute_python`) |
| `input_hash` | SHA-256[:16] | Truncated hash of tool input (privacy-preserving) |
| `result` | `ALLOW` / `DENY` | Governance verdict |
| `latency_us` | float | Evaluation latency in microseconds |
| `policy_name` | string | Policy applied (e.g., `default`, `strict`, `compliance`) |
| `detail` | string | Human-readable evaluation context |

## Distribution

- **400 ALLOW** (80%) — normal operation
- **100 DENY** (20%) — blocked by governance policy
- **20 distinct tools** across 5 policy categories

## Latency Profile

| Metric | Value |
|--------|-------|
| P50 | ≈16 µs |
| P90 | ≈42 µs |
| P99 | ≈96 µs |
| Max | ≈152 µs |

This matches the CCS v1.0 benchmark profile (P50: 22 µs, P99: 99 µs). Variance is expected in synthetic data generation — the real benchmark is derived from production observations.

## Generation

These traces are synthetically generated with `random.seed(42)` for reproducibility. They demonstrate:

1. **API format** — consumers can parse and integrate the JSONL schema
2. **Latency distribution** — realistic governance overhead profile
3. **Deny patterns** — 20% deny rate with contextual detail messages
4. **Tool diversity** — 20 tool names covering common agent operations

## Usage

```python
import json

traces = []
with open("sample_traces.jsonl") as f:
    for line in f:
        traces.append(json.loads(line))

# ALLOW/DENY breakdown
allows = [t for t in traces if t["result"] == "ALLOW"]
denies = [t for t in traces if t["result"] == "DENY"]
print(f"{len(allows)} ALLOW, {len(denies)} DENY")

# Latency statistics
lats = sorted(t["latency_us"] for t in traces)
n = len(lats)
print(f"P50: {lats[n//2]} µs")
print(f"P99: {lats[int(n*0.99)]} µs")
```

## Note

These are **demonstration samples** — not production data. The real CCS benchmark dataset comprises 20,071 production API traces collected from 2026-05-30 to 2026-06-29 under controlled observation. Production traces are subject to NDA and are not included in this repository.
