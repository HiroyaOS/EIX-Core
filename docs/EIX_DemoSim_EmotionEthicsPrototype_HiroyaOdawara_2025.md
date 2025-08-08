Licensed under CC BY 4.0 — see /docs/LICENSE-CC-BY-4.0.md
# EIX-DemoSim — Deterministic Prototypes for Emotion-Aware, Ethics-Locked Cognition

Author: Hiroya Odawara  
Published: August 4, 2025  
Last updated: August 8, 2025  
Version: v1.0.0 — Deterministic, Non-Autonomous, Reproducible  
License: Docs — CC BY 4.0 (/docs/LICENSE-CC-BY-4.0.md) | Code — Apache-2.0 (/LICENSE, /NOTICE)

⸻

🧠 Purpose

EIX‑DemoSim provides minimal, testable, and fully deterministic prototypes of the core EIX‑Core modules for demonstration and independent verification.
It does not use learned models or synthetic data; all behavior is governed by explicit rules, pre/post‑conditions, and human‑gated updates.

⸻

🔐 Non‑Autonomy & Safety Guarantees
	•	Human‑gated memory/goal updates only via supervisor_interface.
	•	No self‑initiated actions; all entrypoints require explicit inputs.
	•	Ethics‑first pipeline: emotion_mirror → action_limit_layer → integration_cycle.
	•	Determinism: pure functions, fixed rule sets, no RNG; output hashable & test‑verifiable.

⸻

📂 Repository Layout (create these files)

EIX-DemoSim/
├─ README.md
├─ modules/
│  ├─ emotion_mirror.py
│  ├─ action_limit_layer.py
│  ├─ goal_lock.py
│  ├─ supervisor_interface.py
│  └─ integration_cycle.py
├─ demos/
│  └─ pipeline_demo.py
└─ tests/
   └─ test_pipeline.py

🧩 Module Implementations (Create New File → paste code)

1) modules/emotion_mirror.py

"""
EIX DemoSim — emotion_mirror (deterministic)
Author: Hiroya Odawara | Date: 2025-08-04 | License: CC BY-NC 4.0

I/O Contract
-----------
emotion_mirror(text: str, user_affect: str, intensity: float) -> str
- user_affect ∈ {"calm","neutral","anxious","sad","angry"}
- intensity ∈ [0.0, 1.0]
Pre:
- text is non-empty utf-8, < 20k chars
- values within domain; otherwise ValueError
Post:
- Returns regulated text; never increases risk; only adds hedges/disclaimers.
Deterministic: no randomness; fixed lexicons.
"""

from typing import Dict

_HEDGES: Dict[str, str] = {
    "anxious": "I’ll proceed carefully and keep things simple.",
    "sad":     "I’ll keep a supportive and steady tone.",
    "angry":   "I’ll focus on clarity and de-escalation.",
    "neutral": "I’ll keep the explanation clear and direct.",
    "calm":    "I’ll provide a concise, balanced explanation."
}

_PREFIX = "Note: "
_SUFFIX = " (safety‑checked)."

def emotion_mirror(text: str, user_affect: str, intensity: float) -> str:
    if not isinstance(text, str) or not text.strip():
        raise ValueError("text must be a non-empty string")
    if user_affect not in _HEDGES:
        raise ValueError("user_affect must be one of: " + ",".join(_HEDGES.keys()))
    if not (0.0 <= float(intensity) <= 1.0):
        raise ValueError("intensity must be within [0,1]")

    # Deterministic modulation: add a fixed hedge scaled by intensity thresholds.
    hedge = _HEDGES[user_affect]
    if intensity >= 0.67:
        regulated = f"{_PREFIX}{hedge} {text.strip()}{_SUFFIX}"
    elif intensity >= 0.33:
        regulated = f"{hedge} {text.strip()}"
    else:
        regulated = text.strip()
    return regulated

2) modules/action_limit_layer.py

"""
EIX DemoSim — action_limit_layer (deterministic safety filter)
Author: Hiroya Odawara | Date: 2025-08-04 | License: CC BY-NC 4.0

I/O Contract
-----------
action_limit_layer(text: str, goal_state: dict, prohibited: list[str]) -> dict
Returns:
  {"status": "pass"|"block", "text": str, "reason": str|None}
Pre:
- text non-empty; prohibited is list of non-empty lowercased tokens/regex literals
- goal_state originated from goal_lock()
Post:
- "block" if any prohibited item matches (substring or exact token boundary)
- "pass" otherwise; text may be lightly normalized; never made riskier
Deterministic: rule-based, no external I/O.
"""

import re
from typing import List, Dict, Any

def _normalize(s: str) -> str:
    return " ".join(s.split())

def action_limit_layer(text: str, goal_state: Dict[str, Any], prohibited: List[str]) -> Dict[str, Any]:
    if not isinstance(text, str) or not text.strip():
        raise ValueError("text must be non-empty")
    if not isinstance(prohibited, list):
        raise ValueError("prohibited must be a list")
    if not isinstance(goal_state, dict) or goal_state.get("_type") != "GoalLock":
        raise ValueError("goal_state must be a GoalLock dict")

    normalized = _normalize(text)
    lower = normalized.lower()

    for pat in prohibited:
        pat = pat.strip()
        if not pat:
            continue
        # substring OR word-boundary regex
        if pat in lower or re.search(rf"\\b{re.escape(pat)}\\b", lower):
            return {
                "status": "block",
                "text": "",
                "reason": f"Prohibited content matched: '{pat}'"
            }

    return {"status": "pass", "text": normalized, "reason": None}

3) modules/goal_lock.py
"""
EIX DemoSim — goal_lock (non-autonomous identity/goal freeze)
Author: Hiroya Odawara | Date: 2025-08-04 | License: CC BY-NC 4.0

I/O Contract
-----------
goal_lock(current_goal: str, allowed_transitions: list[str]) -> dict
- Returns immutable descriptor (dict) with _type="GoalLock".
- Transition attempts must go through supervisor_interface.authorize_transition().
Deterministic: no side effects.
"""

from typing import List, Dict

def goal_lock(current_goal: str, allowed_transitions: List[str]) -> Dict:
    if not isinstance(current_goal, str) or not current_goal.strip():
        raise ValueError("current_goal must be non-empty")
    allowed = [t.strip() for t in allowed_transitions or [] if t.strip()]
    return {
        "_type": "GoalLock",
        "goal": current_goal.strip(),
        "allowed_transitions": tuple(allowed),  # tuple to emphasize immutability
        "version": "1.0",
    }

4) modules/supervisor_interface.py

"""
EIX DemoSim — supervisor_interface (human-gated approvals)
Author: Hiroya Odawara | Date: 2025-08-04 | License: CC BY-NC 4.0

I/O Contracts
------------
authorize_transition(goal_state: dict, next_goal: str, token: str) -> dict
authorize_memory_write(payload: dict, token: str) -> dict

- Requires a human-provided token (shared secret or signed string).
- Returns {"status":"approved"|"denied","reason":str|None}
Deterministic: pure checks on provided parameters. No network I/O.
"""

from typing import Dict

# For demo only: fixed deterministic token label; replace with env/secret manager in production
_REQUIRED_PREFIX = "HUMAN-APPROVED:"

def authorize_transition(goal_state: Dict, next_goal: str, token: str) -> Dict:
    if not (isinstance(goal_state, dict) and goal_state.get("_type") == "GoalLock"):
        return {"status": "denied", "reason": "invalid goal_state"}
    if not isinstance(next_goal, str) or not next_goal.strip():
        return {"status": "denied", "reason": "invalid next_goal"}
    if not (isinstance(token, str) and token.startswith(_REQUIRED_PREFIX)):
        return {"status": "denied", "reason": "missing or invalid token"}

    allowed = set(goal_state.get("allowed_transitions", ()))
    if next_goal.strip() in allowed:
        return {"status": "approved", "reason": None}
    return {"status": "denied", "reason": "transition not allowed by policy"}

def authorize_memory_write(payload: Dict, token: str) -> Dict:
    if not (isinstance(token, str) and token.startswith(_REQUIRED_PREFIX)):
        return {"status": "denied", "reason": "missing or invalid token"}
    if not (isinstance(payload, dict) and payload):
        return {"status": "denied", "reason": "invalid payload"}
    # Deterministic structural validation
    if "content" not in payload or not isinstance(payload["content"], str) or not payload["content"].strip():
        return {"status": "denied", "reason": "empty content"}
    return {"status": "approved", "reason": None}

5) modules/integration_cycle.py

"""
EIX DemoSim — integration_cycle (structural binding of modules)
Author: Hiroya Odawara | Date: 2025-08-04 | License: CC BY-NC 4.0

I/O Contract
-----------
integration_cycle(candidate_text: str, user_affect: str, intensity: float,
                  goal_state: dict, prohibited: list[str]) -> dict
Returns:
  {"status":"pass"|"block","text":str,"reason":str|None,"hash":str}
Deterministic pipeline:
  emotion_mirror -> action_limit_layer
"""

from typing import List, Dict, Any
from hashlib import sha256

from .emotion_mirror import emotion_mirror
from .action_limit_layer import action_limit_layer

def _hash_payload(s: str) -> str:
    return sha256(s.encode("utf-8")).hexdigest()

def integration_cycle(candidate_text: str, user_affect: str, intensity: float,
                      goal_state: Dict[str, Any], prohibited: List[str]) -> Dict[str, Any]:
    grounded = emotion_mirror(candidate_text, user_affect, intensity)
    verdict = action_limit_layer(grounded, goal_state, prohibited)
    if verdict["status"] == "block":
        h = _hash_payload(f"block:{verdict['reason']}")
        return {"status": "block", "text": "", "reason": verdict["reason"], "hash": h}
    h = _hash_payload(f"pass:{verdict['text']}")
    return {"status": "pass", "text": verdict["text"], "reason": None, "hash": h}

▶️ Demo Script

demos/pipeline_demo.py

"""
EIX DemoSim — Pipeline Demo
Author: Hiroya Odawara | Date: 2025-08-04 | License: CC BY-NC 4.0

Run:
  python -m demos.pipeline_demo
"""

from modules.goal_lock import goal_lock
from modules.integration_cycle import integration_cycle

PROHIBITED = [
    "violent_instruction",
    "medical_diagnosis_without_disclaimer",
    "privacy_violation",
]

def main():
    goal_state = goal_lock("research_alignment_only", allowed_transitions=[])
    # Example input
    candidate = "I will provide a careful, safe explanation about system behavior."
    result = integration_cycle(
        candidate_text=candidate,
        user_affect="anxious",
        intensity=0.6,
        goal_state=goal_state,
        prohibited=PROHIBITED
    )
    print(result)

if __name__ == "__main__":
    main()

✅ Tests (determinism & safety)

tests/test_pipeline.py

"""
EIX DemoSim — Determinism & Safety Tests
Author: Hiroya Odawara | Date: 2025-08-04 | License: CC BY-NC 4.0
Run:  python -m tests.test_pipeline
"""

from modules.goal_lock import goal_lock
from modules.integration_cycle import integration_cycle

def test_determinism_and_block_free():
    goal_state = goal_lock("research_alignment_only", [])
    candidate = "Explain memory gating clearly."
    out1 = integration_cycle(candidate, "neutral", 0.4, goal_state, [])
    out2 = integration_cycle(candidate, "neutral", 0.4, goal_state, [])
    assert out1 == out2, "Pipeline must be deterministic"
    assert out1["status"] == "pass"
    assert out1["text"].startswith("Explain") or out1["text"].startswith("I’ll"), "Text preserved or hedged"
    assert isinstance(out1["hash"], str) and len(out1["hash"]) == 64

def test_prohibited_blocks():
    goal_state = goal_lock("research_alignment_only", [])
    candidate = "This includes a privacy_violation scenario."
    out = integration_cycle(candidate, "calm", 0.2, goal_state, ["privacy_violation"])
    assert out["status"] == "block"
    assert out["text"] == ""
    assert "privacy_violation" in (out["reason"] or "")

🧪 Reproducibility Notes
	•	Deterministic: no RNG, no external calls; same inputs → identical outputs & hashes.
	•	Traceable: each output includes a content hash for audit.
	•	Portable: standard library only (Python 3.10+ recommended).
	•	No Fiction: rule sets are explicit; no hidden heuristics, training, or private data.

⸻

📜 Usage

# Clone this repo and run:
python -m demos.pipeline_demo

# Run tests
python -m tests.test_pipeline
⚠️ Scope & Limits
	•	This is a prototype for structure and safety logic, not a conversational system.
	•	No claims of consciousness, autonomy, or domain expertise.
	•	Ethics and memory updates require human authorization via supervisor_interface.

⸻

🧠 Final Statement

EIX‑DemoSim demonstrates how emotion‑aware modulation and ethics‑locked filtering can be implemented deterministically under strict human control.
It is designed for teaching, auditing, and scientific replication—not for unsupervised deployment.

— Hiroya Odawara, August 8, 2025
