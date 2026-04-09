# recon-navi

Agentic web application penetration testing tool specialised for Japanese web infrastructure.

![Tests](https://img.shields.io/badge/tests-164%20passed-brightgreen)
![Version](https://img.shields.io/badge/version-0.8.0-blue)
![Python](https://img.shields.io/badge/python-3.11%2B-blue)
![License](https://img.shields.io/badge/license-MIT-green)

---

## What it does

recon-navi is a **LangGraph-based agentic pipeline** that autonomously performs web application penetration testing with first-class support for Japanese encoding vulnerabilities. It targets bug bounty programs on Japanese platforms (IssueHunt, BugBounty.jp) where character encoding edge cases (Shift-JIS, EUC-JP, ISO-2022-JP) create attack surfaces that generic Western tools miss entirely.

The tool runs a 25-node directed acyclic graph — each node is a discrete recon or exploitation step. A local Ollama LLM (llama3.1 or mistral-nemo) drives analysis and adaptive probing. DynamoDB provides distributed checkpointing for 24/7 unattended scanning across EC2 instances.

---

## Feature highlights

### Japanese encoding as a first-class attack surface
- Encoding priority chain: `utf-8-sig → shift_jis → cp932 → euc-jp → iso-2022-jp → utf-8`
- `Accept-Language: ja` on every request to trigger Japanese error messages
- Full-width digit IDOR bypass (e.g. `１` → `1` after server-side normalisation)
- Multibyte WAF bypass payload generation — exploits charset confusion between WAF (UTF-8) and app (Shift-JIS)
- Keigo politeness-level analysis: `kenjogo` (humble) responses indicate admin-tier gates; `sonkeigo` indicates privilege boundaries worth probing

### 25-node autonomous pipeline
| Phase | Nodes |
|-------|-------|
| Recon | dns_recon, subdomain_enum, passive_recon, path_probe |
| Surface mapping | fetch, js_render, surface_map, login_probe, crawler, param_fuzz, api_discover |
| Exploitation | active_probe, idor_probe, auth_bypass, oob_handler, canary_sweep |
| Analysis | analyze, verify_findings, logic_analyzer, jwt_analyze, cors_probe |
| Deep exploitation | ws_probe, deep_probe, adaptive_probe |
| Reporting | generate → END |

### Stored XSS detection (v0.8.0)
A canary sweep runs after all injection nodes complete. Unique 8-char canaries are planted by `active_probe` and `param_fuzz` during injection; `canary_sweep` re-fetches all crawled URLs and SPA client-side routes (via Playwright) to detect cross-page stored reflections. Confirmed stored XSS findings are automatically escalated to `critical/confirmed`.

### Devil's Advocate verifier (v0.8.0)
A second LLM pass (`verify_findings`) attempts to disprove every non-confirmed finding before it reaches the report. High-conviction benign explanations (score ≥ 0.80) drop findings; mid-conviction (≥ 0.55) demotes severity and marks confidence as `possible`. All demotions/drops are preserved in an audit trail (`verify_skipped`).

### WebSocket injection (v0.7.0)
`ws_probe` discovers WebSocket endpoints from JS source scanning, Playwright network interception, and common path probing. Tests 9 payloads: XSS reflection, error-based SQLi, path traversal, auth-skip token manipulation, full-width digit IDOR.

### LLM-driven logic analysis
`logic_analyzer` runs a 2-turn LLM chain: turn 1 classifies the rejection type (WAF / business logic / auth gate / unknown), turn 2 generates and executes a targeted bypass attempt. Confirmed bypasses are automatically escalated to `critical/confirmed`.

### JWT analysis
Extracts JWTs from cookies, `Authorization` headers, and response bodies. Tests: `alg:none` bypass, HMAC weak-secret bruteforce (30-entry wordlist of common secrets), RS256→HS256 key confusion candidate detection.

### CORS exploitation
Tests arbitrary origin reflection, `null` origin, subdomain wildcard, and destructive-method allowance. All tested against `Access-Control-Allow-Credentials: true`.

### OOB blind injection (v0.4.0)
Blind SSRF and blind SQLi via DNS/HTTP callback. Requires `--oob-domain`. Callbacks are received by an AWS Lambda → DynamoDB pipeline (Terraform in `infra/`). Japanese IP callbacks are automatically flagged as critical (confirms server-side execution on JP infrastructure).

### Reporting
- JSON + Markdown reports with CVSS 4.0 vector and numeric score per finding
- Japanese Enterprise Impact context block in Markdown reports (describes business risk in terms relevant to JP corporate culture)
- `verify_skipped` audit trail so reviewers can inspect LLM demotion decisions

---

## CLI usage

```bash
python main.py                                               # interactive prompt
python main.py -t http://target.co.jp                        # single target
python main.py -t http://target.co.jp -m mistral-nemo --format both -v
python main.py -f targets.txt --loop --loop-delay 300        # 24/7 mode
python main.py -t http://target.co.jp --no-llm               # rule-based only
python main.py -t http://target.co.jp --no-dynamo            # skip DynamoDB
python main.py -t http://target.co.jp --resume               # resume last scan
python main.py -t http://target.co.jp --oob-domain cb.yourdomain.com
python main.py -t http://target.co.jp --quick --depth 0      # fast recon
python main.py -t http://target.co.jp --debug                # state delta per node
```

---

## Tech stack

| Component | Technology |
|-----------|-----------|
| Pipeline orchestration | LangGraph (DAG with conditional edges) |
| LLM inference | Ollama — llama3.1:8b-instruct-q5_K_M (default), mistral-nemo:12b |
| HTTP client | httpx (async-capable, follow redirects, verify=False) |
| JS rendering | Playwright (headless Chromium, network interception, scroll simulation) |
| WebSocket probing | websockets 15.x |
| Encoding detection | charset-normalizer + manual priority chain |
| Checkpointing | AWS DynamoDB (LangGraph BaseCheckpointSaver implementation) |
| OOB callbacks | AWS Lambda + API Gateway + DynamoDB (Terraform-provisioned) |
| Reporting | Custom writer — JSON + Markdown + CVSS 4.0 |
| Testing | pytest, moto (DynamoDB mocking), unittest.mock |

---

## Architecture

See [ARCHITECTURE.md](ARCHITECTURE.md) for the full 25-node graph topology, state contract, and key design decisions.

---

## Test suite

164 tests, 0 failures — covering every node, helper function, and integration path.

```
tests/test_navi_logic.py     — 51 tests  (serialisation, DynamoDB, OOB, CSRF, timed SQLi)
tests/test_v08_and_nodes.py  — 113 tests (all nodes + v0.8.0 canary/verify features)
```

Full pytest output: [tests/results/pytest_output.txt](tests/results/pytest_output.txt)

The test suite caught a production bug during this session: `verify_findings.py` was using attribute-style access (`f.confidence`) on `Finding` TypedDict instances (plain dicts), which would have caused `AttributeError` on every live scan run.

---

## Intentionally-vulnerable test server

`test_server.py` is a local EC-CUBE-style Japanese shop server used for pipeline integration tests. It exposes:

- IDOR (6 variants: QS param, path segment, POST JSON body, method escalation, header injection, auth bypass)
- Reflected XSS and **stored XSS** (guestbook endpoint — planted canaries persist and are detected by `canary_sweep`)
- Error-based SQLi
- CORS origin reflection + `Allow-Credentials: true`
- CSRF (no token on login form)
- JWT with weak secret (`"secret"`) + `alg:none` acceptance
- Info disclosure: `/robots.txt`, `/phpinfo.php`, `/.git/HEAD`, `/api/swagger.json`
- WebSocket echo server on port 8889

---

## Source

Full source available upon request — contact via GitHub or LinkedIn.

> This repository is a portfolio showcase. The implementation is kept private to preserve operational security for active bug bounty work.
