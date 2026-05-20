# STATE — DES Intelligence Platform

*Last updated: 2026-05-20*

---

## Project Reference

**Core value**: `/des-proposal [contact-id]` generates a complete, personalized, client-ready HTML proposal in under 5 minutes — pulling GHL audit data and Data4SEO signals, cross-referenced against the DES knowledge base.

**Current milestone**: Milestone 1 — Proposal Engine (Phases 1–5)

**Current focus**: Phase 1 — Foundation

---

## Current Position

| Field | Value |
|-------|-------|
| Milestone | 1 — Proposal Engine |
| Current phase | 1 — Foundation |
| Current plan | Not started |
| Phase status | Not started |
| Overall progress | 0/10 phases complete |

```
Progress: [          ] 0%
Milestone 1: [          ] 0/5 phases
Milestone 2: [          ] 0/5 phases
```

---

## Performance Metrics

| Metric | Value |
|--------|-------|
| Phases complete | 0/10 |
| Requirements complete | 0/27 |
| Skills built | 0/5 |
| Knowledge base files seeded | 0/6 |

---

## Accumulated Context

### Key Decisions (from research)

- GHL form data is written to contact custom fields — `/des-audit` reads via `contacts_get-contact`
- HTML files cannot be attached to GHL contacts — use custom fields + opportunity stage instead
- Blog publishing requires pre-fetched `blogId`, `authorId`, `categoryId` — stored in `knowledge/ghl-config.json`
- GHL custom field read uses `{ id, value }`; write uses `{ id, field_value }` — asymmetry must be handled in skills
- All 5 skills start as inline Claude Code skills; only extract to subagent if `/des-proposal` hits context limits
- Data4SEO sandbox (`sandbox.dataforseo.com`) used during development; production toggle needed before launch
- GBP posting activity is out of scope — not available via any API; use review count + rating as GBP signal

### SE Florida Multilingual Context

Miami-Dade: 68% Spanish-speaking. Broward: largest Haitian-American community in US. Doral/Aventura: largest Brazilian expat community in US. This is DES's most defensible differentiator — no competitor offers managed multilingual campaigns.

### Active Blockers

None.

### Open TODOs

- [ ] Verify GHL MCP tool names from a live session before writing any skill
- [ ] Confirm Data4SEO sandbox credentials are set in environment (`DATA4SEO_LOGIN`, `DATA4SEO_PASSWORD`)

---

## Session Continuity

**To resume:** Load this file + `ROADMAP.md` + the current phase's `PLAN.md` (once created). Check `REQUIREMENTS.md` traceability for current status.

**Skills operated:**
- `/des-interview` — Company Study interview
- `/des-audit [contact-id]` — Practice audit with Data4SEO enrichment
- `/des-proposal [contact-id]` — Personalized HTML proposal generation
- `/des-trends [--refresh]` — Intelligence monitoring and brief generation
- `/des-content [brief-id|topic]` — Blog post generation and GHL publishing

**Knowledge base files (to be created):**
- `knowledge/services.md` — DES service catalog (single source of truth)
- `knowledge/company-profile.md` — Authoritative DES reference
- `knowledge/market-study.md` — SE Florida market intelligence
- `knowledge/benchmarking.md` — DES vs. competitors
- `knowledge/intelligence/trends.md` — Running trends log
- `knowledge/ghl-config.json` — GHL blog IDs
