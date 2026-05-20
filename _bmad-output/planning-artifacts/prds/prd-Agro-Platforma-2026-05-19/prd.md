---
title: "Agro Agent v1 — Agro-Vision"
status: draft
created: 2026-05-19
updated: 2026-05-19
---

# PRD: Agro Agent v1 — Agro-Vision

## 0. Document Purpose

This PRD defines the v1 scope of **Agro Agent**, the AI capability layer for AgroPlatforma. The primary audience is the engineering team (architecture, development, QA), the product owner, and the client (Agro-Arena / AgroPlatforma). It builds on the [product brief](_bmad-output/planning-artifacts/briefs/brief-Agro-Platforma-2026-05-19/brief.md) and the original client brief. Features are grouped with FRs numbered globally; assumptions are tagged inline and indexed in §9. Glossary terms are used verbatim throughout.

## 1. Vision

AgroPlatforma connects 40,000+ Ukrainian farming enterprises with agricultural input suppliers through a Salesforce-backed, Flutter-fronted B2B platform. Today, when a Consultant encounters a crop problem in the field, the path from diagnosis to product recommendation takes hours: phone calls, manual catalog browsing, warehouse checks, guesswork.

**Agro Agent v1** collapses this to seconds. The flagship feature, **Agro-Vision**, lets the user photograph a crop and receive an expert-level diagnosis (crop, pest, disease, or weed identification) paired with a concrete **Prescription** — ranked products from the platform's 20,000+ SKU Catalog that are Ukraine-approved, in stock, and contextualized to the Farmland's soil and rotation history.

V1 also includes a **Conversational Advisor** that handles follow-up questions after diagnosis, and the **AI Foundation** (orchestration, data connectors, evaluation framework) that future AI features (trading, automation, analytics) plug into at a fraction of the initial cost.

## 2. Target User

### 2.1 Primary Persona: The Consultant

**Ivan, 34, independent agricultural consultant, Poltava region.** Advises 40+ farmers across three districts. His income depends on volume — the faster he can answer field questions, the more farmers he serves. He knows seeds well, but crop protection chemistry and disease diagnosis outside his specialty require calling senior colleagues. He uses AgroPlatforma for ordering but still carries a paper notebook for field observations.

### 2.2 Secondary Persona: The Farmer

**Oksana, 48, owns a 200-hectare mixed farm, Vinnytsia region.** Manages sunflower, wheat, and corn rotations. She calls Ivan when something goes wrong. In v1 she benefits *through* Ivan — he diagnoses faster and prescribes accurately on her behalf. Direct self-service for Oksana is a v2 capability (see §2.4).

### 2.3 Jobs To Be Done

**Consultant (Ivan):**
- When I see a crop problem in the field, I want to identify it instantly so I don't look uncertain in front of the farmer
- When a diagnosis is made, I want specific products I can order right now — not generic advice I have to translate into SKUs
- When the farmer asks follow-up questions (dosage, timing, alternatives), I want expert-level answers without calling the office
- I want to serve more farmers per day without sacrificing quality of advice

**Farmer (Oksana) — served through the Consultant in v1:**
- When I call Ivan about a crop problem, I want a fast, accurate answer and the right product — not a "let me check and call you back"
- I want to trust that the recommendation accounts for my specific field conditions (soil, what I planted last year)
- *(Direct self-service JTBD deferred to v2)*

### 2.4 Non-Users (v1)

- **Farmers (self-service)** — UJ-2 (farmer self-service diagnosis) is **cut from v1**. Consultant-first strategy: farmers are served *through* their consultant in v1. Direct farmer self-service deferred to v2 (different failure tolerance — farmers have no agronomic training; splitting a 3.5-person team across two journeys with different error profiles is the wrong v1 trade-off).
- **Vendors** — served by Block E (Vendor Analytics), not v1
- **Traders** — served by Block B (Trading Agent), not v1
- **Operations staff** — served by Block C (Smart Operator), not v1

### 2.5 Key User Journeys

**UJ-1. Ivan diagnoses corn rust in the field and orders treatment before leaving the row.**
- **Persona + context:** Ivan is walking a farmer's cornfield during a routine visit. He spots rust-like spots on lower leaves.
- **Entry state:** Authenticated in the Flutter app; has GPS location and linked Farmland record for this field.
- **Path:** (1) Taps the Agro-Vision button on the home screen. (2) Points camera at affected leaves and captures a photo. (3) Within seconds, sees a Diagnosis Card: crop identified as corn (97% confidence), disease identified as corn rust / Puccinia sorghi (91% confidence), with visual signs description. (4) Scrolls to Prescription: three ranked products with dosage, vendor, warehouse availability, and stock status. (5) Reads the Agronomic Commentary — application timing, economic threshold, rotation advice. (6) Taps "Add to cart" on the top-ranked fungicide.
- **Climax:** Ivan shows the farmer the diagnosis on his phone screen — credibility established in 15 seconds.
- **Resolution:** Product is in cart, ready for checkout. Ivan can continue walking the field and repeat for other issues.
- **Edge case:** If confidence is below 70%, the system surfaces an uncertainty warning and suggests the Conversational Advisor for clarification ("Can you describe when the spots appeared?").

**UJ-2. [DEFERRED TO v2] Oksana self-diagnoses aphids on sunflower.**
- **Status:** Cut from v1 (architecture-phase decision, 2026-05-19). Farmer self-service has a different failure tolerance than consultant-assisted use — farmers have no agronomic training to catch a wrong diagnosis. v1 is consultant-first; farmers are served through their consultant. This journey returns in v2 once accuracy is field-validated.

**UJ-3. Ivan uses the Conversational Advisor after an ambiguous diagnosis.**
- **Persona + context:** Ivan photographs wheat with unusual discoloration. The diagnosis returns two possibilities with moderate confidence (65% septoria, 55% nitrogen deficiency).
- **Entry state:** Viewing a Diagnosis Card with multiple possibilities flagged.
- **Path:** (1) Taps "Ask the Advisor" below the diagnosis. (2) The Advisor asks clarifying questions: "Are the spots concentrated on lower leaves? When was the last nitrogen application?" (3) Ivan answers. (4) The Advisor narrows the diagnosis and updates the Prescription accordingly.
- **Climax:** Ambiguity resolved through conversation rather than a phone call to a senior agronomist.
- **Resolution:** Updated Prescription displayed; Ivan can order.

## 3. Glossary

- **Agro-Vision** — The photo-to-diagnosis-to-prescription feature. V1 flagship capability.
- **Prescription** — A ranked list of recommended products generated by the AI, including product name, type, dosage, vendor, warehouse availability, and stock status. Sourced from the Catalog.
- **Diagnosis Card** — The UI surface displaying crop identification, problem identification (pest/disease/weed/deficiency), confidence scores, visual signs, and links to the Prescription and Agronomic Commentary.
- **Agronomic Commentary** — Expert-level contextual advice accompanying a Prescription: economic thresholds, application timing, crop rotation considerations, follow-up monitoring schedule.
- **Conversational Advisor** — A chat-based follow-up interface for clarifying ambiguous diagnoses or asking treatment-related questions after an initial Agro-Vision scan.
- **Catalog** — The platform's product database (20,000+ SKUs) managed in Salesforce: seeds, crop protection products, fertilizers, fuel. Each product has regulatory approval status, vendor, category, and dosage metadata.
- **Farmland** — A Salesforce object representing a specific agricultural plot: soil type, crop rotation history, area, GPS coordinates, prior treatments.
- **Consultant** — An independent agricultural advisor or vendor sales manager who uses the platform to serve farmers. Primary user persona.
- **Farmer** — An owner or manager of an agricultural enterprise (SMB). Secondary user persona.
- **AI Foundation** — The shared infrastructure layer (orchestration, data connectors, vector database, evaluation framework) built in v1 that subsequent AI features reuse.
- **Confidence Score** — A percentage indicating the AI's certainty in a crop or problem identification. Displayed on the Diagnosis Card.
- **SKU** — Stock Keeping Unit. A single product entry in the Catalog.

## 4. Features

### 4.1 Photo Capture and Crop Identification

**Description:** The Consultant captures a photo of a crop through the app's camera interface. The system identifies the crop species with a Confidence Score. Realizes UJ-1. If the app cannot determine the crop with sufficient confidence (≥70%), it communicates uncertainty and suggests the Conversational Advisor.

**Functional Requirements:**

#### FR-1: Photo Capture

User can capture or upload a photo of a crop through the Agro-Vision interface in the Flutter app.

**Consequences (testable):**
- Camera opens within 1 second of tapping Agro-Vision button
- Supports both live capture and gallery upload
- Minimum image resolution: 640x480 pixels
- Supported formats: JPEG, PNG, HEIC

#### FR-2: Crop Identification

System identifies the crop species from the submitted photo and returns a Confidence Score.

**Consequences (testable):**
- Returns crop species name (common + Latin) and Confidence Score
- Confidence Score ≥70% required to proceed to problem diagnosis automatically
- If Confidence Score <70%, system displays uncertainty warning and suggests Conversational Advisor
- Response time: ≤10 seconds from photo submission to crop identification result

#### FR-3: Multi-crop Support

System supports identification of crops commercially relevant in Ukraine.

**Consequences (testable):**
- Minimum coverage: corn, sunflower, wheat (winter and spring), barley, soybean, rapeseed, sugar beet
- Accuracy: ≥85% on held-out test set across supported crops

### 4.2 Problem Diagnosis

**Description:** After crop identification, the system detects the specific problem: pest, disease, weed, or nutrient deficiency. Returns the problem name (common + Latin), problem type, Confidence Score, and a description of visual signs. Realizes UJ-1, UJ-3.

**Functional Requirements:**

#### FR-4: Problem Detection

System identifies the crop problem from the photo and classifies it by type (pest, disease, weed, deficiency).

**Consequences (testable):**
- Returns: problem name (common + Latin), type classification, Confidence Score, visual signs description
- Response time: included in the ≤10 second window from FR-2 (crop + problem identified in a single pass)
- If multiple problems are detected, returns ranked list by confidence

#### FR-5: Confidence Thresholding

System communicates diagnostic uncertainty when Confidence Score is below threshold.

**Consequences (testable):**
- Single high-confidence diagnosis (≥80%): displayed as primary result
- Multiple moderate-confidence diagnoses (50-80%): displayed as ranked possibilities with clear labeling
- Low confidence (<50%): system explicitly states it cannot identify the problem and routes to Conversational Advisor
- User always sees the Confidence Score

### 4.3 Prescription Generation

**Description:** The system generates a Prescription — a ranked list of recommended products from the Catalog. Products are filtered by Ukraine regulatory approval, current warehouse availability, and compatibility with the Farmland's soil type and crop rotation history. Each product entry includes dosage, vendor, stock status, and warehouse location. Realizes UJ-1.

**Functional Requirements:**

#### FR-6: Catalog Query and Filtering

System queries the Salesforce Catalog to find products matching the diagnosed problem, filtered by regulatory and availability constraints.

**Consequences (testable):**
- Only Ukraine-approved products are returned
- Products filtered by problem type (fungicide for disease, insecticide for pest, herbicide for weed)
- Real-time stock status from Salesforce warehouse data (in stock / under order / out of stock)
- Stock location (warehouse name) displayed per product

#### FR-7: Farmland Context Integration

System factors the user's Farmland data into the Prescription when available.

**Consequences (testable):**
- If Farmland record is linked: soil type, crop rotation history, and prior treatments are considered in product ranking
- Products chemically incompatible with prior treatments on the same Farmland are deprioritized or flagged
- If no Farmland record is linked: Prescription generated without context, with a note suggesting the user link their Farmland for better results

#### FR-8: Prescription Display

System displays a ranked Prescription with actionable detail for each recommended product.

**Consequences (testable):**
- Each product displays: name, type/category, recommended dosage, vendor name, stock status, warehouse location
- Minimum 2, maximum 5 products per Prescription
- Products ranked by: relevance to diagnosis, availability, Farmland compatibility (when available)
- "Add to cart" action available per product

### 4.4 Agronomic Commentary

**Description:** Each Prescription includes an Agronomic Commentary — contextual expert advice covering economic thresholds, application timing and method, crop rotation considerations, and follow-up monitoring schedule. This is what elevates the output from product search to senior-agronomist-level guidance. Realizes UJ-1.

**Functional Requirements:**

#### FR-9: Commentary Generation

System generates an Agronomic Commentary for each diagnosis + Prescription pair.

**Consequences (testable):**
- Commentary includes: economic damage threshold, recommended application timing, application method notes, crop rotation advice, follow-up monitoring schedule (days until recheck)
- Commentary adapts to user role when role is known (more technical detail for Consultant, simpler language for Farmer)
- Language: Ukrainian
- Length: 3-6 sentences

### 4.5 Conversational Advisor

**Description:** A chat interface that activates when a diagnosis is ambiguous (low confidence, multiple possibilities) or when the user has follow-up questions about a Prescription. The Advisor asks clarifying questions, narrows diagnoses, explains treatment options, and suggests alternatives. Realizes UJ-3.

**Functional Requirements:**

#### FR-10: Advisor Activation

User can invoke the Conversational Advisor from any Diagnosis Card or Prescription view.

**Consequences (testable):**
- "Ask the Advisor" button visible on every Diagnosis Card
- Advisor opens as a chat interface within the app
- Advisor has full context of the current diagnosis, Prescription, and Farmland data

#### FR-11: Diagnostic Clarification

Advisor asks targeted clarifying questions to narrow ambiguous diagnoses.

**Consequences (testable):**
- Questions are specific to the diagnostic ambiguity (e.g., symptom location, timing, recent treatments)
- After clarification, Advisor updates the diagnosis and Prescription
- Conversation is limited to the current diagnostic context (no general-purpose chat)

#### FR-12: Treatment Q&A

Advisor answers follow-up questions about Prescription products: dosage adjustments, application method, product alternatives, compatibility.

**Consequences (testable):**
- Answers grounded in Catalog data and agronomic knowledge
- If the question is outside scope (e.g., general farming advice unrelated to the diagnosis), Advisor states its boundary

### 4.6 AI Foundation Infrastructure

**Description:** The shared infrastructure layer that enables Agro-Vision and is designed for reuse by future AI features. Includes the LLM orchestration layer, Salesforce data connectors, vector database for Catalog retrieval, and the evaluation framework. Not user-facing directly.

**Functional Requirements:**

#### FR-13: LLM Orchestration

System provides a managed orchestration layer for LLM and vision model API calls.

**Consequences (testable):**
- Supports multimodal input (image + text) to LLM
- Handles prompt construction, response parsing, and error recovery
- Rate limiting and retry logic for API failures
- Request/response logging for debugging and evaluation

#### FR-14: Salesforce Data Connectors

System connects to Salesforce REST APIs to retrieve Catalog, Farmland, stock, and account data.

**Consequences (testable):**
- Read access to: Product Catalog (SKUs), Farmland records, warehouse stock levels, account data
- Data refresh: stock levels queried in real-time per Prescription; Catalog and Farmland data cached with configurable TTL
- Connector handles Salesforce API authentication and token refresh

#### FR-15: Catalog Vector Database

System maintains a vector index of the Catalog for retrieval-augmented generation (RAG).

**Consequences (testable):**
- Catalog products indexed with: name, category, active ingredients, approved crops, regulatory status
- Index updated when Catalog changes in Salesforce (batch or event-driven, configurable)
- Retrieval returns top-K relevant products for a given diagnosis context

#### FR-16: Evaluation Framework

System provides a framework for measuring Agro-Vision accuracy using a two-track methodology (resolves Open Question #1).

**Consequences (testable):**
- **Track 1 — Launch gate (blind panel):** Held-out test set of crop/problem photos labeled by a senior agronomist panel that is *independent of the Consultants using the tool*. Panel reviews diagnoses blind (without seeing the AI's answer) to avoid circular validation.
- **Track 2 — Field validation (post-launch):** Actual disease progression / yield outcomes tracked after treatment over the first growing season, validating the panel score against reality and informing v2 targets.
- Evaluation metrics: crop identification accuracy, problem diagnosis accuracy, Prescription relevance score
- Evaluation can be run on demand and results are logged
- Track 1 baseline (≥85%) established and gating before production launch; Track 2 runs continuously post-launch

## 5. Non-Goals (Explicit)

- **Not a general-purpose chatbot.** The Conversational Advisor is scoped to diagnosis clarification and treatment Q&A only. General farming advice, weather queries, market prices, and off-topic conversation are out of scope.
- **Not offline-capable.** V1 requires network connectivity for all AI features. Edge inference is a future consideration.
- **Not a custom CV model.** V1 uses multimodal LLM vision capabilities. Custom-trained computer vision models are only considered if test-set accuracy falls short of targets.
- **Not building Blocks B–E.** Trading Agent, Smart Operator, Market & Mentor, and Vendor Analytics are explicitly deferred to future phases.
- **Not a web version.** V1 is Flutter mobile only (iOS, Android).
- **Not rebuilding the app.** Agro Agent is an AI feature layer integrated into the existing AgroPlatforma app — not a rewrite.
- **Not handling orders end-to-end.** V1 adds products to cart. Checkout, payment, and fulfillment flows are existing platform functionality.

## 6. MVP Scope

### 6.1 In Scope

- Photo capture and upload via Flutter camera interface
- Crop identification (8+ commercially relevant Ukrainian crops)
- Problem diagnosis (pest, disease, weed, deficiency)
- Prescription generation from Salesforce Catalog (20k+ SKU) with stock availability
- Farmland context integration (soil, rotation) where data exists
- Agronomic Commentary per Prescription
- Conversational Advisor for ambiguous diagnoses and treatment Q&A
- AI Foundation: LLM orchestration, Salesforce connectors, vector DB, evaluation framework
- Production hardening: rate limiting, error handling, monitoring, logging
- Ukrainian language for all AI-generated content
- **Consultant-facing only** — the workflow targets UJ-1 (Consultant field diagnosis) and UJ-3 (Advisor). Farmers benefit through their Consultant.

### 6.2 Out of Scope for MVP

- **UJ-2: Farmer self-service diagnosis — deferred to v2.** Different failure tolerance (untrained users); consultant-first strategy. Revisit once accuracy is field-validated (Track 2 of FR-16).
- Natural-language catalog search (v1 Plus tier — deferred) `[NOTE FOR PM: strong demand signal from brief; revisit if timeline permits]`
- Offline/edge inference — requires fundamental architecture change
- Web version of Agro-Vision — Flutter web planned but not in v1
- Custom-trained CV model — only escalate if multimodal LLM accuracy falls short
- Block B: Trading Agent — deferred to Phase 2
- Block C: Smart Operator — deferred to Phase 2
- Block D: Market & Mentor — deferred to Phase 3
- Block E: Vendor Analytics — deferred to Phase 3
- Push notifications for treatment reminders — nice-to-have, not v1
- Multi-photo diagnosis (panoramic field scans) — single photo per diagnosis in v1

## 7. Success Metrics

**Primary**

- **SM-1:** Prescription accuracy ≥85% on held-out test set (crop ID + problem diagnosis + product relevance), measured against an **independent, blind senior agronomist panel** (not the Consultants using the tool — avoids circular validation). Field-outcome validation runs post-launch over the first season. Validates FR-2, FR-3, FR-4, FR-6, FR-7, FR-16.
- **SM-2:** Time to Prescription ≤30 seconds from photo capture to full Prescription display. Validates FR-2, FR-4, FR-8.
- **SM-3:** Consultant weekly active usage ≥50% of active Consultants within 3 months post-launch. Validates UJ-1, UJ-3.

**Secondary**

- **SM-4:** Prescription-to-cart conversion rate — baseline established in v1, growth is a v2 KPI. Validates FR-8.
- **SM-5:** Conversational Advisor resolution rate — % of ambiguous diagnoses successfully clarified without external escalation. Validates FR-11.
- **SM-6:** Consultant NPS lift — measurable improvement vs. pre-AI baseline (survey-based). Validates overall value proposition.

**Counter-metrics (do not optimize)**

- **SM-C1:** False confidence rate — % of high-confidence diagnoses (≥80%) that are incorrect. Counterbalances SM-1. Optimizing for raw accuracy must not inflate Confidence Scores artificially.
- **SM-C2:** LLM API cost per Prescription — must remain within the cost envelope from the pricing model. Counterbalances SM-2. Optimizing for speed must not ignore cost (e.g., redundant API calls).

## 8. Cross-Cutting NFRs

### Performance
- Photo-to-Diagnosis-Card: ≤10 seconds (including network round-trip to LLM API)
- Diagnosis-Card-to-Prescription: ≤5 seconds additional (Salesforce query + RAG)
- Total end-to-end: ≤30 seconds (SM-2)
- Concurrent users: system must handle 500 simultaneous Agro-Vision sessions without degradation

### Security and Data Privacy
- All API calls to LLM and Salesforce over HTTPS/TLS
- User photos are not stored beyond the current session unless the user explicitly opts in
- Farmland data accessed only for the authenticated user's linked records
- No Personally Identifiable Information (PII) sent to LLM APIs beyond what is needed for diagnosis (crop photo, Farmland agronomic data — no names, no financial data)
- Salesforce authentication via OAuth 2.0 with scoped permissions

### Reliability
- Graceful degradation: if LLM API is unavailable, user sees a clear error message — not a silent failure or a wrong diagnosis
- Retry logic with exponential backoff for transient API failures
- Circuit breaker pattern for sustained LLM API outages

### Observability
- Logging: every Agro-Vision session logged (photo hash, diagnosis result, Prescription, latency, user ID)
- Monitoring: real-time dashboard for API latency, error rates, Confidence Score distribution
- Alerting: threshold-based alerts for error rate spikes and latency degradation

### Localization
- All user-facing AI-generated content in Ukrainian
- Crop and product names in Ukrainian with Latin nomenclature where standard (pest/disease names)
- UI strings follow existing AgroPlatforma localization patterns

## 9. Integration and Dependencies

| System | Integration Type | Data | Direction | Risk |
|--------|-----------------|------|-----------|------|
| Salesforce (Catalog) | REST API | Product SKUs, categories, active ingredients, regulatory status | Read | APIs confirmed mostly ready |
| Salesforce (Farmland) | REST API | Soil type, crop rotation, area, GPS, prior treatments | Read | Data completeness varies per farm |
| Salesforce (Stock) | REST API | Warehouse stock levels, locations | Read (real-time) | Latency of stock queries at scale |
| LLM API (Claude) | HTTPS API | Photos (multimodal), text prompts, structured responses | Send/Receive | Cost management, rate limits, latency |
| Vector Database | Internal | Catalog embeddings for RAG retrieval | Read/Write | Index freshness vs. Catalog changes |
| Flutter App | In-app integration | Camera, UI surfaces, cart, user session | Bidirectional | Existing app architecture constraints |

## 10. Risk and Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| LLM diagnostic accuracy below 85% target | Medium | High — blocks launch | Evaluation framework (FR-16) with iterative prompt tuning; escalation path to custom CV model |
| Salesforce API latency degrades Prescription time | Medium | Medium — poor UX | Caching strategy (Catalog, Farmland); stock queries are the only real-time dependency |
| LLM API cost exceeds budget at scale | Medium | Medium — margin erosion | Token usage monitoring (SM-C2); prompt optimization; response caching for repeat diagnoses |
| Farmland data incomplete for many users | High | Low — Prescription still works, just less personalized | Graceful fallback: generate Prescription without Farmland context, suggest user links their farm |
| Photo quality from field conditions (glare, rain, distance) | Medium | Medium — reduced accuracy | Guidance in UI (lighting tips, distance suggestions); low-confidence handling (FR-5) |
| Regulatory data goes stale (product approvals change) | Low | High — legal risk | Catalog sync process with Salesforce as source of truth; flag products with outdated regulatory timestamps |

## 11. Why Now

$250,000 has already been invested building AgroPlatforma — the ordering platform, the Salesforce backend, the Flutter apps. The infrastructure is live and the client base is 40k+ enterprises. But the platform is still primarily an ordering tool — substitutable by a phone call and a spreadsheet.

AI is the capability that makes AgroPlatforma irreplaceable. A consultant who can diagnose in the field and prescribe from the catalog in seconds will not go back to phone calls. The AI Foundation built now is the investment that makes every subsequent AI feature (Blocks B–E) incrementally cheaper. The window is now: multimodal LLM capabilities (vision + language + structured output) have matured to the point where this workflow is technically feasible without custom model training.

## 12. Open Questions

1. ✅ **RESOLVED (2026-05-19):** Accuracy methodology = hybrid two-track (FR-16, SM-1): independent blind senior-agronomist panel as the v1 launch gate; field-outcome tracking post-launch. Still to confirm: panel size, sample count per crop/problem category, and *who staffs the panel*.
2. What is the target LLM API cost per Prescription at the 40k-user scale? What is the acceptable monthly spend?
3. How frequently does the Salesforce Catalog change? What is the acceptable staleness window for the vector index?
4. Is there a regulatory data source for Ukraine-approved crop protection products that can be cross-referenced, or is the Salesforce Catalog the sole source of truth for approval status?
5. What is the existing Flutter app's architecture for adding new feature modules? Are there constraints on navigation patterns or state management?
6. How will the pilot be rolled out? All consultants at once, or a phased rollout by region?

## 13. Assumptions Index

- §4.1 FR-3: Multi-crop support scope (8+ crops) is based on commercially relevant Ukrainian crops. Exact list to be finalized with domain expert input.
- §4.3 FR-7: Farmland data completeness varies. Prescription works without it but is less personalized.
- §8 Performance: 500 concurrent users estimate based on initial rollout; may need revision for full 40k base.
- §9: Salesforce APIs are ready or mostly ready (confirmed by user).
- §9: Ukraine crop/pest taxonomy and approved chemical registry are accessible (confirmed by user).
