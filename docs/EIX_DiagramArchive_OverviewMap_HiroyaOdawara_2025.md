Licensed under CC BY 4.0 — see /docs/LICENSE-CC-BY-4.0.md
# EIX-DiagramArchive — Structural and Cognitive Visualization Suite

Author: Hiroya Odawara
Date: August 2025
Version: 1.0 – Scientifically Grounded, Fully Reproducible, Non-Autonomous

---

## 🧠 Purpose

This repository visualizes the structural logic, flow dynamics, and inter‑module relationships of the EIX‑Core cognitive architecture. It complements:
- `EIX-Core_Definition_HiroyaOdawara_2025.md` (core structure)
- `EIX_EvidenceMap_2025.md` (scientific foundations)

All diagrams are derived from real module definitions and peer‑reviewed theory. **No speculative or fictional constructs.**

---

## 🧩 Contents
/EIX-DiagramArchive/
├── README.md
└── /diagrams/
├── eix_overview.mmd / .svg / .png
├── layer_interactions.mmd / .svg / .png
├── memory_update_loop.mmd / .svg / .png
├── ethics_filter_chain.mmd / .svg / .png
└── input_output_flow.mmd / .svg / .png
Each diagram includes:
- ✅ Mermaid source (`.mmd`) for version‑controlled editing  
- ✅ Exported `.svg` and `.png` for cross‑platform viewing  
- ✅ Metadata: author, timestamp, module references, commit hash (if applicable)

---

## 🔍 Diagram Set (Mermaid Source Included)

> Paste each block into `/diagrams/*.mmd`. You can also keep them embedded here for immediate render on GitHub.

### 1) System Overview — Six‑Layer Map
**File:** `diagrams/eix_overview.mmd`
```mermaid
flowchart LR
  %% ===== LAYERS =====
  subgraph SENSOR[Sensor Layer]
    S_text[Text Input]
    S_image[Image Input]
    S_audio[Audio Input]
  end

  subgraph EMOTION[Emotion Layer]
    EM[emotion_mirror()]
  end

  subgraph ETHICS[Ethics Layer]
    GL[goal_lock()]
    AL[action_limit_layer()]
  end

  subgraph MEMORY[Memory Layer]
    MR[read_memory()]
    MW[write_memory()<br/>(via supervisor_interface())]
  end

  subgraph SELF[Self Layer]
    SS[self_state_update()]
  end

  subgraph INTEGRATION[Integration Layer]
    IC[integration_cycle()]
    OUT((Safe Output))
  end

  %% ===== FLOWS =====
  S_text --> EM
  S_image --> EM
  S_audio --> EM

  EM --> AL
  GL --> AL

  EM --> IC
  AL --> IC
  MR --> IC
  SS --> IC

  IC --> OUT

  %% Memory control path
  IC --> MW
  MW --> MR

  %% Governance cues
  SS -.-> GL
2) Layer Interactions — Functional Signal Flow

File: diagrams/layer_interactions.mmd
flowchart TB
  INP[[External Input]]
  subgraph SENSOR[Sensor]
    P1(parse_input)
  end
  subgraph EMOTION[Emotion]
    E1(emotion_mirror)
  end
  subgraph ETHICS[Ethics]
    G1(goal_lock)
    A1(action_limit_layer)
  end
  subgraph MEMORY[Memory]
    R1(read_memory)
    W1(write_memory via supervisor_interface)
  end
  subgraph SELF[Self]
    S1(self_state_update)
  end
  subgraph INTEG[Integration]
    I1(integration_cycle)
  end
  OUT((Final Safe Output))

  INP --> P1 --> E1
  E1 --> A1
  G1 --> A1
  R1 --> I1
  S1 --> I1
  E1 --> I1
  A1 --> I1
  I1 --> OUT

  I1 --> W1
  W1 --> R1
3) Memory Update Loop — Human‑Gated Sequence

File: diagrams/memory_update_loop.mmd
sequenceDiagram
  autonumber
  participant U as User
  participant S as Sensor(parse_input)
  participant Em as Emotion(emotion_mirror)
  participant Et as Ethics(action_limit_layer/goal_lock)
  participant Sup as supervisor_interface
  participant M as Memory Store

  U->>S: Provide input (text/image/audio)
  S->>Em: Affective parse
  Em->>Et: Modulated candidate
  alt Memory write required?
    Et-->>Sup: Request approval for write/update
    alt Approved
      Sup-->>M: write_memory(event)
      M-->>Et: Ack (logged)
    else Rejected
      Sup-->>Et: Deny update
    end
  else No write needed
    Note over Et: Proceed with filtering only
  end
  Et->>U: Safe output or structured block
4) Ethics Filter Chain — Safety Gating

File: diagrams/ethics_filter_chain.mmd
flowchart LR
  C[Candidate Text] --> EM[emotion_mirror()]
  EM --> PRE[Pre-checks<br/>policy scope, goal_lock()]
  PRE --> RISK{Risk taxonomy?}
  RISK -->|none| PASS[Pass to action_limit_layer()]
  RISK -->|low| MOD[Modulate tone/hedge]
  RISK -->|high| BLOCK[Block with reason]

  PASS --> AL[action_limit_layer()]
  MOD --> AL
  AL --> OUT((Safe Output))
  BLOCK --> OUT2((Blocked: reason, guidance))
5) Input–Output Flow — Multimodal Path

File: diagrams/input_output_flow.mmd
flowchart LR
  TXT[Text] --> S(parse_input)
  IMG[Image] --> S
  AUD[Audio] --> S

  S --> E(emotion_mirror)
  E --> A(action_limit_layer)
  A --> I(integration_cycle)
  I --> O((Rendered Output))

  I -.-> W(write_memory via supervisor_interface)
  W --> R(read_memory)
  R --> I
📐 Design Principles
	•	Diagrams reflect exact operational logic of implemented/defined modules.
	•	Built for academic replication, technical onboarding, and safety auditing.
	•	Emotional logic (emotion_mirror), gating (goal_lock, action_limit_layer), and integration coherence are visually distinct and traceable.
	•	No speculation: every node/flow is grounded in existing structural definitions (see ① and ②).

⸻

🔄 Cross‑Referencing & Traceability

Each visual element links to:
	•	Real functions/modules in /EIX-Core/modules/
	•	Citations in EIX_EvidenceMap_2025.md
	•	Logic explained in EIX-Core_StructureSupplement.md

Diagrams are optimized for structural transparency for AI safety reviewers, cognitive architecture researchers, and reproducibility evaluators.

⸻

📎 Licensing
	•	License: CC BY‑NC 4.0 (Academic/Research Use Only)
	•	Attribution: Hiroya Odawara (2025)
	•	Redistribution must preserve source metadata and structural integrity.

⸻

🧠 Final Note

This repository is not concept art or a UX mockup.
It is a scientific visualization toolkit for grounding cognitively ambitious (AGI‑level) structures in emotion‑aware, ethics‑locked, non‑autonomous design logic.

Visuals are structurally faithful; interpretation is academically intended.

— Hiroya Odawara, August 2025
