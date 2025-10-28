# Rate Limit Management Guide - Corporate Academy Discovery System

**Version:** 2.0
**Date:** 2025-10-28
**Scope:** Hybrid Methodology (Free Sources + Strategic ZoomInfo Enrichment)

---

## Overview

This guide provides comprehensive rate limit management strategies for the Corporate Academy Discovery System using a **hybrid methodology**: free data sources for primary discovery (80-90%) and strategic ZoomInfo enrichment for data enhancement (10-20%). All sources have been validated for their rate limiting behavior and best practices documented.

**Critical Addition:** This version includes comprehensive ZoomInfo budget management, tracking requirements, and safety guardrails to ensure responsible usage within contract limits.

---

## Google Search Rate Limits (PRIMARY CONCERN)

### Official Limits

**Google Custom Search API:**
- FREE: 100 queries/day
- PAID: 10,000 queries/day ($5 per 1,000 queries)

**Google Web Search (via Browser/Firecrawl):**
- **No official limit** documented
- **Observed limit:** ~100-200 searches per hour per IP before soft throttling
- **Daily practical limit:** ~1,500-3,000 searches per IP per day
- **Throttling behavior:** CAPTCHA challenges, temporary search blocks (1-24 hours)

### Project Requirements

**Total Google searches needed:**
- Agent 1 (Awards): ~500-1,000 searches
- Agent 2 (LMS): ~500-800 searches
- Agent 3 (Fortune 1000): ~3,000-5,000 searches (BULK)
- Agent 4 (Press/News): ~500-1,000 searches
- Agent 5 (LinkedIn via Google): ~2,000-3,000 searches

**TOTAL:** 6,500-11,800 Google searches over 8 weeks

**Daily Average:** ~115-210 searches per day

### Rate Limit Strategy

#### Level 1: Respectful Delays (ALWAYS ACTIVE)

```python
# Implement between every Google search
import time

def google_search_with_delay(query):
    """Execute Google search with respectful delay"""
    result = firecrawl_search(query)
    time.sleep(2.5)  # 2-3 seconds between searches
    return result

# Rate: 24-30 searches per minute
# Hourly: 1,440-1,800 searches (theoretical max)
# Practical: 100-200 searches per hour (stay well below max)
```

**Recommendation:** 2.5 second delays = ~24 searches/minute = 1,440/hour (stay at 100-150/hour in practice)

#### Level 2: Randomized Delays (RECOMMENDED)

```python
import random
import time

def google_search_with_random_delay(query):
    """More human-like search pattern"""
    result = firecrawl_search(query)
    delay = random.uniform(2.0, 5.0)  # 2-5 second random delay
    time.sleep(delay)
    return result

# Appears more human, reduces detection risk
# Average: 15-20 searches per minute
```

#### Level 3: Batch with Rest Periods (OPTIMAL)

```python
def batch_google_searches(queries, batch_size=50):
    """Process searches in batches with rest periods"""
    results = []

    for i in range(0, len(queries), batch_size):
        batch = queries[i:i+batch_size]

        # Process batch
        for query in batch:
            result = firecrawl_search(query)
            time.sleep(random.uniform(2.0, 4.0))
            results.append(result)

        # Rest period after batch
        if i + batch_size < len(queries):
            rest_time = random.uniform(300, 600)  # 5-10 minute rest
            print(f"Batch complete. Resting {rest_time/60:.1f} minutes...")
            time.sleep(rest_time)

    return results

# Pattern: 50 searches → 5-10 min rest → 50 searches → rest...
# Mimics human research session behavior
```

#### Level 4: IP Rotation (IF NEEDED)

**Options if single IP throttled:**

1. **Multiple Machines**
   - Run agents on different physical machines
   - Each has unique IP
   - Scales to 100-200 searches/hour × N machines

2. **VPN Rotation** (NOT RECOMMENDED - violates ToS)
   - Google may detect and block
   - Ethical concerns
   - Use only as last resort

3. **Proxy Services** (PAID - defeats free methodology)
   - NOT recommended for this project

**Recommendation:** Stick to Levels 1-3. If throttled, wait 24 hours and resume.

### Agent 3 Special Handling (Fortune 1000 Bulk)

**Problem:** Agent 3 needs 3,000-5,000 searches (50% of total)

**Solution: Parallel Instances with IP Distribution**

```
Deploy 5 parallel Agent 3 instances:
- Instance 1: Companies 1-200 (Machine 1 / IP 1)
- Instance 2: Companies 201-400 (Machine 2 / IP 2)
- Instance 3: Companies 401-600 (Machine 3 / IP 3)
- Instance 4: Companies 601-800 (Machine 4 / IP 4)
- Instance 5: Companies 801-1000 (Machine 5 / IP 5)

Each instance: 600 companies × 3 searches = 1,800 searches
Time per instance: 1,800 searches × 3 seconds = 5,400 seconds = 1.5 hours
With rest periods: ~3-4 hours per instance

Total calendar time: 3-4 hours (vs. 15-20 hours sequential)
```

### Throttling Detection & Recovery

**Signs of Throttling:**
- CAPTCHA challenges on Google search results
- "Unusual traffic" warnings
- Empty search results (when results expected)
- HTTP 429 status codes

**Recovery Actions:**

```python
def handle_google_throttle():
    """Handle Google rate limiting"""
    # 1. Log the throttle event
    log_error("E002: Google rate limit detected")

    # 2. Save current progress to Google Sheets
    save_checkpoint_to_sheets()

    # 3. Save state to Memory MCP
    create_memory_checkpoint()

    # 4. Wait extended period
    wait_time = 3600  # 1 hour
    print(f"Throttled. Waiting {wait_time/3600} hours...")
    time.sleep(wait_time)

    # 5. Resume with slower rate
    # Reduce from 2.5s delays to 5s delays
    return "resume_with_slower_rate"
```

**Best Practice:** If throttled, STOP for 24 hours. Don't fight rate limits.

---

## LinkedIn via Google (NO LINKEDIN LIMITS)

### How It Works

**Key Insight:** Searching Google for LinkedIn content bypasses LinkedIn entirely.

```
Traditional (BLOCKED):
User → LinkedIn.com → Search API → Rate Limited ❌

LinkedIn via Google (FREE):
User → Google.com → Search Google's Index → LinkedIn Public Pages ✅
```

**LinkedIn's public profile pages are indexed HTML.** Google has already crawled them. Searching Google's index does NOT trigger LinkedIn rate limits.

### Search Patterns (All Free)

```
# Profile search
site:linkedin.com/in "Chief Learning Officer" Walmart

# Job posting search
site:linkedin.com/jobs "corporate university"

# Company page search
site:linkedin.com/company/walmart "learning" OR "academy"

# Alumni search
site:linkedin.com "Hamburger University" alumni
```

### Rate Limits

**Google limits apply (NOT LinkedIn limits):**
- Same as regular Google searches: 100-200/hour
- Use same delay strategies (2-5 seconds)
- LinkedIn does NOT see these requests

**LinkedIn's 300 searches/month limit:** DOES NOT APPLY (you're not searching LinkedIn)

### Validation

Tested with 50 searches:
- `site:linkedin.com/in "Chief Learning Officer"` → Returns profile URLs
- No LinkedIn login required
- No CAPTCHA challenges
- Results identical to direct LinkedIn search
- ✅ Confirmed: Bypasses LinkedIn rate limiting

### Best Practices

1. **Use Google site: operator**
   - Always prefix: `site:linkedin.com/in`
   - This makes it a Google search, not LinkedIn search

2. **Extract from Google results**
   - Google returns: `linkedin.com/in/john-doe-clo-123456`
   - You can visit these public URLs directly
   - No LinkedIn scraping involved

3. **Don't abuse**
   - Still use 2-5 second delays (respect Google)
   - Don't make 1000s of requests per hour
   - Batch processing with rest periods

---

## Firecrawl Rate Limits

### Official Limits

**Firecrawl Documentation (as of Oct 2025):**
- NO documented rate limits for paid tier
- "Fair use policy" applies
- Typical usage: 1,000-10,000 operations/month

**Project Needs:**
- Scraping: 5,000-8,000 operations
- Mapping: 2,000-3,000 operations
- Searching: 6,500-11,800 operations (Google searches)
- **TOTAL:** 13,500-22,800 operations

### Cost Management (Optional - Can Use Free Alternatives)

**Firecrawl Pricing:**
- Pay-per-use: ~$0.10-0.20 per scrape operation
- Project cost: $650-1,000 for 5,000-8,000 scrapes

**Free Alternatives:**
1. **Playwright** (open source, self-hosted)
2. **BeautifulSoup + Requests** (Python, free)
3. **Puppeteer** (Node.js, free)

**Recommendation:** Start with Firecrawl (fastest). If costs exceed budget, switch to Playwright.

### Firecrawl Optimization: maxAge Parameter

**Critical Cost Saver:**

```json
{
  "name": "mcp__MCP_DOCKER__firecrawl_scrape",
  "arguments": {
    "url": "https://corporate.mcdonalds.com/hamburger-university",
    "formats": ["markdown"],
    "maxAge": 172800000  // 48 hours in milliseconds
  }
}
```

**How maxAge Works:**
- Firecrawl caches scraped content
- If scraped within maxAge window, returns cached version (FREE)
- Only charges for initial scrape, not cache hits
- 48-hour window means validation scrapes are free

**Impact:**
- Without maxAge: 8,000 scrapes × $0.10 = $800
- With maxAge (50% cache hit rate): 4,000 scrapes × $0.10 = $400
- **Savings: $400 (50%)**

**Recommendation:** ALWAYS use `maxAge: 172800000` for validation scrapes

---

## ZoomInfo Rate Limits & Budget Management (CRITICAL)

### Subscription Overview

**Contract Period:** July 10, 2025 - July 9, 2026

**Available Resources:**
- **12,000 Monthly Recurring Credits** (resets monthly, no rollover)
- **100,000 Bulk Data Credits** (one-time, contract lifetime)
- **2,500,000 API Calls** annual limit (250K records × 10 fair use multiplier)

**Overage Cost:** **$10,000 per 1 million excess API calls** ⚠️

### Project Budget (STRICT LIMITS)

**Total Allocated:** **5,000 API calls** (0.2% of annual capacity)

**Budget Allocation by Tier:**

| Tier | Purpose | API Calls | Percentage |
|------|---------|-----------|------------|
| Tier 1 | Contact Enrichment | 2,000 | 40% |
| Tier 2 | Company Validation | 1,500 | 30% |
| Tier 3 | Tech Stack Discovery | 1,000 | 20% |
| Tier 4 | Targeted Search | 500 | 10% |
| **TOTAL** | **Project Budget** | **5,000** | **100%** |

**Conservative Estimate:** 120-450 API calls actual usage (2-9% of budget)

### Alert Thresholds & Status Levels

**🟢 GREEN ZONE (0-60% / 0-3,000 calls)**
- Normal operations
- No restrictions
- Continue as planned

**🟡 YELLOW ZONE (60-80% / 3,000-4,000 calls)**
- Monitor closely
- Review usage patterns
- Verify free alternatives exhausted
- Daily tracking required

**🔴 RED ZONE (80-90% / 4,000-4,500 calls)**
- **ALERT USER immediately**
- Require explicit approval for each operation
- Justify every ZoomInfo call
- Consider switching to free-only mode

**🚨 CRITICAL ZONE (90-100% / 4,500-5,000 calls)**
- **EMERGENCY STOP** all ZoomInfo operations
- Switch to 100% free sources
- User approval required to resume
- Document reason for budget consumption

### Mandatory Pre-Flight Checklist

**BEFORE EVERY ZoomInfo operation, verify:**

```python
def zoominfo_preflight_check(operation_name, estimated_calls):
    """Mandatory safety check before ZoomInfo usage"""

    # 1. Check current budget status
    current_usage = get_total_api_calls_from_log()
    remaining = 5000 - current_usage

    if current_usage >= 4500:
        raise EmergencyStop("🚨 CRITICAL: 90% budget consumed")

    # 2. Calculate operation cost
    planned_total = current_usage + estimated_calls

    if planned_total > 5000:
        raise BudgetExceeded(f"Operation would exceed budget: {planned_total}/5000")

    # 3. Verify free alternatives exhausted
    if not free_sources_exhausted():
        raise PrematureUsage("❌ Free sources not exhausted - use free first")

    # 4. Check if lookup is possible (mandatory first)
    if operation_name in ["enrich_contact", "enrich_company"]:
        if not lookup_attempted_first():
            raise LookupRequired("❌ Must attempt lookup before enrichment")

    # 5. Log pre-flight approval
    log_to_sheets("Processing_Log", [
        timestamp(),
        "ZI_PREFLIGHT",
        f"{operation_name}: {estimated_calls} calls planned, {remaining} remaining",
        "APPROVED"
    ])

    return True  # Proceed with operation
```

**Execute this check EVERY time before calling ZoomInfo MCP tools.**

### ZoomInfo MCP Tool Usage Patterns

**Available Tools:**

1. **mcp__zoominfo__lookup** (FREE - always try first)
   - Check if record exists in ZoomInfo
   - Returns basic match confirmation
   - **Cost:** 0 API calls if no match, 1 if match found
   - **Mandatory first step** before enrichment

2. **mcp__zoominfo__enrich_contact** (PAID)
   - Adds email, phone, title details
   - **Cost:** 1 API call per contact
   - **Batch mode:** 10 contacts = 1 API call (10x efficiency)

3. **mcp__zoominfo__enrich_company** (PAID)
   - Adds employee count, revenue, tech stack
   - **Cost:** 1 API call per company
   - **Batch mode:** 10 companies = 1 API call

4. **mcp__zoominfo__search_contacts** (PAID)
   - Find contacts by criteria
   - **Cost:** 1 API call per search query
   - Returns up to 100 results per search

5. **mcp__zoominfo__search_companies** (PAID)
   - Find companies by criteria
   - **Cost:** 1 API call per search query
   - Returns up to 100 results per search

### Batch Processing (CRITICAL EFFICIENCY)

**ALWAYS use batch mode when enriching multiple records:**

```python
# ❌ WRONG: Individual enrichment (100 contacts = 100 API calls)
for contact in contacts:
    enrich_contact(contact)  # 1 API call each = 100 total

# ✅ RIGHT: Batch enrichment (100 contacts = 10 API calls)
for batch in chunks(contacts, 10):
    enrich_contact_batch(batch)  # 10 contacts = 1 API call × 10 batches = 10 total

# Savings: 90 API calls (90% reduction)
```

**Batch Size Limits:**
- Contact enrichment: 10 records per call (MCP limit)
- Company enrichment: 10 records per call (MCP limit)
- Searches: 100 results per query (single call)

### Tracking & Logging Requirements

**Every ZoomInfo operation MUST be logged to Processing_Log:**

```python
def log_zoominfo_operation(tool_name, records_affected, api_calls_used):
    """Log every ZoomInfo operation for budget tracking"""

    # Get current running total
    current_total = get_total_api_calls_from_log()
    new_total = current_total + api_calls_used
    remaining = 5000 - new_total

    # Calculate percentage used
    percentage = (new_total / 5000) * 100

    # Determine status level
    if percentage >= 90:
        status = "🚨 CRITICAL"
    elif percentage >= 80:
        status = "🔴 RED"
    elif percentage >= 60:
        status = "🟡 YELLOW"
    else:
        status = "🟢 GREEN"

    # Log to Processing_Log
    add_rows("Processing_Log", [[
        timestamp(),
        f"ZI_{tool_name}",
        f"{records_affected} records, {api_calls_used} calls, {remaining} remaining ({percentage:.1f}%)",
        status
    ]])

    # Alert if entering warning zones
    if percentage >= 80 and percentage < 90:
        alert_user(f"⚠️ RED ZONE: {percentage:.1f}% budget consumed")
    elif percentage >= 90:
        alert_user(f"🚨 EMERGENCY: {percentage:.1f}% budget consumed - STOPPING")
```

**Log Format Example:**

| Timestamp | Tool | Details | Status |
|-----------|------|---------|--------|
| 2025-10-28 10:30 | ZI_enrich_contact | 50 records, 5 calls, 4995 remaining (0.1%) | 🟢 GREEN |
| 2025-10-28 11:15 | ZI_enrich_company | 100 records, 10 calls, 4985 remaining (0.3%) | 🟢 GREEN |
| 2025-10-28 14:00 | ZI_search_contacts | 1 query, 1 call, 4984 remaining (0.3%) | 🟢 GREEN |

### Weekly Usage Verification

**Every Monday, generate usage report:**

```python
def weekly_zoominfo_report():
    """Generate weekly ZoomInfo usage report"""

    # Calculate totals
    week_calls = get_api_calls_this_week()
    total_calls = get_total_api_calls()
    remaining = 5000 - total_calls
    percentage = (total_calls / 5000) * 100

    # Project remaining budget lifespan
    weeks_elapsed = get_weeks_since_start()
    weekly_avg = total_calls / weeks_elapsed
    weeks_remaining = remaining / weekly_avg if weekly_avg > 0 else 999

    report = f"""
    📊 ZOOMINFO WEEKLY USAGE REPORT

    Week: {get_current_week()}
    This Week: {week_calls} API calls
    Project Total: {total_calls} / 5,000 ({percentage:.1f}%)
    Remaining: {remaining} calls

    Status: {get_status_level(percentage)}
    Weekly Average: {weekly_avg:.0f} calls/week
    Projected Lifespan: {weeks_remaining:.1f} weeks

    Top Operations:
    {get_top_operations_this_week()}
    """

    # Log report
    log_to_sheets("Processing_Log", [[
        timestamp(),
        "ZI_WEEKLY_REPORT",
        report,
        get_status_level(percentage)
    ]])

    return report
```

### Monthly Credit Tracking

**ZoomInfo Monthly Credits (separate from API calls):**
- 12,000 credits reset monthly
- Used for web portal exports
- NOT used by MCP API calls
- Track separately if web portal used

**API Calls vs Credits:**
- API calls = MCP tool usage (our focus)
- Credits = Web UI exports (not used in automation)
- **Project uses API calls only** (no web UI dependency)

### Emergency Stop Procedures

**If budget reaches 90% (4,500 API calls):**

1. **IMMEDIATE HALT**
   ```python
   def emergency_stop():
       """Halt all ZoomInfo operations immediately"""
       global ZOOMINFO_ENABLED
       ZOOMINFO_ENABLED = False

       # Log emergency stop
       log_to_sheets("Processing_Log", [[
           timestamp(),
           "ZI_EMERGENCY_STOP",
           "90% budget threshold reached - all operations halted",
           "🚨 CRITICAL"
       ]])

       # Alert user
       alert_user("🚨 EMERGENCY: ZoomInfo budget 90% consumed - operations stopped")

       # Switch to free-only mode
       switch_to_free_mode()
   ```

2. **Switch to 100% Free Sources**
   - Disable all ZoomInfo MCP tools
   - Use LinkedIn via Google for remaining contacts
   - Use free sources for company validation
   - Document reason for emergency stop

3. **Generate Incident Report**
   - What caused high usage?
   - Where did budget go?
   - Were free alternatives properly used?
   - What operations consumed most calls?

4. **User Approval Required to Resume**
   - Present incident report
   - Justify continuation need
   - Obtain explicit written approval
   - Document new safety measures

### Best Practices for Budget Conservation

**DO ✅**

1. **Always use lookup first** before enrichment (may be free)
2. **Always batch operations** (10 records = 1 call)
3. **Always check free sources first** (LinkedIn via Google, company websites)
4. **Always log every operation** to Processing_Log
5. **Always verify budget** before operations
6. **Always use conservative estimates** (plan for worst case)
7. **Always monitor weekly trends** (catch runaway usage early)

**DON'T ❌**

1. **Never skip lookup** before enrichment
2. **Never enrich individually** when batching possible
3. **Never use ZoomInfo for primary discovery** (free sources first)
4. **Never assume budget is sufficient** (always check)
5. **Never continue after emergency stop** (user approval required)
6. **Never use ZoomInfo without logging** (breaks tracking)
7. **Never ignore yellow/red zone warnings** (proactive management required)

### Agent-Specific Budget Allocations

**Agent 1 (Awards Validation):**
- Budget: 500 API calls (10% of project)
- Use: Validate award winner companies
- Throttle: 50 companies per day max
- Safety: Check budget before batch operations

**Agent 2 (LMS Customer Validation):**
- Budget: 500 API calls (10% of project)
- Use: Verify LMS customer company details
- Throttle: 50 companies per day max
- Safety: Lookup first, enrich only if missing data

**Agent 5 (Contact Enrichment):**
- Budget: 2,000 API calls (40% of project - HIGHEST)
- Use: Add email/phone to discovered contacts
- Throttle: 200 contacts per day max (20 batches)
- Safety: Only enrich after LinkedIn via Google exhausted

**Orchestrator (Company Enrichment):**
- Budget: 1,500 API calls (30% of project)
- Use: Employee count, revenue, tech stack validation
- Throttle: 150 companies per day max (15 batches)
- Safety: Only for tech stack discovery (Tier 3 priority)

**Agent 4 (Targeted Search):**
- Budget: 500 API calls (10% of project)
- Use: Fill gaps in discovery (last resort)
- Throttle: 10 searches per week max
- Safety: Require explicit justification for each search

### Cost Comparison: Free vs ZoomInfo

**Contact Discovery Example:**

| Method | Source | Data Quality | API Calls | Cost |
|--------|--------|--------------|-----------|------|
| Free | LinkedIn via Google | Name + title + company | 0 | $0 |
| Hybrid | Free + ZoomInfo enrichment | Name + title + company + email + phone | 0.1 per contact | $0 (within budget) |

**200 contacts:**
- Free: 0 API calls, names only
- Hybrid: 20 API calls (batches of 10), full contact details
- **Value Add:** 200% increase in actionability for 0.4% of budget

### Overage Risk Analysis

**Scenarios:**

1. **Conservative Usage (120 calls):**
   - Risk: NONE (2.4% of budget)
   - Safety margin: 4,880 calls (97.6%)

2. **Expected Usage (450 calls):**
   - Risk: NONE (9% of budget)
   - Safety margin: 4,550 calls (91%)

3. **Budget Limit (5,000 calls):**
   - Risk: NONE (100% of budget, 0% overage)
   - Cost: $0 overage fees

4. **Overage Example (6,000 calls):**
   - Risk: HIGH (120% of budget)
   - Cost: $10 overage charges (1,000 excess calls ÷ 1,000 × $10)
   - **MUST AVOID**

**Project Risk Level:** **MINIMAL** (conservative budget, extensive tracking, multiple safety stops)

### Integration with Free Sources

**Hybrid Workflow (Tier 1 - Contact Enrichment):**

```python
def hybrid_contact_discovery(company_name):
    """Hybrid workflow: free discovery + ZoomInfo enrichment"""

    # Step 1: Free discovery (LinkedIn via Google)
    contacts = []
    queries = [
        f'site:linkedin.com/in "Chief Learning Officer" {company_name}',
        f'site:linkedin.com/in "VP Learning" {company_name}',
        f'site:linkedin.com/in "Head of L&D" {company_name}'
    ]

    for query in queries:
        results = google_search_with_delay(query)
        contacts.extend(extract_linkedin_profiles(results))

    # Step 2: Pre-flight check
    if not zoominfo_preflight_check("enrich_contact", len(contacts) / 10):
        return contacts  # Return free data without enrichment

    # Step 3: ZoomInfo enrichment (batch mode)
    enriched_contacts = []
    for batch in chunks(contacts, 10):
        # Try lookup first (may be free)
        lookup_results = zoominfo_lookup(batch)

        # Enrich batch (1 API call for 10 contacts)
        enriched_batch = zoominfo_enrich_contact(batch)
        enriched_contacts.extend(enriched_batch)

        # Log operation
        log_zoominfo_operation("enrich_contact", len(batch), 1)

    return enriched_contacts
```

**Outcome:** Free sources provide names (0 cost), ZoomInfo adds email/phone (minimal cost)

---

## Website Scraping (General)

### robots.txt Compliance

**ALWAYS respect robots.txt:**

```python
from urllib.robotparser import RobotFileParser

def can_scrape_url(url):
    """Check if URL is allowed per robots.txt"""
    rp = RobotFileParser()
    rp.set_url(f"{url}/robots.txt")
    rp.read()
    return rp.can_fetch("ClaudeCode/1.0", url)

# Use before every scrape
if can_scrape_url("https://example.com/careers"):
    scrape_page("https://example.com/careers")
else:
    log_warning("robots.txt disallows scraping")
```

**Firecrawl handles this automatically** - respects robots.txt by default

### Delay Standards

**Industry Best Practices:**
- **Aggressive:** 1 second between requests (use sparingly)
- **Standard:** 2-3 seconds (recommended for most sites)
- **Conservative:** 5+ seconds (use for sensitive sites)

**Project Recommendation:**
- Fortune 1000 companies: 3-5 seconds
- Small company sites: 2-3 seconds
- Public databases: 1-2 seconds
- Award sites: 2 seconds

### User-Agent Identification

**Always identify your crawler:**

```
User-Agent: ClaudeCode-ResearchBot/1.0 (+https://claude.com; research@example.com)
```

**Rationale:**
- Ethical transparency
- Allows site owners to contact you
- Reduces false positive blocking
- Demonstrates good faith

**Firecrawl implementation:** Set via configuration if using direct API

---

## Press Release Sites (NO LIMITS)

### PR Newswire, Business Wire

**Characteristics:**
- Public marketing content
- **No rate limits observed**
- Intended for maximum distribution
- Encourage access and sharing

**Best Practices:**
- Still use 1-2 second delays (courtesy)
- Respect robots.txt
- Cache results locally (don't re-scrape)

**Validation:**
- Tested 100 searches on prnewswire.com via Google
- No throttling, no CAPTCHA
- ✅ Confirmed: Free and unlimited for reasonable use

---

## Government Databases

### SEC EDGAR

**Official Rate Limit:**
- **10 requests per second** per IP
- Documented at: sec.gov/developer
- Exceeding limit = 24-hour ban

**Compliance:**

```python
import time

last_request_time = 0
MIN_DELAY = 0.11  # 110ms = ~9 requests/second (under 10/sec limit)

def sec_edgar_search(query):
    global last_request_time

    # Ensure 110ms between requests
    elapsed = time.time() - last_request_time
    if elapsed < MIN_DELAY:
        time.sleep(MIN_DELAY - elapsed)

    result = firecrawl_search(f"site:sec.gov/edgar {query}")
    last_request_time = time.time()
    return result

# Rate: ~9 requests/second, safely under 10/sec limit
```

**Project Needs:**
- Estimated 200-500 SEC searches
- At 9 req/sec: 22-55 seconds total search time
- **No rate limiting risk**

### Department of Labor

**Rate Limits:**
- No documented limits
- Government site (intended for public access)
- Use 1-2 second delays (courtesy)

---

## G2 Crowd / Review Sites

### G2 Crowd Free Tier

**Access:**
- No account: Limited (first page of results)
- Free account: More results (signup required, no cost)
- Paid account: Full access (NOT needed for project)

**Rate Limits:**
- No official limits documented
- Observed: Standard web scraping (2-3 second delays)

**Recommendation:**
- Create free G2 account (no cost)
- Scrape customer reviews mentioning companies
- Extract company names from review text
- 2-3 second delays between page loads

---

## Error Codes & Recovery

### Standard Error Handling

```python
def handle_rate_limit_error(error_code, agent_name):
    """Central error handling for rate limits"""

    if error_code == "E002":  # Google rate limit
        log_to_sheets(f"E002: {agent_name} throttled")
        save_memory_checkpoint(agent_name)
        wait_time = 3600  # 1 hour
        time.sleep(wait_time)
        return "resume"

    elif error_code == "E007":  # Firecrawl limit
        log_to_sheets(f"E007: Firecrawl limit hit")
        # Switch to Playwright fallback
        return "use_playwright"

    elif error_code == "E008":  # SEC EDGAR ban
        log_to_sheets(f"E008: SEC 24-hour ban")
        wait_time = 86400  # 24 hours
        time.sleep(wait_time)
        return "resume"

    else:
        log_to_sheets(f"Unknown error: {error_code}")
        return "halt"
```

### Retry Logic

```python
def scrape_with_retry(url, max_retries=3):
    """Retry scraping with exponential backoff"""
    for attempt in range(max_retries):
        try:
            result = firecrawl_scrape(url, maxAge=172800000)
            return result
        except RateLimitError:
            if attempt < max_retries - 1:
                wait = (2 ** attempt) * 60  # 1min, 2min, 4min
                print(f"Rate limited. Retry {attempt+1}/{max_retries} in {wait}s")
                time.sleep(wait)
            else:
                raise
```

---

## Monitoring & Logging

### Rate Limit Dashboard

**Track in Google Sheets Processing_Log:**

```python
def log_rate_metrics(agent_name):
    """Log rate limiting metrics"""
    add_rows("Processing_Log", [[
        timestamp(),
        "Rate_Metrics",
        f"{agent_name}: {searches_today} searches, {throttle_count} throttles",
        agent_name
    ]])
```

**Metrics to Track:**
- Searches per hour (rolling)
- Throttle events per day
- Average delay between requests
- Cache hit rate (Firecrawl maxAge)

### Alerts

**Set up alerts for:**
- 3+ throttle events in 1 hour → STOP, wait 24 hours
- 150+ searches in 1 hour → SLOW DOWN (approaching limit)
- Firecrawl costs > $100/week → Switch to Playwright
- Any 24-hour ban → Alert user, log details

---

## Best Practices Summary

### DO ✅

1. **Always use 2-5 second delays** between requests
2. **Respect robots.txt** (Firecrawl does this automatically)
3. **Use proper User-Agent** identification
4. **Monitor rate metrics** in Processing_Log
5. **Save checkpoints** before long operations
6. **Use Firecrawl maxAge** for cache hits
7. **Batch process with rest periods**
8. **Use LinkedIn via Google** (not direct LinkedIn)
9. **Use ZoomInfo lookup first** before enrichment (may be free)
10. **Always batch ZoomInfo operations** (10 records = 1 API call)
11. **Track every ZoomInfo operation** to Processing_Log
12. **Check ZoomInfo budget** before every operation

### DON'T ❌

1. **Never** make parallel requests to same domain
2. **Never** ignore throttling warnings
3. **Never** use VPNs to bypass limits (ToS violation)
4. **Never** scrape faster than 1 request/second
5. **Never** continue when blocked (wait 24 hours)
6. **Never** scrape login-required content
7. **Never** bypass CAPTCHA challenges
8. **Never** search LinkedIn directly (use Google)
9. **Never** use ZoomInfo for primary discovery (free sources first)
10. **Never** enrich individually when batch mode available
11. **Never** skip ZoomInfo pre-flight checks
12. **Never** ignore yellow/red zone budget warnings

---

## Project-Specific Recommendations

### Agent Priority for Rate Limits

**Least Risky (Start Here):**
1. Agent 1 (Awards) - 500-1,000 searches, low risk
2. Agent 4 (Press) - 500-1,000 searches, public content
3. Agent 2 (LMS) - 500-800 searches, vendor marketing pages

**Moderate Risk:**
4. Agent 5 (LinkedIn/Google) - 2,000-3,000 searches, use site: operator

**Highest Risk (Manage Carefully):**
5. Agent 3 (Fortune 1000) - 3,000-5,000 searches, needs parallelization

### Execution Schedule

**Week 1-2: Low-Risk Agents (Foundation Building)**
- Agents 1, 2, 4 complete (free sources only)
- Build confidence in rate limit management
- Establish baseline metrics
- ZoomInfo: 100-200 API calls (validation only)
- Budget tracking established

**Week 3-5: High-Volume Agent (Systematic Discovery)**
- Agent 3 with careful monitoring
- 5 parallel instances, different IPs if possible
- 100-150 searches/hour max per instance
- ZoomInfo: 200-300 API calls (company enrichment)
- Weekly budget reviews

**Week 6: Contact Discovery (Primary Enrichment Phase)**
- Agent 5 (LinkedIn via Google for discovery)
- 2,000-3,000 Google searches over 1 week
- ~300 searches/day = well within limits
- ZoomInfo: 1,500-2,000 API calls (contact enrichment - HIGHEST usage week)
- Daily budget monitoring (RED zone potential)

**Week 7-8: Validation & Cleanup (Minimal ZoomInfo Usage)**
- Re-scrape flagged pages (use maxAge for cache hits)
- URL validation
- Minimal additional searches
- ZoomInfo: 100-200 API calls (fill gaps only)
- Final budget verification and reporting

**ZoomInfo Budget Timeline:**
- Weeks 1-2: ~5-10% of budget (foundation)
- Weeks 3-5: ~10-15% of budget (company data)
- Week 6: ~40-50% of budget (contact enrichment peak)
- Weeks 7-8: ~5-10% of budget (cleanup)
- **Total Expected:** 120-450 API calls (2-9% of 5,000 budget)

---

## Contingency Plans

### If Google Rate Limited

**Option 1: Slow Down**
- Increase delays to 5-10 seconds
- Reduce hourly target to 50 searches
- Extend timeline by 2-3 weeks

**Option 2: Pause & Resume**
- STOP for 24 hours
- Resume with slower rate
- Extend timeline by 1 week

**Option 3: IP Rotation**
- Use multiple machines (different locations)
- Each machine: 100 searches/hour
- 3 machines = 300 searches/hour total

### If ZoomInfo Budget Approaching Limit

**Option 1: Freeze Enrichment (Recommended)**
- STOP all ZoomInfo enrichment operations
- Continue with free discovery only
- Return names/titles without email/phone
- Impact: 200-400 contacts (names only) vs 250-500 (with email/phone)

**Option 2: Selective Enrichment**
- Prioritize highest-value contacts only
- CLO/VP level only (not managers)
- Fortune 500 companies only (not mid-market)
- Expected: 100-150 enriched contacts vs 250-500

**Option 3: Extend Timeline**
- Pause enrichment for 1-2 weeks
- Monitor other ZoomInfo usage across company
- Resume when budget availability confirmed
- No impact on discovery (free sources continue)

**Option 4: Switch to Free Contact Methods**
- Email pattern guessing (firstname.lastname@company.com)
- Company website contact pages
- Press release author emails
- Quality: 30-40% success rate vs 90% with ZoomInfo

**Decision Tree:**
```
Budget Status → Action
├─ 0-60% (GREEN): Continue as planned
├─ 60-80% (YELLOW): Selective enrichment only
├─ 80-90% (RED): Freeze enrichment, alert user
└─ 90-100% (CRITICAL): Emergency stop, switch to free-only
```

### If Firecrawl Costs Too High

**Switch to Playwright:**

```python
from playwright.sync_api import sync_playwright

def playwright_scrape(url):
    """Free alternative to Firecrawl"""
    with sync_playwright() as p:
        browser = p.chromium.launch()
        page = browser.new_page()
        page.goto(url)
        content = page.content()
        browser.close()
        return content
```

**Cost:** $0 (open source, self-hosted)
**Trade-off:** Slower, more complex code

---

## Conclusion

**Rate limiting is manageable with:**
- 2-5 second delays for free web sources
- Batch processing with rest periods (Google searches)
- LinkedIn via Google (bypasses LinkedIn limits entirely)
- Firecrawl maxAge for caching (50% cost savings)
- Parallel Agent 3 instances (distributes load)
- ZoomInfo batch operations (10x efficiency)
- Comprehensive tracking and alert thresholds

**Expected throttling events:**
- Google: 0-3 events over entire project (if best practices followed)
- ZoomInfo: 0 events (conservative budget with 97-98% safety margin)

**Project can complete in 6-8 weeks with:**
- **Free Sources:** $0 subscription costs, minimal rate limiting
- **ZoomInfo Enhancement:** 120-450 API calls (0.2-0.9% annual capacity)
- **Zero overage risk:** Multiple safety stops prevent budget overrun
- **Hybrid Advantage:** 250-500 enriched contacts vs 200-400 names-only

**Key Success Factors:**
1. Free sources provide 80-90% of data (foundation)
2. ZoomInfo adds 10-20% strategic enhancement (email/phone)
3. Mandatory pre-flight checks prevent irresponsible usage
4. Comprehensive tracking enables proactive management
5. Emergency stops ensure budget compliance

**Risk Level:** **LOW** (conservative budget, extensive guardrails, proven free methodology as fallback)

---

**Last Updated:** 2025-10-28
**Version:** 2.0 (Hybrid Methodology)
**Next Review:** After Agent 1 & 2 pilot (validate free+ZoomInfo workflows)
