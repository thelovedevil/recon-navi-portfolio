# Shikigami

Agentic web application penetration testing pipeline — Japanese infrastructure specialist.

![Tests](https://img.shields.io/badge/tests-164%20passed-brightgreen)
![Version](https://img.shields.io/badge/version-0.9.0-blue)
![Python](https://img.shields.io/badge/python-3.11%2B-blue)
![Stack](https://img.shields.io/badge/stack-LangGraph%20%7C%20Ollama%20%7C%20AWS-orange)

---

## What it is

Shikigami is a **LangGraph-based agentic penetration testing pipeline** built for Japanese web infrastructure. It runs a 28-node directed workflow — each node is a discrete recon or exploitation step — coordinated by a stateful graph that supports parallel execution, iterative self-improvement, and pause/resume across machines via DynamoDB.

Built for bug bounty programmes on Japanese platforms (IssueHunt.jp, BugBounty.jp) where character encoding edge cases (Shift-JIS, EUC-JP, ISO-2022-JP) create attack surfaces that generic Western scanners miss entirely. All LLM inference runs locally via Ollama — no target data leaves the machine.

---

## Track record

Findings submitted to authorised bug bounty programmes on IssueHunt.jp:

| Programme | Finding Class | Severity | Status |
|-----------|--------------|----------|--------|
| Japanese financial platform | SQL error disclosure via authentication endpoint | Medium | Submitted | Accepted |
| Japanese financial platform | Unauthenticated session exposure via WebAuthn flow | High | Submitted | Accepted |
| Japanese financial platform | CORS misconfiguration + reflected XSS chain | High | Submitted | Accepted |
| Japanese financial platform | WAF bypass via multibyte encoding on login | Medium | Submitted | Accepted |
| Japanese media platform | JWT expiry absent + HMAC key exposure | High | Submitted | Pending |
| Japanese media platform | CDN token disclosure via unauthenticated endpoint | Medium | Submitted | Pending |
| Japanese media platform | S3 stack trace disclosure | Medium | Submitted | Pending |
| Japanese media platform | Compound DRM chain vulnerability | High | Submitted | Pending |

All findings produced within authorised scope. Reports include bilingual (EN/JP) write-ups with curl reproduction scripts.

---

## Architecture overview

### Neurosymbolic pipeline

Shikigami uses a two-layer decision system:

**Symbolic layer** (~1ms, 100% precision on matched patterns)
- Definite pattern classifiers fire before any LLM call
- Eliminates ~40% of LLM invocations on unambiguous findings
- Runs at zero cost — pure pattern matching, no model required

**Neural layer** (LLM via Ollama)
- Handles ambiguous evidence the symbolic layer cannot classify
- adaptive exploitation (native tool-calling)
- llama3.1 / mistral-nemo for analysis and verification

**Devil's Advocate verifier**
- Second LLM pass attempts to disprove every non-confirmed finding
- High-conviction benign explanations drop the finding; mid-conviction demotes severity
- Confirmed findings (pattern-level evidence) are never touched

### Parallel execution

```
active_probe
    ├── rce_probe    ─┐
    ├── idor_probe    ├─► oob_handler (barrier)
    └── auth_bypass  ─┘

logic_analyzer
    ├── jwt_analyze  ─┐
    ├── cors_probe    ├─► deep_probe (barrier)
    └── ws_probe     ─┘
```

Three attack nodes and three analysis nodes execute concurrently via LangGraph fan-out. Primary output keys are exclusive (`rce_findings`, `idor_hits`, `auth_bypass_hits`). Shared accumulators (`findings`, `is_vulnerable`, `evidence_bus`) carry `Annotated[T, reducer]` type annotations so LangGraph merges parallel writes correctly rather than raising `InvalidUpdateError`.

### ReAct exploitation agent

The `adaptive_probe` node runs a multi-turn tool-calling ReAct loop using gemma4:e4b. Rather than a one-shot directive, the agent:

1. Reads all accumulated evidence across the full pipeline run
2. Selects and calls tools iteratively: `http_probe`, `fuzz_param`, `playwright_eval`
3. Observes responses and adapts strategy each turn
4. Calls `report_confirmed` when exploitation is proven

The graph loops the node up to 5 times on no-hit, providing fresh evidence context each iteration.

### Evidence bus

Cross-node typed signals propagate via a shared bus in state, enabling chained exploitation across previously independent findings. CORS misconfigs, JWT weaknesses, and auth bypass paths discovered by earlier nodes are available to the ReAct agent as structured signals — not just raw text.

### DynamoDB checkpointing

LangGraph state is persisted after every node. Scans can be paused on a local machine and resumed on EC2. Large text fields are compressed before write — typically 70–85% size reduction, keeping DynamoDB items well under the 400KB limit.

---

## Japanese encoding specialisation

This is the primary differentiator from generic pentest tools.

- **Encoding priority chain**: correct decoding on every response regardless of misconfigured `Content-Type` headers — covers Shift-JIS, CP932, EUC-JP, ISO-2022-JP, UTF-8-SIG
- **`Accept-Language: ja`** on every request — triggers Japanese-language error messages that expose internal stack structure unavailable in English responses
- **Keigo analysis**: Response politeness level is a reliable signal for privilege tier boundaries in Japanese enterprise applications — a pattern unique to JP-language web infrastructure
- **Full-width digit IDOR**: Server-side normalisation bypasses numeric ID validation in many JP frameworks
- **Multibyte WAF bypass**: Charset confusion between WAF and application layer is a consistent source of filter evasion in Japanese infrastructure

---

## Tech stack

| Layer | Technology |
|-------|-----------|
| Workflow orchestration | LangGraph 1.1.4 (StateGraph, conditional edges, fan-out/fan-in) |
| Local LLM inference | Ollama (local, private) |
| LLM fallback | Amazon Bedrock — automatic on local unavailability |
| HTTP client | httpx with JP encoding middleware |
| Browser automation | Playwright (DOM rendering, SPA route extraction, stored XSS detection) |
| State persistence | AWS DynamoDB (LangGraph BaseCheckpointSaver) |
| Infrastructure | Terraform — DynamoDB, Lambda OOB callback listener, EC2 scan runner |
| Reporting | CVSS 4.0 scoring, JSON + Markdown, bilingual EN/JP |
| Testing | pytest 9.0 — 164 tests, zero mocked network for integration suites |

---

## Design principles

**Privacy by default** — all LLM inference is local. Target responses, vulnerability details, and application behaviour never reach a third-party API.

**Precision over recall** — the neurosymbolic gate and Devil's Advocate verifier exist specifically to reduce false positives. A finding that reaches the report has survived two rounds of attempted disproof. Three additional false-positive vectors discovered during live scanning are defended against explicitly:

- *SSTI arithmetic oracle*: The `{{7*7}}→49` check is gated behind a `ssti_oracle=True` flag — it is never applied to arbitrary page bodies, only to probe responses where the arithmetic expression was actually injected.
- *CloudFront SPA soft-404*: Path probe and API discovery fingerprint the CDN's uniform-200 behaviour before scanning (two junk-path requests, body size comparison) and suppress results accordingly.
- *X-Original-URL header ignored*: Auth bypass records the root-path baseline size before testing rewrite headers. A bypass response within ±1000 bytes of the homepage is classified as a non-event, not a confirmed bypass.

**Resumability** — 28-node scans take 15–40 minutes on a live target. Checkpoint-based resume means a crash or network interruption loses at most one node of work.

**Encoding correctness first** — every HTTP response is decoded through the full JP encoding chain before any analysis runs. A finding based on a misread Shift-JIS response is worse than no finding.

---

## Status

Private repository — available for review during hiring or engagement discussions.
