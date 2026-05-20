---
title: "Agro Agent Brief — Addendum"
created: 2026-05-19
updated: 2026-05-19
---

# Brief Addendum: Agro Agent

Supplementary detail from source materials that belongs in downstream documents (PRD, architecture) rather than the executive brief.

## Feature Blocks B–E (Post-V1 Roadmap Detail)

### Block B — Trading Agent (Commercial Brain)
- Proposals for farmer/consultant considering crop rotation and monitoring global/local grain price trends
- AI analyzes past harvests and farmlands, suggests future planning based on local suppliers and international commodity exchanges
- Buy/sell opportunity identification (optional — intent to integrate traders and exchanges in future)

### Block C — Smart Operator & Sales (Operational Excellence)
- Automated commercial proposal (KP) generation in Salesforce
- Analog selection for upsale/cross-sell
- Order automation
- Goal: minimize operations staff, focus on supervisors, enable growth without proportional HR expansion

### Block D — Market & Mentor (Network Development)
- Kabanchik-style task marketplace within the platform
- Gamified consultant training: tests, webinars, supplementary knowledge from vendors
- AI matches tasks to consultant rating, trains on past mistakes and vendor materials

### Block E — Vendor Analytics
- Personalized analytical reports per supplier on product performance
- Regional proposals and issue highlighting in real-time
- Analytical dashboards designed for less digitally-savvy vendor representatives
- Focus consultant recommendations for individual vendor collaboration

## Technical Context (from original brief)

### Tech Stack Detail
- **Salesforce objects in scope:** Catalog of Products (20k+ SKU), Accounts (clients), Farmlands, Selling Opportunity (SO), Buying Opportunity (BO), Tasks, Soils, Weeds, Treatments, Chemistry
- **External integrations via REST API:** crop yield databases, tax registries, weather data
- **Planned catalog expansion:** agricultural machinery, production equipment

### Open Technical Questions (from client brief)
1. Approach recommendation: Salesforce Agentforce vs. custom LLM solution (GPT-5/Claude/Gemini-3) with API integration?
2. Toolset planned for each feature block?
3. Data security model for 40k+ client base — anonymization vs. direct data usage?
4. Team experience with AI integration in Flutter apps, including offline logic where necessary?
5. What datasets are required for the described features?

### Internal Cost Model (Standard V1, 3 months)
- Salesforce developer (UA, mid-senior, contract): $12,500
- ML/CV specialist (UA, senior, part-time advisory): $6,000
- LLM/vision API dev & testing: $2,000
- Infra/hosting (orchestrator, vector DB, staging): $1,200
- Tooling/subscriptions: $1,800
- Contingency (15%): ~$3,500
- **Total cash out-of-pocket: ~$27k–$32k**
- Two Flutter founders not costed (→ margin)
- Negotiation floor: $70k (below this, deal not worth founder time + delivery risk)

### Pricing Tiers Offered
| Tier | Scope | Price | Timeline |
|------|-------|-------|----------|
| Lean v1 | AI foundation + Agro-Vision | $75k–$85k | ~10–12 weeks |
| **Standard v1 (recommended)** | Lean + conversational advisor + hardening | **$95k–$115k** | **~12 weeks** |
| v1 Plus | Standard + NL catalog search | $135k–$155k | ~16 weeks |

Recommended quote: $105k fixed price, ~12 weeks. Payment: 30% signature / 40% mid-milestone / 30% delivery.

## Agro-Vision Demo Scenarios (from agro-demo)
Three pre-built demo scenarios showcasing the photo→diagnosis→prescription flow:
1. **Corn** — Corn rust (Puccinia sorghi), 91% confidence → fungicide + micronutrient prescription
2. **Sunflower** — Aphid infestation (Aphis fabae), 88% confidence → insecticide + surfactant prescription
3. **Wheat** — White goosefoot weed (Chenopodium album), 90% confidence → herbicide + surfactant prescription

Each scenario demonstrates: crop identification, problem diagnosis with Latin name, severity signs, ranked product recommendations with vendor/stock/dosage, and agronomic commentary.
