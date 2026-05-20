# DES Intelligence Platform

## What This Is

A Claude Code-native intelligence platform for DentalEdge Solutions (DES) — a done-for-you dental marketing and automation agency serving independent dental practices in SE Florida (Miami-Dade, Broward, Palm Beach). The platform does two things: (1) maintains a living knowledge base of DES's business, market research, competitive intelligence, and industry trends; (2) runs a per-prospect operational workflow that generates personalized, data-backed HTML service proposals in minutes. Built with GSD phases, operated through custom Claude Code skills.

## Core Value

`/des-proposal [contact-id]` pulls a prospect's GHL audit data and Data4SEO web presence signals, cross-references the DES knowledge base, and generates a complete client-ready HTML proposal — in one command, in under 5 minutes.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] FOUNDATION-01: Project structure initialized with knowledge base directories and all integrations verified (GHL MCP, Data4SEO)
- [ ] FOUNDATION-02: `knowledge/services.md` seeded from existing proposal as single source of truth for DES service catalog
- [ ] COMPANY-01: Structured Company Study interview produces `knowledge/company-profile.md` — authoritative DES reference (services, pricing, positioning, differentiators, target client profile)
- [ ] MARKET-01: Market Study research produces `knowledge/market-study.md` — top problems of SE Florida independent dental practices, competitor profiles, local market dynamics
- [ ] AUDIT-01: Practice Audit form designed from Market Study findings and created in GHL
- [ ] AUDIT-02: `/des-audit [contact-id]` skill pulls GHL form data + enriches with Data4SEO (local SEO rankings, competitor comparison, GBP signals) → structured AuditResult
- [ ] PROPOSAL-01: `/des-proposal [contact-id]` skill generates complete, personalized HTML proposal matching quality of existing `DentalEdgeSolutions_Proposal.html`
- [ ] PROPOSAL-02: Generated proposals save locally to `proposals/[practice-name]-[date].html` and can be pushed to GHL contact record on demand
- [ ] BENCH-01: `knowledge/benchmarking.md` documents DES vs. Weave, RevenueWell, Adit, MaxAssist, Aron AI, DoctorLogic — proposals pull from this for competitive comparison sections
- [ ] PORTFOLIO-01: `knowledge/services.md` validated and updated against Market Study findings — services map to real market problems
- [ ] TRENDS-01: `/des-trends` skill monitors SE Florida dental industry publications, competitor sites, Reddit, LinkedIn, YouTube monthly → updates `knowledge/intelligence/trends.md`
- [ ] CONTENT-01: `/des-content [brief-id|topic]` skill generates SEO-ready blog posts and publishes to GHL blog via GHL MCP

### Out of Scope

- GA4 integration — replaced by Data4SEO for prospect web presence data; no GA4 access needed
- SW Florida market — DES operates exclusively in SE Florida (Miami-Dade, Broward, Palm Beach); source document had an error
- Automated proposal delivery — proposals are reviewed by DES before sending; no auto-send to prospects
- Facebook group monitoring — requires login/membership; not automatable. Manual intake only if needed
- Web application or dashboard UI — this is a Claude Code CLI project, not a browser app
- DSO clients — DES serves independent practices only

## Context

**Company:** DentalEdge Solutions (DES). Done-for-you dental marketing and automation. Primary product: the Practice Growth Engine — 4 tiered packages (Digital Foundation $297/mo, Efficiency Engine $697/mo, Patient Acquisition $1,497/mo, Market Leader $2,497/mo) plus 14+ à la carte services. All built on GoHighLevel.

**Market:** Independent dental practices and small groups (2–5 offices) in SE Florida. Key demographics: multilingual market (Spanish, Haitian Creole, Portuguese). Primary competitive threat: DSOs. Direct service competitors: Weave, RevenueWell, Adit, MaxAssist, Aron AI, DoctorLogic.

**Tech environment:**
- Claude Code with GHL MCP connected (contacts, opportunities, blog, forms)
- Data4SEO credentials in environment (`DATA4SEO_LOGIN`, `DATA4SEO_PASSWORD`)
- uv / Python 3.13 available (markitdown installed)
- context-mode MCP for knowledge base indexing
- claude-mem MCP for session memory

**Workflow integration:**
- Prospect fills Practice Audit form in GHL
- DES runs `/des-proposal [contact-id]`
- Claude pulls form data via GHL MCP + enriches with Data4SEO
- HTML proposal saved locally → reviewed by DES → pushed to GHL contact record
- Trends refresh: monthly automated + on-demand via `/des-trends`
- Content: trend-triggered briefs reviewed by DES → `/des-content [brief-id]` → published to GHL blog

**Existing artifacts:**
- `DentalEdgeSolutions_Proposal.html` — existing proposal (design and content reference)
- `CONTEXT.md` — domain glossary (authoritative terminology)
- `PROJECT.md` (root) — full project plan with phase breakdown and data flows

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| GHL MCP direct pull (not webhook) | No server infrastructure needed; DES is present when proposals are generated anyway | — Pending |
| Data4SEO over GA4 for prospect data | No access grants needed from prospect; pulls public data automatically | — Pending |
| Local HTML file first, then GHL push | Human review step before prospect sees proposal; catches errors and allows edits | — Pending |
| Market Study before audit form design | Form quality depends on knowing which problems to ask about | — Pending |
| services.md as single source of truth | One canonical catalog feeds proposals, content, and GHL; prevents drift | — Pending |
| Reddit + LinkedIn + YouTube for social signals (not Facebook) | Facebook requires private group membership; Reddit/LinkedIn/YouTube are fully automatable | — Pending |
| Proposal engine first (Milestone 1), intelligence layer second (Milestone 2) | Revenue-generating workflow live quickly; trends/content are valuable but not blocking | — Pending |
| GSD phases to build + custom skills as interface | Rigorous phased development; clean reusable commands as the delivered artifact | — Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd-transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd:complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-05-20 after initialization via grill-with-docs + gsd-new-project*
