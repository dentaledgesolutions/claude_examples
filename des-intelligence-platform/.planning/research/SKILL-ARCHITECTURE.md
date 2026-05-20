# Skill Architecture Research — DES Intelligence Platform

**Researched:** 2026-05-20  
**Source:** Existing skill examples in `~/.claude/skills/` (45+ skills), `~/.agents/skills/` (10+ skills), GSD workflow files, agent definitions in `~/.claude/agents/`  
**Confidence:** HIGH — all findings derived from working production skills in this environment

---

## 1. Skill File Format

### Location

Claude Code loads skills from two directories:

| Location | Scope | When to Use |
|----------|-------|-------------|
| `~/.claude/skills/<skill-name>/SKILL.md` | Global — available in every project | Skills used across multiple projects |
| `.claude/skills/<skill-name>/SKILL.md` | Project-local — only in this repo | DES-specific skills belong here |

**Decision for DES:** All five DES skills go in `.claude/skills/` inside the project repo. This keeps them versioned with the platform, shareable with any DES team member who clones the repo, and isolated from unrelated projects.

### Required Directory Structure

```
.claude/
└── skills/
    └── des-audit/
        ├── SKILL.md          ← required; entry point
        ├── ENRICHMENT.md     ← optional reference file (split when SKILL.md > ~100 lines)
        └── scripts/          ← optional utility scripts
            └── fetch-data4seo.py
```

### SKILL.md Frontmatter

Every SKILL.md begins with a YAML frontmatter block:

```yaml
---
name: des-audit
description: "Pull GHL contact + audit form data, enrich with Data4SEO, produce AuditResult markdown. Use when running /des-audit or preparing for proposal generation."
argument-hint: "<contact-id>"
allowed-tools:
  - Read
  - Write
  - Bash
  - mcp__ghl__*
---
```

**Field reference:**

| Field | Required | Purpose | Notes |
|-------|----------|---------|-------|
| `name` | Yes | Skill identifier | Must match directory name; used in `Skill()` calls |
| `description` | Yes | Trigger text — this is what Claude reads to decide whether to load the skill | Max ~1024 chars; must include "Use when [trigger]" |
| `argument-hint` | Recommended | Shows in `/help` output | Not enforced; documents expected argument shape |
| `allowed-tools` | Recommended | Restricts which tools the skill may use | Wildcards like `mcp__ghl__*` work; omit to allow all |

### How Arguments Are Passed

The string after the command name is injected verbatim as `$ARGUMENTS` anywhere it appears in the SKILL.md body:

```
/des-audit contact-abc123
```

Inside SKILL.md, reference it as:

```
Contact ID: $ARGUMENTS
```

For multi-argument parsing, use bash-style extraction in prose instructions:

```
Parse $ARGUMENTS:
- First token → contact ID
- If `--force` is present → bypass cache check
```

Claude parses this in context; there is no automatic shell parsing. The skill instructs Claude how to interpret the string.

---

## 2. Patterns for Reading Multiple Files Before Generating Output

This is the most critical pattern for `/des-proposal`, which must load 4-5 knowledge base files before generating anything.

### Pattern A: `<required_reading>` Block (Recommended for Subagents)

Used extensively in GSD workflows that spawn subagents. The spawning prompt includes a `<files_to_read>` or `<required_reading>` block that the subagent MUST process before acting:

```markdown
<required_reading>
Read ALL of the following before writing a single line of the proposal:

1. `knowledge/company-profile.md` — DES services, pricing, differentiators
2. `knowledge/services.md` — Package definitions and à la carte items
3. `knowledge/market-study.md` — SE Florida dental market context
4. `knowledge/benchmarking.md` — DES vs competitor comparison table
5. `knowledge/intelligence/trends.md` — Current market signals
6. `audit/submissions/$PRACTICE_SLUG.md` — This practice's AuditResult

Do not proceed to generation until all six files are in context.
</required_reading>
```

### Pattern B: `@` File Reference in `execution_context` (Recommended for Global Reference Files)

GSD skills use `<execution_context>` with `@path` syntax to inject workflow files directly into the skill's context at load time:

```markdown
<execution_context>
@$HOME/.claude/get-shit-done/workflows/some-workflow.md
@$HOME/.claude/get-shit-done/references/ui-brand.md
</execution_context>
```

This loads the referenced files as part of the SKILL.md content. Use this for:
- Workflow documents that define multi-step processes
- Reference files that are always needed (style guides, templates)

**Do NOT use** `@` references for per-run data files (audit results, contact data) — those change per invocation and must be loaded dynamically.

### Pattern C: Explicit Read Instructions in the Skill Body

For knowledge base files that are always required, list them as explicit Read instructions in the skill body. This is simpler than subagent delegation and appropriate when the skill runs inline:

```markdown
## Step 1 — Load context

Before any generation, Read these files in order:

1. Read `knowledge/company-profile.md`
2. Read `knowledge/services.md`  
3. Read `knowledge/market-study.md`
4. Read `knowledge/benchmarking.md`
5. Read `knowledge/intelligence/trends.md`

Then Read the AuditResult: `audit/submissions/[practice-slug-from-contact].md`
```

### Recommendation for DES Skills

Use **Pattern C** for `/des-proposal` (inline execution, explicit reads) combined with **Pattern A** (`<required_reading>`) if you later split into a subagent for generation. The explicit list makes the loading order unambiguous and tokens-aware (Claude loads them in order, not all at once).

---

## 3. Handling Optional Arguments and Flags

Claude Code does not parse flags automatically. The skill must instruct Claude how to interpret `$ARGUMENTS`.

### Flag Detection Pattern (from `gsd-phase` and `gsd-autonomous`)

```markdown
<context>
Arguments: $ARGUMENTS

Parse the first token of $ARGUMENTS:
- If it is `--refresh`: execute the full refresh workflow (re-fetch all sources)
- If it is absent or anything else: execute the on-demand synthesis workflow (use cached data if < 30 days old)
</context>
```

### For Boolean Flags (`--refresh`)

```markdown
Parse $ARGUMENTS:
- `--refresh` present → force re-fetch from all tracked sources, bypass cache
- `--refresh` absent → check `knowledge/intelligence/trends.md` modification date;
  if < 30 days old, synthesize from existing cache; if stale, run full refresh
```

### For Value Flags (`--brief-id`)

```markdown
Parse $ARGUMENTS:
- If token matches pattern `brief-[a-z0-9-]+` → treat as brief ID; 
  load `knowledge/intelligence/briefs/$ARGUMENTS.md`
- Otherwise → treat as free-form topic; proceed without loading a brief file
```

### Pattern: Explicit Flag Documentation in Context Block

All GSD skills that have optional flags include an explicit `<context>` block that lists every flag, what "active" means, and what the default behavior is when the flag is absent. This prevents Claude from inferring flags from documentation alone:

```markdown
<context>
**Available flags (documentation only — not automatically active):**
- `--refresh` — Force full source re-fetch. Active ONLY when literal `--refresh` appears in $ARGUMENTS.
- Without flags → on-demand synthesis from cached sources.
</context>
```

---

## 4. Subagents vs. Inline Execution — When to Choose Each

This is the highest-stakes architectural decision for DES skills.

### When to Use Inline Execution (No Subagent)

Run everything in the same Claude context when:
- The workflow is short enough to fit comfortably in one context window
- Interactive steps are required (AskUserQuestion, approval gates)
- The task requires tight feedback loops (show draft → user approves → publish)
- Sequencing is strict with dependencies between steps

**DES skills that should be inline:**
- `/des-interview` — pure interactive Q&A, must be inline
- `/des-audit` — 3-step pipeline (GHL fetch → Data4SEO → write file), short enough for inline
- `/des-content` — show draft → approve → publish loop requires inline approval

### When to Spawn Subagents

Spawn via `Agent(subagent_type="...", prompt="...")` when:
- A discrete, self-contained unit of work is large enough to fill a context window
- Work can be parallelized (multiple independent tasks)
- The orchestrator should stay lean while a worker does heavy lifting
- The subagent needs a fresh context (no contamination from orchestrator state)

**From `execute-phase.md`:** "Orchestrator stays lean: discover plans, analyze dependencies, group into waves, spawn subagents, collect results. Context budget: ~15% orchestrator, 100% fresh per subagent."

**DES skills where subagent spawning makes sense:**
- `/des-proposal` — the actual HTML generation is token-heavy; orchestrator can load audit result + trigger a generation subagent with pre-loaded context
- `/des-trends` — fetching 8-10 sources and synthesizing can be split: fetch subagent + synthesis subagent

### Subagent Invocation Syntax (Claude Code)

```
Agent(
  subagent_type="des-proposal-generator",
  prompt="""
  <required_reading>
  @path/to/audit-result.md
  @knowledge/company-profile.md
  </required_reading>
  
  Generate the HTML proposal for [practice name].
  [full instructions...]
  """
)
```

Subagent definitions live in `.claude/agents/<name>.md` with their own frontmatter (`tools:`, `color:`, etc.).

### Practical Rule for DES

Start inline for all five skills. The workloads are moderate. Move to subagent architecture only if a specific skill's generation step regularly hits context limits or slows down with large knowledge bases. `/des-proposal` is the most likely candidate for future subagent extraction.

---

## 5. Calling External APIs (Data4SEO, GHL MCP)

### GHL MCP Tools

GHL MCP tools are listed in `allowed-tools` using wildcard syntax:

```yaml
allowed-tools:
  - mcp__ghl__*
```

Or specific tools only:

```yaml
allowed-tools:
  - mcp__ghl__get_contact
  - mcp__ghl__get_form_submission
  - mcp__ghl__update_contact
  - mcp__ghl__create_blog_post
```

Within the skill body, reference MCP tools by their full namespaced name. Claude will call them directly as tool invocations — no Bash wrapper needed.

### Data4SEO via HTTP (Bash + curl/Python)

Data4SEO has no MCP server. API calls go through Bash. The credential pattern used in this project:

```bash
# Credentials from environment (set in shell profile or .env)
curl -u "$DATA4SEO_LOGIN:$DATA4SEO_PASSWORD" \
  -X POST https://api.data4seo.com/v3/serp/google/maps/live/regular \
  -H "Content-Type: application/json" \
  -d '{"data": [{"keyword": "dentist miami", "location_code": 1012694}]}'
```

**Pattern for API calls in skills:**

Document the expected API call shape in the skill body so Claude constructs it correctly. Include:
1. The endpoint
2. The credential variables
3. The expected request shape for the DES use case
4. How to parse the response for the fields DES needs

If the API call is complex or reused across skills, extract it to `scripts/fetch-data4seo.sh` or `scripts/enrich.py` and call it from the skill:

```bash
python3 .claude/skills/des-audit/scripts/enrich.py "$DOMAIN" "$CITY"
```

### Pattern: API Calls in Skill Body

For skills that call APIs, structure the skill to show Claude what to do:

```markdown
## Step 2 — Data Enrichment

Call Data4SEO for the practice domain. Use three endpoints:

### 2a — Local keyword rankings
POST https://api.data4seo.com/v3/serp/google/maps/live/regular
Auth: Basic $DATA4SEO_LOGIN:$DATA4SEO_PASSWORD
Body: {"data": [{"keyword": "dentist [city]", "location_code": [GBP location code]}]}
Extract: position, domain, title for top 10 results

### 2b — Competitor comparison  
[...]
```

---

## 6. Interactive Approval Steps (Show Draft → Approve → Publish)

This is the pattern for `/des-content` and the GHL push step of `/des-proposal`.

### The Pattern: Checkpoint Gates

From `gsd-verify-work` and `gsd-autonomous` — gate progress on explicit user confirmation:

```markdown
## Step 3 — Review gate

Display the generated proposal preview:
- Show file path: `proposals/[practice-name]-[date].html`
- Show the first 50 lines of the HTML (so DES can spot-check)
- Show data summary: which Data4SEO findings were included, which knowledge base sections used

Ask: "Ready to push to GHL contact record? (yes/no)"

- If yes → proceed to Step 4 (GHL push)
- If no → stop; inform user the proposal is saved locally at the path above
```

### Using AskUserQuestion

Add `AskUserQuestion` to `allowed-tools` for any skill that has approval steps. This tool surfaces a structured question to the user and blocks until answered:

```yaml
allowed-tools:
  - Read
  - Write
  - Bash
  - AskUserQuestion
  - mcp__ghl__*
```

In the skill body:

```markdown
Use AskUserQuestion to present:
- Question: "Proposal saved to [path]. Push to GHL contact [contact-id] now?"
- Options: ["Push to GHL", "Save locally only", "Regenerate with changes"]
```

### Multi-Stage Approval Pattern

For `/des-content` (generate → review → publish):

```markdown
## Stage 1 — Generate

[generation instructions]

Save draft to `content/drafts/[slug]-draft.md`.

## Stage 2 — Review gate

Display the draft.
Use AskUserQuestion: "Approve for GHL publication, request revisions, or discard?"
- Approve → proceed to Stage 3
- Revisions → ask what to change, regenerate, return to Stage 2
- Discard → delete draft, stop

## Stage 3 — Publish

Call mcp__ghl__create_blog_post with the approved content.
Save published record to `content/published/[slug].md` with GHL post ID.
```

---

## 7. Skill File Patterns — Complete Template for DES

### SKILL.md Template

```markdown
---
name: des-audit
description: "Pull GHL contact and audit form data, enrich with Data4SEO local SEO signals, and write a structured AuditResult markdown file. Use when running /des-audit, preparing for proposal generation, or when contact-id is provided for prospect analysis."
argument-hint: "<contact-id>"
allowed-tools:
  - Read
  - Write
  - Bash
  - mcp__ghl__get_contact
  - mcp__ghl__get_form_submission
---

<objective>
[What this skill produces and why]
</objective>

<context>
Contact ID: $ARGUMENTS

[Flag parsing instructions if any]
</context>

<process>
[Step-by-step numbered process]
</process>
```

### Agent Definition Template (if subagent needed)

```markdown
---
name: des-proposal-generator
description: Generates personalized HTML proposals for dental practices. Spawned by /des-proposal skill.
tools: Read, Write, Bash
color: purple
---

<role>
[Role description]
</role>
```

Agent files live in `.claude/agents/des-proposal-generator.md`.

---

## 8. Recommended Skill Designs for Each DES Command

### `/des-audit [contact-id]`

**Mode:** Inline (no subagent)  
**Tools:** `mcp__ghl__*`, `Bash` (Data4SEO curl), `Write`, `Read`  
**Steps:**
1. Parse `$ARGUMENTS` → contact ID
2. Call GHL MCP: fetch contact record + audit form submission
3. Extract practice domain from contact record
4. Call Data4SEO: 3 endpoints (local rankings, competitors, GBP signals) via Bash
5. Synthesize findings into AuditResult structure
6. Write `audit/submissions/[practice-slug].md`
7. Confirm path to user

**No approval gate** — AuditResult is an internal artifact, not user-facing.

---

### `/des-proposal [contact-id]`

**Mode:** Inline orchestrator, optional subagent for generation  
**Tools:** `Read`, `Write`, `Bash`, `mcp__ghl__*`, `AskUserQuestion`  
**Steps:**
1. Parse `$ARGUMENTS` → contact ID
2. Check if `audit/submissions/[slug].md` exists and is < 24h old; if not, run audit inline
3. Read all knowledge base files (required reading block — see Pattern C above)
4. Generate HTML proposal (inline or via generation subagent)
5. Save to `proposals/[practice-name]-[date].html`
6. Show preview + approval gate via AskUserQuestion
7. On approval: push to GHL via `mcp__ghl__update_contact`

**Required reading block** is the key complexity here — must load 5-6 files before generation.

---

### `/des-trends [--refresh]`

**Mode:** Inline orchestrator  
**Tools:** `Read`, `Write`, `Bash`, `WebFetch`, `WebSearch`  
**Flag handling:**
```
Parse $ARGUMENTS:
- `--refresh` present → force full fetch from all sources in knowledge/intelligence/sources.md
- absent → check trends.md modification date; if > 30 days, auto-refresh; else synthesize only
```
**Steps:**
1. Parse flag
2. Read `knowledge/intelligence/sources.md` for tracked URL list
3. Fetch each source (WebFetch for web pages, Bash for Reddit API, WebSearch for LinkedIn/YouTube)
4. Synthesize: identify themes, emerging signals, SE Florida-specific items
5. Update `knowledge/intelligence/trends.md` with dated entries (append, don't overwrite)
6. Generate 3-5 content briefs → `knowledge/intelligence/briefs/[slug]-[date].md`
7. Report: sources fetched, themes identified, briefs created

**Note:** This is the most token-intensive skill. If context pressure becomes an issue, split into: (a) fetch subagent that fetches + summarizes each source individually, (b) synthesis inline in orchestrator from summaries.

---

### `/des-content [brief-id|topic]`

**Mode:** Inline with approval gate  
**Tools:** `Read`, `Write`, `mcp__ghl__*`, `AskUserQuestion`  
**Argument parsing:**
```
Parse $ARGUMENTS:
- Matches `brief-[a-z0-9-]+` → load brief file from knowledge/intelligence/briefs/
- Otherwise → treat as free-form topic, no brief file loaded
```
**Required reading:** `knowledge/company-profile.md`, `knowledge/services.md`, `knowledge/market-study.md`, `knowledge/intelligence/trends.md` (always), plus brief file if provided.

**Steps:**
1. Parse argument → brief ID or topic
2. Load required knowledge base files
3. Load brief if brief-id provided
4. Generate blog post (SE Florida SEO optimized, DES-positioned)
5. Save draft to `content/drafts/[slug].md`
6. Show draft + AskUserQuestion approval gate
7. On approval: call `mcp__ghl__create_blog_post`
8. Save published record to `content/published/[slug].md`

---

### `/des-interview`

**Mode:** Fully inline, conversational  
**Tools:** `Read`, `Write`, `AskUserQuestion`  
**No `$ARGUMENTS`** — interactive only

**Pattern:** This skill is the simplest in structure but the most interactive. Use AskUserQuestion for each interview question. The skill body should define the interview structure (sections, questions) and write `knowledge/company-profile.md` progressively as answers accumulate.

```markdown
## Interview structure

Conduct the interview section by section. After each section, summarize what was captured 
and confirm before moving to the next.

Sections:
1. Services and scope
2. Pricing (packages + à la carte)
3. Target client profile
4. Differentiators
5. Problem focus
6. Geographic scope
7. Onboarding process
8. HIPAA approach
9. Current clients + testimonials

After all sections: write knowledge/company-profile.md in full.
Show the written document and ask for any corrections.
```

---

## 9. Critical Pitfalls

### Pitfall 1: Loading All Knowledge Base Files Every Time

**Problem:** If `/des-proposal` reads all 5 knowledge base files plus the audit result on every run, that's potentially 50-100k tokens of context loaded before generation even begins. This approaches context limits.

**Prevention:** 
- Keep knowledge base files focused and lean (< 2000 lines each)
- Use context-mode MCP (`ctx_batch_execute`, `ctx_search`) to index and search knowledge base rather than loading full files — load only relevant sections
- The existing `settings.local.json` already allows context-mode MCP tools

### Pitfall 2: `$ARGUMENTS` Parsed Wrong on Edge Cases

**Problem:** If the user passes `/des-content brief-miami-seo-2026` and the skill checks for `brief-` prefix, it works. If they pass `/des-content "miami seo strategy"` the quotes may be stripped or preserved inconsistently.

**Prevention:** Document exact expected argument format in `argument-hint`. Add explicit handling for quoted strings in the skill's argument parsing instructions.

### Pitfall 3: Approval Gate Bypassed in Non-Interactive Mode

**Problem:** If Claude runs in a context without AskUserQuestion (e.g., scheduled execution of `/des-trends`), an AskUserQuestion call will fail or stall.

**Prevention:** For skills that may run unattended, add a `--auto` or `--no-confirm` flag that bypasses approval gates. The `/des-trends` skill in particular should skip approval gates when run scheduled.

### Pitfall 4: GHL MCP Tool Names Unknown at Skill Write Time

**Problem:** The exact GHL MCP tool names (`mcp__ghl__get_contact` etc.) must match what the GHL MCP server actually exposes. Using wrong tool names silently fails.

**Prevention:** Run `mcp__ghl__` in an interactive session first to discover available tools. Document the exact tool names in each skill. Keep a `TOOLS.md` reference in `.claude/skills/` listing verified GHL MCP tool names.

### Pitfall 5: Skill Description Too Generic

**Problem:** Claude loads skills based on description matching. If the description says "Runs dental practice audit", Claude may not load it when the user types `/des-audit abc123` if the command invocation doesn't match natural language triggers.

**Note:** For explicit slash command invocations (`/des-audit`), Claude Code loads the matching skill by name directly — the description is used for ambient/natural language triggering. Both paths matter.

**Prevention:** Keep the name exact, make the description descriptive for natural language use.

### Pitfall 6: Subagent Context Isolation Terminates Background Bash

**Problem:** Documented in `gsd-graphify`: "Sub-agent isolation terminates background bash when the agent exits." If a subagent spawns a long-running Bash process, the process dies when the agent exits.

**Prevention:** Run critical Bash operations (Data4SEO fetches, file writes) in foreground within the agent, not as background processes. Never pass `run_in_background: true` for operations that must complete before the skill finishes.

---

## 10. File Naming Convention for DES Output Artifacts

Based on the project structure established in `PROJECT.md`:

| Output | Path Pattern | Example |
|--------|-------------|---------|
| AuditResult | `audit/submissions/[practice-slug].md` | `audit/submissions/south-miami-dental.md` |
| Proposal | `proposals/[practice-slug]-[YYYY-MM-DD].html` | `proposals/south-miami-dental-2026-05-20.html` |
| Content Draft | `content/drafts/[slug].md` | `content/drafts/ai-in-dentistry-2026.md` |
| Published Record | `content/published/[slug].md` | `content/published/ai-in-dentistry-2026.md` |
| Content Brief | `knowledge/intelligence/briefs/[slug]-[YYYY-MM-DD].md` | `knowledge/intelligence/briefs/invisalign-trends-2026-05-20.md` |
| Trends Document | `knowledge/intelligence/trends.md` | (single file, append-only) |

Use kebab-case for all slugs derived from practice names or content titles.
