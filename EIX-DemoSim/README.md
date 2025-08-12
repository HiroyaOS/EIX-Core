# EIX-DemoSim — Deterministic Prototypes for Emotion-Aware, Ethics-Locked Cognition

**Author:** Hiroya Odawara  
**Published:** August 4, 2025  
**Last updated:** August 13, 2025  
**Version:** v1.0.0 — Deterministic, Non-Autonomous, Reproducible (Freeze Edition)  
**License:** [Docs — CC BY 4.0](https://github.com/HiroyaOS/EIX-Core/blob/main/docs/LICENSE-CC-BY-4.0.md) | [Code — Apache-2.0](https://github.com/HiroyaOS/EIX-Core/blob/main/LICENSE)

---

## 🧠 Purpose

EIX‑DemoSim provides minimal, testable, and fully deterministic prototypes of the core EIX‑Core modules for demonstration and independent verification.  
It does not use learned models or synthetic data; all behavior is governed by explicit rules, pre/post‑conditions, and human‑gated updates.

This module is part of `/docs/EIX-DemoSim/` in the **EIX-Core** repository.

---

## 🔐 Non‑Autonomy & Safety Guarantees

* **Human-gated updates only** — Memory and goal updates occur solely via `supervisor_interface` calls.
* **No self‑initiated actions** — Self-initiated actions are defined as any state transition or output generation triggered without an explicit supervisor call.
* **Ethics-first pipeline** — `emotion_mirror` → `action_limit_layer` → `integration_cycle`.
* **Determinism** — Pure functions, fixed rule sets, no RNG; output is hashable & test‑verifiable.

---

## 🧪 Determinism Verification

To confirm deterministic behavior across environments:

```bash
pytest --seed=42
```
All output hashes must match across independent runs and environments.

---

## 📂 Repository Layout

```text
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
```

---

## 📜 Implementation Notes

* **Part of EIX-Core/docs/EIX-DemoSim/** — This ensures all demo prototypes remain under version-controlled documentation for traceability.
* Designed for **academic replication** — All modules follow strict naming, logging, and deterministic constraints.
* Intended for **auditors, researchers, and safety engineers** evaluating EIX-Core's emotion-aware, ethics-locked cognition.

---

**Hiroya Odawara — August 13, 2025 (Freeze Edition)**
