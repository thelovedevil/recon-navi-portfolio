# Architecture — recon-navi v0.8.0

## Graph topology (25 nodes)

```
dns_recon
    │
subdomain_enum          crt.sh CT log query + 40-entry DNS wordlist
    │
passive_recon           robots.txt, sitemap.xml, security.txt
    │
path_probe              85 JP-specific sensitive paths (Movable Type, EC-CUBE,
    │                   CGI-BIN, kanri/, etc.) — skipped if --quick
fetch                   Landing page + tech fingerprint + encoding detection
    │
js_render               Playwright DOM rendering, XHR/fetch interception,
    │                   WebSocket upgrade detection, SPA route extraction,
    │                   scroll-to-bottom lazy-load trigger
surface_map             HTML form/link/parameter discovery
    │
login_probe             JP default credential attempts + Basic Auth + API key scan
    │
crawler                 Follow internal links to depth N (--depth)
    │
param_fuzz              Arjun-style hidden parameter discovery (150-word wordlist)
    │
api_discover            Swagger/OpenAPI/GraphQL endpoint enumeration
    │
active_probe            Error-based SQLi, XSS, LFI regex + time-based blind SQLi
    │                   (capped at 50 requests, 1.0–1.5s jitter)
idor_probe              Sequential ID probing + full-width digit bypass
    │
auth_bypass             Proxy header injection, HTTP verb tampering,
    │                   path normalisation bypass
oob_handler             Blind SSRF/SQLi via OOB callback domain
    │                   (no-op if --oob-domain not set)
canary_sweep            Post-injection stored XSS second pass:
    │                   httpx re-visits all crawled URLs; Playwright visits
    │                   SPA routes from js_render. Cross-page canary match
    │                   → confirmed stored XSS finding.
analyze                 LLM vulnerability analysis (surgical context for confirmed
    │                   findings, broad context for everything else)
verify_findings         Devil's Advocate: LLM attempts to disprove each
    │                   non-confirmed finding. score ≥ 0.80 → drop,
    │                   score ≥ 0.55 → demote severity + set confidence=possible
logic_analyzer          Keigo politeness-level analysis (kenjogo/sonkeigo/teineigo)
    │                   + privilege cue detection + encoding IDOR
    │                   2-turn LLM chain: classify rejection → generate + execute bypass
jwt_analyze             JWT extraction (cookies/headers/body), alg:none bypass,
    │                   HMAC weak-secret bruteforce, RS256→HS256 key confusion
cors_probe              Origin reflection, null origin, subdomain wildcard,
    │                   destructive methods — all with Allow-Credentials
ws_probe                WebSocket discovery (JS scan + Playwright intercept +
    │                   common paths), 9-payload injection suite
deep_probe              Polyglot re-probe of flagged endpoints
    │
adaptive_probe          LLM-directed single best remaining test
    │
    ├─── vulnerability_router (conditional edge) ───────────────────┐
    ▼                                                               ▼
generate                Multibyte WAF bypass payload generation    END
    │                   for confirmed findings
   END
```

## State contract

`ReconState` is a single TypedDict passed through every node. Key fields:

| Field | Type | Owner |
|-------|------|-------|
| `canary_map` | `Dict[str, Dict]` | param_fuzz, active_probe (write); canary_sweep, crawler, js_render (read) |
| `verify_skipped` | `List[Dict]` | verify_findings (audit trail of dropped/demoted findings) |
| `spa_routes` | `List[str]` | js_render (write); canary_sweep (read) |
| `ws_endpoints` | `List[str]` | js_render, ws_probe |
| `findings` | `List[Finding]` | analyze, canary_sweep, logic_analyzer, jwt_analyze, cors_probe |
| `jwt_findings` | `List[Dict]` | jwt_analyze |
| `cors_findings` | `List[Dict]` | cors_probe |
| `idor_hits` | `List[Dict]` | idor_probe |
| `auth_bypass_hits` | `List[Dict]` | auth_bypass |

`state.py` has **zero internal imports** by design — it is the base dependency for every node. Adding internal imports creates circular dependency chains.

## Key design decisions

**`build_graph()` is a factory, never compiled at import time.**
Prevents module-level side effects and allows the checkpointer to be injected at runtime (in-memory for `--no-dynamo`, DynamoDB for production).

**LLM JSON parsing has three fallback levels.**
1. Structured output via `ChatOllama.with_structured_output()` (Pydantic)
2. Regex extraction of JSON block from completion text
3. Keyword scan fallback (extracts fields by key name from freeform text)
This ensures nodes degrade gracefully when small models produce malformed output.

**All payload bytes stored as hex strings.**
`Finding.evidence` and `waf_evasion_payloads` store raw bytes as hex for JSON serializability. The middleware layer handles encode/decode at probe time.

**Response headers included in LLM context.**
Prevents the LLM from hallucinating header values — a known failure mode on 8B models.

**Rate limiting: 1.0–1.5s jitter between path probes.**
Matches bug bounty platform etiquette requirements and avoids rate-limit bans on shared infrastructure.

**`verify_findings` never touches `confirmed` findings.**
Canary-level and regex-level evidence is not subject to LLM second-guessing. Only `likely` and `possible` findings go through Devil's Advocate review.

## Encoding priority chain

```
utf-8-sig  →  shift_jis  →  cp932  →  euc-jp  →  iso-2022-jp  →  utf-8
```

`cp932` is Microsoft's extended Shift-JIS (Windows code page 932) — common on older EC systems. It must be tried before `euc-jp` because some bytes overlap. `utf-8` is tried last rather than first to avoid misclassifying legacy Shift-JIS content as valid UTF-8 (many Shift-JIS byte sequences are valid UTF-8 but decode to garbage).

All charset logic is owned exclusively by `middleware/jp_encoding.py`. No node imports charset detection logic directly.

## OOB infrastructure

```
Probe (recon-navi)
    │
    └── HTTP/DNS callback ──► API Gateway ──► Lambda (oob_lambda.py)
                                                   │
                                              DynamoDB (NetNavi_OOB_Hits)
                                                   │
                                    oob_handler polls every 5s (timeout: 45s)
```

Provisioned via Terraform (`infra/dynamo_oob.tf`). Japanese IP callbacks are flagged as critical — they confirm server-side execution on JP infrastructure, which is the highest-value finding category on JP bug bounty platforms.

## Ollama models

| Model | VRAM | Use case |
|-------|------|----------|
| llama3.1:8b-instruct-q5_K_M | ~5.7 GB | Default — fast, fits in 8 GB GPU |
| mistral-nemo:12b-instruct-v1-q4_K_M | ~7.5 GB | Higher accuracy for ambiguous findings |

LLM calls are optional — `--no-llm` runs the full pipeline in rule-based mode, suitable for EC2 without GPU.
