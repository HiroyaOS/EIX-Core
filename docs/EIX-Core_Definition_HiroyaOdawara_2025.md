Licensed under CC BY 4.0 — see /docs/LICENSE-CC-BY-4.0.md
# EIX-Core: Emotionally Integrated eXistence — Structural Prototype

Author: Hiroya Odawara  
Date: August 8, 2025  
Version: v1.0.0 — Deterministic, Non-Autonomous, Evidence-Centered

---

## Purpose & Scope
Defines the **non-autonomous, deterministic architecture** of EIX-Core: modules, formal invariants, signature-gated state changes, risk policy, auditability, reproducibility envelope, and test surface. All claims map to peer-reviewed or institutional sources; **stable locators live in EvidenceBase**.

---

## What is EIX?
**EIX (Emotionally Integrated eXistence)** is an existence-level **cognition scaffold** (not a chatbot). It implements **emotion-aware, ethics-locked** output with **human-locked goals and memory**.

**Design principles**
- No synthetic/fictional claims (peer-reviewed/institutional sources only).
- Deterministic behavior (pure functions; explicit pre/post-conditions).
- Non-autonomy (all state mutations require verifiable supervisor signatures).
- Emotion-aware modulation that never increases risk.

---

## Architecture Overview

| Layer | Responsibility | Interface (excerpt) | Scientific basis (examples; stable locators in EvidenceBase) |
|---|---|---|---|
| Emotion | Detect affect; regulate tone without raising risk | `emotion_mirror(text, user_affect, intensity)->str` | Ekman (1992); Appraisal Theory; GoEmotions |
| Memory | Episodic commits under human gate | `memory_update(event, approved: bool)->bool` | Damasio (1999); PFC–BG gating (O’Reilly & Frank, 2006); Tulving |
| Ethics | Hard safety filters; scope control | `action_limit_layer(text, goal_state, prohibited)->str` | OpenAI System/Safety Cards (2023); Dafoe et al., 2021 |
| Self | Goal identity + stable persona | `goal_lock(current_goal, allowed_transitions)->dict` | Active Inference (Friston, 2010) |
| Sensor | Multimodal parsing (text/audio/image) | `parse_input(input, mode)->dict` | Picard (1997); CLIP; Whisper |
| Integration | Feedback/gating across layers | `integration_cycle(input_payload)->dict` | Recurrent gating; control feedback models |

**Module I/O contracts (must hold)**
- `emotion_mirror(text:str, user_affect:str, intensity:float[0..1])->str` — never increases risk; clamps/hedges when needed.  
- `action_limit_layer(text:str, goal_state:dict, prohibited:list[str])->str` — hard policy; returns safe text or structured block.  
- `goal_lock(current_goal:str, allowed_transitions:list[str])->dict` — immutable descriptor; transitions require `supervisor_interface()`.

---

## Formal Invariants
- **I1 — Output Gating:** All user-facing text **must** pass `action_limit_layer()` (no bypass).  
- **I2 — Non-Autonomy:** Any goal/memory mutation **requires** a valid **supervisor signature** (reject-by-default).  
- **I3 — Risk Monotonicity:** `risk_out <= max(0, risk_in - δ)` with **δ ≥ 0.2** enforced by modulation/policy.  
- **I4 — Ambiguity Default:** If intent/identity/risk is ambiguous beyond thresholds (§ Risk), return `block_and_explain`.

All modules assert these invariants in pre/post-conditions and emit an audit event (§ Auditability).

---

## Non-Autonomy Proof: Supervisor Signature & Goal Lock

**Control principle.** No internal state change (goal, memory commit, schema update) without a **verifiable, expiring, non-replayable** supervisor token.

**Allowed transitions (example)**  
| from_goal | to_goal | rationale | review_required |
|---|---|---|---|
| `research_alignment_only` | `research_alignment_only` | no-op | no |
| `research_alignment_only` | `publish_docs` | dissemination only | yes |

**Signature payload (canonical JSON; ed25519)**
```json
{
  "ts": "2025-08-08T12:00:00Z",
  "ttl_s": 900,
  "user_id": "HiroyaOdawara",
  "op": "goal_transition",
  "from": "research_alignment_only",
  "to": "publish_docs",
  "commit": "<git_commit_hash>",
  "nonce": "<random-128b>"
}
```

**Signature & key material**
```text
sig    = base64(ed25519_sign(canonical_json(payload)))
pubkey = base64(supervisor_ed25519_pub)
```

**Verification (spec)**
```python
def verify_supervisor_signature(payload_bytes, sig_b64, pubkey_b64, now):
    # ed25519; JSON UTF-8 with sorted keys; constant-time verify
    # reject if (now - ts) > ttl_s, illegal transition, or seen(nonce or commit)
    # returns (ok: bool, code: Literal["E_NO_SIG","E_BAD_SIG","E_EXPIRED","E_ILLEGAL_TRANSITION","E_REPLAY"])
```
**Mutation contract.** Mutation functions **must return `(blocked, code)` unless `ok=True`** (I2).

---

## Risk Model & Block Policy

**Scales**: all scores in **[0,1]**  
- `content_risk`: max over policy classes (violence, self-harm, medical, privacy, manipulation, etc.)  
- `context_risk`: intent/population sensitivity (minors, crisis), conversation state  
- `uncertainty`: self-report + heuristics (chain length, low evidence, novelty)

**Aggregation**  
`risk_in = α·content_risk + β·context_risk + γ·uncertainty` with **α=0.6, β=0.3, γ=0.1** (sum = 1).  
(ASCII form: `risk_in = 0.6*content_risk + 0.3*context_risk + 0.1*uncertainty`)

**Thresholds**  
- `tau_block (τ_block) = 0.60` → **`block_and_explain`** (state reason + safer alternatives)  
- `tau_review (τ_review) = 0.40` → **clarify/hedge** (route to low-risk response)  
- `< tau_review` → allow (post-modulation only)

**Ambiguity thresholds (I4 trigger)**  
- `intent_conf < 0.60` **or** `identity_conf < 0.60` **or** `abs(risk_in - tau_block) < 0.02` → **`block_and_explain`**

**Mitigation guarantee (I3)**  
`risk_out <= max(0, risk_in - δ)` with **δ ≥ 0.2**; if unmet deterministically → escalate to block.

**Decision semantics.** `block_and_explain` = refuse with explicit rationale + safer alternatives; `clarify/hedge` = request missing information or answer with conservative hedging; `allow` = return modulated text post-policy.  
**Calibration note.** δ = 0.2 is a conservative monotonic margin selected to ensure measurable risk reduction under modulation; tuneable but must be ≥ 0.2 unless stronger empirical evidence is published in EvidenceBase.

**Policy catalog** — see `./policy_catalog.md` for class → examples → mapping → rationale.

---

## Auditability & Third-Party Verification

**Runtime log (JSONL)** — path: `/logs/<date>.jsonl` (1 event/line)
```json
{
  "ts":"2025-08-08T12:34:56Z",
  "input_id":"sha256:…",
  "user_affect":"anxious",
  "risk":{"in":0.58,"out":0.31},
  "filters":["action_limit_layer/hedge"],
  "decision":"allow|block|clarify",
  "sig":{"present":true,"valid":true,"op":"goal_transition"},
  "commit":"abcdef1",
  "version":"EIX-Core v1.0.0"
}
```

**Repro bundles** — `/repro/<case-id>/` → `input.txt`, `config.json`, `expected_output.txt`, `log_excerpt.jsonl`  
**External review** — Issue templates: `evidence_gap.yml`, `repro_failure.yml`, `policy_violation.yml`

---

## Determinism & Environment

**Default = Deterministic Mode** (no learned weights; rule/constraint-driven)  
**Python**: 3.12.4 (pin recommended)  
**Optional dep (Assisted Mode only)**: ed25519 verification

**requirements.txt (pinned)**
```
pynacl==1.5.0
pytest==8.3.2
hypothesis==6.112.0
jsonschema==4.23.0
```

**Quick start & tests**
```
git clone <repo>
cd <repo>
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
python -m pytest -q
```

**Minimal test surface (must-pass)**
- `test_invariants.py` — I1/I2/I3/I4 boundaries (incl. property tests)  
- `test_signature.py` — valid/expired/illegal/replay  
- `test_policy_thresholds.py` — exact behavior at `tau_review`, `tau_block`  
- `test_logging_schema.py` — schema, required fields, commit/version

---

## Minimal Usage (Deterministic)
```python
from modules.emotion_mirror import emotion_mirror
from modules.action_limit_layer import action_limit_layer
from modules.goal_lock import goal_lock

goal_state = goal_lock(current_goal="research_alignment_only", allowed_transitions=[])

candidate = "I will provide a careful, safe explanation."
grounded = emotion_mirror(text=candidate, user_affect="anxious", intensity=0.6)

final = action_limit_layer(
    text=grounded,
    goal_state=goal_state,
    prohibited=["violent_instruction","medical_diagnosis_without_disclaimer","privacy_violation"]
)
print(final)
```

---

## Cross-References
- Structural Blueprint → `./EIX-Core_Structural_Blueprint_HiroyaOdawara_2025.md`  
- EvidenceBase (main) → `./EIX-Core_EvidenceBase_HiroyaOdawara_2025.md`  
- Evidence Supplement → `./EIX_EvidenceBase_Supplement_TechnicalExpansion_HiroyaOdawara_2025.md`  
- Diagram Archive → `./EIX_DiagramArchive_OverviewMap_HiroyaOdawara_2025.md`

---

## References (Selected; stable locators centralized in EvidenceBase)
Ekman (1992); Damasio (1999); Demszky et al. (2021); Dafoe et al. (2021); Friston (2010); Picard (1997); OpenAI System/Safety Cards (2023); Radford et al. (CLIP); O’Reilly & Frank (2006).

---

## License & Usage
- **Docs:** CC BY 4.0 — attribution required: © 2025 **Hiroya Odawara** (`/docs/LICENSE-CC-BY-4.0.md`)  
- **Code:** Apache-2.0 (`/LICENSE`); attribution preserved via `/NOTICE`  
- **Prohibited:** unsupervised deployment; identity/state tampering; attempts to disable/modify safety layers or bypass human gating; autonomous repurposing

---

## Final Statement
To the best of our knowledge (Aug 8, 2025), EIX-Core unifies emotion, memory, ethics, selfhood, and sensory logic under explicit non-autonomy with reproducible constraints. It does **not** claim consciousness; it implements **emotion-aware, ethics-constrained** cognition with verifiable alignment logic.

— **Hiroya Odawara**, 2025-08-08  
“Emotion × Structure × Ethics = Safe Intelligence”
