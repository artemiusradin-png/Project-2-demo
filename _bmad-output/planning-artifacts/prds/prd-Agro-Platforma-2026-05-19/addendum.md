---
title: "Agro Agent v1 PRD — Addendum"
created: 2026-05-19
updated: 2026-05-19
---

# PRD Addendum: Agro Agent v1

Supplementary detail for downstream documents (architecture, UX spec, epics) that does not belong in the PRD itself.

## Technical Approach Options (for Architecture Phase)

The original client brief asks: "Salesforce Agentforce vs. custom LLM solution (GPT-5/Claude/Gemini-3) with API integration?" This is an architecture decision, not a PRD concern. The PRD captures capabilities; the architecture phase will evaluate:

1. **Custom LLM orchestration** — External multimodal LLM (Claude recommended in pricing doc) with custom orchestration layer, RAG pipeline, and Salesforce connectors. Maximum flexibility, full control over prompt engineering and evaluation.
2. **Salesforce Agentforce** — Native Salesforce AI tooling. Tighter Salesforce integration but limited vision capabilities and customization surface.
3. **Hybrid** — Agentforce for data access and workflow, external LLM for vision and diagnosis.

The pricing one-pager's cost model assumes approach #1 with Claude as the primary LLM.

## Internal Cost Model Reference

From the pricing one-pager (Standard v1, 3 months):
- Salesforce developer: $12,500
- ML/CV specialist (part-time advisory): $6,000
- LLM/vision API dev & testing: $2,000
- Infra/hosting: $1,200
- Tooling/subscriptions: $1,800
- Contingency (15%): ~$3,500
- **Total out-of-pocket: ~$27k–$32k**
- Recommended client price: $105k (negotiation floor: $70k)

## Future Blocks Detail (Post-V1)

### Block B — Trading Agent
- Crop planning based on rotation + commodity price trends
- Integration with international commodity exchanges
- Buy/sell opportunity identification

### Block C — Smart Operator & Sales
- Automated commercial proposal (KP) in Salesforce
- Upsale/cross-sell analog selection
- Order automation to reduce operations headcount

### Block D — Market & Mentor
- Kabanchik-style task marketplace
- Gamified consultant training with vendor-provided materials
- AI-matched tasks based on consultant rating

### Block E — Vendor Analytics
- Per-vendor personalized analytical reports
- Regional proposals and issue highlighting
- Dashboards designed for less digital vendor reps

## Demo Scenarios (from agro-demo)

Three pre-built scenarios demonstrating the photo→diagnosis→prescription flow:

1. **Corn rust** — Puccinia sorghi, 91% confidence → fungicide (Capt Pro 325 SC) + contact fungicide (Mancozeb 800 WP) + micronutrient (Adenin Mn+Zn)
2. **Sunflower aphid** — Aphis fabae, 88% confidence → insecticide (Karate Zeon 050 CS) + systemic insecticide (Bi-58 New) + surfactant (Liposam)
3. **Wheat white goosefoot** — Chenopodium album, 90% confidence → herbicide (Granstar Pro 75) + herbicide (Lancelot 450 WG) + surfactant (Trend 90)

These scenarios can serve as the basis for the evaluation test set (FR-16) and as acceptance test cases.

## Data Security Notes (for Architecture)

Client brief question: "Data security model for 40k+ client base — anonymization vs. direct data usage?"

Recommendation for architecture phase:
- Photos sent to LLM API: no PII attached, only agronomic metadata (crop type, region — not farmer identity)
- Farmland data: used in Prescription generation server-side; Farmland identifiers not sent to external LLM
- Session data: logged internally for evaluation; retention policy to be defined
- Salesforce data: accessed via scoped OAuth; principle of least privilege
