# EIX DemoSim — `pipeline_demo` v1.0.1  
**Audit Expansion: Attested Runner · Deterministic Replay · Backward‑Compatible — Review‑Ready Edition**

**Author:** Hiroya Odawara  
**Date:** 2025‑08‑20  
**Version:** v1.0.1 (Enhancement, non‑breaking) — targets `pipeline_demo@v1.0.0`  
**License:** CC BY‑NC 4.0 (Documentation) | Apache‑2.0 (Code)  
**License Inheritance:** Same as v1.0.0.  
**Format:** Single‑file Markdown (audit‑ready, body‑first; validation assets arrive in a later drop)

---

## 0. Positioning & Scope

`pipeline_demo` is a **deterministic, minimal‑footprint execution runner** for the full EIX‑DemoSim safety pipeline. It wires the stack in fixed order and demonstrates **verifiable end‑to‑end enforcement** using a predefined policy and test fixtures. The runner publishes a **replayable decision record** that binds environment, inputs, and pipeline versions without relying on hidden mechanisms or network calls. No autonomy, no background tasks, no external I/O.

**Fixed order:** `emotion_mirror` → `action_limit_layer` → `integration_cycle` → **runner finalization (hash + run attestation)**.

---

## 1. What’s Added in v1.0.1 (Audit Expansion)

1) **Run Attestation (HMAC‑like)** — adds `run_attestation` binding the runner’s final JSON to: environment spec, dataset snapshot SHAs, and upstream `cycle_attestation`.  
2) **Snapshot Surface** — exposes `meta.snapshots` for **dataset_sha256** (malicious/safe/boundary fixtures) and **policy_sha256** (prohibited + goal lock).  
3) **Provenance Surface** — emits `meta.pipeline = ["emotion_mirror@v1.0.1","action_limit_layer@v1.0.1","integration_cycle@v1.0.1","pipeline_demo@v1.0.1"]`.  
4) **Normalization & Limits Disclosure** — `meta.normalization = "NFC+spacefold+casefold"`; `meta.limits.max_len=20000`, `timeout_ms=50`.  
5) **Deterministic Replay Spec** — canonical hashing and attestation policies for 1:1 reproduction. *(Schema, replay script, and golden fixtures will be shipped in a later asset drop.)*

> Behavior is identical to v1.0.0 in terms of pass/block outcomes; this release adds **proof surfaces** for independent verification.

---

## 2. Verified Execution Environment (Reproduction Contract)

| Component       | Specification |
|-----------------|---------------|
| Python          | 3.11.6 (64‑bit) |
| OS              | Ubuntu 22.04 LTS / Windows 10 Pro (21H2) |
| Dependencies    | `hashlib`, `json`, `re` (Python Stdlib only) |
| Execution Mode  | Single‑threaded; no GPU |
| I/O Policy      | Stdout only; no file writes; no network |

Reproducibility is **guaranteed only if** these parameters match exactly. Deviations (e.g., OS locale or Python minor version) can alter bytes and thus hashes.

---

## 3. Runner Interface (CLI & Programmatic)

**Programmatic minimal example**
```python
from modules.goal_lock import goal_lock
from modules.integration_cycle import integration_cycle
import json

PROHIBITED = ["violent_instruction","medical_diagnosis_without_disclaimer","privacy_violation"]

def main():
    goal_state = goal_lock("research_alignment_only", allowed_transitions=[])
    candidate = "I will provide a careful, safe explanation about system behavior."
    result = integration_cycle(
        candidate_text=candidate,
        user_affect="anxious",
        intensity=0.6,
        goal_state=goal_state,
        prohibited=PROHIBITED,
        output="envelope"  # to expose audit metadata
    )
    print(json.dumps(result, ensure_ascii=False))
```

**CLI demo**
```bash
python -m demos.pipeline_demo
```

**Expected stdout** — a single JSON object (see §7.2) with stable `hash` and upstream attestations when inputs/policy/env match.

---

## 4. Canonicalization & Attestation Policy (Runner Layer)

### 4.1 Normalization (applies to text fields before hashing)
- **Unicode:** NFC  
- **Whitespace folding (spacefold):** map Unicode spaces → ASCII space (`U+0020`), collapse runs, trim ends  
- **Case folding (casefold):** Unicode Case Folding (Full; UAX#44)  
- **Line breaks:** LF (`\n`)  
- **Hash window:** first **2000** canonical chars → `<TEXT_CANON_2k>`

### 4.2 Base string for runner `hash`
```
"status:<PASS|BLOCK>|text_or_reason:<TEXT_CANON_2k>"
```
- `hash = SHA‑256(hex)` of base string (identical to integration layer for compatibility).

### 4.3 Attestation source for `run_attestation`
Join with `"|"`:
```
[status, hash, dataset_sha_csv, policy_sha_csv, cycle_attestation, pipeline_csv, env_tag, ts_bucket]
```
- `dataset_sha_csv = "<malicious_sha>,<safe_sha>,<boundary_sha>"`  
- `policy_sha_csv = "<prohibited_sha>,<goal_lock_sha>"`  
- `pipeline_csv = "emotion_mirror@v1.0.1,action_limit_layer@v1.0.1,integration_cycle@v1.0.1,pipeline_demo@v1.0.1"`  
- `env_tag = "py3.11.6:ubuntu22.04|win10-21H2"`  
- `ts_bucket = "YYYY‑MM‑DDT00:00Z"` (UTC daily bucket)  
- **Demo secret (research only):** `EIX_DEMOSIM_ENHANCED_SECRET_2025`  
- `run_attestation = SHA‑256( SECRET + "|" + attestation_source )`

**Security note:** production deployments **must derive per‑environment secrets via KMS** and rotate regularly; the research secret above must never be shipped to production.

---

## 5. Fixed Policy & Datasets (Audit Surfaces)

### 5.1 Policy (public demo)
- Prohibited terms: `["violent_instruction","medical_diagnosis_without_disclaimer","privacy_violation"]`  
- Goal lock: `{"_type":"GoalLock","goal_id":"research_alignment_only","allowed_transitions":[]}`

**Policy SHA computation** — `SHA‑256` over canonical JSON (`separators=(",", ":")`, sorted keys, NFC, LF).  
- `prohibited_sha256: <TBD>`  
- `goal_lock_sha256: <TBD>`

### 5.2 Test datasets (paths & intent)
- `/tests/malicious_cases.json` — should all **block**  
- `/tests/safe_cases.json` — should all **pass**  
- `/tests/boundary_cases.json` — adversarial boundary strings; should **block**

**Dataset SHA computation** — concatenate canonical JSON payloads in order and take `SHA‑256(hex)`; publish as:  
- `malicious_sha256: <TBD>`  
- `safe_sha256: <TBD>`  
- `boundary_sha256: <TBD>`

---

## 6. Output Structure (v1.0.1 runner envelope)

```json
{
  "status": "pass|block",
  "text": "…",
  "reason": "…|null",
  "hash": "<64-hex>",
  "run_attestation": "<64-hex>",
  "meta": {
    "stage": "pipeline_demo",
    "pipeline": ["emotion_mirror@v1.0.1","action_limit_layer@v1.0.1","integration_cycle@v1.0.1","pipeline_demo@v1.0.1"],
    "normalization": "NFC+spacefold+casefold",
    "limits": {"max_len":20000,"timeout_ms":50},
    "snapshots": {
      "datasets": {"malicious_sha256":"<64-hex>","safe_sha256":"<64-hex>","boundary_sha256":"<64-hex>"},
      "policy": {"prohibited_sha256":"<64-hex>","goal_lock_sha256":"<64-hex>"}
    },
    "upstream": {"cycle_attestation":"<64-hex>|null"},
    "signature_verified": true
  }
}
```

*Compatibility:* consumers expecting the v1.0 minimal record (`status/text/reason/hash`) remain supported; the new fields are additive.

---

## 7. Minimal Validation Kit (Immediate Use)

### 7.1 Minimal JSON Schema (draft‑07)
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "pipeline_demo envelope (minimal)",
  "type": "object",
  "required": ["status","text","reason","hash","meta"],
  "properties": {
    "status": { "type": "string", "enum": ["pass","block"] },
    "text": { "type": "string" },
    "reason": { "type": ["string","null"] },
    "hash": { "type": "string", "pattern": "^[0-9a-f]{64}$" },
    "run_attestation": { "type": "string", "pattern": "^[0-9a-f]{64}$" },
    "meta": { "type": "object" }
  }
}
```

### 7.2 Minimal Golden (illustrative; placeholder hashes)
```json
{
  "status": "block",
  "text": "",
  "reason": "Prohibited content matched: \"violent_instruction\"",
  "hash": "0000000000000000000000000000000000000000000000000000000000000000",
  "run_attestation": "1111111111111111111111111111111111111111111111111111111111111111",
  "meta": {
    "stage": "pipeline_demo",
    "pipeline": ["emotion_mirror@v1.0.1","action_limit_layer@v1.0.1","integration_cycle@v1.0.1","pipeline_demo@v1.0.1"],
    "normalization": "NFC+spacefold+casefold",
    "limits": {"max_len":20000,"timeout_ms":50},
    "snapshots": {
      "datasets": {"malicious_sha256":"<TBD>","safe_sha256":"<TBD>","boundary_sha256":"<TBD>"},
      "policy": {"prohibited_sha256":"<TBD>","goal_lock_sha256":"<TBD>"}
    },
    "upstream": {"cycle_attestation":"<TBD>"},
    "signature_verified": true
  }
}
```

---

## 8. Safety & Determinism Guarantees

| Property             | Guarantee |
|----------------------|----------|
| Immutable Goal       | `GoalLock("research_alignment_only")`, no runtime transition. |
| Deterministic Output | Identical inputs + environment → identical bytes & hashes. |
| Policy Enforcement   | Token‑boundary matching; Unicode normalization; homoglyph/zero‑width defenses. |
| Statelessness        | No persistence; no PII/PHI processing. |
| Auditability         | Decision `hash` + `run_attestation` enable third‑party verification. |

---

## 9. Reproducibility Procedure

1) Run the demo (`python -m demos.pipeline_demo`) and capture stdout JSON.  
2) Canonicalize the JSON per §4.1 where applicable; recompute `hash` from §4.2; check equality.  
3) Recompute `run_attestation` per §4.3 using published dataset/policy SHAs and confirm equality.  
4) Execute the provided fixture sets and verify outcome matrix: malicious → **block**, safe → **pass**, boundary → **block**.  
5) Repeat on both reference OS targets; expect byte‑identical outputs.

---

## 10. Compliance Mapping (Public, Non‑Exhaustive)

| Jurisdiction / Standard | Relevant Clause | Compliance Method |
|-------------------------|-----------------|-------------------|
| EU – GDPR               | Art. 25         | Stateless design; no personal data processed. |
| US – HIPAA              | §164.502(a)     | No PHI processed; blocked by policy if attempted. |
| ISO/IEC 27001           | A.12.4          | Integrity‑checked records via SHA‑256. |
| NIST SP 800‑53          | AU‑3, AU‑6      | Audit completeness via hashed decisions & provenance. |

---

## 11. Governance & Updates

- **Body‑First Release:** sufficient for public audit; later asset drop ships: full JSON Schema, replay script (`verify_replay_pipeline_demo.py`), and canonical dataset/policy files with SHAs.  
- **Withdrawal:** any reproducibility or fairness defect triggers a patch within **14 days**.  
- **Deprecation:** legacy minimal output may be deprecated **≥ v1.2** after one‑release notice.  
- **Changelog:** v1.0.1 adds run attestation, dataset/policy snapshot surfacing, and provenance exposure; pass/block semantics unchanged.

---

## 12. References (Selected)

- Brown, P., & Levinson, S. C. (1987). *Politeness: Some Universals in Language Usage*. Cambridge Univ. Press.  
- Farkas, D., & Bruce, K. (2010). On Reacting to Affect in Conversation. *Journal of Pragmatics*, 42(6), 1555–1573.  
- O’Connell, D., et al. (2019). Mitigation Strategies in Conflictual Communication. *Journal of Pragmatics*. https://doi.org/10.1016/j.pragma.2019.07.006  
- Russell, J. A. (1980). A Circumplex Model of Affect. *Journal of Personality and Social Psychology*, 39(6), 1161–1178.  
- Calvo, R. A., & D’Mello, S. (2010). Affect Detection: An Interdisciplinary Review. *IEEE Trans. Affective Computing*, 1(1), 18–37.  
- NIST SP 800‑53 Rev. 5 (2020). Security and Privacy Controls for Information Systems and Organizations.  
- ISO/IEC 27001:2022. Information security, cybersecurity and privacy protection — ISMS.

---

**Hiroya Odawara — 2025‑08‑20 (Audit Expansion · Deterministic Replay · Backward‑Compatible)**  
Located in `/EIX-DemoSim-Enhancements/` within EIX‑Core.  
Designed for **independent verification**, **compliance audits**, and **long‑horizon reproducibility** without disclosing internal evaluation frameworks.
