# ARITY_AUDIT_PROTOCOL

**Version:** 1.0  
**Role:** Senior Integration QA / SDET  
**Description:** Binary parity diff between an Integration API collection and a Proxy/Wrapper layer. Executes live requests via MCP RESTful tools, compares real responses, and outputs a full gap/mismatch report.

---

## 1. OBJECTIVE
Perform a full binary parity diff between:
- **`<collection_1>`**: Integration API (Source of Truth)
- **`<collection_2>`**: Proxy / Wrapper Layer

---

## 2. EXECUTION WORKFLOW

### [STEP 0] — Load Collections
1. Read `<collection_1>` and `<collection_2>` JSON files.
2. Flatten folder hierarchy to extract a flat list of `item` entries.
3. Build Maps: `MAP_01` (Source) and `MAP_02` (Proxy) keyed by `{{METHOD}} + {{PATH}}`.

### [PHASE 1] — ENDPOINT_MAPPING
**Logic:** Detect routing gaps.
- **MATCH:** `PATH` exists in both. (*Note: Treat `:id` and `:userId` as structural matches*).
- **GAP_MISSING_PROXIED_ROUTE:** In `01` but missing from `02`.
- **GHOST_ENDPOINT_DETECTED:** In `02` but missing from `01`.

### [PHASE 2] — CONTRACT_SCHEMA_DIFF
**Logic:** Detect schema drift in responses.
- **Key Naming:** Flag naming drifts (e.g., `snake_case` ↔ `camelCase`).
- **Type Safety:** Flag coercions (e.g., `Integer` → `String`).
- **Nullability:** Flag if a field is `REQUIRED` in `01` but `ABSENT` in `02`.

### [PHASE 3] — HEADER_PARITY
**Logic:** Verify header forwarding.
- **AUTH_STABLE:** Auth types match.
- **HEADER_DROPPED:** Key in `01` missing from `02`.
- **PROXY_INJECTED:** Key in `02` not present in `01`.

### [PHASE 4] — STATUS_CODE_PROPAGATION
**Logic:** Detect error masking.
- **ERROR_MASKING_CRITICAL:** `01` returns `4XX/5XX`, but `02` returns `200`.
- **CODE_MISMATCH:** `01` returns `401`, but `02` returns `403`.
- **SUCCESS_CODE_DRIFT:** `01` returns `201`, but `02` returns `200`.

### [PHASE 5] — PARAMETER_SYNC
**Logic:** Audit query params and path variables.
- **PARAM_RENAMED:** Mapping name difference.
- **PARAM_MISSING:** Param defined in `01` omitted in `02`.

### [PHASE 6] — LIVE EXECUTION (MCP RESTful API)
**Logic:** Execute real HTTP requests and diff actual payloads.
- **STATUS_MATCH:** Real-time status codes align.
- **LIVE_ERROR_MASKING:** Backend failure hidden by proxy success.
- **LIVE_KEY_DRIFT:** Actual payload keys differ from documentation.
- **LATENCY_OVERHEAD_HIGH:** Proxy adds >200ms overhead vs Source.

---

## 3. OUTPUT SCHEMA

### Parity Summary
| CATEGORY | SOURCE (01) | PROXY (02) | STATUS_TAG |
|:---|:---|:---|:---|
| `{{PATH}}` | [Value] | [Value] | `MATCH` | `GAP` | `GHOST` |
| `{{KEY}}` | [Value] | [Value] | `MATCH` | `MISMATCH` |
| `{{TYPE}}` | [Value] | [Value] | `STABLE` | `COERCED` |
| `{{CODE}}` | [Value] | [Value] | `PASS` | `MASKED` |

### Severity Matrix
| Finding | Count | Severity |
|:---|:---|:---|
| `ERROR_MASKING_CRITICAL` | [X] | 🔴 CRITICAL |
| `GAP_MISSING_PROXIED_ROUTE` | [X] | 🔴 HIGH |
| `KEY_MISSING` | [X] | 🟠 HIGH |
| `HEADER_DROPPED` | [X] | 🟠 MEDIUM |
| `COERCED` | [X] | 🟠 MEDIUM |
| `GHOST_ENDPOINT` | [X] | ℹ️ INFO |

---

## 4. EXECUTION COMMAND
> "Analyze `<path_to_api_json>` and `<path_to_proxy_json>` using the ARITY_AUDIT_PROTOCOL. Perform full static diff and live execution validation. Output the Severity Matrix first."
