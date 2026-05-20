# Roadmap — DES Intelligence Platform

*Generated: 2026-05-20 | Mode: MVP | Granularity: Medium*

---

## Milestones

- **Milestone 1: Knowledge Base** — Phases 1–7 — Full intelligence foundation ready (company profile, market study, audit system, benchmarking, trends, validated services)
- **Milestone 2: Operational Skills** — Phases 8–10 — Operational skills live using the complete knowledge base (`/des-proposal`, `/des-content`, all skills integrated)

---

## Phases

### Milestone 1: Knowledge Base

- [ ] **Phase 1: Foundation** — Project structure, verified integrations, and seeded knowledge base
- [ ] **Phase 2: Company Study** — Authoritative DES company profile in `knowledge/company-profile.md`
- [ ] **Phase 3: Market Study** — SE Florida market intelligence in `knowledge/market-study.md`
- [ ] **Phase 4: Practice Audit System** — GHL form live + `/des-audit` produces complete AuditResult
- [ ] **Phase 5: Benchmarking** — `benchmarking.md` complete with DES vs. all 6 direct competitors
- [ ] **Phase 6: Trends Engine** — `/des-trends` monitors sources and produces monthly content briefs
- [ ] **Phase 7: Service Portfolio Validation** — `services.md` cross-referenced with market problems, benchmarking, and trends

### Milestone 2: Operational Skills

- [ ] **Phase 8: Proposal Engine** — `/des-proposal` generates personalized HTML proposal in under 5 minutes using full knowledge base
- [ ] **Phase 9: Content Pipeline** — `/des-content` generates and queues GHL blog posts
- [ ] **Phase 10: Integration and Polish** — All 5 skills documented and working end-to-end

---

## Phase Details

### Phase 1: Foundation
**Goal**: Establish the project directory structure, verify all external integrations, and seed the knowledge base so every subsequent phase has a working foundation to build on.
**Mode:** mvp
**Depends on**: Nothing
**Requirements**: FOUND-01, FOUND-02, FOUND-03, FOUND-04, FOUND-05
**Success Criteria** (what must be TRUE):
  1. All directories exist: `knowledge/`, `audit/submissions/`, `proposals/`, `content/briefs/`, `.claude/skills/`
  2. `knowledge/services.md` contains all DES packages and à la carte services extracted from `DentalEdgeSolutions_Proposal.html`
  3. `audit/field-map.json` documents all GHL custom field IDs and keys for the location
  4. Data4SEO sandbox returns valid results for a test domain via Maps SERP and Business Data endpoints
  5. `knowledge/ghl-config.json` contains `blogId`, `authorId`, and default category ID
**Plans**: 4 plans

Plans:
- [ ] 01-01-PLAN.md — Directory scaffold (all project output paths)
- [ ] 01-02-PLAN.md — Seed knowledge/services.md from proposal HTML
- [ ] 01-03-PLAN.md — GHL field map + blog config IDs
- [ ] 01-04-PLAN.md — Data4SEO sandbox verification

### Phase 2: Company Study
**Goal**: Produce an authoritative, structured DES company profile that every skill can reference for services, pricing, positioning, and differentiators.
**Mode:** mvp
**Depends on**: Phase 1
**Requirements**: COMPANY-01, COMPANY-02
**Success Criteria** (what must be TRUE):
  1. `/des-interview` skill runs a structured interview covering services, pricing, ideal client, differentiators, geography, onboarding, and HIPAA approach
  2. `knowledge/company-profile.md` exists and contains all four Practice Growth Engine tiers with pricing, all à la carte services, the multilingual differentiator, and the target client profile
  3. Any DES team member can answer "what do we charge for X?" by reading `company-profile.md` alone
**Plans**: TBD

### Phase 3: Market Study
**Goal**: Document the SE Florida independent dental practice market — top problems, DSO landscape, direct competitor profiles — so audit forms and proposals are grounded in real market intelligence.
**Mode:** mvp
**Depends on**: Phase 2
**Requirements**: MARKET-01, MARKET-02
**Success Criteria** (what must be TRUE):
  1. `knowledge/market-study.md` documents the top 10 problems of SE Florida independent dental practices with supporting data
  2. Market study includes DSO competitor profiles (Aspen, Heartland, Dental Care Alliance, Great Expressions, Western, MB2) and direct service competitor profiles (Weave, RevenueWell, Adit, MaxAssist, Aron AI, DoctorLogic)
  3. Market study addresses SE Florida multilingual demographics (Spanish, Haitian Creole, Portuguese) and their implications for dental marketing
  4. Market study findings are validated against `DentalEdgeSolutions_Proposal.html` and any gaps or new market problems are documented
**Plans**: TBD

### Phase 4: Practice Audit System
**Goal**: Design and deploy the GHL Practice Audit form and build `/des-audit` so DES can generate a structured, data-enriched audit for any prospect from a single command.
**Mode:** mvp
**Depends on**: Phase 3
**Requirements**: AUDIT-01, AUDIT-02, AUDIT-03, AUDIT-04
**Success Criteria** (what must be TRUE):
  1. `audit/form-design.md` maps every audit question to a named GHL custom field key, derived from the top 10 market problems
  2. The Practice Audit form exists in GHL with custom fields matching `audit/form-design.md`
  3. `/des-audit [contact-id]` reads contact custom fields via `contacts_get-contact`, enriches with Data4SEO (local keyword ranks, nearest 3 competitors with ratings, GBP review count + rating), and produces `audit/submissions/[practice-name].md`
  4. The AuditResult includes: practice profile, top 3–5 prioritized problems, Data4SEO findings, and recommended DES services matched to each detected problem
**Plans**: TBD

### Phase 5: Benchmarking
**Goal**: Build a structured competitive comparison document covering DES vs. all 6 direct competitors so proposals and the service portfolio have accurate, current competitive positioning.
**Mode:** mvp
**Depends on**: Phase 3
**Requirements**: BENCH-01, BENCH-02
**Success Criteria** (what must be TRUE):
  1. `knowledge/benchmarking.md` documents DES vs. Weave, RevenueWell, Adit, MaxAssist, Aron AI, and DoctorLogic across: capabilities table, pricing comparison, SE Florida presence, done-for-you vs. DIY positioning, and multilingual capability
  2. Benchmarking data is verified against current competitor websites and pricing pages (not just training data)
  3. `benchmarking.md` is structured so `/des-proposal` can load it and generate the competitive comparison section dynamically
**Plans**: TBD

### Phase 6: Trends Engine
**Goal**: Build `/des-trends` to monitor SE Florida dental industry sources monthly and produce actionable content briefs that keep DES ahead of market shifts.
**Mode:** mvp
**Depends on**: Phase 3
**Requirements**: INTELL-01, INTELL-02, INTELL-03, INTELL-04
**Success Criteria** (what must be TRUE):
  1. `knowledge/intelligence/sources.md` documents all tracked sources: ADA News, Dental Economics, Dentistry Today, Dental Products Report, competitor blog/pricing pages, Reddit (r/dentistry, r/DentalHygiene, r/smallbusiness), LinkedIn public posts, YouTube dental channels
  2. `/des-trends` fetches and synthesizes content from tracked sources, updates `knowledge/intelligence/trends.md` with dated entries, and produces 3–5 content briefs in `knowledge/intelligence/briefs/`
  3. `/des-trends --refresh` forces a fresh fetch; without the flag the skill shows current `trends.md` without refetching
  4. An initial `trends.md` is populated and ready for the Proposal Engine to reference
**Plans**: TBD

### Phase 7: Service Portfolio Validation
**Goal**: Validate `services.md` against all three intelligence sources — market problems, competitive benchmarking, and current trends — to produce a fully cross-referenced, accurate service catalog before any proposal is generated.
**Mode:** mvp
**Depends on**: Phase 5, Phase 6
**Requirements**: PORTFOLIO-01, PORTFOLIO-02
**Success Criteria** (what must be TRUE):
  1. Every service in `knowledge/services.md` is annotated with: the market problem(s) it solves (from `market-study.md`), how it compares to what competitors offer (from `benchmarking.md`), and any trend context that makes it more or less relevant now (from `trends.md`)
  2. Services without a clear market problem match are flagged for review
  3. Any new service opportunities identified by cross-referencing market, benchmarking, and trends are documented in `knowledge/service-opportunities.md`
  4. `services.md` is finalized and ready to serve as the authoritative source for all proposal generation
**Plans**: TBD

---

### Phase 8: Proposal Engine
**Goal**: Build `/des-proposal` so DES can generate a complete, personalized, client-ready HTML proposal for any audited prospect in under 5 minutes — using the full validated knowledge base.
**Mode:** mvp
**Depends on**: Phase 4, Phase 7
**Requirements**: PROPOSAL-01, PROPOSAL-02, PROPOSAL-03, PROPOSAL-04, PROPOSAL-05
**Success Criteria** (what must be TRUE):
  1. `/des-proposal [contact-id]` runs `/des-audit` internally (or loads a fresh AuditResult), loads all knowledge base context (`company-profile.md`, `services.md`, `market-study.md`, `benchmarking.md`, `trends.md`), and generates a complete HTML proposal
  2. The generated proposal matches the quality and structure of `DentalEdgeSolutions_Proposal.html`: fully self-contained HTML, interactive tabs and CTAs, mobile-responsive, print/export button
  3. The proposal is personalized: uses the practice name throughout, cites their actual Data4SEO findings (keyword ranks, review count/rating, competitor names), and recommends specific services matched to their detected problems
  4. The competitive comparison section is generated dynamically from `benchmarking.md` (not a static table)
  5. The proposal saves to `proposals/[practice-name]-[date].html` and outputs the local file path
  6. On user approval, the skill writes proposal summary data to 10 GHL contact custom fields, moves the opportunity to "Proposal Sent" stage, and adds an activity note
**UI hint**: yes
**Plans**: TBD

### Phase 9: Content Pipeline
**Goal**: Build `/des-content` so DES can generate and queue SEO-ready blog posts for GHL from any trend brief or free-form topic in a single command.
**Mode:** mvp
**Depends on**: Phase 6, Phase 7
**Requirements**: CONTENT-01, CONTENT-02, CONTENT-03
**Success Criteria** (what must be TRUE):
  1. `/des-content [brief-id|topic]` accepts a brief ID or free-form topic, loads knowledge base context, and generates a complete blog post with SE Florida local SEO optimization and FAQ schema markup
  2. The skill shows the draft and requires user approval before publishing; on approval it publishes to GHL blog via `blogs_create-blog-post` with status DRAFT for final GHL review
  3. Each published post is logged in `content/published/` with title, date, GHL post ID, and source brief
**Plans**: TBD

### Phase 10: Integration and Polish
**Goal**: Verify all 5 skills work together end-to-end, document every skill's usage and edge cases, and confirm the full workflow from audit to proposal to content is repeatable without friction.
**Mode:** mvp
**Depends on**: Phase 8, Phase 9
**Requirements**: (integration — no new REQ-IDs; validates all prior phases)
**Success Criteria** (what must be TRUE):
  1. Running the full workflow — `/des-interview` → `/des-audit [id]` → `/des-proposal [id]` → `/des-trends` → `/des-content [brief]` — completes without errors using a real GHL contact
  2. Each skill has a `SKILL.md` documenting: purpose, usage, inputs, outputs, dependencies, and known edge cases
  3. The knowledge base (`company-profile.md`, `market-study.md`, `services.md`, `benchmarking.md`, `trends.md`) is current, internally consistent, and cross-referenced
  4. All architectural decisions from `research/SUMMARY.md` are reflected in the running skills (custom field read/write asymmetry, pre-fetched GHL IDs, sandbox vs. production Data4SEO toggle)
**Plans**: TBD

---

## Progress Table

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Foundation | 0/4 | In progress | - |
| 2. Company Study | 0/0 | Not started | - |
| 3. Market Study | 0/0 | Not started | - |
| 4. Practice Audit System | 0/0 | Not started | - |
| 5. Benchmarking | 0/0 | Not started | - |
| 6. Trends Engine | 0/0 | Not started | - |
| 7. Service Portfolio Validation | 0/0 | Not started | - |
| 8. Proposal Engine | 0/0 | Not started | - |
| 9. Content Pipeline | 0/0 | Not started | - |
| 10. Integration and Polish | 0/0 | Not started | - |
