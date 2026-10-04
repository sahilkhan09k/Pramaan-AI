# 🏅 Pramaan AI
### AI-Assisted NSQF Skill Assessment Platform — *Smart India Hackathon 2026*
**Team:** AXIOM

> **Pramaan AI** takes an informal worker from self-declaration to a signed, NSQF-aligned Skill Passport — fully offline. *The AI suggests; the human assessor decides.*

---

## 📋 Table of Contents

1. [Overview](#overview)
2. [Proposed Solution](#proposed-solution)
3. [System Architecture](#system-architecture)
4. [Technical Stack](#technical-stack)
5. [Component Breakdown](#component-breakdown)
   - [Client Layer (PWA)](#1-client-layer-pwa)
   - [Application Layer (API Gateway)](#2-application-layer-api-gateway)
   - [AI Service Layer (FastAPI)](#3-ai-service-layer-fastapi)
   - [Data Layer](#4-data-layer)
   - [Trust & Compliance Layer](#5-trust--compliance-layer)
6. [Offline-First Design](#offline-first-design)
7. [Assessment Workflow](#assessment-workflow)
8. [AI vs. Human Decision Boundaries](#ai-vs-human-decision-boundaries)
9. [Calibration & Scoring](#calibration--scoring)
10. [Impact & Benefits](#impact--benefits)
11. [Feasibility Analysis](#feasibility-analysis)
12. [Challenges & Mitigations](#challenges--mitigations)
13. [Key Performance Targets](#key-performance-targets)
14. [Research Foundation](#research-foundation)
15. [Future Roadmap](#future-roadmap)

---

## Overview

Pramaan AI is a full-stack, offline-first assessment platform designed for the recognition of prior learning (RPL) of India's informal workforce. The platform aligns with:

- **NCVET RPL Guidelines (2023 & 2024)**
- **NSQF / NCrF Credit Framework**
- **PMKVY 4.0 RPL Component**
- **DPDP Act 2023** (data privacy)
- **W3C Verifiable Credentials** (Ed25519)
- **Skill India Digital Hub**

The platform bridges the gap between *manual RPL today* (slow, assessor-dependent, hard to reach) and *consistent, scalable, auditable NSQF certification* for the informal workforce, using a **human-in-the-loop** AI architecture.

---

## Proposed Solution

Six core innovations power Pramaan AI:

### 1. Pictorial + Voice Self-Declaration
Workers tap **picture cards** of tools and tasks, or speak in their own language (Hindi, Marathi, English, and more). The interface is purpose-built for **low-literacy workers** with no typing required.

### 2. Offline QP Mapping Engine
On-device **multilingual embeddings** (transformers.js / Sentence-BERT + FAISS) rank the **top-3 NSQF Qualification Packs (QPs)** with reasons, matching NOS/PCs — all with **zero network dependency**.

### 3. Guided Practical + Evidence-Linked AI
NOS-to-PC checklists with anchored 0–3 rubrics. On-device **YOLOv8n** vision model flags PPE, tools, and step order with **clip timestamps** and SHA-256 hashed evidence.

### 4. Calibration Gym
Assessors score **gold anchor clips**. A **Many-Facet Rasch Model (MFRM)** automatically corrects for rater leniency or severity and flags drift over time.

### 5. Confidence Triage + Blind Double-Scoring
Borderline cases get a **second assessor**; a 10% random re-score gives live **inter-assessor agreement (κ/α/ICC)**. Escalation is automatic when score gap > 1 point.

### 6. Verifiable Skill Passport
NOS-wise profile, NCVET threshold maths, human sign-off, **W3C Verifiable Credential** (QR + DigiLocker-ready), and bridge-course advice when gaps are found.

---

## System Architecture

```
+---------------------------------------------------------------------+
|                        CLIENT LAYER (PWA)                           |
|  +------------------+  +------------------+  +-------------------+  |
|  |   Worker PWA     |  |  Assessor PWA    |  |Moderator Dashboard|  |
|  | picture/voice    |  | calibration gym  |  | κ/α severity map  |  |
|  | declaration      |  | checklist        |  | double-score queue|  |
|  | offline QP map   |  | capture          |  | fraud flags       |  |
|  | self-assessment  |  | AI flags         |  |                   |  |
|  |                  |  | sign-off         |  |                   |  |
|  +------------------+  +------------------+  +-------------------+  |
|                                                                     |
|         ON-DEVICE (offline): embeddings · MediaPipe pose           |
|                   YOLOv8n PPE · IndexedDB                          |
+-----------------------------+---------------------------------------+
                              | (sync when online)
                              v
+---------------------------------------------------------------------+
|               Node.js API GATEWAY                                   |
|   JWT · RBAC · resumable sync + conflict resolution · rate limiting |
+-------+------------------+-------------------+---------------------+
        |                  |                   |
        v                  v                   v
+--------------+ +--------------+ +---------------+ +--------------+
|  QP Mapping  | | Speech &     | |  Scoring &    | |  Integrity   |
|  FAISS + LLM | |  Language    | |  Calibration  | |  Layer       |
|  rerank,     | |  Bhashini /  | |  rubric, MFRM | |  liveness,   |
|  top-3 with  | |  Whisper ASR | |  severity,    | |  video hash, |
|  confidence  | |  TTS, viva   | |  κ/α/ICC,     | |  geo-time,   |
|              | |  drafts      | |  drift        | |  audit chain |
+--------------+ +--------------+ +---------------+ +--------------+
        |                  |                   |               |
        +------------------+-------------------+---------------+
                              |
                              v
+---------------------------------------------------------------------+
|                          DATA LAYER                                 |
|  MongoDB Atlas · MinIO/S3 evidence (SHA-256) · QP/NOS vector index  |
|                         · VC registry                               |
+---------------------------------------------------------------------+
                              |
                              v
+---------------------------------------------------------------------+
|                       LEARNING LOOP                                 |
|    overrides + anchor clips → monthly retraining + bias audit       |
+---------------------------------------------------------------------+
```

---

## Technical Stack

| Layer | Technologies |
|---|---|
| **Frontend** | React · Vite · Tailwind CSS · Workbox (Service Worker) |
| **On-Device AI** | transformers.js embeddings · MediaPipe Pose · TF.js vision |
| **Offline Storage** | Dexie / IndexedDB outbox |
| **API Gateway** | Node.js · Express · JWT · RBAC |
| **AI Service** | Python · FastAPI |
| **Speech & Language** | Bhashini · Whisper ASR · MeitY TTS |
| **Computer Vision** | YOLOv8n (PPE + tool detection) · MediaPipe (pose) |
| **NLP / Embeddings** | Sentence-BERT · FAISS · LLM rerank |
| **Scoring** | MFRM (Many-Facet Rasch) · κ / α / ICC statistics |
| **Database** | MongoDB Atlas |
| **Object Storage** | MinIO / AWS S3 (SHA-256 hashed evidence) |
| **Credentials** | W3C VC (Ed25519) · DigiLocker · QR |
| **Containerisation** | Docker |
| **Security** | TLS · AES-at-rest encryption · hash-chained audit log |

---

## Component Breakdown

### 1. Client Layer (PWA)

Three purpose-built Progressive Web Apps share the same offline-first runtime:

#### Worker PWA
- **Picture / Voice Declaration** — tap tool cards or speak in Hindi / Marathi / English (offline)
- **Offline QP Mapping** — on-device FAISS embeddings suggest top-3 NSQF QPs with explanations
- **PC Self-Assessment** — worker flags likely gaps before the assessor arrives
- **Ustad Attestation** — master / employer OTP verification

#### Assessor PWA
- **On-device AI Flags** — real-time PPE, tool, and step-order detection with clip timestamps
- **Evidence Capture** — video/photo + geo-time stamping, SHA-256 hashed to IndexedDB outbox
- **Guided Checklist** — NOS → PC anchored rubric, scores 0–3
- **Calibration Gym** — score anchor clips; MFRM severity computed server-side

#### Moderator Dashboard
- **κ/α Severity Map** — live inter-assessor agreement visualisation
- **Double-Score Queue** — borderline and random re-score cases
- **Fraud Flags** — reused-video detection, face-liveness results, geo-time anomalies

All three apps use a **Workbox service worker** to cache assets and queue writes in an **IndexedDB outbox** when offline, syncing via resumable protocol once connectivity is restored.

---

### 2. Application Layer (API Gateway)

**Node.js + Express** gateway with:

| Feature | Detail |
|---|---|
| **Authentication** | JWT (signed tokens, short-lived + refresh) |
| **Authorisation** | RBAC — four roles: Worker / Assessor / Moderator / Admin |
| **Sync Protocol** | Resumable sync with server-side conflict resolution; signed scores are append-only |
| **Versioning** | Assessment sessions versioned; immutable once signed |
| **Rate Limiting** | Per-role API throttling to prevent abuse |

---

### 3. AI Service Layer (FastAPI)

Python **FastAPI** microservice exposing four sub-modules:

#### QP Mapping Engine
1. Worker declaration (text/audio) → multilingual sentence embeddings (Sentence-BERT)
2. ANN search over FAISS index of all NCVET QP/NOS/PC descriptions (sourced from NQR)
3. Top-K candidates re-ranked by a lightweight **LLM** with chain-of-thought reasons
4. Returns **top-3 QPs** with confidence scores → presented to worker + assessor for confirmation

#### Speech & Language Module
- **ASR** — Bhashini / Whisper for multi-language transcription
- **TTS** — MeitY Bhashini TTS for voice prompts in local languages
- **Viva draft generation** — question bank seeded from NOS PCs

#### Scoring & Calibration Module
- Per-PC rubric scoring (0–3) with **anchored descriptors**
- **MFRM severity correction** — corrects per-assessor bias; raw + adjusted scores both stored
- **κ / α / ICC** inter-rater statistics computed live
- **Drift detection** — flags assessors whose severity shifts across batches

#### Integrity Layer
- **Liveness detection** — face-liveness model flags proxy fraud
- **Video perceptual hash** — cross-centre reused-video detection (SHA-256 chain)
- **Geo-time window check** — GPS + timestamp plausibility
- **Hash-chained audit log** — append-only, tamper-evident log of all events

---

### 4. Data Layer

| Store | Purpose |
|---|---|
| **MongoDB Atlas** | Candidates, sessions, scores, overrides, audit trails |
| **MinIO / S3** | Evidence blobs (video, photo) — SHA-256 content addressing |
| **FAISS Vector Index** | QP/NOS sentence embeddings for offline + online search |
| **VC Registry** | Issued W3C Verifiable Credentials (Ed25519 signed) |

**Data flow:** `QP → NOS → PC → marks JSON` parsed from NQR; stored in MongoDB Atlas; evidence in S3 / MinIO with SHA-256.

---

### 5. Trust & Compliance Layer

| Requirement | Implementation |
|---|---|
| **Transport security** | TLS everywhere |
| **Data at rest** | AES encryption |
| **Privacy** | DPDP Act 2023 consent flow; anonymised analytics |
| **Credential integrity** | W3C VC (Ed25519) — QR + DigiLocker-ready |
| **Audit** | SHA-256 hash-chained, append-only log; every override recorded with reason code |
| **Regulatory** | NCVET RPL 2023 & AA Guidelines 2024; PMKVY 4.0 RPL |

---

## Offline-First Design

Pramaan AI is built for **rural and semi-urban assessment centres** with unreliable or zero connectivity. The table below shows exactly what works offline and what is deferred to sync:

| Feature | No Network | After Sync |
|---|---|---|
| Picture / text declaration | Yes | — |
| QP mapping | On-device (FAISS) | LLM rerank |
| Voice declaration | Recorded | Transcription (Whisper/Bhashini) |
| Checklist, capture, AI flags, scoring | Yes | — |
| Viva (cached question bank + TTS) | Yes | New questions fetched |
| Sign-off / credential | Signed locally | Registry + DigiLocker upload |

On-device models are kept **< 100 MB** to run on low-cost Android phones. Evidence is queued in the **IndexedDB outbox** and synced using a resumable protocol; conflicts are resolved server-side with append-only semantics.

---

## Assessment Workflow

The end-to-end journey across three actor roles:

```
WORKER
  |
  +-> Pictorial / Voice Declaration (hi · mr · en, offline)
  |
  +-> Offline QP Mapping  ->  Top-3 QPs + reasons  (worker confirms)
  |
  +-> PC Self-Assessment  ->  Likely gaps flagged early
  |
  +-> Ustad Attestation   ->  Master / employer OTP

ASSESSOR
  |
  +-> Calibration Gym  ->  Anchor clips scored -> MFRM severity computed
  |
  +-> On-device AI Flags  ->  PPE · tools · step order (real-time)
  |
  +-> Evidence Capture  ->  Video/photo · geo-time · SHA-256
  |
  +-> Guided Checklist  ->  NOS->PC · anchored 0-3 rubric

OUTCOME
  |
  +-> Score / Override  ->  Reason code mandatory for any override
  |
  +-> Confidence Triage  ->  Borderline -> 2nd assessor
  |
  +-> Human Sign-off  ->  NCVET 70/30 threshold maths + pass marks
  |
  +-> Skill Passport  ->  VC (Ed25519) + QR + bridge-course advice

DATA & SYNC
  IndexedDB outbox -> MongoDB Atlas | hashed evidence | audit trail | offline export

FEEDBACK LOOP
  Overrides + anchor clips -> refine QP mapping and vision aids each batch
```

---

## AI vs. Human Decision Boundaries

A key design principle of Pramaan AI is that **AI supports; it never decides**. This protects certification integrity:

| Step | AI Supports | Assessor Decides | Guardrail |
|---|---|---|---|
| QP Mapping | Suggests top-3 QPs with reasons | Worker / assessor confirms | No auto-enrolment |
| Checklist & Viva | Drafts from NOS / PCs | SME approves bank once per QP | Versioned, frozen per batch |
| Practical Observation | Flags PPE, tools, step order + timestamp | Scores every PC | Flags never become marks |
| Scoring | Suggests per-PC score | Accepts or overrides | Reason code required; >1-point gap escalates |
| Rater Calibration | Estimates severity (MFRM) | Moderator applies adjustment | Raw + adjusted both stored |
| Certification | Recommends from NCVET thresholds | Assessor signs; awarding body certifies | AI cannot issue a credential |

---

## Calibration & Scoring

### NCVET Scoring Rules

| Rule | Value |
|---|---|
| Core / Non-core NOS weightage | 70 / 30 |
| Pass mark — NSQF L1–3 | 50% |
| Pass mark — NSQF L4+ | 70% |
| Bridge course triggered when | Gaps found (≥ 68 hrs guidance) |
| Learning-outcome mapping | Manual (NCVET) — to be automated |

### MFRM Rater Calibration
- Assessors score **gold anchor clips** in the Calibration Gym before each batch
- Many-Facet Rasch Model estimates per-assessor severity/leniency
- Raw score + MFRM-adjusted score both stored; moderator applies adjustment
- Drift flagged if severity shifts beyond threshold across batches

### Confidence Triage
- Borderline candidates (score near pass mark) → **blind 2nd assessor review**
- Random 10% re-score for live κ/ICC computation
- Gap > 1 point → mandatory escalation with reason code

---

## Impact & Benefits

### Quantitative Benefits

| Metric | Target |
|---|---|
| Top-3 QP mapping accuracy | 85% |
| Inter-assessor α (× 100) | ≥ 75 |
| Pass/fail agreement | 90% |
| Scoring effort saved | ~40% |
| Offline task completion | 100% |

### Qualitative Benefits

1. **Consistency** — anchored rubrics + MFRM severity correction; target inter-assessor α ≥ 0.75
2. **Speed** — declaration to QP match in minutes; triage targets ~40% less scoring effort
3. **Reach** — entire assessment runs offline at rural and semi-urban sites
4. **Transparency** — every AI suggestion links to a clip timestamp and rubric line
5. **Inclusion** — picture cards + voice in local languages for low-literacy workers
6. **Fewer disputes** — evidence + logged overrides make appeals quick to resolve

### Societal Impacts

| Impact Area | Description |
|---|---|
| **Workforce inclusion** | Informal workers earn an NSQF credential without repeating training |
| **Certification integrity** | Human sign-off, liveness, hashing, and fraud checks |
| **Assessor productivity** | More candidates per assessor; less rework and travel |
| **Employability** | Verifiable Skill Passport opens access to jobs and credit |
| **Policy alignment** | NCVET RPL, NSQF, NCrF, and Skill India Digital Hub compliant |
| **Data for policy** | Anonymised skill-gap maps by trade and region |

---

## Feasibility Analysis

### Technical Feasibility
- Proven open stack; NCVET QP/NOS files are public on NQR — mapping corpus exists on day one
- On-device models (< 100 MB) run on low-cost Android phones
- FAISS + transformers.js proven for offline semantic search

### Operational Feasibility
- Assessor stays in control; picture-card + voice UI minimises training burden
- Short assessor onboarding; full workflow offline at rural centres
- Calibration Gym embeds training into the workflow

### Economic Feasibility
- Open-source models and a small cloud footprint
- Confidence triage cuts review time — each assessor covers more candidates per day
- No proprietary dependencies; deployable on low-cost infrastructure

### Regulatory Feasibility
- Follows NCVET RPL Guidelines 2023 (NOS-wise assessment, self-assessment)
- Follows AA Guidelines 2024 (geo-tagged video, archived logs, integrity files)
- DPDP Act 2023 consent captured in onboarding flow

---

## Challenges & Mitigations

| Challenge | Risk | Mitigation |
|---|---|---|
| **Limited training data** | Synthetic declarations, self-recorded anchor clips, ITI-instructor gold labels | Federated data collection; use NQR public corpus as seed |
| **Over-reliance on AI** | Flags must never become marks; override gap > 1 point escalates | Hard guardrails: AI flags are advisory only; mandatory reason codes |
| **Fraud** | Proxy, reused video, ghost tests | Face liveness match; perceptual video hash across centres; GPS + time-window check; hash-chained log |
| **Connectivity** | Rural dead zones | Offline-first; 720p compressed video; resumable sync; encrypted offline export |
| **Bias & privacy** | Language/gender accuracy gaps | Accuracy audit by language and gender; picture-card fallback for ASR errors; encryption + RBAC |
| **Assessor resistance** | Distrust of AI suggestions | Pilot with assessors; clear "why" behind every suggestion; assessor always has final say |

---

## Key Performance Targets

These targets are to be validated in the pilot (25 recorded electrician practicals × 6 PCs, 6+ raters, crossover design):

| Metric | Target |
|---|---|
| Top-3 QP mapping accuracy | 85% |
| Inter-assessor α (×100) | 75 |
| Pass/fail agreement | 90% |
| Scoring effort saved | 40% |
| Offline task completion | 100% |

**Test methodology:**
- **Test set:** 25 recorded electrician practicals × 6 PCs, deliberately varied quality; gold scores from 2 expert raters
- **Raters:** 6+ raters, crossover design: unassisted paper checklist vs. Pramaan-assisted
- **Metrics:** Krippendorff α (ordinal), ICC(2,k), pass/fail agreement, deviation from gold, time per candidate

---

## Research Foundation

| # | Reference |
|---|---|
| 1 | NCVET (2023). Guidelines for Recognition of Prior Learning |
| 2 | NCVET (2024). Comprehensive Revised Guidelines for Assessment Agencies |
| 3 | MSDE (2024). PMKVY 4.0 Guidelines (RPL component) |
| 4 | NCVET (2023). NSQF notification; National Credit Framework |
| 5 | Linacre, J. M. (1989). Many-Facet Rasch Measurement |
| 6 | Krippendorff, K. (2004). Content Analysis; Cohen, J. (1960) |
| 7 | Radford, A. et al. (2022). Whisper; MeitY Bhashini |
| 8 | Reimers & Gurevych (2019). Sentence-BERT; FAISS (2019) |
| 9 | Lugaresi, C. et al. (2019). MediaPipe; Ultralytics YOLOv8 |
| 10 | W3C (2022). Verifiable Credentials Data Model; DPDP Act 2023 |

---

## Future Roadmap

| Priority | Feature | Description |
|---|---|---|
| High | **Multi-trade scaling** | New QP = JSON template → auto checklist and picture cards |
| High | **SIDH + DigiLocker** | Push credentials; NCrF credits into APAAR / ABC |
| Medium | **IVR / missed-call** | Self-declaration for feature-phone users |
| Medium | **Adaptive viva (IRT)** | Fewer questions at the same confidence level |
| Medium | **National Calibration Network** | Cross-state assessor benchmarking |
| Long-term | **Smart matching** | Bridge courses and employers near the worker |
| Long-term | **Policy heatmaps** | Skill gaps by trade and district (e-Shram linkage) |

---

## Demo

> **Demo trade:** Assistant Electrician (CON/Q0602, NSQF L3)

The demo runs the full end-to-end flow:
Worker declaration → Offline QP mapping → Assessor guided checklist → AI flags → Score + override → Confidence triage → Verifiable Skill Passport generation.

---

## Architecture Evolution

```
Manual RPL today        Pramaan AI assist         Consistent, scalable,
(slow, assessor-        (human-in-the-loop,   ->  auditable NSQF
dependent, hard         evidence-linked)           certification for the
to reach)                                          informal workforce
```

---

## What's New in Pramaan AI

| Innovation | Description |
|---|---|
| **Rater-severity correction (MFRM)** | First RPL platform to apply Many-Facet Rasch to India vocational assessment |
| **AI mapping with zero network** | FAISS + transformers.js embedded QP search — no internet needed |
| **Picture-card UI for non-literate workers** | Tap-to-declare with voice; no reading required |
| **Reused-video fraud detection** | Perceptual hash chain across centres; hash-chained audit log |

---

*Built for Smart India Hackathon 2026 by Team AXIOM.*  
*Licensed for public benefit under NCVET RPL framework.*
