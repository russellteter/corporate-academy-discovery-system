# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Corporate Academy Discovery System** - A multi-agent research automation system designed to discover, validate, and catalog 500+ corporate academies/universities at large enterprises (5,000+ employees). The system uses 5 specialized research agents operating in parallel, coordinated by an orchestrator, with Google Sheets as the central data store.

**Key Characteristics:**
- Multi-agent parallel execution architecture
- Google Sheets MCP server for data persistence (FREE tier)
- Automated research, validation, and deduplication workflows
- Target: 500+ academies with 60% CONFIRMED confidence, 200+ contacts
- 6-8 week automated execution timeline
- **100% FREE DATA SOURCES** - No paid subscriptions required (Apollo.io, ZoomInfo, etc.)

## System Architecture

### Multi-Agent Structure

```
Orchestrator (Monitoring & Merge)
├── Agent 1: Awards Mining (Brandon Hall, Training Top 125, CLO, ATD BEST)
├── Agent 2: LMS Reverse Engineering (Cornerstone, Docebo, SAP, Workday)
├── Agent 3: Website Discovery (8,000+ company universe systematic search)
├── Agent 4: Press Release Mining (PR Newswire, Business Wire, news archives)
└── Agent 5: LinkedIn Intelligence (Profile search, job postings, contact discovery)
```

### Data Flow

1. **Input**: Company_Universe sheet (8,000+ companies to research)
2. **Processing**: Each agent writes discoveries to dedicated sheet
3. **Orchestration**: Orchestrator merges duplicates, elevates confidence
4. **Output**: Consolidated_Academies + Contacts_Master sheets

## Google Sheets Configuration

**Required Sheets** (9 total):
1. `Consolidated_Academies` - Master output (merged, validated)
2. `Awards_Discoveries` - Agent 1 raw output
3. `LMS_Discoveries` - Agent 2 raw output
4. `Website_Discoveries` - Agent 3 raw output
5. `Press_Discoveries` - Agent 4 raw output
6. `LinkedIn_Discoveries` - Agent 5 raw output
7. `Contacts_Master` - All L&D contacts discovered
8. `Company_Universe` - Input list (companies to research)
9. `Processing_Log` - System events and agent status

**Google Sheets MCP Access:**
- Spreadsheet ID: Extract from URL after creating sheets
- Permissions: Edit access required for automated writes
- API: Use `mcp__google-sheets__*` tools for all read/write operations

## Core Data Schemas

### Academy Record (Primary)
```
Academy_ID (String, Required) - Unique: AC-0001, AC-0002...
Academy_Name (String, Required) - Exact branded name
Parent_Company (String, Required) - Parent corporation
Parent_Domain (String, Required) - Company domain
Academy_URL (URL, Optional) - Direct link to academy site
Discovery_Source (String, Required) - Source(s): "Awards Agent, LMS Agent"
Confidence_Score (Enum, Required) - CONFIRMED / HIGH / MEDIUM
Evidence_Description (Text, Required) - Summary of all evidence
LMS_Platform (String, Optional) - Cornerstone, Docebo, SAP, etc.
VILT_Platform (String, Optional) - Zoom, Microsoft Teams, etc.
Contact_Count (Integer, Optional) - Number of contacts found
Discovery_Date (Date, Required) - ISO 8601: 2025-10-28
```

### Contact Record
```
Contact_ID (String, Required) - Unique: C-0001, C-0002...
Academy_ID (String, Required) - Links to academy
Academy_Name (String, Required) - For reference
Parent_Company (String, Required) - For reference
Contact_Name (String, Required) - Full name
Contact_Title (String, Required) - Job title
Contact_LinkedIn_URL (URL, Optional) - LinkedIn profile
Discovery_Source (String, Required) - Which agent found
Discovery_Date (Date, Required) - ISO 8601
```

## Critical Search Terminology

**ALL agents MUST search these three terms equally:**
- "corporate university" / "[Company] University"
- "corporate academy" / "[Company] Academy"
- "learning institute" / "[Company] Institute"

**Standard Search Pattern:**
```
("corporate university" OR "corporate academy" OR "learning institute")
OR
("[Company] University" OR "[Company] Academy" OR "[Company] Institute")
OR
(inurl:university OR inurl:academy OR inurl:institute)
```

## Agent Execution Specifications

### Agent 1: Awards & Recognition Mining (Priority: HIGH)
- **Sources**: Brandon Hall Excellence Awards, Training Magazine Top 125, CLO LearningElite, ATD BEST Awards (ALL FREE)
- **Methodology**: Extract award winners from public winner lists → validate academy exists → capture details + CLO names
- **Expected Yield**: 300-400 academies, 50-100 CLO contacts
- **Stop**: All 4 sources + archives (2020-2025) searched OR 400 academies found
- **100% FREE**: All award sites publish winner lists publicly

**⚠️ ZOOMINFO SAFETY CHECK (MANDATORY):**

**Before ANY ZoomInfo usage in Agent 1:**
1. ✅ Verify award program data exhausted
2. ✅ Check Processing_Log running total < 200,000 API calls
3. ✅ Calculate planned API calls for validation
4. ✅ Confirm: running_total + planned_calls < 250,000
5. ✅ Log operation to Processing_Log before execution

**ZoomInfo Usage Rules for Agent 1:**
- ✅ **APPROVED**: Validate award winner company details (employee count, industry)
- ✅ **APPROVED**: Enrich CLO contacts found in award profiles
- ✅ **APPROVED**: Comprehensive enrichment of all award winners
- **Budget Allocation**: Up to 25,000 API calls (10% of project budget)

**MCP Tool Stack:**
1. `firecrawl_search` - Find award pages: `"Brandon Hall Excellence Awards 2023 corporate university"`
2. `firecrawl_scrape` - Extract winner details from award pages
3. `firecrawl_map` - Discover academy URLs on company domains
4. `firecrawl_scrape` - Validate academy website with `maxAge` for caching
5. `mcp__google-sheets__add_rows` - Write to Awards_Discoveries sheet immediately
6. **OPTIONAL**: `mcp__zoominfo__enrich_company` - Only for high-priority winners

**Workflow Example:**
```
1. firecrawl_search("Brandon Hall 2023 corporate university winners")
2. Extract company names and academy mentions
3. For each company:
   - firecrawl_map("https://[company-domain].com", search="university OR academy")
   - firecrawl_scrape(academy_url, maxAge=172800000)
   - Validate: Is this a real corporate academy? (use Sequential if uncertain)
   - add_rows(spreadsheet_id, "Awards_Discoveries", [[academy_data]])
4. Log to Processing_Log every 10 discoveries
```

### Agent 2: LMS Vendor Customer Mining (Priority: HIGH)
- **Sources**: Cornerstone OnDemand, Docebo, SAP SuccessFactors, Workday, G2 reviews (FREE customer showcase pages)
- **Methodology**: Scrape public customer case studies → validate academy → capture tech stack
- **Expected Yield**: 200-300 academies
- **Stop**: All major LMS vendors searched OR 300 academies found
- **100% FREE**: All vendor customer pages are public marketing content

**⚠️ ZOOMINFO SAFETY CHECK (MANDATORY):**

**Before ANY ZoomInfo usage in Agent 2:**
1. ✅ Verify LMS vendor pages checked first
2. ✅ Check Processing_Log running total < 200,000 API calls
3. ✅ Calculate planned API calls for tech stack discovery
4. ✅ Confirm: running_total + planned_calls < 250,000
5. ✅ Log operation to Processing_Log before execution

**ZoomInfo Usage Rules for Agent 2:**
- ✅ **APPROVED**: Comprehensive tech stack discovery for all academies
- ✅ **APPROVED**: Cross-reference and validate LMS platform data
- ✅ **APPROVED**: Identify VILT platforms and training technologies
- **Budget Allocation**: Up to 50,000 API calls (20% of project budget)

**MCP Tool Stack:**
1. `firecrawl_search` - Find LMS customer pages: `"Cornerstone OnDemand customers corporate university"`
2. `firecrawl_scrape` - Extract customer case studies
3. `firecrawl_map` - Find academy pages on customer domains
4. `firecrawl_extract` - Extract structured tech stack data (LMS, VILT platforms)
5. `mcp__google-sheets__add_rows` - Write to LMS_Discoveries with tech stack
6. **OPTIONAL**: `mcp__zoominfo__search_companies` - Only for tech stack validation

**Workflow Example:**
```
1. firecrawl_search("Docebo customer success stories corporate academy")
2. firecrawl_scrape(customer_page_url)
3. Extract: Company name, academy mentions, tech platforms mentioned
4. For each customer:
   - firecrawl_map("https://[company].com", search="university OR academy OR institute")
   - firecrawl_scrape(academy_url)
   - firecrawl_extract(academy_url, schema={academy_name, programs, lms_platform})
   - add_rows(spreadsheet_id, "LMS_Discoveries", [[academy_data_with_lms]])
5. IMPORTANT: Always populate LMS_Platform field (e.g., "Docebo")
```

### Agent 3: Website Discovery (Priority: CRITICAL)
- **Input**: Read from Company_Universe sheet
- **Methodology**: Batch process 20 companies → 3 search patterns per company
- **Time Budget**: 3-5 minutes per company maximum
- **Expected Yield**: 400-600 academies from 8,000-10,000 companies
- **Stop**: All companies processed OR 600 academies found

**MCP Tool Stack:**
1. `mcp__google-sheets__get_sheet_data` - Read Company_Universe in batches (100 rows)
2. `firecrawl_map` - Map each company domain for learning/training pages
3. `firecrawl_scrape` - Validate academy pages found
4. `Sequential` (if needed) - Determine if content qualifies as academy
5. `mcp__google-sheets__add_rows` - Write to Website_Discoveries
6. `mcp__google-sheets__batch_update_cells` - Mark companies as "Completed" in Company_Universe

**Workflow Example:**
```
1. get_sheet_data(spreadsheet_id, "Company_Universe", "A2:F101") # Batch of 100
2. For each company:
   - firecrawl_map(company_domain, search="university OR academy OR institute OR learning")
   - If URLs found:
     * firecrawl_scrape(each_url, maxAge=172800000)
     * Analyze: Branded academy? Programmatic? (Sequential if ambiguous)
     * Extract: academy_name, programs, contacts, evidence
     * add_rows(spreadsheet_id, "Website_Discoveries", [[academy_data]])
   - Else: Log "No academy found"
3. batch_update_cells(spreadsheet_id, "Company_Universe",
     ranges={"E2:E101": [["Completed"], ["Completed"], ...]})
4. Log batch completion to Processing_Log
5. Repeat with next 100 companies
```

### Agent 4: Press Mining (Priority: MEDIUM)
- **Sources**: PR Newswire, Business Wire, Google News (2018-2025)
- **Methodology**: Search press releases → validate academy still active
- **Expected Yield**: 60-100 academies
- **Stop**: 10+ searches with no findings OR 100 academies found

**MCP Tool Stack:**
1. `firecrawl_search` - Search press release sites with date filters
2. `firecrawl_scrape` - Extract press release details
3. `firecrawl_map` - Validate academy still exists on company site
4. `mcp__google-sheets__add_rows` - Write to Press_Discoveries

**Workflow Example:**
```
1. firecrawl_search("site:prnewswire.com launches corporate university 2020..2025")
2. For each press release:
   - firecrawl_scrape(press_url)
   - Extract: Company, academy name, launch date, quoted executives
   - Validate still active: firecrawl_map(company_domain, search="[academy_name]")
   - If active: Confidence=HIGH, if not: Confidence=MEDIUM, note "may be defunct"
   - add_rows(spreadsheet_id, "Press_Discoveries", [[academy_data_with_launch_date]])
3. Capture contacts from press release (media contacts, quoted execs)
4. Write contacts to Contacts_Master
```

### Agent 5: LinkedIn Intelligence via Google (Priority: HIGH - Contacts)
- **Primary Mission**: Contact discovery (200+ contacts minimum)
- **Methodology**: Search LinkedIn via Google (bypasses rate limits), job postings, profile discovery
- **KEY TECHNIQUE**: Use Google `site:linkedin.com` searches instead of direct LinkedIn access
- **Expected Yield**: 100+ academies, 200+ contacts
- **Stop**: 300 contacts captured OR exhausted search variations
- **100% FREE**: No LinkedIn subscription required

**⚠️ ZOOMINFO SAFETY CHECK (MANDATORY):**

**Before ANY ZoomInfo usage in Agent 5:**
1. ✅ Use LinkedIn via Google for initial discovery baseline
2. ✅ Check Processing_Log running total < 200,000 API calls
3. ✅ Calculate planned API calls: [contacts to enrich] ÷ 10 = [batches]
4. ✅ Confirm: running_total + planned_calls < 250,000
5. ✅ Log planned operation to Processing_Log before execution

**ZoomInfo Usage Rules for Agent 5:**
- ✅ **APPROVED**: PRIMARY - Mass contact enrichment (5,000-20,000 contacts)
- ✅ **APPROVED**: Targeted contact discovery searches (1,000+ searches)
- ✅ **APPROVED**: Hard-to-find roles (Dean, CLO, VP Learning)
- ✅ **APPROVED**: Comprehensive L&D leadership contact database building
- **Budget Allocation**: Up to 125,000 API calls (50% of project budget)

**MCP Tool Stack:**
1. `firecrawl_search` - Search Google with `site:linkedin.com` patterns
2. `firecrawl_scrape` - Extract profile/job details from LinkedIn public pages
3. `firecrawl_map` - Validate academy on company site
4. `mcp__google-sheets__add_rows` - Write to BOTH LinkedIn_Discoveries AND Contacts_Master
5. `mcp__memory__create_entities` - Store contacts for cross-session persistence
6. **OPTIONAL**: `mcp__zoominfo__enrich_contact` - Only AFTER free sources exhausted

**LinkedIn via Google Search Patterns:**
```
site:linkedin.com/in "Chief Learning Officer" [Company]
site:linkedin.com/in "VP Learning" OR "Director Learning" [Company]
site:linkedin.com/in "Dean" "[Company] University"
site:linkedin.com/in "[Company] Academy" OR "[Company] University"
site:linkedin.com/jobs "corporate university" OR "corporate academy"
site:linkedin.com/jobs "[Company]" "learning" "director"
```

**Workflow Example:**
```
1. Google search via Firecrawl (NOT direct LinkedIn):
   - firecrawl_search('site:linkedin.com/in "Chief Learning Officer" Walmart')
   - firecrawl_search('site:linkedin.com/in "Dean" "Walmart University"')
   - firecrawl_search('site:linkedin.com/jobs "corporate university"')
2. For each profile found:
   - firecrawl_scrape(linkedin_public_profile_url)
   - Extract: Name, title, company, academy mentions
   - Validate academy: firecrawl_map(company_domain, search="[academy_name]")
   - add_rows(spreadsheet_id, "Contacts_Master", [[contact_data]])
   - If academy found: add_rows(spreadsheet_id, "LinkedIn_Discoveries", [[academy_data]])
3. For job postings:
   - Extract: Company, job title, academy name, tech platforms mentioned
   - add_rows for both academy and contacts
4. Rate Management:
   - Google rate limits (not LinkedIn) - 2-3 second delays between searches
   - If throttled: Wait 600 seconds, resume
   - Use memory MCP to persist progress: create_entities(processed_companies)
```

**Why This Works:**
- LinkedIn profile pages are public and indexed by Google
- Searching Google for `site:linkedin.com` bypasses LinkedIn's rate limits
- No LinkedIn login or subscription required
- Google allows ~100-200 searches/hour (sufficient for project)
- LinkedIn job postings are public and Google-indexed

## Orchestrator Responsibilities

**⚠️ ZOOMINFO SAFETY CHECK (MANDATORY):**

**Before ANY ZoomInfo usage in Orchestrator:**
1. ✅ Verify company/contact data not available via free sources
2. ✅ Check Processing_Log running total < 200,000 API calls
3. ✅ Calculate planned API calls: [records to enrich] ÷ 10 = [batches]
4. ✅ Confirm: running_total + planned_calls < 250,000
5. ✅ Prioritize CONFIRMED academies over MEDIUM confidence
6. ✅ Log all operations to Processing_Log immediately

**ZoomInfo Usage Rules for Orchestrator:**
- ✅ **APPROVED**: Batch enrich Consolidated_Academies (company validation)
- ✅ **APPROVED**: Enrich Contacts_Master (email/phone for known contacts)
- ✅ **APPROVED**: Tech stack validation for CONFIRMED academies
- ✅ **APPROVED**: Comprehensive company data validation (revenue, employee count, industry)
- ✅ **APPROVED**: Duplicate detection and merge operations using ZoomInfo data
- ❌ **PROHIBITED**: Re-enriching already-validated records
- **Budget Allocation**: Up to 50,000 API calls (20% of project budget)

**Monitoring Loop (Every 5 Minutes):**
1. Scan for new records in all agent sheets
2. Detect duplicates using fuzzy matching (case-insensitive, name variations)
3. Merge duplicates into Consolidated_Academies
4. Auto-elevate confidence scores (2 agents → HIGH, 3+ agents → CONFIRMED)
5. Link contacts to academies (update Academy_ID, Contact_Count)
6. Log dashboard statistics
7. **Check ZoomInfo usage: Alert if running_total > 4,000 API calls**

**Duplicate Merging Logic:**
- Combine Discovery_Source from all agents
- Merge all Evidence_Description fields
- Use most complete Academy_Name, Academy_URL
- Take LMS/VILT Platform from any agent that found it
- Apply confidence boosting rules

**Auto-Elevation Rules:**
- Multiple evidence (3+ sources) + 10,000+ employees → HIGH
- 3+ LinkedIn profiles mention same academy → HIGH
- Press release + active website → HIGH
- Tech stack identified + website → HIGH
- If 2 agents found same academy → Upgrade MEDIUM → HIGH
- If 3+ agents found same academy → Upgrade HIGH → CONFIRMED

## Confidence Score Framework

**CONFIRMED (Target: 60% / 300+ records):**
- 3+ independent sources validate academy exists
- Example: Award + LMS case study + Dedicated website

**HIGH (Target: 30% / 150+ records):**
- 2 strong sources validate academy
- Example: LMS case study + Website validation

**MEDIUM (Target: 10% / 50+ records):**
- 1-2 weaker sources suggest academy
- Example: LinkedIn mentions (1-2 profiles) + size threshold

## Google Sheets MCP Operations

**Reading Data:**
```
get_sheet_data(spreadsheet_id, sheet="Company_Universe", range="A1:F1000")
```

**Writing Discoveries:**
```
add_rows(spreadsheet_id, sheet="Awards_Discoveries", data=[
  ["AC-0001", "Hamburger University", "McDonald's Corporation", ...]
])
```

**Batch Updates (Orchestrator):**
```
batch_update_cells(spreadsheet_id, sheet="Consolidated_Academies", ranges={
  "A2:A10": [[academy_ids]],
  "N2:N10": [[contact_counts]]
})
```

**Logging:**
```
add_rows(spreadsheet_id, sheet="Processing_Log", data=[
  ["2025-10-28 14:32:18", "Batch_Complete", "20 companies processed, 5 academies found", "Agent_3"]
])
```

## Error Handling

**Standard Error Codes:**
- **E001**: Website 404/not found → Skip, note in Evidence
- **E002**: Rate limit hit → Wait 1 hour, retry
- **E003**: Unable to parse academy name → Best guess + flag
- **E004**: Spreadsheet write failed → Retry 3x
- **E005**: Search returned zero results → Move to next search
- **E006**: Academy appears defunct → Mark LOW confidence

**Critical Errors (Stop Execution):**
- E004 fails after 3 retries → Halt agent, alert
- Google Sheets API access denied → Halt all agents
- Company_Universe sheet missing → Halt Agent 3

## MCP Server Research Tools

This project leverages multiple MCP servers for automated research. Here's how to use each for corporate academy discovery:

### Firecrawl MCP (Primary Web Scraping)

**Purpose**: Advanced web scraping, content extraction, and website mapping for academy discovery and validation.

**Available Tools:**
1. **firecrawl_scrape** - Single page content extraction (FASTEST, most reliable)
2. **firecrawl_search** - Web search with content extraction
3. **firecrawl_map** - Discover all URLs on a website
4. **firecrawl_crawl** - Multi-page crawling with depth control
5. **firecrawl_extract** - Structured data extraction using LLM

**When to Use Each:**

**Use `firecrawl_scrape` for:**
- Extracting academy website content after finding URL
- Validating academy pages exist (HTTP 200 check)
- Pulling program details, leadership, curriculum info
- Single-page content needs

```json
{
  "name": "mcp__MCP_DOCKER__firecrawl_scrape",
  "arguments": {
    "url": "https://corporate.mcdonalds.com/hamburger-university",
    "formats": ["markdown"],
    "maxAge": 172800000
  }
}
```

**Use `firecrawl_search` for:**
- Finding academy mentions across the web
- Discovering press releases and news articles
- Locating award announcements
- General web research when URL unknown

```json
{
  "name": "mcp__MCP_DOCKER__firecrawl_search",
  "arguments": {
    "query": "McDonald's Hamburger University corporate training",
    "limit": 10,
    "sources": [{"type": "web"}],
    "scrapeOptions": {
      "formats": ["markdown"],
      "onlyMainContent": true
    }
  }
}
```

**Use `firecrawl_map` for:**
- Discovering all pages on a company domain
- Finding hidden academy pages or subdomains
- Identifying learning/training section URLs
- Before deciding what to scrape

```json
{
  "name": "mcp__MCP_DOCKER__firecrawl_map",
  "arguments": {
    "url": "https://mcdonalds.com",
    "search": "university OR academy OR learning OR training"
  }
}
```

**Use `firecrawl_crawl` for:**
- Extracting content from multiple related pages
- When comprehensive coverage needed
- WARNING: Can be slow and token-heavy - use limit/maxDepth carefully

```json
{
  "name": "mcp__MCP_DOCKER__firecrawl_crawl",
  "arguments": {
    "url": "https://corporate.mcdonalds.com/hamburger-university/*",
    "maxDiscoveryDepth": 2,
    "limit": 10,
    "deduplicateSimilarURLs": true
  }
}
```

**Use `firecrawl_extract` for:**
- Extracting structured academy data (name, programs, contacts)
- Pulling specific fields from academy pages
- When you need consistent data format

```json
{
  "name": "mcp__MCP_DOCKER__firecrawl_extract",
  "arguments": {
    "urls": ["https://corporate.mcdonalds.com/hamburger-university"],
    "schema": {
      "type": "object",
      "properties": {
        "academy_name": {"type": "string"},
        "programs": {"type": "array"},
        "leadership": {"type": "array"}
      }
    }
  }
}
```

**Best Practices:**
- Always use `maxAge` parameter with scrape for 500% faster cached results
- Use `firecrawl_search` without formats first, then scrape specific URLs
- Limit crawl depth (maxDiscoveryDepth: 2-3) to avoid token overflow
- Use `firecrawl_map` before `firecrawl_crawl` for better control

### GitHub MCP (Press Release & Job Mining)

**Purpose**: Search code repositories, job postings, and organizational content for academy mentions.

**Relevant Tools:**
1. **search_code** - Search across ALL GitHub repositories
2. **search_repositories** - Find company repositories
3. **get_file_contents** - Read specific files (job descriptions, docs)

**Use Cases:**

**Finding Job Postings:**
```json
{
  "name": "mcp__github__search_code",
  "arguments": {
    "query": "corporate university director OR corporate academy dean path:careers",
    "perPage": 20
  }
}
```

**Finding Academy Documentation:**
```json
{
  "name": "mcp__github__search_repositories",
  "arguments": {
    "query": "corporate training academy in:description stars:>10",
    "perPage": 20
  }
}
```

**Best Practices:**
- Use language filters: `language:markdown` for documentation
- Path filters: `path:careers OR path:jobs` for postings
- Combine with company names: `org:mcdonalds academy`

### Notion MCP (Project Management)

**Purpose**: Track research progress, manage findings, coordinate agents (if using Notion instead of Sheets).

**Available Tools:**
- `post-search` - Search Notion workspace
- `post-page` - Create new pages for discoveries
- `post-database-query` - Query academy database
- `patch-block-children` - Add findings to pages

**Use Case:** If you prefer Notion over Google Sheets for data storage, can be configured as alternative.

### ZoomInfo MCP (Optional Acceleration)

**Purpose**: Rapid B2B contact and company data enrichment for accelerated research workflows.

**Status**: ✅ Configured and ready (requires OAuth authentication via `/mcp`)

**⚠️ CRITICAL: Usage Limits & Guardrails ⚠️**

**Subscription Details (Contract: 7/10/2025 - 7/9/2026):**
- **12,000 Monthly Recurring Credits** (12 users × 1,000/user)
  - Resets monthly, non-transferable, no rollover
- **100,000 Bulk Data Credits** (one-time, entire contract)
- **2,500,000 API Calls** (annual limit, 10x database size)
  - Fair Use: 250,000 records × 10 = 2.5M calls
  - **OVERAGE COST: $10,000 per 1 million excess API calls**

**Per-User Monthly Limits (Advanced+ Edition):**
- 2,000 Company/Contact views per user
- 100,000 Intent exports per user
- 500,000 bulk export per action

**PROJECT USAGE GUARDRAILS (MANDATORY):**

```yaml
zoominfo_usage_limits:
  # HARD LIMITS - Never Exceed
  max_api_calls_per_session: 5000       # Daily safety limit
  max_api_calls_per_week: 25000         # Weekly safety limit
  max_api_calls_project_total: 250000   # Total for entire project (10% of annual capacity)

  # SAFE MONTHLY BUDGET
  safe_monthly_api_calls: 200000        # Well below 2.5M annual / 12
  safe_monthly_credits: 10000           # Leave 2000 buffer for other projects

  # BATCH LIMITS
  max_enrichment_batch_size: 10         # MCP server limit
  max_search_results_per_query: 100     # Allow deeper pagination

  # USAGE TRACKING (REQUIRED)
  log_every_api_call: true              # Must log to Processing_Log
  check_usage_before_batch: true        # Weekly admin portal verification
  alert_threshold: 200000               # Alert at 80% of project limit
```

**MANDATORY PRE-FLIGHT CHECKS:**

Before ANY ZoomInfo operation, verify:
1. **Check Running Total**: Read last entry in Processing_Log ZoomInfo_Usage column
2. **Verify Budget**: Ensure running_total + planned_calls < 250,000
3. **Confirm Necessity**: Can free sources (LinkedIn via Google, awards, LMS vendors) provide this data?
4. **Use Lookup First**: Always call `lookup` before `search_*` to avoid wasted API calls
5. **Weekly Review**: Check admin portal to verify tracking accuracy

**APPROVED USE CASES (Priority Order):**

✅ **Tier 1 - Contact Enrichment (Highest Priority)**
- Enrich already-found contacts with email/phone
- Batch enrichment (10 contacts per call)
- Budget: Up to 100,000 API calls
- Estimated: 5,000-20,000 contacts = 500-2,000 API calls

✅ **Tier 2 - Targeted Contact Discovery**
- Search for hard-to-find roles (Dean, VP Learning, CLO)
- Accelerate Agent 5 contact discovery
- Budget: Up to 80,000 API calls
- Estimated: 1,000-5,000 searches = 1,000-5,000 API calls

✅ **Tier 3 - Company Validation**
- Verify employee counts for academies in Consolidated_Academies
- Validate revenue, industry classification, tech stacks
- Budget: Up to 50,000 API calls
- Estimated: 500-5,000 companies = 50-500 API calls

✅ **Tier 4 - Tech Stack Discovery**
- Identify LMS/VILT platforms for confirmed academies
- Cross-reference with Agent 2 findings
- Budget: Up to 20,000 API calls
- Estimated: 500-2,000 companies = 500-2,000 API calls

❌ **PROHIBITED USE CASES:**
- ❌ Repeated enrichment of same records (check cache first)
- ❌ Exploratory searches without specific criteria (use lookup first)
- ❌ Operations without logging to Processing_Log

**COST-AWARE WORKFLOW PATTERN:**

```
1. FREE SOURCES FIRST (Foundation):
   - Awards programs (Agent 1): 300-400 academies
   - LMS vendors (Agent 2): 200-300 academies
   - Website discovery (Agent 3): 400-600 academies
   - Press releases (Agent 4): 60-100 academies
   - LinkedIn via Google (Agent 5): 200+ contacts baseline

2. ZOOMINFO ACCELERATION (Primary Data Source):
   - APPROVED: Mass contact enrichment (5,000-20,000 contacts)
   - APPROVED: Targeted contact discovery (1,000+ searches)
   - APPROVED: Comprehensive company validation (500-5,000 companies)
   - APPROVED: Full tech stack discovery (500-2,000 companies)

3. TRACK EVERY CALL:
   add_rows(spreadsheet_id, "Processing_Log", [[
     timestamp,
     "ZoomInfo",
     tool_name,
     records_affected,
     estimated_api_calls,
     running_total,
     remaining_budget
   ]])
```

**USAGE MONITORING (REQUIRED):**

**Daily Checks:**
- Review Processing_Log ZoomInfo entries
- Calculate running total API calls
- No strict daily limit - monitor trends

**Weekly Checks:**
- Log into admin portal: https://admin.zoominfo.com/#/api-and-webhooks
- Check "API Usage" dashboard
- Verify total calls reasonable progress toward goals
- Check remaining monthly recurring credits (target: use 25-50% = 3,000-6,000)

**Monthly Checks:**
- Review total API calls used (target: 10,000-50,000 for project scale)
- Calculate remaining annual capacity: 2,500,000 - total_used
- Document findings in Project Status report
- Verify staying within 250,000 project cap (10% of annual)

**OVERAGE PREVENTION:**

**Automatic Safeguards:**
```python
# Pseudo-code for every ZoomInfo operation
def before_zoominfo_call(tool_name, estimated_calls):
    running_total = get_latest_running_total("Processing_Log")

    if running_total + estimated_calls > 250000:
        ALERT("STOP: Project limit (10% of annual) would be exceeded!")
        ALERT(f"Current: {running_total}, Planned: {estimated_calls}, Limit: 250000")
        return ABORT

    if running_total + estimated_calls > 200000:
        WARN("Approaching 80% of project budget - review if on track")

    return PROCEED

def after_zoominfo_call(tool_name, actual_calls):
    new_total = running_total + actual_calls
    log_to_processing_log(tool_name, actual_calls, new_total, 250000 - new_total)

    if new_total >= 200000:
        ALERT("NOTICE: 80% of project budget used - verify on track for goals")
```

**AUTHENTICATION:**
1. In Claude Code, type `/mcp`
2. Select "Authenticate" next to ZoomInfo
3. Browser opens for OAuth login
4. Grant permissions and return to Claude Code
5. Tools become available as `mcp__zoominfo__*`

**AVAILABLE TOOLS:**

**Must Use First:**
- `mcp__zoominfo__lookup` - Get valid field values (ALWAYS call before search)

**Search Tools (Use Sparingly):**
- `mcp__zoominfo__search_contacts` - Contact search (1+ API call per page)
- `mcp__zoominfo__search_companies` - Company search (1+ API call per page)

**Enrichment Tools (Preferred):**
- `mcp__zoominfo__enrich_contact` - Enrich contacts (1 API call per batch of 10)
- `mcp__zoominfo__enrich_company` - Enrich companies (1 API call per batch of 10)

**EXAMPLE USAGE WITH GUARDRAILS:**

```json
// STEP 1: Check budget
check_running_total("Processing_Log") // Returns: 245 API calls used

// STEP 2: Use lookup first (minimal API cost)
{
  "name": "mcp__zoominfo__lookup",
  "arguments": {
    "fieldNames": ["management-levels", "job-functions"]
  }
}

// STEP 3: Targeted enrichment (batch of 10 = 1 API call)
{
  "name": "mcp__zoominfo__enrich_contact",
  "arguments": {
    "contacts": [
      {"email": "john.smith@company.com"},
      {"email": "jane.doe@company.com"},
      // ... 8 more (total 10)
    ]
  }
}
// Cost: 1 API call for 10 enrichments

// STEP 4: Log usage immediately
add_rows("Processing_Log", [[
  "2025-10-28 14:30:00",
  "ZoomInfo",
  "enrich_contact",
  10,
  1,
  246,
  4754
]])
```

**INTEGRATION WITH AGENTS:**

**Agent 1 (Awards Mining):**
- ✅ Validate award winner company details (company enrichment)
- ❌ Do NOT use for primary discovery

**Agent 2 (LMS Vendors):**
- ✅ Cross-reference tech stack data (targeted validation)
- ❌ Do NOT replace LMS vendor customer pages

**Agent 5 (LinkedIn Intelligence):**
- ✅ Enrich contacts found via LinkedIn via Google
- ✅ Fill gaps for hard-to-find roles (Dean, CLO)
- ❌ Do NOT replace LinkedIn via Google as primary discovery

**Orchestrator:**
- ✅ Batch enrich Consolidated_Academies (company validation)
- ✅ Enrich Contacts_Master (email/phone discovery)
- ✅ Validate employee counts, revenue, tech stacks
- ❌ Do NOT perform exhaustive searches

**COST PROJECTION FOR THIS PROJECT:**

```
Conservative Estimate (25% monthly credits):
- Contact enrichment: 5,000 contacts / 10 per batch = 500 API calls
- Company validation: 1,000 companies / 10 per batch = 100 API calls
- Tech stack discovery: 500 companies = 500 API calls
- Targeted searches: 200 searches × 3 pages = 600 API calls
- Lookup operations: 200 lookups = 20 API calls
TOTAL: ~1,720 API calls (0.69% of project budget, ~15% monthly credits)
OVERAGE RISK: NONE

Recommended Usage (50% monthly credits):
- Contact enrichment: 10,000 contacts / 10 per batch = 1,000 API calls
- Company validation: 2,500 companies / 10 per batch = 250 API calls
- Tech stack discovery: 1,500 companies = 1,500 API calls
- Targeted searches: 1,000 searches × 5 pages = 5,000 API calls
- Lookup operations: 500 lookups = 50 API calls
TOTAL: ~7,800 API calls (3.1% of project budget, ~50% monthly credits)
OVERAGE RISK: NONE

Maximum Approved Usage (Project Cap):
- Contact enrichment: 20,000 contacts / 10 = 2,000 API calls
- Targeted contact discovery: 5,000 searches × 5 pages = 25,000 API calls
- Company validation: 5,000 companies / 10 = 500 API calls
- Tech stack discovery: 2,000 companies = 2,000 API calls
TOTAL: ~29,500 API calls (11.8% of project budget)
NOTE: Exceeds single-month credits, spans 2-3 months

Hard Limit (Do Not Exceed):
- 250,000 API calls = Project hard cap (10% of annual capacity)
- At 200,000 calls: Review progress and remaining goals
- At 225,000 calls: Final stretch - verify necessity of remaining operations
```

**FALLBACK STRATEGY:**

If approaching limits (>200,000 calls used):
1. **REVIEW progress toward project goals**
2. **PRIORITIZE remaining operations** by business value
3. **COMPLETE highest-value enrichments first**
4. **DOCUMENT any scope adjustments**

**SUCCESS METRICS (With ZoomInfo as Primary Tool):**
- 500+ academies validated (awards + website discovery + ZoomInfo validation)
- 500-1,000+ contacts discovered (LinkedIn via Google + ZoomInfo targeted search + enrichment)
- 80%+ academies with validated employee counts (ZoomInfo company enrichment)
- 60%+ academies with identified tech stacks (ZoomInfo + LMS vendors)
- Total ZoomInfo usage: 5,000-30,000 API calls (2-12% of project budget, well within limits)

**For detailed implementation, see:** `docs/ZoomInfo_Usage_Guide.md`

### Google Sheets MCP (Primary Data Storage)

**Purpose**: All data persistence, agent coordination, and output management.

**Critical Tools:**

**Reading:**
```
get_sheet_data(spreadsheet_id, sheet="Company_Universe", range="A1:F1000")
get_multiple_sheet_data([{spreadsheet_id, sheet, range}, ...])
```

**Writing:**
```
add_rows(spreadsheet_id, sheet="Awards_Discoveries", data=[[row1], [row2]])
update_cells(spreadsheet_id, sheet="Consolidated_Academies", range="A2:Z100", data)
batch_update_cells(spreadsheet_id, sheet, ranges={range1: data1, range2: data2})
```

**Best Practices:**
- Use `batch_update_cells` for multiple updates (more efficient)
- Read in batches of 100-500 rows to manage token usage
- Write immediately after discovery (don't buffer)
- Use `add_rows` for appending (no range needed)

### Sequential Thinking MCP (Complex Analysis)

**Purpose**: Multi-step reasoning for ambiguous situations, confidence scoring, and strategic decisions.

**When to Use:**
- Uncertain if website content qualifies as "academy"
- Complex confidence scoring decisions (MEDIUM vs HIGH)
- Duplicate detection with fuzzy matching
- Strategic planning for search approaches

```json
{
  "name": "mcp__sequential-thinking__sequentialthinking",
  "arguments": {
    "thought": "Analyzing if 'Leadership Institute' qualifies as corporate academy",
    "thoughtNumber": 1,
    "totalThoughts": 5,
    "nextThoughtNeeded": true
  }
}
```

**Best Practices:**
- Use for edge cases, not routine validation
- Estimate 3-7 thoughts for most analysis
- Mark `nextThoughtNeeded: false` when conclusion reached

### Memory MCP (Cross-Session Context)

**Purpose**: Persist knowledge across long research sessions, track patterns, remember company characteristics.

**Available Tools:**
- `create_entities` - Store companies, academies as entities
- `create_relations` - Link companies to academies
- `add_observations` - Add evidence notes
- `search_nodes` - Find related entities
- `read_graph` - View full knowledge graph

**Use Cases:**
- Remember companies already researched
- Track recurring evidence patterns
- Link multiple academies to same parent company
- Persist findings between session breaks

```json
{
  "name": "mcp__memory__create_entities",
  "arguments": {
    "entities": [{
      "name": "McDonald's Corporation",
      "entityType": "ParentCompany",
      "observations": [
        "Has Hamburger University (established academy)",
        "QSR industry, 50,000+ employees",
        "Found via Brandon Hall Award 2023"
      ]
    }]
  }
}
```

### Playwright MCP (Dynamic Content)

**Purpose**: Browser automation for JavaScript-heavy sites, form submissions, login-required content.

**When to Use:**
- Academy pages require JavaScript to render
- Need to interact with search forms or filters
- Content behind "Load More" buttons
- Verifying visual presence of academy branding

**Available Tools:**
- `browser_navigate` - Load pages
- `browser_snapshot` - Capture page state
- `browser_click` - Interact with elements
- `browser_take_screenshot` - Visual validation

**Use Cases:**
- LMS vendor customer pages with dynamic loading
- Award sites with interactive filters
- LinkedIn search (if direct search blocked)

**Best Practices:**
- Use as fallback when Firecrawl fails
- More expensive (slower, more tokens)
- Good for visual validation of findings

### n8n Workflows MCP (Automation)

**Purpose**: Create repeatable research workflows, schedule batch processing, coordinate agent tasks.

**Not Critical for Initial Implementation** - This project uses direct agent logic, but n8n could automate:
- Scheduled daily batches (Agent 3 processes 100 companies/day)
- Retry logic for rate-limited searches
- Data quality checks and alerts

**Consider for:** Production deployment with ongoing monitoring.

## Search Operators Reference

```
site:[domain].com          - Search specific domain
inurl:[text]               - URL contains text
intitle:[text]             - Page title contains text
"exact phrase"             - Exact match
OR                         - Boolean OR
- (minus)                  - Exclude term
.. (two periods)           - Date range: 2020..2025
```

**Example Combined Search:**
```
site:mcdonalds.com ("corporate university" OR "academy") -customer -product
```

## Quality Assurance Indicators

**System is functioning correctly if:**
- Each agent sheet shows 50+ rows within first week
- Consolidated_Academies shows Discovery_Source listing 2+ agents
- Confidence distribution: ~60% CONFIRMED/HIGH, ~40% MEDIUM
- Contacts_Master accumulates 100+ contacts in first week
- Processing_Log shows regular "Batch_Complete" entries
- No persistent E004 errors

## Execution Timeline

**Parallel Execution (6-8 weeks total):**
- **Week 1-2**: All agents launch, Agents 1,2,4,5 complete searches, Agent 3 starts
- **Week 3-4**: Agent 3 continues, cross-validation boosts confidence
- **Week 5-6**: Agent 3 completes, diminishing returns, quality focus
- **Week 7-8**: Final validation, URL checks, spot-checking, exports

**Can compress to 4-5 weeks with additional parallelization**

## Success Criteria

**Minimum Targets:**
- 500 validated academies total
- 300+ CONFIRMED confidence (60%)
- 150+ HIGH confidence (30%)
- 50+ MEDIUM confidence (10%)
- 200+ L&D leadership contacts
- 40%+ with LMS platform identified
- 30%+ with VILT platform identified

## Key ICP Qualifiers (Reference)

**THIS IS THE #1 QUALIFIER:** Target companies must demonstrate synchronous VILT as core delivery model - NOT just webinars or async e-learning.

**Must-Have Indicators:**
- Named academy/university with dedicated web presence
- Published programs with cohort-based delivery
- Virtual/hybrid delivery model established or emerging
- 7,500+ employees (core) or 5,000+ (extended research)

**Exclude:**
- Only compliance or async e-learning
- Ad-hoc training without programmatic structure
- Webinar-only model (one-way broadcast)
- No evidence of live instruction requirements
- Higher education institutions (colleges/universities)

## Common Technology Platforms

**LMS Platforms:**
Cornerstone OnDemand, Docebo, SAP SuccessFactors Learning, Workday Learning, Degreed, EdCast, 360Learning, Absorb LMS

**VILT Platforms:**
Zoom (Webinar, Meetings), Microsoft Teams, Adobe Connect, Webex Training, GoToTraining

## Best Practices

1. **Parallel Tool Calls**: Use parallel Google Sheets operations when reading/writing independent records
2. **Rate Limiting Awareness**: Monitor for E002 errors, especially LinkedIn
3. **Fuzzy Matching**: "McDonald's Hamburger University" = "Hamburger University"
4. **Exact Branding**: Record names exactly as they appear on academy websites
5. **Evidence Thoroughness**: Always capture WHY confidence score assigned
6. **Contact Linkage**: Every contact must have valid Academy_ID
7. **Incremental Logging**: Write to Processing_Log every batch (not just end)

## Deliverables

**Primary:** Google Sheet with 9 populated tabs
**Exports:**
- `corporate_academies_export.csv` (Consolidated_Academies)
- `contacts_export.csv` (Contacts_Master)
- `execution_summary.md` (Statistics, insights, recommendations)

## Project-Specific Instructions

- **Framework Override**: This is a research automation project, NOT a software development project. Traditional coding standards (tests, linting, etc.) do NOT apply.
- **Persistence Model**: All state lives in Google Sheets - no local databases or files
- **Agent Autonomy**: Each agent operates independently with minimal coordination
- **Context Management**: Expect long-running sessions (hours/days) - use checkpoint/resume patterns
- **Quality Over Speed**: Validate every discovery before writing to sheets
- **Contact Priority**: Agent 5's primary mission is contact discovery - academies are secondary

## MCP Integration Summary

### Tool Priority Matrix

**CRITICAL (Use for every agent):**
- ✅ **Firecrawl** - Primary web scraping and validation
- ✅ **Google Sheets** - All data persistence and coordination

**HIGH VALUE (Use frequently):**
- ✅ **Sequential Thinking** - Complex decisions, confidence scoring
- ✅ **Memory** - Cross-session persistence for long research runs

**SITUATIONAL (Use as needed):**
- ⚠️ **Playwright** - Fallback for JavaScript-heavy sites
- ⚠️ **GitHub** - Job postings, company repos (optional)

**OPTIONAL (Not required):**
- ❌ **Notion** - Alternative to Google Sheets (not recommended)
- ❌ **n8n** - Future automation (Phase 2)
- ❌ **Magic** - UI generation (not applicable)
- ❌ **IDE** - Not applicable for research project

### Typical Agent Workflow Pattern

```
1. Read → get_sheet_data (Google Sheets) or firecrawl_search (web)
2. Discover → firecrawl_map (find URLs) or firecrawl_search (general)
3. Extract → firecrawl_scrape (single page) or firecrawl_extract (structured)
4. Analyze → Sequential (if ambiguous) or direct validation
5. Persist → add_rows (Google Sheets) + create_entities (Memory)
6. Log → add_rows (Processing_Log sheet)
```

### Performance Optimization Tips

**Firecrawl Caching:**
- Always use `maxAge: 172800000` (48 hours) for repeat scrapes
- Reduces API calls by 500% for re-validation
- Critical for Agent 3 (processes 8,000+ companies)

**Google Sheets Batching:**
- Read 100-500 rows at once, not row-by-row
- Use `batch_update_cells` for multiple updates
- Write discoveries immediately (don't buffer in memory)

**Parallel Operations:**
- Agent 1-5 run in parallel (not sequential)
- Firecrawl allows concurrent requests (use parallel tool calls)
- Each agent has independent Google Sheets write access

**Rate Limiting Strategy:**
- Firecrawl: No documented limits, but respect fair use
- LinkedIn (via Firecrawl): May hit rate limits → E002, wait 1 hour
- Google Sheets: 500 requests/100 seconds (well within capacity)

### Error Recovery Patterns

**E002 (Rate Limit):**
```
1. Log error: add_rows(Processing_Log, [["E002", "Rate limited", agent_name]])
2. Save progress: create_entities(Memory, processed_companies)
3. Wait 3600 seconds
4. Resume from last processed item
5. Use memory MCP to check: search_nodes("last_processed")
```

**E004 (Sheet Write Failed):**
```
1. Retry with exponential backoff (1s, 2s, 4s)
2. If 3 failures: Log critical error
3. Continue processing (don't halt)
4. Flag records as "MANUAL REVIEW REQUIRED"
```

**Firecrawl 404/Timeout:**
```
1. Mark as E001 in Evidence_Description
2. Set Confidence to MEDIUM (if other evidence exists)
3. Continue processing (don't block on one failure)
```

### Cross-Session Persistence Strategy

**Problem:** Research takes 6-8 weeks, sessions may disconnect

**Solution:** Use Memory MCP as checkpoint system

**Checkpoint Pattern:**
```
Every 100 discoveries or 4 hours:
1. create_entities(Memory, {
     entityType: "CheckpointState",
     observations: [
       "Agent3: Processed 2,500/8,000 companies",
       "Discoveries so far: 287 academies",
       "Current batch: rows 2501-2600 in Company_Universe",
       "Last successful write: 2025-10-28 14:32:18"
     ]
   })
2. add_observations("Agent3State", ["checkpoint saved"])
3. Continue processing
```

**Resume Pattern:**
```
On agent restart:
1. read_graph(Memory) - Get full state
2. search_nodes("CheckpointState") - Find last checkpoint
3. get_sheet_data("Company_Universe", check Status column)
4. Resume from first "Not_Started" or "In_Progress" row
```

### MCP Tool Call Examples (Copy-Paste Ready)

**Firecrawl Search with LinkedIn:**
```json
{
  "name": "mcp__MCP_DOCKER__firecrawl_search",
  "arguments": {
    "query": "site:linkedin.com/in Chief Learning Officer walmart",
    "limit": 20,
    "sources": [{"type": "web"}]
  }
}
```

**Firecrawl Map Company Domain:**
```json
{
  "name": "mcp__MCP_DOCKER__firecrawl_map",
  "arguments": {
    "url": "https://walmart.com",
    "search": "university OR academy OR institute OR learning"
  }
}
```

**Firecrawl Scrape with Caching:**
```json
{
  "name": "mcp__MCP_DOCKER__firecrawl_scrape",
  "arguments": {
    "url": "https://corporate.walmart.com/learning-university",
    "formats": ["markdown"],
    "onlyMainContent": true,
    "maxAge": 172800000
  }
}
```

**Google Sheets Write Discovery:**
```json
{
  "name": "mcp__google-sheets__add_rows",
  "arguments": {
    "spreadsheet_id": "YOUR_SHEET_ID",
    "sheet": "Awards_Discoveries",
    "data": [[
      "AC-0001",
      "Walmart University",
      "Walmart Inc.",
      "walmart.com",
      "https://corporate.walmart.com/university",
      "Awards Agent",
      "CONFIRMED",
      "Brandon Hall Award 2023; Dedicated website with 50+ programs",
      "Retail",
      "50000+",
      "",
      "",
      0,
      "2025-10-28",
      "UNIQUE",
      ""
    ]]
  }
}
```

**Sequential Thinking for Validation:**
```json
{
  "name": "mcp__sequential-thinking__sequentialthinking",
  "arguments": {
    "thought": "Need to determine if 'Walmart Leadership Institute' qualifies as corporate academy. Evidence: dedicated website, cohort-based programs, 10,000+ employees. Concern: focus is leadership development, not broad training.",
    "thoughtNumber": 1,
    "totalThoughts": 4,
    "nextThoughtNeeded": true
  }
}
```

**Memory Checkpoint:**
```json
{
  "name": "mcp__memory__create_entities",
  "arguments": {
    "entities": [{
      "name": "Agent3_State_2025-10-28",
      "entityType": "CheckpointState",
      "observations": [
        "Processed: 2,500/8,000 companies",
        "Found: 287 academies",
        "Current range: A2501:F2600 in Company_Universe",
        "Last write: 2025-10-28 16:45:22",
        "Next action: Continue batch processing"
      ]
    }]
  }
}
```

### Troubleshooting Common Issues

**Issue: "Firecrawl returning empty results"**
- Check URL format (must be full https://)
- Try without `onlyMainContent: true`
- Use `firecrawl_map` first to verify URL exists

**Issue: "Google Sheets permission denied"**
- Verify spreadsheet ID correct
- Check OAuth token valid (~/.claude/google-sheets-token.json)
- Re-authenticate if needed

**Issue: "Agent finding too many false positives"**
- Tighten search patterns (add exclusions: -customer -product)
- Use Sequential MCP for validation
- Raise confidence threshold (require 2+ sources)

**Issue: "Rate limits on LinkedIn"**
- Switch to indirect search: GitHub job postings
- Use press releases for contact names
- Reduce search frequency (batch process)

**Issue: "Context window filling up"**
- Use Memory MCP for checkpointing
- Start new session, resume from Memory state
- Process in smaller batches (50 companies instead of 100)

---

## 100% Free Data Sources Methodology

### Budget: $0 Subscriptions Required

**This project uses ONLY free, publicly available data sources:**

✅ **Award Programs** (FREE)
- Brandon Hall Excellence Awards winner lists
- Training Magazine Top 125 rankings
- CLO LearningElite profiles
- ATD BEST Awards directory

✅ **LMS Vendor Pages** (FREE)
- Cornerstone customer showcases
- Docebo case studies
- SAP SuccessFactors customers
- Workday customer stories
- G2 Crowd reviews (free tier)

✅ **Systematic Web Discovery** (FREE)
- Fortune 1000 from Wikipedia
- Google `site:` searches (100-200/hour free)
- Company career pages
- Company newsrooms

✅ **Press & News** (FREE)
- PR Newswire via Google
- Business Wire via Google
- Google News search
- Company press releases

✅ **LinkedIn via Google** (FREE)
- `site:linkedin.com/in` searches via Google
- LinkedIn job postings via Google
- Bypasses LinkedIn rate limits
- No subscription required

✅ **Government Databases** (FREE)
- SEC EDGAR filings
- DOL apprenticeship registry
- State education registrations

### Rate Limit Management (Free Sources)

**Google Search Limits:**
- ~100-200 searches per hour per IP
- Mitigation: 2-3 second delays between searches
- Rotate user agents if needed

**LinkedIn via Google:**
- No LinkedIn limits (searching Google, not LinkedIn)
- Google limits apply (generous)
- Public profile pages are indexed

**Website Scraping:**
- Respect robots.txt
- 2-5 second delays between requests
- Proper user-agent identification

### Expected Outcomes (Free Methodology)

**Academy Discovery:**
- 500-750 validated academies
- Confidence: 50-60% CONFIRMED, 30-35% HIGH, 10-15% MEDIUM

**Contact Discovery:**
- 200-400 L&D contacts
- Sources: Award profiles (50-100) + LinkedIn via Google (200+) + Press (50+)

**Data Quality:**
- Comparable to paid sources (awards and websites are primary)
- LMS platform identification: 40%+
- Legal compliance: 100% (all public data)

### Paid Data Sources (OPTIONAL - Available if Needed)

**ZoomInfo Integration (CONFIGURED):**
- ✅ **Status**: MCP server configured and ready
- ✅ **Authentication**: Requires OAuth (use `/mcp` command)
- ✅ **Use Case**: Optional accelerated contact/company discovery
- ✅ **Access**: Available via `mcp__zoominfo__*` tools

**When to Use ZoomInfo:**
- Need rapid contact enrichment (200+ contacts in hours vs weeks)
- Require verified phone numbers and direct emails
- Need comprehensive tech stack intelligence
- Want to validate company data (revenue, employee count)

**Free Alternative Methodology (DEFAULT):**
- ❌ Apollo.io ($59/month) → Use LinkedIn via Google (free)
- ⚠️ ZoomInfo (AVAILABLE) → Can still use award programs + web discovery (free)
- ❌ TheirStack.com → Use LMS vendor customer pages (free)

**Why Free Works:**
- Award programs are primary validation sources regardless
- LMS vendor customer pages are public marketing content
- Google indexes LinkedIn public profiles
- Company websites are the ultimate source of truth
- Government databases provide verified data

**Project Mode Selection:**
- **FREE MODE (default)**: 6-8 weeks, 100% free sources, 200-400 contacts
- **HYBRID MODE (ZoomInfo enabled)**: 4-6 weeks, paid enrichment, 400-800 contacts

---

**Note:** This project requires sustained, autonomous multi-agent execution over 6-8 weeks. Focus is on research quality, data validation, and systematic discovery - NOT rapid prototyping or MVP development.

**MCP Dependencies:**
- **Critical**: Firecrawl, Google Sheets
- **Recommended**: Sequential Thinking, Memory
- **Optional**: Playwright (fallback), ZoomInfo (acceleration)

**Budget:**
- **Free Mode**: $0 subscriptions (Firecrawl usage minimized with caching)
- **Hybrid Mode**: ZoomInfo API credits (user already has subscription)
