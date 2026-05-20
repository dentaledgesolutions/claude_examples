# Research Summary — DES Intelligence Platform

*Synthesized from 4 research agents: DATA4SEO.md, GHL-MCP.md, SKILL-ARCHITECTURE.md, MARKET.md*

---

## Critical Findings That Change the Plan

### 1. GBP Posting Activity Is Not Available
Data4SEO (or any third-party API) cannot provide how often a practice posts to Google Business Profile. GBP post activity requires OAuth from the business owner. **Remove GBP posting activity from Data4SEO enrichment scope.** Keep: local rankings, competitor comparison, review count/rating.

### 2. GHL Form Data → Use Contact Custom Fields
There is no clean "get form submission by contact ID" endpoint. The correct pattern is: GHL form writes answers to contact custom fields → `/des-audit` reads contact via `contacts_get-contact` → maps field IDs using `locations_get-custom-fields`. This means the Practice Audit form must be designed to map every question to a named GHL custom field.

### 3. GHL Proposal Delivery → Custom Fields + Opportunity Stage (Not File Attachment)
HTML files cannot be attached to GHL contacts. The correct architecture (already used in prior DES work): write proposal data to 10 custom contact fields, move opportunity stage, add note. GHL Documents & Contracts template with merge fields handles the formatted output in GHL.

### 4. Blog Publishing Requires Pre-fetched IDs
Creating a GHL blog post requires `blogId`, `authorId`, and `categoryId` — none are auto-resolved. Must call `blogs/site/all`, `blogs/authors`, `blogs/categories` once at setup and cache. This is a one-time setup step, not per-post overhead.

### 5. Custom Field Read/Write Asymmetry in GHL
Contact read response uses `{ id, value }`. Contact update payload uses `{ id, field_value }`. Easy to get wrong. Must be documented in the `/des-audit` and `/des-proposal` skills.

### 6. All Skills Start Inline (Not Subagents)
All 5 custom skills (`/des-audit`, `/des-proposal`, `/des-trends`, `/des-content`, `/des-interview`) should be inline Claude Code skills. Only extract to subagent if `/des-proposal` hits context limits from loading the full knowledge base.

### 7. SE Florida Multilingual Is DES's Most Defensible Differentiator
No competitor offers Spanish/Haitian Creole/Portuguese campaign management. Miami-Dade 68% Spanish-speaking; Broward largest Haitian-American community in US; Doral/Aventura largest Brazilian expat community in US. Proposals for multilingual markets should always surface this.

---

## What Is Confirmed Available

| Capability | Tool/Endpoint | Notes |
|-----------|---------------|-------|
| Local keyword rankings for a domain | Data4SEO Maps SERP | $0.001-0.002/call |
| Competitor list for a city | Data4SEO Maps SERP | Same call returns full competitor set |
| GBP review count + rating | Data4SEO Business Data | $0.00375/call |
| Read contact + custom fields | `contacts_get-contact` | One call, all audit data |
| Write proposal data to contact | `contacts_update-contact` | Custom fields via `field_value` |
| Move opportunity stage | `opportunities_update-opportunity` | Pipeline automation |
| Publish blog post to GHL | `blogs_create-blog-post` | Requires pre-fetched IDs |
| Custom field key mapping | `locations_get-custom-fields` | One-time at setup |

---

## SE Florida Market Intelligence

**Top practice problems (for audit form design):**
1. Revenue leakage from no-shows/cancellations (12–18% no-show rate in SE FL vs 5–15% national)
2. Inconsistent new patient flow (expensive Google Ads market; avg CPC $12–20 for "dentist miami")
3. Front desk inefficiency and after-hours missed calls
4. No-show/cancellation recovery — manual processes, no automation
5. Weak online reputation (Google reviews not systematically generated)
6. Poor local SEO visibility (most practices don't rank for their neighborhood keywords)
7. Language barrier with patients (Spanish/Haitian Creole/Portuguese)
8. DSO competition stealing market share (30–38% DSO penetration in SE FL)
9. Treatment plan backlog — diagnosed patients who never scheduled
10. Outdated or no website

**Competitor positioning summary:**
- Weave, RevenueWell, Adit, MaxAssist: DIY tools, $200–700/mo, no managed service, no multilingual
- DoctorLogic: Best-in-class website + SEO + ads ($800–1,500/mo), managed but no CRM/automation/AI
- DES unique position: Only fully managed, multilingual, done-for-you, GHL-native stack in SE FL

**Key DSOs in SE FL:** Aspen Dental, Heartland Dental, Dental Care Alliance, Great Expressions, Western Dental, MB2 Dental

---

## Technology Risks and Mitigations

| Risk | Mitigation |
|------|-----------|
| Wrong GHL MCP tool names fail silently | Verify all tool names from live session before writing any skill |
| GBP posting activity unavailable | Remove from scope; use review count/rating as GBP signal instead |
| Context limit on /des-proposal (large knowledge base) | Keep knowledge base files lean; use context-mode search to load relevant sections only |
| Data4SEO sandbox vs production | Always use sandbox (`sandbox.dataforseo.com`) during dev |
| GHL custom field ID mapping | Cache field map at startup via `locations_get-custom-fields` |
