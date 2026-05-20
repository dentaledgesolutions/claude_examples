# Requirements — DES Intelligence Platform

---

## v1 Requirements (Milestone 1 — Proposal Engine)

### Foundation

- [ ] **FOUND-01** — Project directory structure initialized: `knowledge/`, `audit/submissions/`, `proposals/`, `content/briefs/`, `.claude/skills/`
- [ ] **FOUND-02** — `knowledge/services.md` seeded from `DentalEdgeSolutions_Proposal.html` with all packages, à la carte services, and pricing
- [ ] **FOUND-03** — GHL custom field map documented: `locations_get-custom-fields` called and all field IDs + keys recorded in `audit/field-map.json`
- [ ] **FOUND-04** — Data4SEO sandbox connection verified with a test domain lookup (Maps SERP + Business Data endpoints)
- [ ] **FOUND-05** — GHL blog pre-fetch IDs documented: `blogId`, `authorId` (Erick), default category ID stored in `knowledge/ghl-config.json`

### Company Study

- [ ] **COMPANY-01** — `/des-interview` skill runs structured Company Study interview covering: services, pricing, ideal client profile, key differentiators, geographic boundaries, onboarding process, HIPAA approach
- [ ] **COMPANY-02** — Interview produces `knowledge/company-profile.md` — authoritative DES reference used by all other skills

### Market Study

- [ ] **MARKET-01** — Market Study research produces `knowledge/market-study.md` documenting: top 10 problems of SE FL independent dental practices, DSO competitive landscape (Aspen, Heartland, Dental Care Alliance, Great Expressions, Western, MB2), competitor analysis (Weave, RevenueWell, Adit, MaxAssist, Aron AI, DoctorLogic), multilingual market demographics, local search dynamics
- [ ] **MARKET-02** — Market Study findings validated against existing `DentalEdgeSolutions_Proposal.html` — any new problems or competitor data surfaced for service portfolio update

### Practice Audit

- [ ] **AUDIT-01** — Practice Audit form designed in `audit/form-design.md`: questions derived from top 10 market problems, each question mapped to a named GHL custom field key
- [ ] **AUDIT-02** — Practice Audit form created in GHL with custom fields matching `audit/form-design.md`
- [ ] **AUDIT-03** — `/des-audit [contact-id]` skill: reads contact custom fields via `contacts_get-contact`, maps field IDs using cached field map, calls Data4SEO for domain (Maps SERP + Business Data), produces structured `audit/submissions/[practice-name].md`
- [ ] **AUDIT-04** — AuditResult includes: practice profile, top 3–5 detected problems (prioritized by impact), Data4SEO findings (local rank for top 3 keywords, nearest 3 competitors with ratings, GBP review count + rating), recommended DES services matched to each problem

### Proposal Engine

- [ ] **PROPOSAL-01** — `/des-proposal [contact-id]` skill: runs `/des-audit` internally (or loads fresh AuditResult), loads `company-profile.md` + `services.md` + `market-study.md`, generates complete personalized HTML proposal
- [ ] **PROPOSAL-02** — Generated proposal matches quality and structure of `DentalEdgeSolutions_Proposal.html`: fully self-contained HTML, interactive tabs and CTAs, mobile-responsive, print/export button
- [ ] **PROPOSAL-03** — Proposal is personalized: uses practice name throughout, cites their specific Data4SEO findings (actual keyword ranks, actual review count/rating, actual competitor names), recommends specific services matching their detected problems
- [ ] **PROPOSAL-04** — Proposal saves to `proposals/[practice-name]-[date].html` and outputs local file path
- [ ] **PROPOSAL-05** — On user approval, skill writes proposal summary data to 10 GHL contact custom fields, moves opportunity to "Proposal Sent" stage, adds activity note via `contacts_update-contact` + `opportunities_update-opportunity`

---

## v2 Requirements (Milestone 2 — Intelligence Layer)

### Service Portfolio Validation

- [ ] **PORTFOLIO-01** — `knowledge/services.md` reviewed and updated against Market Study findings: each service cross-referenced with identified market problems, gaps noted, services without market problem match flagged
- [ ] **PORTFOLIO-02** — Any new service opportunities identified by Market Study documented as proposals in `knowledge/service-opportunities.md` (not yet added to catalog)

### Benchmarking

- [ ] **BENCH-01** — `knowledge/benchmarking.md` documents DES vs. Weave, RevenueWell, Adit, MaxAssist, Aron AI, DoctorLogic across: capabilities table, pricing comparison, SE Florida presence, done-for-you vs. DIY positioning, multilingual capability
- [ ] **BENCH-02** — Proposals auto-pull from `benchmarking.md` for the competitive comparison section (replaces the static table in the existing proposal)

### Intelligence Layer

- [ ] **INTELL-01** — `knowledge/intelligence/sources.md` documents all tracked sources: ADA News, Dental Economics, Dentistry Today, Dental Products Report, competitor blog/pricing pages, Reddit (r/dentistry, r/DentalHygiene, r/smallbusiness), LinkedIn public posts, YouTube dental channels
- [ ] **INTELL-02** — `/des-trends [--refresh]` skill: fetches and synthesizes content from all tracked sources, updates `knowledge/intelligence/trends.md` with dated entries, produces 3–5 content briefs saved to `knowledge/intelligence/briefs/`
- [ ] **INTELL-03** — Trends refresh cadence: monthly automated (via scheduled agent) + on-demand manual trigger. `--refresh` flag forces a fresh fetch; without it, skill shows current `trends.md` without refetching
- [ ] **INTELL-04** — Proposals automatically load current `knowledge/intelligence/trends.md` as additional context for relevant trend references in proposal copy

### Content Pipeline

- [ ] **CONTENT-01** — `/des-content [brief-id|topic]` skill: accepts a brief ID from `knowledge/intelligence/briefs/` or a free-form topic, loads knowledge base context, generates complete blog post with SE Florida local SEO optimization and FAQ schema markup
- [ ] **CONTENT-02** — Content generation includes AskUserQuestion approval gate before publishing: shows draft → user approves → skill publishes to GHL blog via `blogs_create-blog-post` with status DRAFT for final GHL review
- [ ] **CONTENT-03** — Published content logged in `content/published/` with title, date, GHL post ID, and source brief

---

## Out of Scope

- **GA4 integration** — replaced by Data4SEO for prospect web presence data
- **SW Florida market** — DES operates in SE Florida (Miami-Dade, Broward, Palm Beach) only; source document error
- **Automated proposal delivery** — proposals always reviewed by DES before reaching prospects
- **Facebook group monitoring** — requires private group membership; not automatable
- **GBP posting activity monitoring** — not available via any third-party API; requires OAuth from business owner
- **Web application or dashboard UI** — Claude Code CLI project only
- **DSO clients** — DES serves independent practices only
- **Direct HTML file attachment to GHL contacts** — unsupported file type; use custom field merge approach instead

---

## Traceability

| REQ-ID | Phase | Status |
|--------|-------|--------|
| FOUND-01 | Phase 1: Foundation | — |
| FOUND-02 | Phase 1: Foundation | — |
| FOUND-03 | Phase 1: Foundation | — |
| FOUND-04 | Phase 1: Foundation | — |
| FOUND-05 | Phase 1: Foundation | — |
| COMPANY-01 | Phase 2: Company Study | — |
| COMPANY-02 | Phase 2: Company Study | — |
| MARKET-01 | Phase 3: Market Study | — |
| MARKET-02 | Phase 3: Market Study | — |
| AUDIT-01 | Phase 4: Practice Audit System | — |
| AUDIT-02 | Phase 4: Practice Audit System | — |
| AUDIT-03 | Phase 4: Practice Audit System | — |
| AUDIT-04 | Phase 4: Practice Audit System | — |
| PROPOSAL-01 | Phase 5: Proposal Engine | — |
| PROPOSAL-02 | Phase 5: Proposal Engine | — |
| PROPOSAL-03 | Phase 5: Proposal Engine | — |
| PROPOSAL-04 | Phase 5: Proposal Engine | — |
| PROPOSAL-05 | Phase 5: Proposal Engine | — |
| PORTFOLIO-01 | Phase 6: Portfolio Validation | — |
| PORTFOLIO-02 | Phase 6: Portfolio Validation | — |
| BENCH-01 | Phase 7: Benchmarking | — |
| BENCH-02 | Phase 7: Benchmarking | — |
| INTELL-01 | Phase 8: Trends Engine | — |
| INTELL-02 | Phase 8: Trends Engine | — |
| INTELL-03 | Phase 8: Trends Engine | — |
| INTELL-04 | Phase 8: Trends Engine | — |
| CONTENT-01 | Phase 9: Content Pipeline | — |
| CONTENT-02 | Phase 9: Content Pipeline | — |
| CONTENT-03 | Phase 9: Content Pipeline | — |
