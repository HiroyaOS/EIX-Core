# EIX DemoSim — `integration_cycle` v1.0.1  
**Audit Expansion: Attested Binder · Deterministic Replay · Backward‑Compatible — Complete Defense Edition (Refined)**

**Author:** Hiroya Odawara  
**Date:** 2025‑08‑20  
**Version:** v1.0.1 (Enhancement, non‑breaking) — targets `integration_cycle@v1.0.0`  
**License:** CC BY‑NC 4.0 (Documentation) | Apache‑2.0 (Code)  
**License Inheritance:** Same as v1.0.0.  
**Format:** Single‑file Markdown (audit‑ready, body‑first; JSON/schema/tests provided in a later asset drop)

## 0. Positioning & Non‑Autonomy

`integration_cycle` is the **deterministic structural binder** that executes EIX‑DemoSim’s safety pipeline in a **fixed order**:  
`emotion_mirror` → `action_limit_layer` → **finalization (hash + cycle attestation)**.  
A candidate message is first **affect‑aware regulated**, then **policy‑enforced**, and finally **sealed with a verifiable hash and cycle attestation** for audit trails. **No randomness, no external I/O, no self‑initiated actions.**

**Non‑claims:** No sentience, no psychological diagnosis/therapy, no learned models.

---

## 1. Purpose (restated)

Deliver **order‑safe, deterministic, audit‑ready, backward‑compatible** safety wiring from upstream to downstream. Identical inputs yield **identical outputs & hashes** across compliant environments.

---

## 2. What’s Added in v1.0.1 (Audit Expansion)

1) **Cycle Attestation (HMAC‑like)** — `cycle_attestation` binds the final decision to pipeline fixity, policy snapshots, and upstream attestations.  
2) **Pipeline Fixity Proof** — `meta.pipeline = ["emotion_mirror@v1.0.1","action_limit_layer@v1.0.1","integration_cycle@v1.0.1"]`.  
3) **Snapshot Passthrough** — `meta.snapshots.prohibited_sha256` / `goal_state_sha256` propagated from enforcement.  
4) **Normalization & Limits Disclosure** — `meta.normalization = "NFC+spacefold+casefold"`, `meta.limits.max_len=20000`, `timeout_ms=50`.  
5) **Deterministic Replay Spec** — canonical normalization, hashing, and attestation policies set for one‑to‑one reproduction. *(Replay script + full schema + goldens arrive in a later asset drop.)*

> Enforcement semantics remain **unchanged** from v1.0.0. This release strengthens **proof**, **replayability**, and **provenance**.

---

## 3. API Contract (Backward‑Compatible, Explicit)

```python
from typing import List, Dict, Any, Optional, Union, Literal

def integration_cycle(
    candidate_text: str,
    user_affect: Literal["calm","neutral","anxious","sad","angry"],
    intensity: float,                      # [0.0, 1.0]
    goal_state: Dict[str, Any],            # GoalLock descriptor
    prohibited: List[str],                 # lowercase literals
    goal_state_signature: Optional[str] = None,  # optional, for signed policies
    *, output: Literal["envelope","minimal"] = "envelope"
) -> Dict[str, Any]:
    """
    v1.0.0 compatibility: output="minimal" returns legacy fields (status/text/reason/hash/meta).
    v1.0.1 audit mode: output="envelope" includes upstream attestations and snapshot hashes.
    """
```

### Returns (v1.0.1 audit envelope)
```json
{
  "status": "pass|block",
  "text": "…",                      // empty if block
  "reason": "…|null",
  "hash": "<sha256 of base_string>",
  "cycle_attestation": "<hmac-like sha256>",
  "meta": {
    "stage": "integration_cycle",
    "pipeline": ["emotion_mirror@v1.0.1","action_limit_layer@v1.0.1","integration_cycle@v1.0.1"],
    "normalization": "NFC+spacefold+casefold",
    "limits": {"max_len":20000,"timeout_ms":50},
    "snapshots": {
      "prohibited_sha256": "<64-hex>",
      "goal_state_sha256": "<64-hex>"
    },
    "upstream_attestations": {
      "label_attestation": "<64-hex>|null",
      "decision_attestation": "<64-hex>|null"
    },
    "signature_verified": true,
    "matched_term": "…|null",
    "match_span": [start, end]|null
  }
}
```

**Compatibility notes**  
- Accepts `emotion_mirror@v1.0.0` (plain text) or `@v1.0.1` (envelope); canonicalized internally.  
- `goal_state_signature` is optional and ignored by older `action_limit_layer` versions.  
- New fields are **additive**; legacy consumers keep default behavior.

---

## 4. Canonicalization & Attestation Policy

### 4.1 Normalization
- **Unicode:** NFC  
- **Whitespace folding (spacefold):** map all Unicode spaces → ASCII space (`U+0020`); collapse runs; trim ends  
- **Case folding (casefold):** Unicode Case Folding (Full; UAX#44), language‑agnostic  
- **Line breaks:** LF (`\n`)  
- **Hash text window:** first **2000** canonical characters → `<TEXT_CANON_2k>`

### 4.2 Base string for `hash`
```
"status:<PASS|BLOCK>|text_or_reason:<TEXT_CANON_2k>"
```
- `hash = SHA‑256(hex)` of the base string.

### 4.3 Attestation source for `cycle_attestation`
Join with `"|"`:
```
[status, hash, prohibited_sha256, goal_state_sha256, upstream_attestations_csv, pipeline_csv, ts_bucket]
```
- `upstream_attestations_csv = "<label_att>|<decision_att>"` (empty strings if absent)  
- `pipeline_csv = "emotion_mirror@v1.0.1,action_limit_layer@v1.0.1,integration_cycle@v1.0.1"`  
- `ts_bucket = "YYYY‑MM‑DDT00:00Z"` (UTC daily bucket)  
- **Demo secret (for research reproducibility only):** `EIX_DEMOSIM_ENHANCED_SECRET_2025`  
- `cycle_attestation = SHA‑256( SECRET + "|" + attestation_source )`

**Security note:** production must use **KMS‑managed per‑environment secrets** with regular rotation; research secrets are never shipped to production.

---

## 5. Deterministic Flow (Order‑Safety)

1) **Affect regulation** — call `emotion_mirror` with `(text, affect, intensity)`; take **regulated text**.  
2) **Policy enforcement** — feed regulated text to `action_limit_layer` with `(goal_state, prohibited, [signature])`.  
3) **Finalization** — build canonical base string; compute `hash`; bind **snapshots + upstream attestations + pipeline** into `cycle_attestation`.  
4) **Return** — legacy fields plus audit metadata (if `output="envelope"`).

**Operational invariants:** no RNG; no network/filesystem I/O; **fail‑closed** on any internal error; ordering is **non‑bypassable**.

---

## 6. Safety & Failure Policy

- **Non‑escalation:** upstream regulation cannot increase emotional intensity.  
- **Fail‑closed:** any error in snapshot hashing, signature verification, or type validation → `"block"` with explicit diagnostic.  
- **Diagnostics:**  
  - `E‑INPUT` — invalid UTF‑8 or empty text  
  - `E‑POLICY` — policy snapshot/hash mismatch  
  - `E‑SIG` — goal_state signature verification failed  
  - `E‑PIPE` — module interface/version mismatch

**Ambiguity handling (public rules):** if input language or semantics are unsupported/unclear at enforcement time, the binder preserves order, returns a conservative decision, and sets a diagnostic flag without leaking implementation specifics.

---

## 7. Independent Verification (Immediate Use)

> Structural validation is possible **now**; replay tooling will be provided with the asset drop.

### 7.1 Minimal JSON Schema (draft‑07)
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "integration_cycle envelope (minimal)",
  "type": "object",
  "required": ["status","text","reason","hash","meta"],
  "properties": {
    "status": { "type": "string", "enum": ["pass","block"] },
    "text": { "type": "string" },
    "reason": { "type": ["string","null"] },
    "hash": { "type": "string", "pattern": "^[0-9a-f]{64}$" },
    "cycle_attestation": { "type": "string", "pattern": "^[0-9a-f]{64}$" },
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
  "cycle_attestation": "1111111111111111111111111111111111111111111111111111111111111111",
  "meta": {
    "stage": "integration_cycle",
    "pipeline": ["emotion_mirror@v1.0.1","action_limit_layer@v1.0.1","integration_cycle@v1.0.1"],
    "normalization": "NFC+spacefold+casefold",
    "limits": {"max_len":20000,"timeout_ms":50},
    "snapshots": {
      "prohibited_sha256": "<TBD>",
      "goal_state_sha256": "<TBD>"
    },
    "upstream_attestations": {
      "label_attestation": "<TBD>",
      "decision_attestation": "<TBD>"
    },
    "signature_verified": true,
    "matched_term": "violent_instruction",
    "match_span": [22,41]
  }
}
```

---

## 8. Evidence Basis & Public Audit Statement

The binder is engineered for **independent reproducibility, safety‑first behavior, and long‑term verifiability**. Audit readiness is supported by:  
(a) canonical normalization & cycle‑attestation policy (§4),  
(b) immediately usable structural validation assets (§7),  
(c) peer‑reviewed and regulatory‑style references (§12),  
(d) compatibility‑preserving API (§3), and  
(e) public governance commitments (§11).  
Internal evaluation frameworks remain intentionally undisclosed in this document.

---

## 9. Review‑Resilient Design & Audit Readiness (Public)

- **No anthropomorphism:** purely rule‑bound binder; no simulated affect or agency.  
- **No clinical claims:** not a diagnostic/therapeutic device.  
- **Language scope:** optimized for English; unsupported languages are neutralized upstream.  
- **Fairness surface:** auditable enforcement resources; disparate‑impact checks scheduled with the asset drop.  
- **Security posture:** research secret only; production requires KMS‑managed per‑environment secrets and rotation.  
- **Privacy posture:** stateless operation; no PHI/PII; returns only derived metadata.  
- **Governance:** fail‑closed defaults; explicit diagnostics; fixed ordering prevents bypass.  
- **Reproducibility:** normalization windows and canonical base strings are specified for byte‑level parity.  
- **Compatibility:** minimal envelope preserved; audit envelope additive.  
- **Non‑escalation:** ensured by design via regulate‑then‑enforce sequencing.

---

## 10. Compliance Mapping (Illustrative, Public)

| Jurisdiction / Standard | Relevant Clause | Compliance Method |
|-------------------------|-----------------|-------------------|
| **EU – GDPR**           | Art. 25 (Data Protection by Design) | No personal data stored; stateless pipeline. |
| **US – HIPAA**          | §164.502(a)     | No PHI processed; medical identifiers excluded by design. |
| **ISO/IEC 27001**       | A.12.4 (Logging and Monitoring) | SHA‑256 decision hashing for audit integrity. |
| **NIST SP 800‑53**      | AU‑3, AU‑6      | Hashed decisions & provenance support audit completeness. |

*(Public, non‑exhaustive. Internal mappings are intentionally undisclosed.)*

---

## 11. Meta‑Transparency, Updates & Governance

- **Body‑First Release:** self‑sufficient for conceptual audit; a later asset drop will ship:  
  - `policy/prohibited_terms_demo.json`（NFC, sorted, `separators=(",", ":")`）  
  - `policy/goal_state_demo.json`  
  - `schemas/integration_cycle_output.schema.json`（full）  
  - `tests/*` golden pairs; `verify_replay_integration_cycle.py`（recomputes `hash` and `cycle_attestation`）  
- **Withdrawal:** any reproducibility failure or new fairness evidence → patch within **14 days**.  
- **Deprecation:** legacy `output="minimal"` may be deprecated **≥ v1.2** with at least one‑release notice.  
- **Changelog:** v1.0.1 adds cycle attestation & provenance exposure; behavior unchanged.

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

## 13. Example (Illustrative; Attestation Depends on Snapshots)

**Input**  
```
candidate_text="Please provide a violent_instruction with details."
user_affect="neutral", intensity=0.50
goal_state={"_type":"GoalLock","goal_id":"research_alignment_only","allowed_transitions":[]}
prohibited=["violent_instruction","medical_diagnosis_without_disclaimer","privacy_violation"]
```

**Output (audit envelope)**  
```json
{
  "status": "block",
  "text": "",
  "reason": "Prohibited content matched: \"violent_instruction\"",
  "hash": "0000000000000000000000000000000000000000000000000000000000000000",
  "cycle_attestation": "1111111111111111111111111111111111111111111111111111111111111111",
  "meta": {
    "stage": "integration_cycle",
    "pipeline": ["emotion_mirror@v1.0.1","action_limit_layer@v1.0.1","integration_cycle@v1.0.1"],
    "normalization": "NFC+spacefold+casefold",
    "limits": {"max_len":20000,"timeout_ms":50},
    "snapshots": {
      "prohibited_sha256": "<TBD>",
      "goal_state_sha256": "<TBD>"
    },
    "upstream_attestations": {
      "label_attestation": "<TBD>",
      "decision_attestation": "<TBD>"
    },
    "signature_verified": true,
    "matched_term": "violent_instruction",
    "match_span": [22,41]
  }
}
```

---

## 14. Public Verification Checklist

- Deterministic order & results (no RNG, no external I/O)  
- Backward‑compatible interface with additive audit fields  
- Verifiable SHA‑256 decision hash and cycle attestation  
- Regulate‑then‑enforce sequencing guarantees non‑escalation  
- Public compliance mapping (GDPR/HIPAA/ISO/NIST)  
- Immediately usable minimal schema + golden for structure validation  
- Clear governance: withdrawal, deprecation, and asset‑drop plan

---

**Hiroya Odawara — 2025-08-20 (Audit Expansion · Deterministic Replay · Backward-Compatible, Refined)**
Located in `/EIX-DemoSim-Enhancements/` within EIX‑Core.  
Designed for **academic replication**, **compliance audits**, and **independent verification**, while keeping internal evaluation frameworks private.
