# GHL MCP Capabilities Research

**Project:** DES Intelligence Platform
**Researched:** 2026-05-20
**Sources:** Official GHL API docs (GoHighLevel/highlevel-api-docs), open-ghl-mcp, tenfoldmarc/ghl-mcp, BusyBee3333/Go-High-Level-MCP-2026-Complete
**Confidence:** HIGH — verified against official OpenAPI spec

---

## MCP Connection Pattern

The GHL MCP server runs at `https://services.leadconnectorhq.com/mcp/` and is configured directly in `~/.claude/settings.json` (global) or a project-level `.mcp.json`. No npm package required — just the URL and auth headers:

```json
{
  "mcpServers": {
    "prod-ghl-mcp": {
      "url": "https://services.leadconnectorhq.com/mcp/",
      "headers": {
        "Authorization": "Bearer YOUR_PIT_TOKEN",
        "locationId": "YOUR_LOCATION_ID"
      }
    }
  }
}
```

API version header required on all requests: `Version: 2021-07-28`

---

## 1. Form Submissions

### Capability: YES — with a critical limitation

The GHL API exposes form submissions at `GET /forms/submissions`. The tool exposed by the MCP is `get_form_submissions` (or `get_all_form_submissions` in some implementations).

**Important:** There is NO `contactId` query parameter in the official API spec. The official spec shows only `q` (free-text search) for contact-based filtering. The `q` parameter description says: "Filter by contactId, name, email or phone no."

This means to pull form data for a specific contact, you pass their contact ID as the `q` search param:

```
GET /forms/submissions?locationId=LOC_ID&q=CONTACT_ID&formId=FORM_ID
```

Some community MCP implementations expose a `contactId` parameter that maps to `q` internally. Others expose `q` directly. Either works — pass the contact ID as the value.

### Official API Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `locationId` | Yes | Your sub-account location ID |
| `page` | No | Page number (default: 1) |
| `limit` | No | Records per page (max 100, default 20) |
| `formId` | No | Filter by specific form ID |
| `q` | No | Filter by contactId, name, email, or phone |
| `startAt` | No | Start date filter (YYYY-MM-DD) |
| `endAt` | No | End date filter (YYYY-MM-DD) |

**Note:** `GET /forms/{id}` returns 401 "Route not supported by IAM Service." `GET /forms/{id}/submissions` returns 404. Only the `/forms/submissions` bulk endpoint works.

### Response Shape

```json
{
  "submissions": [
    {
      "id": "38303ec7-629a-49e2-888a-cf8bf0b1f97e",
      "contactId": "DWQ45t2IPVxi9LDu1wBl",
      "createdAt": "2021-06-23T06:07:04.000Z",
      "formId": "YSWdvS4Is98wtIDGnpmI",
      "name": "test",
      "email": "test@test.com",
      "others": {
        "__submissions_other_field__": "john@deo.com",
        "__custom_field_id__": "20",
        "eventData": { ... },
        "fieldsOriSequance": ["full_name", "first_name", "last_name", "phone", "email"]
      }
    }
  ],
  "meta": {
    "total": 1,
    "currentPage": 1,
    "nextPage": null,
    "prevPage": null
  }
}
```

**Key insight:** Custom form fields land in `others` under their field ID as key (not the field label). Standard GHL fields (`name`, `email`, `phone`, `first_name`, `last_name`) appear as top-level keys. The `fieldsOriSequance` array gives the field order. For the DES intake form, the practice-specific custom fields will be under `others.__FIELD_ID__`.

### DES Workflow Implication

The safest approach is to use `contacts_get-contact` (which returns the full contact record including all custom fields already mapped) rather than parsing form submission `others`. Custom fields are written to the contact record when GHL processes the form submission. The contact record is the clean source of truth.

---

## 2. Contact Data (`contacts_get-contact`)

### Capability: Full — HIGH confidence

`GET /contacts/{contactId}` returns the complete contact record.

### Full Field List (from official spec `GetContectByIdSchema`)

```
id, name, locationId, firstName, lastName, email, emailLowerCase,
timezone, companyName, phone, dnd, dndSettings, type, source,
assignedTo, address1, city, state, country, postalCode, website,
tags, dateOfBirth, dateAdded, dateUpdated, attachments, ssn,
keyword, firstNameLowerCase, fullNameLowerCase, lastNameLowerCase,
lastActivity, customFields, businessId, attributionSource,
lastAttributionSource, visitorId
```

### Custom Fields in Contact Response

Custom fields are returned as an array on the contact:

```json
{
  "contact": {
    "id": "CONTACT_ID",
    "firstName": "Carlos",
    "lastName": "Mendez",
    "email": "carlos@coralwaydental.com",
    "companyName": "Coral Way Dental",
    ...
    "customFields": [
      { "id": "FIELD_ID", "value": "General Dentistry" },
      { "id": "FIELD_ID_2", "value": "New patients" },
      { "id": "FIELD_ID_3", "value": "Weave" }
    ]
  }
}
```

The `id` in each custom field object is the GHL custom field definition ID (not the key). The `value` is the submitted value.

To map custom field IDs to human-readable keys, call `locations_get-custom-fields` (`GET /locations/{locationId}/customFields`) which returns:

```json
{
  "fields": [
    {
      "id": "FIELD_ID",
      "name": "Practice Specialty",
      "key": "practice_specialty",
      "dataType": "DROPDOWN",
      "locationId": "LOC_ID",
      "options": [...]
    }
  ]
}
```

**Best practice for DES workflow:** Call `get_custom_fields` once on setup to build a field ID → key mapping. Cache it. Then when reading a contact, look up each custom field's `id` against the mapping to get the human-readable key.

Alternatively — and simpler — use `contacts_update-contact` with the field key directly when writing. When reading, parse by field ID using a cached map. The existing `ghl-workflow-spec.md` uses field keys like `practice_specialty`, `biggest_challenge`, etc. — these are the `key` values from the custom field definitions.

### Opportunity Data (`opportunities_get-opportunity`)

`GET /opportunities/{opportunityId}` returns:

```json
{
  "id": "OPP_ID",
  "name": "Coral Way Dental — Practice Audit",
  "pipelineId": "PIPELINE_ID",
  "pipelineStageId": "STAGE_ID",
  "status": "open",
  "contactId": "CONTACT_ID",
  "contact": {
    "id": "CONTACT_ID",
    "name": "Carlos Mendez",
    "companyName": "Coral Way Dental",
    "email": "carlos@...",
    "phone": "+1...",
    "tags": []
  },
  "monetaryValue": null,
  "assignedTo": "USER_ID",
  "createdAt": "2026-05-20T...",
  "updatedAt": "2026-05-20T...",
  "customFields": [],
  "pipeline": { "id": "...", "name": "Proposal Pipeline" },
  "stage": { "id": "...", "name": "New Lead", "position": 0 }
}
```

To move the opportunity to a new stage: `opportunities_update-opportunity` with `pipelineStageId`.

---

## 3. Blog Publishing (`blogs_create-blog-post`)

### Capability: Full — HIGH confidence

Verified against official GHL API spec at `GoHighLevel/highlevel-api-docs`.

### Required Workflow: 3 lookups before creating

Creating a blog post requires IDs that must be fetched first:

1. **Blog site ID** — `GET /blogs/site/all?locationId=X&skip=0&limit=10`
2. **Author ID** — `GET /blogs/authors?locationId=X&limit=10&offset=0`
3. **Category IDs** — `GET /blogs/categories?locationId=X&limit=10&offset=0`

These IDs are stable for a given location — fetch once and store in config/constants.

### Create Blog Post Parameters

**Endpoint:** `POST /blogs/posts`
**Scope required:** `blogs/post.write`

```json
{
  "title": "5 Reasons Miami Dental Practices Are Losing Patients to Competitors",
  "locationId": "YOUR_LOCATION_ID",
  "blogId": "BLOG_SITE_ID",
  "rawHTML": "<h1>Full HTML content here...</h1>",
  "description": "Short excerpt/meta description for this post",
  "imageUrl": "https://example.com/featured-image.jpg",
  "imageAltText": "Alt text for SEO",
  "urlSlug": "miami-dental-practices-losing-patients",
  "author": "AUTHOR_ID",
  "categories": ["CATEGORY_ID_1"],
  "tags": ["dental marketing", "SE Florida"],
  "status": "DRAFT",
  "publishedAt": "2026-05-20T10:00:00.000Z",
  "canonicalLink": ""
}
```

**Required fields:** `title`, `blogId`, `rawHTML`, `description`, `imageUrl`, `imageAltText`, `urlSlug`, `author`, `categories`, `publishedAt`

**Status values:** `DRAFT`, `PUBLISHED`, `SCHEDULED`, `ARCHIVED`

### Blog Post Response

```json
{
  "data": {
    "_id": "66f429b8afdce84227a4610d",
    "title": "5 Reasons...",
    "description": "Short excerpt...",
    "imageUrl": "https://...",
    "imageAltText": "Alt text",
    "urlSlug": "miami-dental-...",
    "author": "AUTHOR_ID",
    "categories": ["CATEGORY_ID"],
    "tags": ["dental marketing"],
    "status": "DRAFT",
    "publishedAt": "2026-05-20T...",
    "updatedAt": "2026-05-20T...",
    "archived": false,
    "rawHTML": "<h1>...</h1>"
  }
}
```

### URL Slug Validation

Before publishing, verify slug uniqueness: `GET /blogs/posts/url-slug-exists?urlSlug=SLUG&locationId=LOC_ID`
Returns: `{ "exists": true/false }`

### Update Blog Post

`PUT /blogs/posts/{postId}` — all fields optional except `locationId` and `blogId`.

### Important Note on MCP Tool Names

The named tools in the prompt (`blogs_create-blog-post`, `blogs_update-blog-post`, `blogs_get-blog-post`) correspond to the native GHL MCP streamable HTTP endpoint. These map to:
- `POST /blogs/posts` → `blogs_create-blog-post`
- `PUT /blogs/posts/{postId}` → `blogs_update-blog-post`
- `GET /blogs/posts/all` → `blogs_get-blog-post`

The tenfoldmarc implementation (which most closely mirrors what a direct GHL MCP exposes) uses `rawHTML` as the content field name — not `content` or `html`. Use `rawHTML`.

---

## 4. Document / File Attachment to Contacts

### Capability: PARTIAL — important constraints

#### What works: Notes

`POST /contacts/{contactId}/notes` — attach a text note to a contact record.

```json
{ "body": "Proposal generated 2026-05-20. See custom fields for details." }
```

Notes appear in the contact's activity timeline in GHL. This is the simplest way to log that a proposal was generated.

#### What works: Custom field file uploads

`POST /forms/upload-custom-files` — upload a file to a custom field on a contact. Supports PDF, DOCX, JPG, PNG, CSV, XLSX, MP4, ZIP, TXT, SVG. Max 50 MB.

The file field format is multipart: key is `{custom_field_id}_{file_id}` (where file_id is a UUID). Query params: `contactId` and `locationId`.

This endpoint returns the updated contact object. HTML files are NOT in the supported type list — so you cannot attach an HTML proposal file directly to a contact's file upload field.

#### What does NOT work for DES proposals

- Attaching an HTML file to a contact record is not supported (not in allowed file types).
- GHL Documents & Contracts is the correct mechanism for proposals — this is outside the standard Contacts API and is managed through the GHL UI, not the API.

#### Recommended approach for DES

The existing `ghl-workflow-spec.md` already has the correct pattern: write proposal data to 10 custom fields on the contact (`proposal__google_rating`, `proposal__pain_analysis`, etc.), then in GHL, create the document from the "DentalEdge Practice Growth Proposal" template which pulls those custom fields as merge variables. This is the right architecture.

To signal completion from Claude Code:
1. Write the 10 proposal custom fields via `contacts_update-contact`
2. Move the opportunity stage via `opportunities_update-opportunity`
3. Add a note via `contacts/{id}/notes` summarizing what was written and when

---

## 5. Key Scopes Required (Private Integration Token)

| Scope | Required For |
|-------|-------------|
| `contacts.readonly` | Reading contact data |
| `contacts.write` | Updating contacts (custom fields, proposal data) |
| `opportunities.readonly` | Reading pipeline/stage |
| `opportunities.write` | Moving opportunity stage |
| `forms.readonly` | Reading form submissions |
| `blogs/post.write` | Creating blog posts |
| `blogs/posts.readonly` | Reading/listing blog posts |
| `blogs/author.readonly` | Getting author IDs |
| `blogs/category.readonly` | Getting category IDs |
| `blogs/list.readonly` | Getting blog site IDs |
| `blogs/check-slug.readonly` | Verifying URL slug availability |
| `locations.readonly` | Getting custom field definitions |
| `conversations.readonly` | Reading conversations/messages |

---

## 6. Workflow Patterns for DES

### Reading a contact's audit form data

The cleanest pattern is to read the contact directly — the GHL workflow writes form fields to contact custom fields during ingestion:

```
1. contacts_get-contact(contactId) 
   → returns customFields array with field IDs and values
2. locations_get-custom-fields(locationId)
   → returns field definitions with id, key, name
3. Map: for each custom field in contact, find key by field ID
```

One `get_custom_fields` call at startup, cached. One `get_contact` call per lead. No form submission parsing needed.

### Writing proposal data back to contact

`PUT /contacts/{contactId}` with `customFields` array:

```json
{
  "customFields": [
    { "id": "FIELD_ID_FOR_proposal__google_rating", "field_value": "4.8" },
    { "id": "FIELD_ID_FOR_proposal__review_count", "field_value": "286" },
    { "id": "FIELD_ID_FOR_proposal__pain_analysis", "field_value": "..." }
  ]
}
```

Note: GHL uses `id` + `field_value` format in the update payload (not `id` + `value`). The read response uses `id` + `value`. This is a known asymmetry in the GHL API.

### Publishing a blog post

```
1. Get blog site ID: blogs/site/all → cache in config
2. Get author ID: blogs/authors → cache (Erick's user = author)
3. Get category ID: blogs/categories → cache
4. Check slug: blogs/posts/url-slug-exists
5. Create: POST /blogs/posts with rawHTML, all required fields, status: "DRAFT"
6. Review in GHL UI, then update status to "PUBLISHED"
```

---

## 7. Known Limitations and Gotchas

| Issue | Description | Workaround |
|-------|-------------|------------|
| No `contactId` filter on form submissions | Official spec only has `q` for contact search | Use `q=CONTACT_ID` or read contact directly |
| `GET /forms/{id}` not supported | Returns 401 | Use `GET /forms/submissions` with `formId` param |
| Custom field ID vs key | Response uses `id`+`value`; update uses `id`+`field_value` | Map IDs via `get_custom_fields` at startup |
| Blog requires blogId/authorId/categoryId | Cannot create post without these pre-fetched | One-time lookup and cache in config |
| HTML not supported for file attachments | `upload-custom-files` does not accept HTML | Use Documents & Contracts template with merge fields |
| `notes` field on opportunity | Not accepted on create or update (returns 422) | Use contact notes endpoint instead |
| `locationId` in opportunity update | Must NOT be in request body (returns 422) | Only include in the update body when explicitly documented |

---

## 8. GHL MCP Tool Name Reference

Based on the native GHL MCP server at `services.leadconnectorhq.com/mcp/`, the tool names follow the pattern `{resource}_{operation}`:

| MCP Tool Name | HTTP Endpoint | Notes |
|---------------|---------------|-------|
| `contacts_get-contact` | `GET /contacts/{id}` | Full contact + custom fields |
| `contacts_get-contacts` | `GET /contacts/` | Search/list |
| `contacts_update-contact` | `PUT /contacts/{id}` | Write custom fields |
| `opportunities_get-opportunity` | `GET /opportunities/{id}` | Full opportunity |
| `opportunities_update-opportunity` | `PUT /opportunities/{id}` | Move stage, update status |
| `locations_get-custom-fields` | `GET /locations/{id}/customFields` | Field definitions |
| `conversations_get-messages` | `GET /conversations/{id}/messages` | Message history |
| `blogs_create-blog-post` | `POST /blogs/posts` | Requires blogId, authorId, categoryId |
| `blogs_update-blog-post` | `PUT /blogs/posts/{id}` | Publish draft |
| `blogs_get-blog-post` | `GET /blogs/posts/all` | List posts by blogId |

Additional tools likely available (verify at runtime):
- `forms_get-submissions` / `get_form_submissions`
- `contacts_create-note` / note creation
- `blogs_get-sites` — to get blogId
- `blogs_get-authors` — to get authorId
- `blogs_get-categories` — to get categoryId
- `blogs_check-url-slug` — to validate slug

---

## Sources

- Official GHL API spec: https://github.com/GoHighLevel/highlevel-api-docs (commit 192cd68)
- tenfoldmarc/ghl-mcp: https://github.com/tenfoldmarc/ghl-mcp (70+ tools, PIT-auth MCP)
- basicmachines-co/open-ghl-mcp: https://github.com/basicmachines-co/open-ghl-mcp (OAuth MCP, confirmed API limitations)
- BusyBee3333/Go-High-Level-MCP-2026-Complete: https://github.com/BusyBee3333/Go-High-Level-MCP-2026-Complete (576-endpoint coverage, TypeScript types)
- Prior DES project research: `/Users/erick/claude-examples/des-strategy-proposals/ghl-workflow-spec.md`
