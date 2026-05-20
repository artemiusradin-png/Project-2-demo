---
stepsCompleted: [1, 2, 3, 4, "4b-multi-agent"]
inputDocuments:
  - '_bmad-output/planning-artifacts/briefs/brief-Agro-Platforma-2026-05-19/brief.md'
  - '_bmad-output/planning-artifacts/briefs/brief-Agro-Platforma-2026-05-19/addendum.md'
  - '_bmad-output/planning-artifacts/prds/prd-Agro-Platforma-2026-05-19/prd.md'
  - '_bmad-output/planning-artifacts/prds/prd-Agro-Platforma-2026-05-19/addendum.md'
workflowType: 'architecture'
project_name: 'Agro Platforma - Project 2'
user_name: 'Artem'
date: '2026-05-19'
---

# Architecture Decision Document

_This document builds collaboratively through step-by-step discovery. Sections are appended as we work through each architectural decision together._

## Project Context Analysis

### Requirements Overview

**Functional Requirements (16 FRs across 6 features):**

| Feature | FRs | Architectural Implication |
|---------|-----|--------------------------|
| Photo Capture & Crop ID | FR-1 to FR-3 | Camera integration in Flutter, image preprocessing, multimodal LLM API call |
| Problem Diagnosis | FR-4, FR-5 | Chained inference (crop + problem in single pass), confidence thresholding logic, multi-result handling |
| Prescription Generation | FR-6 to FR-8 | RAG pipeline, Salesforce real-time stock queries, Farmland context enrichment, product ranking algorithm |
| Agronomic Commentary | FR-9 | LLM generation with structured agronomic prompts, role-aware output |
| Conversational Advisor | FR-10 to FR-12 | Stateful chat session, context window management, scoped conversation boundaries |
| AI Foundation | FR-13 to FR-16 | Orchestration layer, Salesforce connectors, vector DB, evaluation framework |

**Non-Functional Requirements:**
- Performance: ≤10s photo-to-diagnosis, ≤30s end-to-end, 500 concurrent sessions
- Security: HTTPS/TLS, no PII to LLM, scoped OAuth, photo retention policy, Farmland data isolation
- Reliability: Graceful degradation, retry with backoff, circuit breakers
- Observability: Session logging, latency/error dashboards, Confidence Score distribution monitoring, alerting
- Localization: All AI output in Ukrainian, Latin nomenclature for taxonomy

**Scale & Complexity:**
- Primary domain: Full-stack (Flutter mobile + AI orchestration backend + Salesforce integration)
- Complexity level: High
- Estimated architectural components: 8-12

### Technical Constraints & Dependencies

- **Salesforce is Single Source of Truth** — all product, account, farmland, and stock data lives there. Architecture must treat Salesforce as the authoritative data source, not a replication target.
- **Existing Flutter app** — new features must integrate into the existing navigation, state management, and auth patterns. Not a greenfield mobile build.
- **External LLM dependency** — Claude API for vision + language. Latency, rate limits, and cost are external constraints. No local inference in v1.
- **Ukraine regulatory compliance** — product approval filtering is a hard requirement, not a nice-to-have.
- **Team constraint** — 2 Flutter founders + 1 Salesforce dev (contract) + 1 ML advisor (part-time). Every architectural choice must be operable by this team.

### Cross-Cutting Concerns Identified

1. **Security infrastructure** — Enterprise-grade data protection across 40k clients: photo handling, PII isolation from LLM, Salesforce scoped access, session data governance, API authentication chain
2. **LLM cost management** — Vision tokens are expensive at scale; need caching, prompt optimization, and usage monitoring
3. **Data freshness strategy** — Real-time for stock levels, cached for Catalog/Farmland, event-driven vector index updates
4. **Error cascade prevention** — LLM API failure must not cascade to Salesforce queries or crash the client app
5. **Evaluation pipeline** — Must run independently of production to measure accuracy without affecting users

### Party Mode Enhancements (Roundtable Insights)

The following architectural concerns were surfaced by the agent roundtable and are adopted into the context:

| Concern | Source | Action |
|---------|--------|--------|
| **Salesforce API rate limits** are the real bottleneck at 500 concurrent users | Winston (Architect) | Caching layer is day-one infrastructure, not optional |
| **Photo EXIF/GPS stripping** before any data leaves the device or reaches LLM | Winston, Amelia | Security preprocessing pipeline sits between camera and AI |
| **Photo compression** before LLM ingestion | Winston | Image preprocessing to manage cost + latency (no 12MB field photos to API) |
| **Consultant override/audit trail** — trust infrastructure | Mary (Analyst), John (PM) | If AI contradicts consultant judgment, the override must be logged. Adoption fails from distrust, not bad AI |
| **Recommendation explainability** — commercial liability | John (PM) | Every prescription must be auditable: why this product, what data drove the ranking. Regulatory and legal requirement |
| **Consider cutting UJ-2 (farmer self-service) from v1** | John (PM) | Focus all 12-week effort on making UJ-1 (consultant) excellent. Consultant-first was the declared strategy |
| **Catalog data quality** is a dependency time bomb | Mary (Analyst) | Catalog freshness SLA must be formalized with Agro-Arena. Prescription accuracy depends on it |
| **RAG injection surface** | Amelia (Dev) | If catalog descriptions contain user-contributed content, sanitization before embedding is required |
| **Failure state UX** at the 30-second boundary | John (PM) | Architecture must define graceful timeout behavior — partial results, retry, clear error messaging |
| **Regulatory and data residency** in wartime Ukraine | Mary (Analyst) | Data residency constraints must be captured before choosing hosting/cloud provider |

## Starter Template Evaluation

### Primary Technology Domain

This is not a greenfield project. The Flutter mobile app and Salesforce backend already exist. The new component is an **AI orchestration backend** — a service layer that receives photo requests from Flutter, orchestrates LLM vision + RAG + Salesforce queries, and returns structured prescriptions.

### Starter Options Considered

| Option | Fit | Verdict |
|--------|-----|---------|
| **Python + FastAPI** | Async-native, AI/ML ecosystem, OpenAPI docs for Flutter consumption | ✅ Selected |
| Node.js + Express/Fastify | Familiar to JS devs, good API layer, weaker AI tooling | ❌ AI ecosystem gap |
| Dart (shelf/serverpod) | Unified language with Flutter, but minimal AI/ML ecosystem | ❌ Ecosystem limitation |
| Salesforce Agentforce | Native SF integration, but no vision capabilities, limited customization | ❌ Cannot meet FR-1 to FR-5 |

| Vector DB Option | Fit | Verdict |
|-----------------|-----|---------|
| **pgvector on PostgreSQL 17** | Full SQL filtering, <20ms at 20k scale, one DB to operate | ✅ Selected |
| Pinecone | Managed, fast, but overkill for 20k products + adds vendor dependency | ❌ Over-engineered |
| Weaviate / ChromaDB | Good for larger scale, but adds operational complexity for small team | ❌ Team constraint |

| Cloud Option | Fit | Verdict |
|-------------|-----|---------|
| **GCP (Cloud Run + Cloud SQL + Vertex AI)** | Claude via Vertex AI, zero-K8s deployment, best Flutter integration | ✅ Selected |
| AWS (ECS + RDS + Bedrock) | Broader services but more operational overhead, less Flutter-native | ❌ Operational complexity |

| LLM Orchestration | Fit | Verdict |
|-------------------|-----|---------|
| **Anthropic Python SDK (direct)** | Single-provider app, linear pipeline, no abstraction overhead | ✅ Selected |
| LangChain | Adds abstraction without value for single-provider linear pipeline | ❌ Unnecessary complexity |
| LlamaIndex | Better for complex RAG, but overkill for structured catalog retrieval | ❌ Over-engineered |

### Selected Stack

**Rationale:** Every component is a managed service (Cloud Run, Cloud SQL, Vertex AI). The team operates application code, not infrastructure. Python is the lingua franca for AI work, and FastAPI's async capabilities handle the parallel Claude + Salesforce call pattern naturally.

**Core Dependencies:**

```
fastapi==0.136.1
anthropic==0.102.0
pgvector==0.3.6
asyncpg==0.30.0
httpx==0.28.1
Pillow==11.1.0
uvicorn==0.34.0
```

**Infrastructure:**

| Component | Service | Purpose |
|-----------|---------|---------|
| API Server | GCP Cloud Run | Containerized FastAPI, auto-scaling, HTTPS, load balancing |
| Database | GCP Cloud SQL (PostgreSQL 17 + pgvector) | Vector index for catalog RAG + session/evaluation data |
| LLM API | Vertex AI Model Garden (Claude) | Multimodal vision + language, billed through GCP |
| Image Storage | GCP Cloud Storage | Temporary photo upload from Flutter (session-scoped TTL) |
| Monitoring | GCP Cloud Logging + Cloud Monitoring | Latency, error rates, usage dashboards |
| Containerization | Docker → Cloud Run | Single Dockerfile, `gcloud run deploy` |

**Architectural Decisions Provided by Stack:**

- **Language & Runtime:** Python 3.12, async/await throughout
- **API Design:** FastAPI with auto-generated OpenAPI spec consumed by Flutter
- **Image Preprocessing:** Pillow for EXIF stripping, compression, format normalization
- **Salesforce Integration:** httpx async HTTP client for REST API calls
- **Database Access:** asyncpg for non-blocking PostgreSQL queries
- **Deployment:** Docker container → Cloud Run (zero Kubernetes)

**Note:** Project initialization (GCP setup, Cloud SQL provisioning, FastAPI scaffold, Docker config) should be the first implementation story.

## Core Architectural Decisions

### Decision Priority Analysis

**Critical Decisions (Block Implementation):**
1. Auth: Firebase Auth (JWT validated by backend middleware)
2. Photo pipeline: Cloud Storage signed URL + EXIF strip + 24h TTL cleanup
3. SF caching: PostgreSQL cache tables, tiered by data volatility
4. Chat state: PostgreSQL session table (auditable)
5. Accuracy methodology: Hybrid — senior agronomist blind panel for v1 launch gate, field-outcome tracking in parallel

**Important Decisions (Shape Architecture):**
6. CI/CD: GitHub Actions → Cloud Run, Dev + Staging + Production
7. Connection pooling: PgBouncer / Cloud SQL pooling from day one (roundtable: Winston)
8. Audit log captures reasoning chain, not just outputs (roundtable: John — liability)

**Scope Decision:**
- **UJ-2 (farmer self-service) CUT from v1.** Focus on UJ-1 (consultant field diagnosis) + UJ-3 (advisor). Consultant-first strategy. Farmer self-service deferred to v2. (roundtable: John)

**Deferred Decisions (Post-MVP):**
- Redis caching layer (only if PostgreSQL becomes bottleneck)
- Offline photo queuing on Flutter (v2)
- Multi-region deployment (post-launch scaling)

### Data Architecture

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Primary DB | PostgreSQL 17 + pgvector on Cloud SQL | Vector index + cache + sessions + audit in one DB. Team operates one service. |
| Connection pooling | PgBouncer or Cloud SQL built-in pooling — configured day one | pgvector ANN is CPU-hungry; pooling is not a retrofit (roundtable: Winston) |
| Catalog cache | PostgreSQL table, hourly refresh + cache-miss fallback | Catalog changes infrequently. Stale-read window during sync must have a fallback test (roundtable: Amelia) |
| Farmland cache | PostgreSQL table, daily/on-demand refresh | Farmland data is mostly static (soil, rotation) |
| Stock levels | Live query to Salesforce per prescription | Stock is time-sensitive — no caching. Rate-limit aware with retry |
| Vector index | pgvector HNSW on catalog products | <20ms retrieval at 20k scale. Metadata filtering via SQL WHERE |
| Chat sessions | PostgreSQL table (session_id, messages JSONB, created_at, ttl) | Auditable, persistent, supports explainability |
| Migrations | Alembic | Standard Python migration tool for PostgreSQL |

### Authentication & Security

| Decision | Choice | Rationale |
|----------|--------|-----------|
| User auth | Firebase Auth | Flutter-native SDK, GCP-native, JWT validation on backend |
| Backend auth | Firebase JWT verification middleware + token-refresh retry/backoff spec | Every request validated. Refresh race under poor mobile (3G Ukraine) handled (roundtable: Amelia) |
| SF data access | Scoped service-account Connected App | Minimum required read permissions. No user-level SF tokens. Eliminates token-refresh nightmare (roundtable: Winston) |
| Photo security | EXIF/GPS stripped server-side (Pillow) before LLM | Original never sent to external API. Mandatory CI gate `test_exif_strip` blocks merge on fail (roundtable: Amelia) |
| Photo storage | Cloud Storage signed URL + 24h TTL auto-delete | Backend reads, processes, deletes. Signed-URL-expiry → explicit 403 handling test (roundtable: Amelia) |
| PII isolation | No user identity sent to Claude API | Only crop photo + agronomic context (soil, rotation). No names, accounts, financials |
| RAG injection defense | Input sanitization before embedding lookup + prompt boundary enforcement | Attack surface moved to pgvector catalog cache, not closed — must sanitize (roundtable: Amelia) |
| Confidence pre-flight gate | Validate confidence threshold BEFORE surfacing prescription to user | Pre-flight validation, not just post-hoc consultant override (roundtable: Amelia) |
| API security | HTTPS only, per-user + per-endpoint rate limiting, request size limits | Protects against buggy Flutter client looping `/prescriptions` (roundtable: Winston) |
| Audit trail | Log reasoning chain: symptoms → diagnosis → catalog rule → SKU, plus session_id, confidence | Liability requires *why*, not just *what* (roundtable: John) |
| Consultant override | Override flag + timestamp logged when consultant modifies a recommendation | Liability transfer paper trail — human made final call (roundtable: John, kept as-is) |

### API & Communication Patterns

| Decision | Choice | Rationale |
|----------|--------|-----------|
| API style | REST (FastAPI auto-generated OpenAPI) | Flutter consumes OpenAPI spec directly |
| Contract tests | Pact (or equivalent) between Flutter and FastAPI | Schema drift bites week 6 without it (roundtable: Amelia) |
| Error handling | Structured errors with codes + specific Ukrainian user-facing messages | Partial-result message must state *what's missing and why* — actionable, not "try again later" (roundtable: John) |
| Rate limiting | Per-user, per-endpoint (e.g., 10 Agro-Vision req/min) | Prevents abuse, manages LLM cost |
| Request timeout | 35s gateway (5s buffer over 30s SLA) | Cloud Run max, allows full pipeline |
| Latency observability | Structured per-stage timing logs before production | Know where the 30s goes before deploying, not after (roundtable: Winston) |
| Partial results | LLM ok but SF stock fails → return diagnosis without stock status | Graceful degradation — don't fail whole request for a stock lookup |

### Frontend Architecture (Flutter Integration)

| Decision | Choice | Rationale |
|----------|--------|-----------|
| API client | Generated from OpenAPI spec (openapi-generator for Dart) | Type-safe, auto-updated on backend change |
| Photo upload | Camera → Cloud Storage signed URL → API call with storage ref | Async upload, no base64 bloat |
| State management | Existing app patterns (don't introduce new) | Integrating into an existing app, not greenfield |
| New UI surfaces | Diagnosis Card, Prescription view, Advisor chat — in existing nav | Lives inside the existing app shell |

### Infrastructure & Deployment

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Environments | Dev + Staging + Production | Enterprise client expects staging validation |
| CI/CD | GitHub Actions → Docker build → Cloud Run deploy | Repo on GitHub. test → build → deploy |
| Mandatory CI gates | `test_exif_strip`, signed-URL-expiry, cache-stale-read, p99<25s load test (k6/locust in staging) | If a gate doesn't block merge, it doesn't exist (roundtable: Amelia) |
| Monitoring | GCP Cloud Logging + Monitoring + custom dashboards | Latency per stage, error rates, Confidence Score distribution, LLM cost/request |
| Alerting | Alerts: error rate >5%, p99 >25s, LLM cost spike | Early warning before SLA breach |
| Secrets | GCP Secret Manager | API keys, SF creds, Firebase config — not in env/code |

### Accuracy Validation (resolves PRD Open Question #1)

| Phase | Method | Gate |
|-------|--------|------|
| V1 launch gate | Senior agronomist **blind** panel — independent agronomists (NOT the consultants using the tool) review diagnoses against photo + context | ≥85% on held-out test set before production launch |
| Post-launch (first season) | Field-outcome tracking — actual disease progression / yield data after treatment | Validates the panel score against reality; informs v2 accuracy targets |

Rationale: consultant-agreement is circular (consultants trained on same patterns agree with the model's mistakes). Blind panel breaks the circularity for the launch gate; field outcomes provide eventual ground truth (roundtable: John).

### Decision Impact — System Component Map

```
┌─────────────────────────────────────────────────────────┐
│  Flutter App (existing)                                  │
│  ┌──────────┐ ┌──────────┐ ┌───────────┐               │
│  │ Camera   │ │ Diagnosis│ │ Advisor   │               │
│  │ Capture  │ │ Card UI  │ │ Chat UI   │               │
│  └────┬─────┘ └──────────┘ └───────────┘               │
│       │  Firebase Auth (JWT)                             │
└───────┼──────────────────────────────────────────────────┘
        │ signed URL upload    │ REST API (OpenAPI + Pact)
        ▼                      ▼
┌──────────────┐    ┌──────────────────────────────────────┐
│ GCP Cloud    │    │  AI Orchestration Backend (FastAPI)   │
│ Storage      │◄───│                                      │
│ (photos,24h) │    │  ┌────────────┐  ┌───────────────┐  │
└──────────────┘    │  │ Photo      │  │ LLM Adapter   │  │
                    │  │ Preprocessor│  │ (Anthropic SDK)│  │
                    │  │ EXIF strip │  │ + confidence  │  │
                    │  │ + compress │  │   pre-flight  │  │
                    │  └────────────┘  └───────┬───────┘  │
                    │  ┌────────────┐  ┌───────▼───────┐  │
                    │  │ Salesforce │  │ RAG / Vector  │  │
                    │  │ Connector  │  │ + injection   │  │
                    │  │ (httpx)    │  │   sanitize    │  │
                    │  └──────┬─────┘  └───────┬───────┘  │
                    │         │   PgBouncer pool│           │
                    │  ┌──────▼────────────────▼────────┐  │
                    │  │  PostgreSQL 17 (Cloud SQL)     │  │
                    │  │  • Vector index (catalog)      │  │
                    │  │  • Catalog/Farmland cache      │  │
                    │  │  • Chat sessions               │  │
                    │  │  • Audit log (reasoning chain) │  │
                    │  └───────────────────────────────┘  │
                    └──────────────────────────────────────┘
                              │  scoped service account
                    ┌─────────▼──────────┐
                    │  Salesforce        │
                    │  (Source of Truth)  │
                    │  • Catalog (20k SKU)│
                    │  • Farmland/Soils  │
                    │  • Stock (live)    │
                    │  • Accounts        │
                    └────────────────────┘
```

### Open Architecture Risk (carried forward)

**RAG corpus ownership** — Winston's top concern, unresolved by architecture alone: *who owns and curates the agronomic knowledge base?* Garbage in, garbage out — no architecture compensates for a poor corpus. This is a staffing/ownership decision for the planning phase, flagged as a project risk.

## Multi-Agent Solution Architectures (A, C, E)

The client offer is scoped to the three agents with the **highest customer efficiency**: Agro-Vision (A), Agro-Flow (C), Agro-Insight (E). These were selected over B and D because A→C→E forms a self-reinforcing data flywheel and each reuses the shared v1 foundation. This section specifies the proposed architecture per agent and the shared substrate.

### Selection Rationale (customer-value, technical)

| Agent | Customer value | Technical argument | Verdict |
|-------|----------------|--------------------|---------|
| **A · Agro-Vision** | Highest-frequency revenue moment; every consultant performs at senior-agronomist level | Diagnosis is *immediately verifiable* in the field → fastest trust build (customer's #1 goal). Builds the shared foundation + generates the data exhaust C/E need. | **Include — #1, mandatory** |
| **C · Agro-Flow** | Directly delivers "grow without growing opex"; converts A's prescription into an automated quote/order | Rides A's Salesforce connectors (low marginal build). Quote/analog assembly is *deterministic and verifiable against the system of record* — not a probabilistic guess. | **Include — #2** |
| **E · Agro-Insight** | Monetizes A+C data; strengthens vendor terms → better inventory → better A prescriptions | Read-only analytics; numbers computed in SQL (deterministic, auditable), LLM only narrates. No external dependency, no real-time/liability surface. Lowest-risk agent. | **Include — #3** |
| B · Agro-Markets | High strategic value (what to plant / when to sell) | Needs *new* external integrations (commodity exchanges, price feeds); a market prediction is unfalsifiable until a season passes — cannot build a held-out test set. Worst trust-build profile. | Defer to Phase 2 |
| D · Agro-Mentor | Consultant retention/upskilling | Product/workflow engineering more than AI; value diffuse and lagging. | Defer to Phase 3 |

### Shared Foundation (built once in v1, reused by C and E)

```
┌──────────────────────── SHARED SUBSTRATE ────────────────────────┐
│ FastAPI orchestration · Anthropic SDK (Claude via Vertex AI)      │
│ PostgreSQL 17 + pgvector  ·  Salesforce connector (scoped)        │
│ Firebase Auth  ·  Cloud Storage  ·  Cloud Run  ·  Audit/Event log │
└───────────────────────────────────────────────────────────────────┘
        ▲ reuse                ▲ reuse                ▲ reuse
   ┌────┴─────┐          ┌─────┴─────┐          ┌─────┴──────┐
   │ A Vision │  events  │ C Flow    │  events  │ E Insight  │
   │ diagnose │ ───────▶ │ transact  │ ───────▶ │ analyse    │
   └──────────┘          └───────────┘          └─────┬──────┘
        ▲                                              │ vendor signals
        └──────────── better inventory / terms ────────┘   (the flywheel)
```

Reuse ratio (net-new build vs. reused foundation): **A ≈ 100% net-new (it is the foundation)**, **C ≈ 30% net-new**, **E ≈ 20% net-new**. This is the architectural basis for the customer-efficiency claim.

### A · Agro-Vision — Field Intelligence (v1, recap)

Already specified above (§System Architecture, §Core Architectural Decisions). Summary: Flutter capture → signed-URL upload → EXIF strip + compress → Claude multimodal diagnosis → confidence pre-flight gate → pgvector RAG over the 20k catalog → live Salesforce stock query → ranked prescription + agronomic commentary → reasoning-chain audit log. Read-only against Salesforce.

**Emits to the event log:** `diagnosis`, `prescription`, `confidence`, `consultant_override` — consumed by C and E.

### C · Agro-Flow — Operational Intelligence

**Purpose:** turn a prescription (or a manual consultant request) into an auto-built quote (KP) in Salesforce, with analog substitution and cross-sell, minimizing operations headcount.

```
 A prescription / manual request
        │ event
        ▼
 ┌─────────────────────────────────────────────┐
 │ Quote Builder (FastAPI module)              │
 │  • map prescription lines → SF Quote/SO     │
 │  • Analog/Cross-sell engine ── pgvector ────┼──▶ REUSES A's catalog
 │    (substitute when OOS, complementary up-  │     vector index
 │     sell) ranked by relevance + margin      │
 │  • Deterministic Rules layer:               │
 │    margin floor · pack size · regulatory    │
 │    compatibility · stock (live SF)          │
 └───────────────┬─────────────────────────────┘
                 │ DRAFT write (scoped, draft-only)
                 ▼
        Salesforce  Quote / Selling Opportunity
                 │
                 ▼
        Consultant / Supervisor APPROVAL gate ──▶ binding
                 │ event
                 ▼
        Audit/Event log  (accepted vs. overridden)  ──▶ feeds E
```

**Key decisions:**
- **Net-new vs. reuse:** reuses A's Salesforce connector, pgvector catalog index, LLM orchestration, auth, audit. Net-new = Quote Builder, deterministic Rules layer, scoped SF *write* path, approval UI.
- **Security delta from A:** A is read-only against Salesforce; C requires a **separate scoped service-account permission with draft-only write** to Quote/SO objects. No auto-binding — a human (consultant/supervisor) approves before a quote becomes commercial. Every write audited.
- **Verifiability:** quote totals, analog matches and rule outcomes are deterministic and reconcilable against Salesforce — accuracy is checkable, unlike a probabilistic field diagnosis. LLM is used only for ranking/justification text, never for the numbers.
- **Latency:** not field-real-time; quote generation is async-tolerant (seconds, background job acceptable).

### E · Agro-Insight — Vendor Intelligence

**Purpose:** live, personalized analytics and narrative reports for vendors — regional performance, focus consultants, problem hotspots — for non-technical supplier reps.

```
  A events (diagnosis/prescription)  ┐
  C events (quote/order)             ├─▶ ELT job (Cloud Scheduler → Cloud Run job)
  Salesforce sales data              ┘        │
                                              ▼
                        ┌───────────────────────────────────────┐
                        │ Analytics Data Mart (PostgreSQL)      │
                        │  vendor-scoped materialized views     │
                        │  by region · product · consultant ·   │
                        │  time — numbers computed in SQL        │
                        └───────────────┬───────────────────────┘
                          ┌─────────────┴─────────────┐
                          ▼                           ▼
              Narrative Report Gen            Dashboard API (read-only)
              (Claude narrates the            tenant-scoped endpoints →
               computed aggregates;            vendor dashboard
               never invents numbers)          (Flutter web / web view)
```

**Key decisions:**
- **Net-new vs. reuse:** reuses PostgreSQL, Claude adapter, auth, monitoring. Net-new = data-mart schema, ELT aggregation jobs, tenant-scoped read API, narrative prompts. **Zero new external dependencies.**
- **Trust by construction:** all metrics are computed deterministically in SQL. The LLM only *narrates* pre-computed aggregates (RAG-grounded on the data mart) — it can never fabricate a number. This addresses the "vendor analytics must be trustworthy" concern structurally.
- **Tenant isolation (critical):** vendor A must never see vendor B's data. Row-level scoping enforced at both the data-mart and API layers; every query carries a vendor scope derived from the authenticated principal, not the request body.
- **No liability surface:** E is reporting, not crop-affecting advice — the lowest-risk agent in the suite. No vision, no real-time constraint.

### Cross-Agent Concerns

| Concern | Design |
|---------|--------|
| Event backbone | A and C append domain events to the shared append-only audit/event log in PostgreSQL; E reads from it. Simple, one DB, no message broker needed at this scale. |
| Salesforce write isolation | Only C writes (draft-only, scoped). A and E are read-only. Three distinct scoped service-account permission sets — least privilege per agent. |
| Phasing | A ships in v1 (foundation). C and E attach post-v1, each a contained increment on the proven substrate. |
| Failure isolation | An agent failure is contained: if C is down, A still diagnoses; if E's ELT fails, A and C are unaffected. No agent is in another's critical path. |

### The Data Flywheel — Mechanics, Moat, and v1 Boundary

The A→C→E selection is one compounding loop, not three features. This subsection records the mechanics, what is genuinely defensible vs. what is not, and the explicit v1 architectural boundary — developed and pressure-tested in a BMAD party-mode roundtable (Mary/Analyst, John/PM, Winston/Architect, Sophia/Storyteller).

**Mechanics (the turns):**
1. **A generates ground-truth field data** — every diagnosis is a labelled event (problem → product → accepted/overridden → context). Proprietary; requires the field moment.
2. **C converts decisions into transactions + economics** — quote, margin, substitution, conversion. Adds the economic layer A lacks; informs A's ranking (recommend what is in stock, margin-positive, actually bought — not only clinically correct).
3. **E aggregates A+C into vendor leverage** — better vendor terms/inventory → A can prescribe products that are available and well-priced → richer C data → the loop tightens.

**Architectural credibility fact (why the loop is real, not marketing):** every agent writes to and reads from the **same append-only event log on the same PostgreSQL instance**. This is shared state, not an integration — no API handoff, no file exchange. The loop is real because the substrate is singular. This is the one provable architectural claim that the flywheel is a system, not a slide.

**v1 boundary — the honest caveat (load-bearing scoping decision):** the feedback arrow back to A is **not automatic in v1**. There is no scheduled retraining pipeline, embedding refresh, or model fine-tuning. The loop closes through **human action**: agronomists review Agro-Insight, procurement adjusts inventory, the prescription catalog is updated. The correct framing is "the data compounds in one place; the team acts on it" — *not* "self-improving AI". Presenting the feedback loop as autonomous software is overclaiming and out of v1 scope. Automatic feedback (retraining/relevance learning) is a deliberate post-v1 item.

**Moat reality (what is defensible vs. not — for strategy/positioning consumers of this doc):**

| Claim | Status | Basis |
|-------|--------|-------|
| Dataset geometry — value needs distribution network + field moment + transaction *together* | **Real moat** | Porter activity-system logic; unreplicable by a standalone app or an ERP; ~5-year replication barrier |
| Switching cost grows once Agro-Flow is embedded in the Salesforce workflow | **Real moat** | Classic enterprise lock-in (retraining + data migration + relationship risk) |
| "Lowers CAC every turn" | **Slogan until proven** | Plausible mechanism, no cohort evidence; do not assert without acquisition-cost-by-vintage data |
| "Loop self-funds" | **Conditional** | Holds only if A's diagnostic accuracy crosses the consultant-trust threshold fast; otherwise the loop stalls at A. This is a sequencing risk, and the reason v1 is A-first |

### Carried-forward open items (unchanged)

RAG corpus ownership (A) remains the top project risk. C adds: the deterministic rules catalog (margin floors, pack sizes, regulatory compatibility) must be sourced and owned. E adds: vendor data-sharing scope must be agreed contractually before the data mart is designed. Flywheel adds: **automatic A-feedback (retraining/relevance learning) is explicitly deferred post-v1** — v1 closes the loop via human action by design.
