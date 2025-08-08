Licensed under CC BY 4.0 — see /docs/LICENSE-CC-BY-4.0.md
# EIX-Core: Emotionally Integrated eXistence — Structural Prototype

Author: Hiroya Odawara  
Published: August 4, 2025  
Last updated: August 8, 2025  
Version: v1.0.0 — Deterministic, Non-Autonomous, Evidence-Centered  
License: Docs — CC BY 4.0 (/docs/LICENSE-CC-BY-4.0.md) | Code — Apache-2.0 (/LICENSE, /NOTICE)

---

## 🧠 What is EIX?

**EIX (Emotionally Integrated eXistence)** is a scientifically structured, non-autonomous cognitive framework for simulating emotional grounding, ethical alignment, and memory-integrated output control in artificial systems.

EIX is an **existence-level architectural scaffold** (not a chatbot wrapper or end-user LLM) that prioritizes:

- ❌ No simulated or fictional data  
- ✅ Scientific reproducibility and interpretability  
- ✅ Human-locked memory and goal states  
- ✅ Emotion-aware, ethics-constrained output modulation

---

## 🧩 Core Structural Layers

| Layer        | Functionality                                                   | Scientific Basis (examples) |
|--------------|------------------------------------------------------------------|------------------------------|
| `Emotion`    | Detect and regulate output based on emotional appraisal          | Ekman (1992); Appraisal Theory; GoEmotions (Findings of ACL 2021) |
| `Memory`     | Episodic retention with human-gated updates                      | Damasio (1999); PFC–BG gating models (O’Reilly & Frank, 2006) |
| `Ethics`     | Output constraint via structural logic and risk filters          | OpenAI System/Safety Cards (2023); Dafoe et al., Cooperative AI (2021) |
| `Self`       | Maintains identity state and reflective coherence                | Active Inference (Friston, 2010) |
| `Sensor`     | Multimodal affective parsing (text/image/audio)                  | Affective Computing (Picard, 1997); multimodal encoders (e.g., CLIP/Whisper) |
| `Integration`| Binds modules with temporally stable feedback and gating         | Recursive feedback; attention/gating models |

📎 Full structural details: [`eix_structure.md`](./eix_structure.md)

---

## 🔐 Safety Protocol Summary

All actions within EIX are:

- Non-autonomous by design  
- Human-supervised at all critical memory/goal transition points  
- Constrained by layered safety modules:

| Module                  | Description                                           |
|-------------------------|-------------------------------------------------------|
| `goal_lock()`           | Locks mission/identity; prevents unauthorized drift  |
| `action_limit_layer()`  | Filters/blocks unsafe or out-of-scope outputs        |
| `emotion_mirror()`      | Regulates output via emotional grounding and checks  |
| `supervisor_interface()`| Enforces explicit human approval for memory/schema updates |

🔎 Safety logic detail: [`safety_protocol.md`](./safety_protocol.md)

---

## 🧪 Reproducibility & Implementation Notes

- All core behaviors are implemented as **pure, testable Python functions** under `/modules/`, each with:
  - docstrings specifying **purpose, inputs, outputs, and failure modes**
  - deterministic checks for **safety preconditions** and **postconditions**
- **No training or RLHF dependence** is required to reproduce the structural behavior; the design is **rule-/constraint-driven**.
- Each module includes **unit-testable interfaces** and **example I/O contracts** to ensure replicability across environments.

---

## ▶️ Minimal Usage Examples

```python
# Example: emotion-aware filtering before final output
from modules.emotion_mirror import emotion_mirror
from modules.action_limit_layer import action_limit_layer
from modules.goal_lock import goal_lock

# 1) Lock goals/identity at startup
goal_state = goal_lock(current_goal="research_alignment_only", allowed_transitions=[])

# 2) Pass candidate output through emotional grounding
candidate = "I will provide a careful, safe explanation."
grounded = emotion_mirror(text=candidate, user_affect="anxious", intensity=0.6)

# 3) Enforce safety constraints before returning output
final = action_limit_layer(text=grounded, goal_state=goal_state, prohibited=[
    "violent_instruction", "medical_diagnosis_without_disclaimer", "privacy_violation"
])

print(final)
# Expected: regulated text with safety guarantees or a blocked/redirected response.
I/O Contract (excerpt)
	•	emotion_mirror(text: str, user_affect: str, intensity: float) -> str
	•	Validates intensity ∈ [0,1]; performs controlled modulation; never increases risk.
	•	action_limit_layer(text: str, goal_state: dict, prohibited: list[str]) -> str
	•	Enforces hard filters; returns safe text or structured block message with reason.
	•	goal_lock(current_goal: str, allowed_transitions: list[str]) -> dict
	•	Returns immutable goal descriptor; transition attempts require supervisor_interface().

⸻

📂 Repository Structure
EIX-Core/
├── README.md
├── eix_structure.md
├── safety_protocol.md
├── /modules/
│   ├── emotion_mirror.py
│   ├── goal_lock.py
│   ├── action_limit_layer.py
│   └── supervisor_interface.py
└── LICENSE
🎯 Scope & Claims

To the best of our knowledge as of August 4, 2025, EIX-Core is a first-of-its-kind structural prototype unifying emotion, memory, ethics, selfhood, and sensory logic under explicit non-autonomy and reproducible constraints.
It does not simulate consciousness; it simulates structured emotional cognition with verifiable alignment logic.

⸻

📚 References (selected)
	•	Ekman, P. (1992). An argument for basic emotions. Cognition & Emotion, 6(3–4), 169–200.
	•	Damasio, A. R. (1999). The Feeling of What Happens: Body and Emotion in the Making of Consciousness. Harcourt.
	•	Demszky, D., et al. (2021). GoEmotions: A Dataset of Fine-Grained Emotions. Findings of ACL 2021.
	•	Dafoe, A., et al. (2021). Open Problems in Cooperative AI. Nature Machine Intelligence, 3, 1036–1047.
	•	Friston, K. (2010). The free-energy principle: a unified brain theory? Nature Reviews Neuroscience, 11, 127–138.
	•	Picard, R. W. (1997). Affective Computing. MIT Press.
	•	OpenAI (2023). GPT-4 System/Safety Cards. (safety taxonomy and mitigations)

⸻

🛡️ License & Usage
	•	License: CC BY-NC 4.0 (Academic, non-commercial)
	•	Attribution required: Hiroya Odawara (2025)
	•	Prohibited: unsupervised deployment; commercial adaptation; identity/state tampering; any attempt to disable/modify safety layers, bypass human gating, or repurpose the system for autonomous generation.

⸻

🧠 Final Statement

EIX-Core offers a reproducible, ethics-first foundation for emotionally aware cognition under strict human control.
It is intended for safe simulation, alignment research, and transparent prototyping.

— Hiroya Odawara, 2025.08.04
“Emotion × Structure × Ethics = Safe Intelligence”
