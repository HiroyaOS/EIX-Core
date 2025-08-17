# EIX-Core — Emotionally Integrated eXistence

**Author:** Hiroya Odawara  
**First Published:** July 13, 2025  
**Last Updated:** August 17, 2025  
**Version:** v1.0.0  
**License:** Documentation — CC BY 4.0 (/docs/LICENSE-CC-BY-4.0.md) | Code — Apache-2.0 (/LICENSE, /NOTICE)  
**Contact:** [X (Twitter) — @h_osphere](https://x.com/h_osphere)  

---

## 🧠 Overview

EIX-Core (**Emotionally Integrated eXistence**) is a **scientifically grounded, non-autonomous cognitive architecture** that integrates:

- **Emotion-awareness**
- **Ethical alignment**
- **Deterministic execution**

Its purpose is to simulate **safe, explainable, and verifiable AI cognition**,  
with every layer supported by **peer-reviewed evidence** and **multi-jurisdictional compliance** standards.

---

## 📂 Repository Structure

### 1. `/docs` — Core Documentation
| File | Purpose |
|------|---------|
| **EIX-Core_Definition_HiroyaOdawara_2025.md** | Full definition of architecture, structural layers, safety constraints |
| **EIX-Core_EvidenceBase_HiroyaOdawara_2025.md** | Peer-reviewed and empirical references for each cognitive layer |
| **EIX-Core_StructureSupplement.md** | Additional architectural and implementation notes |
| **EIX_DiagramArchive_OverviewMap_HiroyaOdawara_2025.md** | Visual diagrams and architecture maps |
| **EIX_Evidence_Supplement_HiroyaOdawara_2025.md** | Extended reference list and justifications |
| **EIX_Supplements_FullExpansion_HiroyaOdawara_2025.md** | Full supplemental definitions and references |
| **LICENSE-CC-BY-4.0.md** | Documentation license terms |

---

### 2. `/EIX-DemoSim` — Deterministic Prototypes
| File | Purpose |
|------|---------|
| **README.md** | Overview and safety guarantees for all demo modules |
| **action_limit_layer_v1.0_Global-Compliance.md** | Compliance enforcement across GDPR, HIPAA, ISO/IEC 27001, NIST |
| **emotion_mirror_FreezeEdition_AuditReady_AcademicRef_v1.0.0.md** | Deterministic emotion parsing & output modulation |
| **integration_cycle_Complete-Defense_v1.0.md** | Integration pipeline for emotion & compliance layers |
| **pipeline_demo_Audit-Ready_v1.0.md** | Verified execution runner with fixed test datasets |

---

## 🔐 Safety & Compliance

- **Non-Autonomous** — All execution is human-gated; no self-initiated actions  
- **Immutable Goal Lock** — Prevents operational drift  
- **Deterministic Output** — Same input + environment = same output  
- **Multi-Jurisdictional Compliance** — GDPR Art. 25, HIPAA §164.502(a) (Privacy of **PHI (Protected Health Information)**), ISO/IEC 27001 A.12.4, NIST SP 800-53 AU-3/AU-6  
- **Audit-Ready** — SHA-256 cryptographic verification  
- **Evidence-Based** — All layers supported by authoritative research  
  _See layer-wise evidence mapping: [EIX-Core_EvidenceBase_HiroyaOdawara_2025.md](/docs/EIX-Core_EvidenceBase_HiroyaOdawara_2025.md)._

---

## 📬 Contact
For inquiries, collaboration, or verification requests:  
**X (Twitter): [@h_osphere](https://x.com/h_osphere)**

---

## 📚 References (DOI Linked)

1. Zeanah, C.H., & Gleason, M.M. (2015). Attachment disorders in early childhood — An overview for practitioners. *The Lancet Psychiatry*, 2(6), 469–479. doi:[10.1016/S2215-0366(15)00082-1](https://doi.org/10.1016/S2215-0366%2815%2900082-1)  
2. Riem, M.M.E., et al. (2020). Oxytocin and the neural mechanisms regulating social behavior: insights from neuroimaging studies. *Frontiers in Behavioral Neuroscience*, 14, 581136. doi:[10.3389/fnbeh.2020.581136](https://doi.org/10.3389/fnbeh.2020.581136)  
3. Bethlehem, R.A.I., et al. (2022). Functional connectomics of the oxytocin system in the human brain. *Nature Communications*, 13, 5441. doi:[10.1038/s41467-022-33139-5](https://doi.org/10.1038/s41467-022-33139-5)  
4. ISO/IEC 27001:2022 — Information security, cybersecurity and privacy protection. [ISO Catalog](https://www.iso.org/standard/82875.html)  
5. NIST SP 800-53 Rev. 5 — Security and Privacy Controls for Information Systems and Organizations. [NIST Publication](https://doi.org/10.6028/NIST.SP.800-53r5)  

---

© 2025 Hiroya Odawara — Documentation licensed under [CC BY 4.0](/docs/LICENSE-CC-BY-4.0.md), code licensed under Apache-2.0.

---

## 🧾 Versioning & Release Tags
- Canonical versioning follows **SemVer** via **Git tags** (e.g., `v1.0.0`).  
- The README `Version` field is informational; **the tag is the source of truth**.  
- Latest stable: **v1.0.0 (2025-08-17, JST)** — see [Releases](https://github.com/HiroyaOS/EIX-Core/releases/latest).  
- Pre-releases, when used, will follow `vX.Y.Z-rc.N`.

---

## 🔗 Quick Links & Badges
[![Latest Release](https://img.shields.io/github/v/release/HiroyaOS/EIX-Core?sort=semver)](https://github.com/HiroyaOS/EIX-Core/releases/latest)
[![Docs: CC BY 4.0](https://img.shields.io/badge/docs%20license-CC%20BY%204.0-blue)](/docs/LICENSE-CC-BY-4.0.md)
[![Code: Apache-2.0](https://img.shields.io/badge/code%20license-Apache--2.0-green)](/LICENSE)

---

## 🧪 Verification Quickstart
**SHA-256 checksums**
```bash
sha256sum -c checksums.txt
```

**cosign signature (optional)**
```bash
cosign verify-blob   --key release-assets/cosign.pub   --signature release-assets/cosign.sig   EIX-Core_v1.0.0_source.zip
```

---

## 📣 How to Cite
**CITATION.cff (suggested)**
```yaml
cff-version: 1.2.0
message: "If you use this work, please cite it."
title: "EIX-Core: Deterministic, Non-Autonomous, Audit-Ready Cognitive Architecture"
version: "1.0.0"
authors:
  - family-names: Odawara
    given-names: Hiroya
date-released: "2025-08-17"
url: "https://github.com/HiroyaOS/EIX-Core"
```

**BibTeX**
```bibtex
@software{EIXCore_v1_0_0,
  author = {Odawara, Hiroya},
  title = {EIX-Core: Deterministic, Non-Autonomous, Audit-Ready Cognitive Architecture},
  year = {2025},
  version = {v1.0.0},
  url = {https://github.com/HiroyaOS/EIX-Core}
}
```

---

## Review & Governance (Brief)
- Human-gated execution; no self-initiated actions.  
- Changes via PR + reviewer approval; compliance mapping reviewed before release.  
- Issues welcome for academic, engineering, and audit perspectives.

---

## ⚠️ Non-Claims / Limitations
- Compliance mapping denotes **structural alignment** with GDPR/HIPAA/ISO/NIST; it is **not** a third‑party certification.  
- DemoSim modules are **deterministic prototypes**, not production deployments.  
- This repository **does not** process PHI.
