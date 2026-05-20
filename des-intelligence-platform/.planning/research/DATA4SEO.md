# DataForSEO API Research

**Project:** DES Intelligence Platform
**Researched:** 2026-05-20
**Source:** Context7 / DataForSEO v3 official docs (library ID: `/llmstxt/dataforseo_v3_llms_txt`)
**Overall confidence:** HIGH — all findings verified against official DataForSEO v3 documentation

---

## Authentication

DataForSEO uses **HTTP Basic Authentication** everywhere. You pass your login and password (the credentials from https://app.dataforseo.com/api-access) as base64-encoded `login:password`.

**Python pattern (official RestClient):**
```python
from client import RestClient
# Download from https://cdn.dataforseo.com/v3/examples/python/python_Client.zip
client = RestClient("your_login", "your_password")
response = client.post("/v3/serp/google/organic/live/regular", post_data)
```

**Raw requests pattern (no SDK):**
```python
import requests
import base64

creds = base64.b64encode(b"login:password").decode()
headers = {
    "Authorization": f"Basic {creds}",
    "Content-Type": "application/json"
}
response = requests.post(
    "https://api.dataforseo.com/v3/serp/google/organic/live/regular",
    headers=headers,
    json=[{"keyword": "dentist miami", "location_code": 1015116, "language_code": "en"}]
)
```

**Key detail:** The `login` is your account email, and the `password` is the API password shown in the dashboard — it is NOT your account login password.

---

## Use Case 1: Local SEO Rankings

### What's Actually Being Measured

There are two distinct ranking signals for a dental practice:

1. **Organic (blue links)** — the practice's website ranking in standard web results
2. **Local Pack / Google Maps** — the 3-5 business listings in the map box (the most important for dental)

DataForSEO handles both, but through different endpoints. For dental practices, the Maps/Local Pack is the primary signal.

---

### Endpoint A — Google Maps SERP (primary for dental local rankings)

```
POST https://api.dataforseo.com/v3/serp/google/maps/live/advanced
```

This endpoint simulates a Google Maps search for a keyword + location and returns the ranked local business results. This is the correct endpoint for "where does Miami Smiles Dental rank for 'dentist miami'?"

**Request:**
```json
[
  {
    "keyword": "dentist miami",
    "location_name": "Miami,Florida,United States",
    "language_code": "en",
    "device": "desktop",
    "os": "windows"
  }
]
```

You can also use `location_code` (integer) instead of `location_name`. Miami location code is `1015116`.

**Key response fields per result item:**
```json
{
  "type": "maps_search",
  "rank_group": 1,
  "rank_absolute": 1,
  "title": "Miami Smiles Dental",
  "domain": "miamismilesdental.com",
  "website": "https://miamismilesdental.com",
  "address": "123 Brickell Ave, Miami, FL 33131",
  "address_info": {
    "city": "Miami",
    "region": "Florida",
    "country_code": "US"
  },
  "phone": "(305) 555-1234",
  "rating": 4.8,
  "reviews_count": 342,
  "price_level": "$$",
  "place_id": "ChIJp6_4d78-j4AR...",
  "cid": "12368344280360434538",
  "latitude": 25.7617,
  "longitude": -80.1918,
  "category": "Dentist",
  "is_claimed": true,
  "work_hours": { ... }
}
```

**To find a specific practice's rank:** iterate the `items` array and match on `domain` or `title`.

**Important limitation:** Google Maps results return up to ~20 local results per query. If the practice isn't in the top 20, it won't appear. For rank tracking beyond position 20, you'd need to paginate or use a different approach.

---

### Endpoint B — Google Local Finder SERP

```
POST https://api.dataforseo.com/v3/serp/google/local_finder/live/advanced
```

Local Finder is what appears when a user clicks "More places" from the local pack. Returns more results (up to 45+) than Maps for the same keyword/location.

**Request:**
```json
[
  {
    "keyword": "dental implants fort lauderdale",
    "location_name": "Fort Lauderdale,Florida,United States",
    "language_code": "en",
    "min_rating": 0,
    "device": "desktop",
    "os": "windows"
  }
]
```

Response structure is similar to Maps. Use this when you need deeper rank penetration (positions 4–45) or when a practice doesn't appear in the top 3.

---

### Endpoint C — Google Organic SERP (website rankings)

```
POST https://api.dataforseo.com/v3/serp/google/organic/live/regular
```

This returns standard web search results — useful for tracking website rankings (blue links), but **not** for tracking Google Business Profile / Maps pack rankings.

**Request:**
```json
[
  {
    "keyword": "dentist miami",
    "location_code": 1015116,
    "language_code": "en",
    "device": "desktop",
    "os": "windows"
  }
]
```

Each item in the response has:
- `rank_group` and `rank_absolute` — position in SERP
- `domain` — the domain (e.g., `miamismilesdental.com`)
- `url` — specific page URL
- `title` — page title
- `description` — meta description

To check if a practice's domain appears: filter `items` where `domain == "practicewebsite.com"`.

---

### Location Codes for Florida Markets

Common DataForSEO location codes (use these instead of `location_name` for reliability):

| City | location_code |
|------|--------------|
| Miami, FL | 1015116 |
| Fort Lauderdale, FL | 1013625 |
| Boca Raton, FL | 1012873 |
| West Palm Beach, FL | 1016321 |
| Orlando, FL | 1015584 |
| Tampa, FL | 1016166 |
| Jacksonville, FL | 1014625 |

You can also use GPS coordinates via `location_coordinate` parameter: `"25.7617,-80.1918,14z"` — the zoom level (`z`) controls the search radius. `14z` ≈ city-level. This is useful for hyper-local targeting.

---

### Task Methods: Live vs. Standard

Every endpoint has two execution modes:

| Mode | How it works | Speed | Cost |
|------|-------------|-------|------|
| **Live** (`/live/regular`, `/live/advanced`) | Synchronous — returns data in the same HTTP call | ~5–30 seconds | Higher per-call cost |
| **Standard** (`task_post` + `task_get`) | Async — POST creates task, GET retrieves when ready | Minutes | Lower cost |

**Recommendation for dental platform:** Use **Live** endpoints for on-demand rank checks triggered by user action. Use **Standard** (task_post) for scheduled nightly batch jobs tracking many keywords across many practices.

---

## Use Case 2: Competitor Comparison

### Finding Nearby Competitors

DataForSEO does NOT have an endpoint that takes one domain and automatically discovers geographically nearby competitors. You have to derive competitors through SERP data.

**The correct workflow:**

**Step 1 — Run a Maps/Local Finder SERP query for the target keyword + city.** The result set IS the local competitor list. All businesses ranking for "dentist miami" in Google Maps are the competition.

```python
# Returns up to 20 local businesses ranked for this keyword in this city
response = client.post("/v3/serp/google/maps/live/advanced", [{
    "keyword": "dentist miami",
    "location_name": "Miami,Florida,United States",
    "language_code": "en"
}])
# Extract: title, domain, rating, reviews_count, rank_absolute for each item
```

**Step 2 — Pull the target practice's rank from the same response.** If `miamismilesdental.com` is at `rank_absolute: 4`, and two competitors are at 1, 2, 3, you have a direct comparison.

**Step 3 — For domain-level SEO metric comparison** (organic traffic estimate, keyword count), use DataForSEO Labs:

```
POST https://api.dataforseo.com/v3/dataforseo_labs/google/competitors_domain/live
```

This endpoint takes your target domain and an array of competitor domains you already know, and returns intersecting keyword metrics.

**Request:**
```python
post_data = [{
    "target": "miamismilesdental.com",
    "intersecting_domains": [
        "southfloridadental.com",
        "coraldentistry.com"
    ],
    "language_name": "English",
    "location_name": "United States",
    "limit": 50
}]
```

**What it returns:** For each keyword where both domains rank, you get organic position, traffic estimates, and keyword metrics. Good for identifying keyword gaps.

**Important caveat:** `competitors_domain/live` uses DataForSEO's historical keyword index — it reflects index data (updated weekly), NOT a real-time SERP check. Small local practices may have sparse index coverage. This endpoint works better for multi-location DSOs or practices with established web presence.

---

### Organic Ranked Keywords by Domain

```
POST https://api.dataforseo.com/v3/dataforseo_labs/google/ranked_keywords/live
```

Pass a single domain and get back all keywords it ranks for in Google organic, with positions. Filter by `location_code` to restrict to a specific metro.

```python
post_data = [{
    "target": "miamismilesdental.com",
    "location_code": 2840,   # United States (Labs uses national codes)
    "language_name": "English",
    "limit": 100,
    "filters": [["keyword_data.keyword_info.search_volume", ">", 10]]
}]
```

**What it returns per keyword:** keyword text, search volume, competition, current rank, URL that ranks, traffic estimate.

This gives you a "keyword portfolio" view — useful for comparing two dental practices' organic visibility side by side.

---

## Use Case 3: Google Business Profile Signals

### Reviews + Rating + Review Count

**Primary endpoint:**
```
POST https://api.dataforseo.com/v3/business_data/google/reviews/task_post
GET  https://api.dataforseo.com/v3/business_data/google/reviews/task_get/{id}
```

This is an async endpoint (task_post + task_get). No live version.

You query by **business name + location** (not by domain or place_id):

```python
post_data = [{
    "keyword": "Miami Smiles Dental",          # Business name
    "location_name": "Miami,Florida,United States",
    "language_name": "English",
    "depth": 100,                              # Number of reviews to fetch
    "sort_by": "most_relevant"                 # or "newest", "highest_rating", "lowest_rating"
}]
response = client.post("/v3/business_data/google/reviews/task_post", post_data)
task_id = response["tasks"][0]["id"]
# Wait for completion, then:
result = client.get(f"/v3/business_data/google/reviews/task_get/{task_id}")
```

**Task completion time:** Typically 1–5 minutes. Use pingback_url for async notification.

**Top-level response fields:**
- `reviews_count` — total review count (integer)
- `rating` — aggregate rating object: `{"rating_type": "Max5", "value": 4.8, "votes_count": 342}`

**Per-review fields:**
- `review_text`, `original_review_text`
- `timestamp` — exact UTC datetime of review
- `rating.value` — 1–5
- `local_guide` — boolean
- `owner_answer` — practice's reply text
- `owner_timestamp` — when practice replied
- `review_highlights` — structured tags like `{"feature": "Service", "assessment": "Excellent"}`

---

### Google My Business Info (Full Business Profile)

```
POST https://api.dataforseo.com/v3/business_data/google/my_business_info/task_post
GET  https://api.dataforseo.com/v3/business_data/google/my_business_info/task_get/{id}
```

This returns the full GBP profile snapshot for a business. More comprehensive than reviews.

**Key fields returned:**
```json
{
  "title": "Miami Smiles Dental",
  "website": "https://miamismilesdental.com",
  "rating": 4.8,
  "rating_count": 342,
  "reviews_total": 342,
  "reviews_per_rating": {"1": 3, "2": 1, "3": 8, "4": 47, "5": 283},
  "phone_number": "(305) 555-1234",
  "address": "123 Brickell Ave, Miami, FL 33131",
  "latitude": 25.7617,
  "longitude": -80.1918,
  "work_hours": [...],
  "categories": ["Dentist", "Cosmetic Dentist"],
  "is_claimed": true,
  "cid": "12368344280360434538",
  "place_id": "ChIJp6_4d78-j4AR...",
  "description": "...",
  "photos": [...],
  "featured_photo": "https://..."
}
```

**What it does NOT provide:** GBP post activity (how often the practice posts Google Posts), Q&A activity, or Google Post content. Those signals are not available via DataForSEO Business Data API.

---

### GBP Post Activity — NOT Available via DataForSEO

DataForSEO does not have an endpoint for Google Business Profile post history or posting frequency. This is a known gap in the API. To track GBP posting activity, the options are:

1. **Google My Business API** (now Google Business Profile API) — requires OAuth per-business and is only accessible to the business owner or delegated manager. Not usable for competitor monitoring.
2. **Manual Google Maps scraping** — posts appear on the business's Google Maps listing under "Updates" tab. Not available via DataForSEO.
3. **Accept the gap** — track reviews + ratings as GBP health proxy. Post frequency correlates with engagement but isn't directly measurable for competitors via any API.

**Recommendation:** For the intelligence platform, track `reviews_total`, `rating`, `reviews_per_rating` distribution, and `owner_answer` presence (response rate) as GBP health signals. Skip post frequency for competitor profiles — it requires Google's own API and business owner access.

---

## API Execution Patterns

### Synchronous (Live) — Use for On-Demand Checks

All `/live/` endpoints return synchronous results. The HTTP connection stays open until DataForSEO fetches the data (typically 5–30 seconds). Maximum 100 tasks per POST request.

```python
response = client.post("/v3/serp/google/maps/live/advanced", [{
    "keyword": "dentist miami",
    "location_name": "Miami,Florida,United States",
    "language_code": "en"
}])
if response["status_code"] == 20000:
    items = response["tasks"][0]["result"][0]["items"]
```

**Timeout risk:** Live endpoints can take up to 30 seconds. Set `requests.post(..., timeout=60)`.

### Async (Task-Based) — Use for Batch/Scheduled Jobs

Standard workflow for batch operations:

```python
# Step 1: Submit up to 100 tasks
response = client.post("/v3/business_data/google/reviews/task_post", tasks_list)
task_ids = [t["id"] for t in response["tasks"]]

# Step 2: Poll for completion (or use pingback_url)
import time
for task_id in task_ids:
    while True:
        result = client.get(f"/v3/business_data/google/reviews/task_get/{task_id}")
        if result["tasks"][0]["status_code"] == 20000:
            break
        time.sleep(15)  # check every 15 seconds
```

Better pattern: set `pingback_url` on task_post — DataForSEO will GET your URL when a task completes. This eliminates polling.

Results are stored for **30 days** after task creation.

---

## Costs

DataForSEO uses a credit-based model. Costs in USD are returned in each API response in the `cost` field.

| Endpoint | Approximate Cost |
|----------|----------------|
| `serp/google/organic/live/regular` | ~$0.003 per call |
| `serp/google/maps/live/advanced` | ~$0.001–0.002 per call |
| `serp/google/local_finder/live/advanced` | ~$0.001 per call |
| `business_data/google/reviews/task_post` | ~$0.00375 per task (100 reviews depth) |
| `business_data/google/my_business_info` | ~$0.001 per task |
| `dataforseo_labs/google/competitors_domain/live` | ~$0.01 per request + $0.0001 per result |
| `dataforseo_labs/google/ranked_keywords/live` | ~$0.00714 per request |

**Cost control strategies:**

1. **Batch tasks** — each POST can contain up to 100 task objects. One HTTP call for 100 keywords costs ~100x less in overhead than 100 individual calls.

2. **Use Standard (async) over Live for bulk jobs** — async tasks are cheaper than live. Reserve live endpoints for user-triggered actions where latency matters.

3. **Use `location_code` over `location_name`** — more reliable, avoids name-resolution overhead.

4. **Cache aggressively** — rankings for a dental practice don't change minute-to-minute. Daily refresh is sufficient for most metrics. Weekly is fine for Labs data.

5. **Use `depth` parameter wisely on reviews** — `depth: 10` for a quick aggregate rating check. `depth: 200` only when doing full review analysis.

6. **Sandbox environment** — DataForSEO provides `https://sandbox.dataforseo.com/` which returns mock data for free. Use it for all development and testing to avoid burning credits.

---

## Response Times

| Endpoint type | Typical response time |
|--------------|----------------------|
| Live SERP (Maps, Organic, Local Finder) | 5–30 seconds |
| Live Labs (competitors, ranked keywords) | 1–5 seconds |
| Async task_post | Immediate (returns task ID) |
| Async task completion | 1–10 minutes (Business Data); 15–60 min for large depth |

---

## Recommended Endpoints Per Use Case (Summary)

### For Local SEO Rankings

| Goal | Endpoint | Notes |
|------|----------|-------|
| Find practice rank in Google Maps for keyword + city | `POST /v3/serp/google/maps/live/advanced` | Primary endpoint for dental |
| Find practice rank in Local Finder (positions 4–45) | `POST /v3/serp/google/local_finder/live/advanced` | Use when practice not in top 3 |
| Find practice rank in organic web results | `POST /v3/serp/google/organic/live/regular` | For website SEO tracking |
| Keyword portfolio for a domain | `POST /v3/dataforseo_labs/google/ranked_keywords/live` | Weekly batch job |

### For Competitor Comparison

| Goal | Endpoint | Notes |
|------|----------|-------|
| Get all local competitors for keyword + city | `POST /v3/serp/google/maps/live/advanced` | Competitors ARE the SERP results |
| Compare keyword overlap between two domains | `POST /v3/dataforseo_labs/google/competitors_domain/live` | Best for established practices |
| Get ratings + review count for any competitor | `POST /v3/business_data/google/reviews/task_post` | Query by business name |
| Get full GBP profile for competitor | `POST /v3/business_data/google/my_business_info/task_post` | Includes category, hours, photos |

### For GBP Signals

| Goal | Endpoint | Notes |
|------|----------|-------|
| Aggregate rating + review count | `POST /v3/business_data/google/my_business_info/task_post` | Returns `rating`, `reviews_total` |
| Full review text + timestamps + owner replies | `POST /v3/business_data/google/reviews/task_post` | Set `depth` to desired count |
| GBP post activity | NOT AVAILABLE | No DataForSEO endpoint exists |

---

## Practical Workflow for Dental Platform

### Daily/Nightly Batch Job

```python
KEYWORDS = ["dentist miami", "dental implants miami", "cosmetic dentist miami",
            "emergency dentist miami", "teeth whitening miami"]
LOCATION_CODE = 1015116  # Miami, FL
PRACTICE_DOMAIN = "miamismilesdental.com"

tasks = []
for kw in KEYWORDS:
    tasks.append({
        "keyword": kw,
        "location_code": LOCATION_CODE,
        "language_code": "en",
        "device": "desktop",
        "os": "windows"
    })

# One POST call, 5 tasks
response = client.post("/v3/serp/google/maps/live/advanced", tasks)

# Extract ranks
for i, task_result in enumerate(response["tasks"]):
    items = task_result["result"][0]["items"]
    for item in items:
        if item.get("domain") == PRACTICE_DOMAIN or item.get("website", "").contains(PRACTICE_DOMAIN):
            print(f"{KEYWORDS[i]}: rank {item['rank_absolute']}")
```

### GBP Health Check (Weekly)

```python
# Step 1: Post task
response = client.post("/v3/business_data/google/my_business_info/task_post", [{
    "keyword": "Miami Smiles Dental",
    "location_name": "Miami,Florida,United States",
    "language_name": "English"
}])
task_id = response["tasks"][0]["id"]

# Step 2: After task completes (use pingback in production)
result = client.get(f"/v3/business_data/google/my_business_info/task_get/{task_id}")
data = result["tasks"][0]["result"][0]
rating = data["rating"]           # float
reviews_total = data["reviews_total"]  # int
reviews_by_star = data["reviews_per_rating"]  # {"1": N, "2": N, ...}
```

---

## Known Gaps and Limitations

1. **GBP post activity** — No endpoint. Accept this gap or supplement with GBP Management API (requires practice owner auth).

2. **Domain-to-practice matching in Maps** — The Maps SERP returns a `website` field but it's not always normalized. A practice at `www.miamismilesdental.com/` might return as `miamismilesdental.com`. Always match on normalized domain (strip `www.`, trailing slash).

3. **Small practice index coverage in Labs** — `ranked_keywords/live` and `competitors_domain/live` use DataForSEO's keyword index. Small local practices with low organic traffic may show 0–5 indexed keywords even if they rank locally. The Maps/Local Finder SERP endpoints are always reliable for local pack tracking regardless of index coverage.

4. **Location specificity** — `location_code: 1015116` (Miami) triggers a search from within Miami but does not simulate sub-city geolocation (e.g., "from Brickell neighborhood"). Use `location_coordinate` with a specific lat/long for hyper-local accuracy. This matters for dental — a practice in Coral Gables may rank differently for "dentist near me" depending on where the simulated user is located.

5. **Review task matching ambiguity** — Reviews are queried by business name keyword, not by place_id. If a practice has a common name ("Family Dental Care"), the API may return results for a different business. Verify match using `place_id` or `cid` in the response against a known value.

6. **Rate limits** — DataForSEO documents 2000 concurrent tasks per account. In practice, the bottleneck is the synchronous live endpoint timeout (up to 30s), not a hard rate limit. For a platform tracking 50+ practices across 5 keywords, async task_post is essential to avoid sequential bottlenecks.

---

## Sources

All findings verified against official DataForSEO v3 documentation via Context7 (HIGH confidence):

- https://docs.dataforseo.com/v3/serp/google/maps/live/advanced
- https://docs.dataforseo.com/v3/serp/google/local_finder/live/advanced
- https://docs.dataforseo.com/v3/serp/google/organic/live/regular
- https://docs.dataforseo.com/v3/business_data/google/reviews/task_post
- https://docs.dataforseo.com/v3/business_data/google/reviews/task_get
- https://docs.dataforseo.com/v3/business_data/google/my_business_info/task_get
- https://docs.dataforseo.com/v3/dataforseo_labs/google/competitors_domain/live
- https://docs.dataforseo.com/v3/dataforseo_labs/google/ranked_keywords/live
- https://docs.dataforseo.com/v3/auth
- https://docs.dataforseo.com/v3/appendix/user_data (pricing)
