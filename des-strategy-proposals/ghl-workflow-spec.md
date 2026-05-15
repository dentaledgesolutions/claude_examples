# GHL Workflow Specification — Personalized Proposal Generator
**Version:** 3.0 (Claude Code + GHL MCP — no middleware)
**Trigger:** Stage 1 intake form submission  
**Output:** Draft proposal in GHL Documents & Contracts + pipeline opportunity + Erick notification  

---

## Architecture

GHL handles form intake and notifications. Claude Code (connected to GHL via MCP) handles everything else — reading the contact, calling Data4SEO, generating the proposal, and writing it back to GHL.

```
Prospect fills Stage 1 form
         ↓
GHL creates contact + opportunity (New Lead stage)
GHL sends notification to Erick (email + in-app)
         ↓
Erick opens Claude Code → runs proposal generation command
         ↓
Claude Code reads contact from GHL via MCP
Claude Code calls Data4SEO API (Google rating, reviews, local rank, top competitor)
Claude Code generates personalized proposal (using competitive-intelligence.md)
Claude Code creates document draft in GHL Documents & Contracts via MCP
Claude Code moves opportunity to "Proposal Drafted"
Claude Code creates review task assigned to Erick
         ↓
Erick reviews draft in GHL → sends
```

No HTTP requests from GHL. No middleware. No custom server.

---

## Prerequisites

- [ ] Documents & Contracts accessible — navigate to **Payments → Documents & Contracts** in GHL left menu (no separate activation required)
- [ ] Stage 1 form live with all 10 fields
- [ ] GHL MCP configured in Claude Code (PIT token + Location ID — see setup below)
- [ ] Data4SEO credentials available as environment variables
- [ ] GHL pipeline created: Proposal Pipeline — stages: New Lead → Proposal Drafted → Proposal Sent → Won → Lost
- [ ] GHL custom fields created (list at bottom of this document)
- [ ] `competitive-intelligence.md` and `claude-system-prompt.md` in project directory

---

## Part 1 — GHL Workflow (Simple)

This workflow has one job: capture the form, create the records, and notify Erick.

### TRIGGER — Form Submitted
- Trigger type: Form Submitted
- Select: Stage 1 intake form

### STEP 1 — Create Contact

Map all form fields to GHL contact + custom fields:

| Form Field | GHL Field |
|---|---|
| First Name | contact.first_name |
| Last Name | contact.last_name |
| Email | contact.email |
| Phone | contact.phone |
| Dental Office Name | contact.company_name |
| Role | custom: practice_role |
| Website URL | custom: practice_website |
| City / Area | custom: practice_city |
| County | custom: practice_county |
| Specialty | custom: practice_specialty |
| Biggest Challenge | custom: biggest_challenge |
| Current Tools | custom: current_tools |
| Languages | custom: practice_languages |

### STEP 2 — Create Opportunity

- Pipeline: Proposal Pipeline
- Stage: New Lead
- Name: `{{contact.company_name}} — Practice Audit`
- Assigned to: Erick Sicard

### STEP 3 — Notify Erick

**Internal email:**
- To: erick.sicard@dentaledgesolutions.com
- Subject: `New Practice Audit Request — {{contact.company_name}}`
- Body:
```
New lead ready for proposal generation.

Practice: {{contact.company_name}}
Contact: {{contact.first_name}} {{contact.last_name}} ({{custom_field.practice_role}})
Location: {{custom_field.practice_city}}, {{custom_field.practice_county}}
Specialty: {{custom_field.practice_specialty}}
Challenge: {{custom_field.biggest_challenge}}
Current Tools: {{custom_field.current_tools}}
Website: {{custom_field.practice_website}}

Open Claude Code and run: generate proposal for {{contact.company_name}}
```

**In-app notification:** New lead — `{{contact.company_name}}` — generate proposal in Claude Code

That's the entire GHL workflow. Three steps.

---

## Part 2 — Claude Code Runbook

What Erick does after receiving the notification.

### Setup (one-time)

**1. Configure GHL MCP in Claude Code**

GHL's MCP server uses direct HTTP — no npm package or SDK required.

In GHL: Settings → Private Integrations → Create New Integration

Enable these scopes:
- View/Edit Contacts
- View/Edit Opportunities
- View/Edit Conversations + Conversation Messages
- View Calendars & Calendar Events
- View Locations
- View Custom Fields
- View Forms

Copy your PIT token (`pit_...`) and your Location ID (Settings → Business Profile → bottom of page).

Add to `~/.claude/settings.json`:
```json
{
  "mcpServers": {
    "prod-ghl-mcp": {
      "url": "https://services.leadconnectorhq.com/mcp/",
      "headers": {
        "Authorization": "Bearer YOUR_PIT_TOKEN_HERE",
        "locationId": "YOUR_LOCATION_ID_HERE"
      }
    }
  }
}
```

No `npx`. No environment variables. Just the URL and headers.

**2. Set Data4SEO credentials as environment variables**

Add to your shell profile (`~/.zshrc`):
```bash
export DATA4SEO_LOGIN="your_data4seo_email"
export DATA4SEO_PASSWORD="your_data4seo_password"
```

**3. Confirm project directory is set**

Claude Code should be opened from `/Users/erick/claude-examples/des-strategy-proposals/` — this gives it access to `competitive-intelligence.md` and `claude-system-prompt.md`.

---

### Generating a Proposal (per lead)

When you receive the notification email, open Claude Code and type:

```
Generate a proposal for [Practice Name] — they just submitted the practice audit form.
```

Claude Code will then:

1. **Read the contact from GHL** via MCP — pulls all custom fields (city, specialty, challenge, current tools, website, languages)

2. **Call Data4SEO** for two lookups:
   - Google Business Profile → rating, review count
   - Local pack search → local ranking position, top competitor name + rating + review count

3. **Read** `competitive-intelligence.md` for competitor comparison language

4. **Generate** a personalized proposal following the structure in `claude-system-prompt.md`

5. **Create the document draft** in GHL Documents & Contracts via MCP — all sections populated (saved as Draft, not sent)

6. **Move the opportunity** from New Lead → Proposal Drafted

7. **Create a task** in GHL: "Review and send proposal — [Practice Name]" due in 4 hours

8. **Report back** to you with a summary:
   - Practice's Google rating and review count found
   - Top local competitor identified
   - Which competitor comparison was triggered (based on current tools)
   - Document draft created — navigate to **Payments → Documents & Contracts** in GHL to review

**Total time from typing the command to proposal ready:** ~60–90 seconds

---

### Reviewing and Sending

1. Click the GHL link Claude Code provides (or navigate to **Payments → Documents & Contracts** in GHL)
2. Review the draft — check that:
   - The practice name and doctor name are correct throughout
   - The Google data matches what you know about this practice
   - The competitor comparison matches their actual tools
   - The ROI numbers are reasonable for their size
3. Make any edits directly in the Documents & Contracts editor
4. Send when satisfied — choose **Send via Email** or **Share via Link**
   - Note: contact must have a valid email address on file for email delivery
5. Move opportunity to Proposal Sent

---

### If Data4SEO Returns No Data

Some practices have inconsistent Google Business Profile names or low online presence. If Data4SEO can't find the practice, Claude Code will:
- Tell you what it searched for and what it found (or didn't find)
- Generate the proposal using industry benchmark language instead of real numbers
- Flag which sections used benchmarks vs. real data

You can then manually look up their Google rating and ask Claude Code to regenerate the affected sections with the correct numbers before sending.

---

## Part 3 — GHL Custom Fields to Create

Settings → Custom Fields → Contacts:

| Field Label | Field Key | Type |
|---|---|---|
| Practice Role | practice_role | Dropdown |
| Practice Website | practice_website | Text |
| Practice City | practice_city | Text |
| Practice County | practice_county | Dropdown |
| Practice Specialty | practice_specialty | Dropdown |
| Biggest Challenge | biggest_challenge | Dropdown |
| Current Tools | current_tools | Text |
| Practice Languages | practice_languages | Text |

---

## Testing Before Going Live

Run one end-to-end test with fake data:

1. Submit the Stage 1 form with Test 1 data:
   - Practice: Coral Way Dental
   - Contact: Carlos Mendez
   - City: Coral Way, Miami-Dade
   - Specialty: General
   - Challenge: New patients
   - Tools: Weave
   - Website: coralwaydental.com (fake — Data4SEO may return null, which is fine for testing)

2. Confirm GHL created the contact, opportunity, and sent the notification email

3. Open Claude Code and type: `Generate a proposal for Coral Way Dental — they just submitted the practice audit form.`

4. Watch Claude Code execute each step and report back

5. Open GHL → **Payments → Documents & Contracts** → find the draft — verify all sections are populated

6. Compare to the manual Test 1 output you already approved

If output matches quality of the manual test: system is ready. Publish the Stage 1 form.
