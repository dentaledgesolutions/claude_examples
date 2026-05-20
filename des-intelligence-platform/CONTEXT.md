# DES Intelligence Platform — Domain Context

This is the authoritative glossary for the DES Intelligence Platform project.
All modules, skills, and documents must use these terms consistently.

---

## Core Entities

### DES
DentalEdge Solutions. A done-for-you dental marketing and automation agency serving independent dental practices in SE Florida. DES builds and manages GoHighLevel-native systems so practice owners can focus on clinical work. DES does not sell software subscriptions — it delivers fully managed implementations.

### Target Market
Independent dental practices and small groups (2–5 offices) in Southeast Florida — specifically Miami-Dade, Broward, and Palm Beach counties. DES does not serve DSOs or practices outside SE Florida.

### Practice
A dental practice in the Target Market. The prospect or client DES is selling to or serving. A Practice has an owner (often the dentist), a practice manager, and front-desk staff — each with distinct problems.

### Practice Growth Engine
DES's branded service system. Four tiered packages (Digital Foundation, Efficiency Engine, Patient Acquisition, Market Leader) plus à la carte services. The Practice Growth Engine is the primary commercial offering documented in `knowledge/services.md`.

---

## Platform Layers

### Knowledge Base
The collection of research documents that give the platform its intelligence. Lives in `/knowledge/`. Includes `company-profile.md`, `market-study.md`, `benchmarking.md`, `services.md`, and `intelligence/trends.md`. Read by all operational skills at runtime. Updated periodically, not per-proposal.

### Intelligence Layer
The trends monitoring subsystem. Tracks SE Florida dental industry publications, competitor websites, and social signals (Reddit, LinkedIn, YouTube) on a monthly cadence. Produces `intelligence/trends.md` and content opportunity briefs. Accessed via `/des-trends`.

### Operational Workflow
The per-practice pipeline: Practice Audit → Data Enrichment → Proposal Generation. Triggered on demand for each new prospect. Accessed via `/des-audit` and `/des-proposal`.

---

## Key Workflows

### Practice Audit
A structured assessment of a specific dental practice's operational and digital marketing situation. Conducted via a GHL form completed by or about the prospect. The form is designed from Market Study findings — questions map directly to the problems most common in the Target Market. Audit data is the primary input for Proposal generation.

### Data Enrichment
Automated augmentation of Practice Audit data using Data4SEO. Pulls three signals for the prospect's domain: (1) local SEO rankings for dental keywords in their city, (2) competitor comparison against 2–3 nearby practices, (3) Google Business Profile signals (reviews, ratings, posting activity). Enrichment requires no action from the prospect.

### Service Proposal
A personalized HTML document generated for a specific Practice. Combines Practice Audit data, Data Enrichment signals, and the Knowledge Base to argue which DES services will solve the practice's specific problems. Saved locally as `/proposals/[practice-name]-[date].html`, reviewed by DES, then pushed to the GHL contact record when ready to send.

### Content Brief
A structured content opportunity produced by the Intelligence Layer during a trends refresh. Contains a suggested title, angle, target keyword, and connection to current trends. Reviewed and approved by DES before a full piece is written and published to the GHL blog.

---

## Integrations

### GHL MCP
The GoHighLevel MCP server connected to Claude Code. Used to: pull Practice Audit form submissions by contact ID, push completed proposals to contact records, publish approved content to the GHL blog. The primary bridge between the platform and the CRM.

### Data4SEO
The SEO data provider. Used exclusively for prospect Data Enrichment during the Practice Audit workflow. Credentials stored as environment variables (`DATA4SEO_LOGIN`, `DATA4SEO_PASSWORD`).

---

## Competitors

Direct competitors referenced in Market Study and Benchmarking: Weave, RevenueWell, Adit, MaxAssist, Aron AI, DoctorLogic. Competitive comparison table maintained in `knowledge/benchmarking.md`.

---

## Geographic Scope

**SE Florida only.** Miami-Dade, Broward, Palm Beach. The document `DES Plan Basics.docx` incorrectly said "Southwest Florida" — the correct market is Southeast Florida. All market research, local SEO analysis, and demographic assumptions (multilingual: Spanish, Haitian Creole, Portuguese) are scoped to SE Florida.
