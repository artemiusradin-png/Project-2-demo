---
title: "Agro Agent — AI Layer for AgroPlatforma"
status: final
created: 2026-05-19
updated: 2026-05-19
---

# Product Brief: Agro Agent

## Executive Summary

AgroPlatforma is a B2B platform that digitizes the Ukrainian agricultural supply chain — connecting 40,000+ farming enterprises with seed, crop-protection, fertilizer, and fuel suppliers through a single digital layer built on Salesforce and Flutter. The platform is backed by Agro-Arena, a distributor with 20 years in Ukrainian agribusiness.

**Agro Agent** is an AI capability layer integrated into the existing app — not a rebuild. The v1 flagship is **Agro-Vision**: a consultant or farmer photographs a crop in the field, and within seconds receives an expert-level diagnosis (crop identification, pest/disease/weed detection) paired with a concrete "prescription" — specific products from the platform's 20,000+ SKU catalog that are approved for use in Ukraine, with real-time warehouse availability and agronomic context (soil type, crop rotation history).

This is the capability that turns AgroPlatforma from an ordering tool into an indispensable field companion. The AI foundation built for v1 is designed to be reused: subsequent AI features (trading intelligence, sales automation, vendor analytics) attach to it at a fraction of the initial cost.

## The Problem

Agricultural consultants in Ukraine serve as the critical link between input suppliers and farmers. Today, when a consultant encounters a crop problem in the field, the workflow is slow and manual:

1. **Diagnosis is experience-dependent.** Junior consultants lack the expertise to confidently identify diseases, pests, or nutrient deficiencies across the full crop spectrum. They call senior agronomists, send photos via Viber, wait for responses. Misdiagnosis means wrong product recommendations and lost farmer trust.

2. **Product selection is fragmented.** Even after diagnosis, matching the right treatment to what is actually available — legally approved in Ukraine, in stock at nearby warehouses, compatible with the farmer's soil and rotation — requires cross-referencing multiple systems. Consultants browse the catalog manually, call warehouses, guess at compatibility.

3. **Time kills deals.** A consultant who cannot answer in the field loses the sale to whoever answers first. The current workflow takes hours to days. The farmer moves on, calls another supplier, or does nothing — all of which cost revenue.

4. **Expertise doesn't scale.** Agro-Arena's competitive advantage is deep agronomic knowledge. But that knowledge lives in the heads of a few senior people. The platform has 40k+ potential clients; the expert bench has single-digit depth.

## The Solution

**Agro-Vision** collapses the field-to-prescription workflow from hours to seconds:

- **Point. Shoot. Diagnose.** The consultant opens the app, photographs the affected crop. A multimodal LLM (Claude) identifies the crop species, detects the problem (disease, pest, weed, deficiency), and reports confidence levels.

- **Instant prescription.** The system queries the platform's Salesforce catalog via RAG (retrieval-augmented generation), filtering by: Ukraine-approved products, current warehouse stock levels, the farmer's registered farmland properties (soil type, crop rotation, prior treatments). It returns a ranked list of recommended products with dosage, vendor, and availability.

- **Agronomic context.** Each prescription includes a "senior agronomist commentary" — economic thresholds, application timing, follow-up monitoring guidance, rotation considerations. This is what elevates the output from a product search to genuine expert advice.

- **Foundation for future AI.** The orchestration layer, Salesforce data connectors, vector database, and evaluation framework built for Agro-Vision are reusable. Blocks B through E (Trading Agent, Smart Operator, Market & Mentor, Vendor Analytics) plug into this foundation.

## What Makes This Different

- **Closed-loop prescription, not generic advice.** Most agro-AI tools stop at diagnosis. Agro-Vision goes from photo to purchasable product from the existing catalog — with real stock levels and Ukraine regulatory compliance baked in. The user doesn't leave the app.

- **Built on a real distribution network.** Agro-Arena's 20-year supplier relationships, 20k+ SKU catalog, and warehouse network are the moat. The AI makes this catalog actionable in the field — something a standalone AI app cannot replicate without the supply chain behind it.

- **Consultant-first economics.** The platform's growth model is consultant-driven: one consultant brings a pool of farmers. The AI makes consultants more effective (and more loyal to the platform), which drives organic acquisition at lower cost than farmer-direct marketing.

- **No direct competitor in Ukraine.** Existing tools are either generic global apps (Plantix, Agrio) without local catalog integration, or Salesforce-native tools (Agentforce) without vision capabilities. No one offers the closed-loop photo-to-purchasable-product workflow tied to a real distribution network.

## Who This Serves

**Primary: Agricultural Consultants**
Independent experts and vendor sales managers who advise farmers on inputs. They need to look knowledgeable in the field, respond fast, and close orders on the spot. Success = they earn more, spend less time on paperwork, and their farmers trust them more.

**Secondary: Farmers (SMB)**
Owners and managers of small-to-medium farming operations. They want a fast answer when something is wrong with their crop and a product they can order immediately. Success = problem identified and solution ordered in one session, no phone tag.

**Downstream (future phases): Vendors and Traders**
Suppliers get data on what products are being recommended and where. Traders get market intelligence. These personas are served by Blocks B–E, not v1.

## Success Criteria

| Metric | Target | Notes |
|--------|--------|-------|
| Prescription accuracy | ≥85% on held-out test set | Measured against senior agronomist panel review |
| Time to prescription | <30 seconds | From photo capture to full recommendation |
| Consultant adoption | 50%+ of active consultants using weekly within 3 months post-launch | Tracked via app analytics |
| Prescription-to-order conversion | Baseline + track | V1 establishes baseline; growth is a v2 KPI |
| Consultant NPS lift | Measurable improvement vs. pre-AI baseline | Survey-based |

## Scope

### V1 — In (AI Foundation + Agro-Vision)

- Multimodal LLM integration (Claude) for crop/pest/disease/weed identification from photos
- RAG pipeline connecting to Salesforce product catalog (20k+ SKU)
- Prescription engine: product matching with dosage, warehouse availability, agronomic context
- Ukraine regulatory compliance filtering (approved products only)
- Farmland context integration (soil type, crop rotation) via Salesforce REST APIs
- Evaluation framework with held-out test set for accuracy measurement
- Production hardening: rate limiting, error handling, monitoring
- Conversational agronomist advisor (follow-up questions after diagnosis)
- Flutter UI integration (iOS, Android)

### V1 — Out

- Block B: Trading Agent (commodity intelligence, crop planning)
- Block C: Smart Operator (automated KP generation, upsell/cross-sell)
- Block D: Market & Mentor (task marketplace, gamified training)
- Block E: Vendor Analytics (supplier dashboards and reports)
- Web version of Agro-Vision
- Offline/edge inference (requires network connectivity for v1)
- Custom-trained computer vision models (v1 uses multimodal LLM; escalate to custom CV only if accuracy falls short)
- Natural-language catalog search (v1 Plus scope, not Standard v1)

### Dependencies & Risks

- **Salesforce APIs are ready or mostly ready.** Catalog, stock, farmland, and soil data can be exposed via REST APIs with moderate effort.
- Ukraine-specific crop/pest taxonomy and approved chemical registry are available as structured reference data for the RAG pipeline.
- LLM/vision API costs are usage-based and billed to the client. Vision is token-heavy at scale (40k users).

## Vision

If Agro-Vision succeeds, the AI foundation it builds becomes the nervous system of AgroPlatforma:

**Year 1:** Agro-Vision is live. Consultants treat it as their pocket senior agronomist. The prescription-to-order loop tightens — consultants who use AI close faster and handle more farmers. Platform stickiness increases measurably.

**Year 2:** Trading Agent and Smart Operator come online. The platform doesn't just help consultants react to field problems — it proactively suggests what to plant next season based on price trends, soil data, and rotation history. Order processing becomes semi-automated, reducing operations staff needs and letting the business scale without proportional headcount growth.

**Year 3:** The full five-block agent suite is operational. AgroPlatforma evolves from a B2B e-commerce tool into an AI-powered agricultural operating system. Vendors get real-time market intelligence. The task marketplace and training system create a self-improving consultant network. The platform becomes the default digital infrastructure for Ukrainian agricultural commerce — and the model is exportable to other emerging agricultural markets.

The $250k already invested built the platform. The AI layer is what makes it irreplaceable.
