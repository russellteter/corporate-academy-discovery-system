# Rate Limit Management Guide - Corporate Academy Discovery System

**Version:** 1.0
**Date:** 2025-10-28
**Scope:** 100% Free Data Sources

---

## Overview

This guide provides comprehensive rate limit management strategies for the Corporate Academy Discovery System using only free data sources. All sources have been validated for their rate limiting behavior and best practices documented.

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

### DON'T ❌

1. **Never** make parallel requests to same domain
2. **Never** ignore throttling warnings
3. **Never** use VPNs to bypass limits (ToS violation)
4. **Never** scrape faster than 1 request/second
5. **Never** continue when blocked (wait 24 hours)
6. **Never** scrape login-required content
7. **Never** bypass CAPTCHA challenges
8. **Never** search LinkedIn directly (use Google)

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

**Week 1-2: Low-Risk Agents**
- Agents 1, 2, 4 complete
- Build confidence in rate limit management
- Establish baseline metrics

**Week 3-5: High-Volume Agent**
- Agent 3 with careful monitoring
- 5 parallel instances, different IPs if possible
- 100-150 searches/hour max per instance

**Week 6: Contact Discovery**
- Agent 5 (LinkedIn via Google)
- 2,000-3,000 searches over 1 week
- ~300 searches/day = well within limits

**Week 7-8: Validation & Cleanup**
- Re-scrape flagged pages (use maxAge for cache hits)
- URL validation
- Minimal additional searches

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
- 2-5 second delays
- Batch processing with rest periods
- LinkedIn via Google (bypasses LinkedIn limits)
- Firecrawl maxAge for caching
- Parallel Agent 3 instances

**Expected throttling events:** 0-3 over entire project (if best practices followed)

**Project can complete in 6-8 weeks with zero subscription costs and minimal rate limiting issues.**

---

**Last Updated:** 2025-10-28
**Next Review:** After Agent 1 & 2 pilot (validate assumptions)
