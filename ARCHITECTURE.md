# Architecture — Shikigami v0.9.0

## Graph topology (28 nodes)

```
dns_recon
    │
subdomain_enum          CT log query + DNS wordlist
    │
passive_recon           robots.txt, sitemap.xml, security.txt
    │
path_probe              JP-specific sensitive paths — skipped if --quick
    │
fetch                   Landing page, tech fingerprint, encoding detection
    │
js_render               Playwright DOM rendering, XHR/fetch interception,
    │                   WebSocket upgrade detection, SPA route extraction
surface_map             HTML form/link/parameter discovery
    │
login_probe             JP default credential attempts + Basic Auth
    │
crawler                 Follow internal links to depth N
    │
param_fuzz              Hidden parameter discovery
    │
api_discover            Swagger/OpenAPI/GraphQL endpoint enumeration
    │
mad_probe               Mass Assignment Discovery
    │
active_probe            Multi-technique injection suite
    │
    ├── rce_probe    ─┐
    ├── idor_probe    ├──────────────── PARALLEL PHASE A ─────────────────┐
    └── auth_bypass  ─┘                                                    │
                                                                           ▼
                                                                     oob_handler
                                                                           │
                                                                     canary_sweep
                                                                           │
                                                                        analyze
                                                                           │
                                                                  verify_findings
                                                                           │
                                                                  logic_analyzer
                                                                           │
    ┌── jwt_analyze  ─┐                                                    │
    ├── cors_probe    ├──────────────── PARALLEL PHASE B ─────────────────┘
    └── ws_probe     ─┘
                    │
               deep_probe
                    │
           adaptive_probe  ◄──────────────────────────┐
                    │                                  │ (loop up to 5×
                    │                                  │  on no-hit)
          adaptive_loop_router                         │
                    │                                  │
          ┌─────────┼──────────┐                       │
          ▼         ▼          ▼                       │
     continue   generate_poc  end ──► END              │
          │         │                                  │
          └─────────┘─────────────────────────────────┘
                    │
                 generate
                    │
                   END
```

## State contract

`ReconState` is a single TypedDict passed through every node. Nodes return `{**state, <changed_keys>}` — immutable update pattern.

Key fields added in v0.9.0:

| Field | Type | Purpose |
|-------|------|---------|
| `evidence_bus` | `List[Dict]` | Cross-node typed signals (trusted_origin, jwt_alg_none, auth_bypass_path, etc.) |
| `adaptive_iterations` | `int` | Loop counter for adaptive_probe (max 5) |
| `verify_skipped` | `List[Dict]` | Audit trail of findings dropped/demoted by Devil's Advocate |

`state.py` has **zero internal imports** by design — it is the base dependency for every node. Internal imports here create circular dependency chains.

## Key design decisions

**`build_graph()` is a factory, never compiled at import time.**
Prevents module-level side effects and allows the checkpointer to be injected at runtime (in-memory for local runs, DynamoDB for production/EC2).

**Neurosymbolic gate eliminates ~40% of LLM calls.**
A symbolic pattern classifier runs first (~1ms). If it produces a confirmed finding, the LLM call is skipped entirely. The LLM only sees ambiguous evidence that requires reasoning. This matters on an RTX 4070 where each LLM call costs 2–8 seconds of GPU time.

**Three-level LLM JSON parsing.**
1. Structured output via `with_structured_output()` (Pydantic schema)
2. Regex extraction of JSON block from completion text
3. Keyword scan fallback (field extraction from freeform text)
Nodes degrade gracefully when small models produce malformed output.

**Devil's Advocate verifier never touches `confirmed` findings.**
Pattern-level and canary-level evidence is not subject to LLM second-guessing. Only `likely` and `possible` findings go through the adversarial review pass.

**Parallel phases write to exclusive state keys.**
`rce_probe` → `rce_findings`, `idor_probe` → `idor_hits`, `auth_bypass` → `auth_bypass_hits`. No reducer logic required — LangGraph merges by key, and keys don't overlap.

**Evidence bus enables cross-node chaining.**
When `cors_probe` finds a trusted origin or `jwt_analyze` finds a weak secret, those signals are written to `evidence_bus`. The `adaptive_probe` ReAct agent reads the bus summary in its context window, allowing it to chain attacks across findings from previously independent nodes.

**Bedrock fallback is health-check gated.**
`llm_router.py` pings Ollama's `/api/tags` before every LLM invocation. If Ollama is down, the call routes to Amazon Bedrock (Claude Haiku) transparently. The `FallbackLLM` wrapper also catches mid-call `ConnectionRefusedError` and `TimeoutError` and retries on Bedrock.

**DynamoDB compression keeps items under 400KB.**
`clean_text` and `vulnerability_details` can be hundreds of KB on large targets. Both are gzip+base64 compressed before write (70–85% reduction). `raw_content` (fetch bytes buffer) is pruned entirely — it's not needed for resume.

**Rate limiting: 1.0–1.5s jitter between probes.**
Matches bug bounty platform etiquette and avoids rate-limit bans on shared infrastructure.

## Encoding priority chain

```
utf-8-sig  →  shift_jis  →  cp932  →  euc-jp  →  iso-2022-jp  →  utf-8
```

`cp932` (Windows Shift-JIS extension) is tried before `euc-jp` because some byte sequences overlap. `utf-8` is tried last — many Shift-JIS byte sequences are technically valid UTF-8 but decode to mojibake, so trying UTF-8 first produces false confidence on legacy JP systems.

All charset logic is owned exclusively by `middleware/jp_encoding.py`. No node imports charset detection logic directly.

## OOB infrastructure

```
Probe (shikigami)
    │
    └── HTTP/DNS callback ──► API Gateway ──► Lambda (oob_lambda.py)
                                                   │
                                              DynamoDB (NetNavi_OOB_Hits)
                                                   │
                                    oob_handler polls every 5s (timeout: 45s)
```

Provisioned via Terraform. Japanese IP callbacks are flagged as critical — they confirm server-side execution on JP infrastructure, the highest-value finding category on JP bug bounty platforms.

## LLM configuration

Shikigami supports configurable local models via Ollama with automatic Bedrock fallback. Model selection is intentionally not documented here.

`--no-llm` runs the full pipeline in rule-based mode (symbolic layer only), suitable for EC2 without GPU.
