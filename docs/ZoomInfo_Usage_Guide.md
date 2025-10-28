# ZoomInfo MCP Server - Complete Usage Guide

**Document Version**: 1.0
**Last Updated**: 2025-10-28
**Contract Period**: July 10, 2025 - July 9, 2026
**Project**: Corporate Academy Discovery System

---

## Table of Contents

1. [Subscription Overview](#subscription-overview)
2. [Usage Limits & Constraints](#usage-limits--constraints)
3. [Cost Structure & Overage Risks](#cost-structure--overage-risks)
4. [Project Budget & Guardrails](#project-budget--guardrails)
5. [Mandatory Pre-Flight Checklist](#mandatory-pre-flight-checklist)
6. [Approved Use Cases](#approved-use-cases)
7. [Prohibited Use Cases](#prohibited-use-cases)
8. [Usage Tracking & Monitoring](#usage-tracking--monitoring)
9. [API Call Estimation Guide](#api-call-estimation-guide)
10. [Batch Operations Best Practices](#batch-operations-best-practices)
11. [Emergency Procedures](#emergency-procedures)
12. [Weekly/Monthly Reporting](#weeklymonthly-reporting)

---

## Subscription Overview

### **Contract Details**

| Item | Details |
|------|---------|
| **Contract Period** | July 10, 2025 - July 9, 2026 (12 months) |
| **Client** | Class Technologies Inc. |
| **Edition** | Advanced+ (12 users) |
| **Database Size** | 250,000 records |

### **Credit Allocations**

**Monthly Recurring Credits: 12,000**
- Structure: 12 users × 1,000 credits per user
- Usage: Company/contact views, searches, basic API queries
- Reset: 1st of each month
- **CRITICAL**: Non-transferable between users, NO rollover

**Bulk Data Credits: 100,000 (One-Time)**
- Usage: Large exports via OperationsOS platform
- Contract Lifetime: Entire 12-month period
- Use Cases: Bulk exports, mass enrichment operations

---

## Usage Limits & Constraints

### **API Call Limits (Fair Use Policy)**

**Annual API Call Limit: 2,500,000**
- Calculation: 250,000 records (database size) × 10 = 2.5M calls
- Source: RingLead/Enrich Premium Fair Use clause (Contract Page 5)
- Safe Monthly Budget: ~200,000 API calls (leaves 100K buffer)

**Per-User Monthly Limits (Advanced+ Edition):**

| Metric | Limit | Total (12 Users) |
|--------|-------|------------------|
| Company/Contact Views | 2,000/user/month | 24,000/month |
| Intent Exports | 100,000/user/month | 1,200,000/month |
| Bulk Export (per action) | 500,000 records | N/A (per operation) |

### **Credit Consumption Rates**

**Monthly Recurring Credits Usage:**
- Individual contact search: 5-10 credits per search
- Company profile view: 3-5 credits per view
- Contact enrichment: 5-10 credits per contact
- Company enrichment: 10-20 credits per company

**Bulk Data Credits Usage:**
- Bulk export (1,000 contacts): 100-200 credits
- Mass enrichment (1,000 records): 500-1,000 credits
- Large dataset export: Variable based on size

---

## Cost Structure & Overage Risks

### **Overage Charges**

**API Call Overages:**
- **Cost**: $10,000 per 1 million API calls beyond 2.5M annual limit
- **Example**: 3.5M total API calls = 1M overage = $10,000 charge
- **Warning Threshold**: 2.0M calls (80% of limit)

**Data Credit Overages:**
- Monthly Recurring Credits: Cannot purchase additional credits mid-cycle
- Bulk Data Credits: Additional purchases available at premium rates (not specified in contract)

### **Risk Scenarios**

**Low Risk (This Project - Recommended):**
- API Calls: < 5,000 (0.2% of annual limit)
- Monthly Credits: < 1,000 (8.3% of monthly allocation)
- Overage Cost: $0

**Medium Risk:**
- API Calls: 10,000-50,000 (0.4%-2% of annual limit)
- Monthly Credits: 3,000-6,000 (25%-50% of monthly allocation)
- Overage Cost: $0 (but reduces capacity for other projects)

**High Risk (DO NOT EXCEED):**
- API Calls: > 100,000 (4%+ of annual limit)
- Monthly Credits: > 10,000 (83%+ of monthly allocation)
- Overage Cost: Potential if repeated monthly

**Critical Risk (EMERGENCY STOP):**
- API Calls: > 2,000,000 (80% of annual limit)
- Approaching overage threshold
- Overage Cost: $10,000+ likely

---

## Project Budget & Guardrails

### **Corporate Academy Discovery System - Approved Budget**

```yaml
project_budget:
  name: "Corporate Academy Discovery System"
  duration: "6-8 weeks"
  approved_api_call_limit: 5000
  approved_monthly_credit_limit: 1000
  buffer_percentage: 20

  hard_limits:
    max_api_calls_per_day: 500
    max_api_calls_per_week: 2000
    max_api_calls_total: 5000
    alert_threshold: 4000  # 80% of project limit
    emergency_stop: 4500   # 90% of project limit

  usage_allocation:
    contact_enrichment: 2000    # 40%
    company_validation: 1500    # 30%
    tech_stack_discovery: 1000  # 20%
    targeted_searches: 500      # 10%
```

### **Mandatory Guardrails**

**Pre-Operation Checks:**
1. ✅ Verify running total in Processing_Log < 5,000
2. ✅ Confirm free sources exhausted for this data
3. ✅ Calculate planned API calls for operation
4. ✅ Ensure running_total + planned_calls < 5,000
5. ✅ Use `lookup` tool before any `search_*` operation

**Post-Operation Logging:**
1. ✅ Log API call count to Processing_Log immediately
2. ✅ Calculate new running total
3. ✅ Update remaining budget
4. ✅ Alert if > 4,000 calls used

**Emergency Stops:**
- At 4,000 calls (80%): ALERT user, review remaining operations
- At 4,500 calls (90%): STOP all ZoomInfo operations, require user approval
- At 5,000 calls (100%): HARD STOP, no further calls this project

---

## Mandatory Pre-Flight Checklist

**Before EVERY ZoomInfo MCP operation, complete this checklist:**

### **Step 1: Check Current Usage**

```python
# Read Processing_Log to get running total
last_entry = get_sheet_data(
    spreadsheet_id,
    sheet="Processing_Log",
    range="A2:G" # All ZoomInfo entries
)

# Find last ZoomInfo entry
zoominfo_entries = [row for row in last_entry if row[1] == "ZoomInfo"]
if zoominfo_entries:
    running_total = int(zoominfo_entries[-1][5])  # Column F: running_total
else:
    running_total = 0

print(f"Current ZoomInfo API calls used: {running_total}/5000")
```

### **Step 2: Estimate Planned Operation Cost**

| Operation Type | API Call Cost |
|----------------|---------------|
| `lookup` | ~0.1 calls (minimal) |
| `search_contacts` (1 page, 25 results) | 1 call |
| `search_contacts` (5 pages, 125 results) | 5 calls |
| `search_companies` (1 page, 25 results) | 1 call |
| `enrich_contact` (batch of 10) | 1 call |
| `enrich_contact` (100 contacts, batched) | 10 calls |
| `enrich_company` (batch of 10) | 1 call |
| `enrich_company` (100 companies, batched) | 10 calls |

### **Step 3: Verify Budget Availability**

```python
planned_calls = 50  # Example: enriching 500 contacts = 50 calls
remaining_budget = 5000 - running_total

if planned_calls > remaining_budget:
    print("❌ STOP: Insufficient budget for this operation")
    print(f"Planned: {planned_calls}, Available: {remaining_budget}")
    return ABORT

if running_total + planned_calls > 4000:
    print("⚠️ WARNING: Operation would exceed 80% of project budget")
    print("Review necessity with user before proceeding")
    return REVIEW_REQUIRED

print(f"✅ APPROVED: Sufficient budget available")
print(f"After operation: {running_total + planned_calls}/5000 ({(running_total + planned_calls)/50}%)")
return PROCEED
```

### **Step 4: Confirm Free Alternatives Exhausted**

**Ask yourself:**
- ❓ Can this contact be found via LinkedIn via Google?
- ❓ Can this company data be validated via their website?
- ❓ Can this tech stack be confirmed via LMS vendor customer pages?
- ❓ Can this information be found via awards programs or press releases?

**If YES to any:** Use free source first, ZoomInfo only for enrichment/validation

### **Step 5: Use Lookup First (Always)**

```json
// NEVER skip this step - prevents wasted API calls
{
  "name": "mcp__zoominfo__lookup",
  "arguments": {
    "fieldNames": ["management-levels", "industries", "metro-regions"]
  }
}

// Use exact values from lookup response in search
{
  "name": "mcp__zoominfo__search_contacts",
  "arguments": {
    "managementLevel": "Vice President",  // From lookup
    "industryCodes": "Computer Software",  // From lookup
    "metroRegion": "San Francisco-Oakland-Hayward, CA"  // From lookup
  }
}
```

---

## Approved Use Cases

### **Tier 1: Contact Enrichment (HIGHEST PRIORITY)**

**Purpose**: Add email/phone to already-discovered contacts

**Approved Scenarios:**
- ✅ Contacts found via LinkedIn via Google (missing email/phone)
- ✅ Contacts found via awards programs (missing direct contact info)
- ✅ Contacts found via press releases (need verification)

**Budget Allocation**: 2,000 API calls (40% of project budget)

**Implementation:**
```json
{
  "name": "mcp__zoominfo__enrich_contact",
  "arguments": {
    "contacts": [
      {"firstName": "John", "lastName": "Smith", "company": "Walmart Inc."},
      {"email": "jane.doe@target.com"},
      // ... up to 10 contacts per batch
    ]
  }
}
```

**Estimated Yield:**
- 200 contacts enriched = 20 API calls
- 500 contacts enriched = 50 API calls

### **Tier 2: Company Validation**

**Purpose**: Verify employee counts, revenue, industry classification

**Approved Scenarios:**
- ✅ Academies in Consolidated_Academies with missing data
- ✅ Validation of employee count thresholds (5,000+)
- ✅ Revenue verification for qualification

**Budget Allocation**: 1,500 API calls (30% of project budget)

**Implementation:**
```json
{
  "name": "mcp__zoominfo__enrich_company",
  "arguments": {
    "companies": [
      {"companyName": "Walmart Inc.", "domain": "walmart.com"},
      {"companyId": "12345"},
      // ... up to 10 companies per batch
    ]
  }
}
```

**Estimated Yield:**
- 100 companies validated = 10 API calls
- 500 companies validated = 50 API calls

### **Tier 3: Tech Stack Discovery**

**Purpose**: Identify LMS/VILT platforms for confirmed academies

**Approved Scenarios:**
- ✅ Academies confirmed via other sources (need tech details)
- ✅ Cross-reference Agent 2 LMS vendor findings
- ✅ VILT platform identification for qualified prospects

**Budget Allocation**: 1,000 API calls (20% of project budget)

**Implementation:**
```json
{
  "name": "mcp__zoominfo__search_companies",
  "arguments": {
    "companyName": "Walmart Inc.",
    "techAttributeTagList": "Cornerstone OnDemand,Docebo,SAP SuccessFactors"
  }
}
```

**Estimated Yield:**
- 200 companies checked = 200 API calls

### **Tier 4: Targeted Contact Search (USE WITH CAUTION)**

**Purpose**: Find hard-to-locate L&D leadership contacts

**Approved Scenarios:**
- ✅ ONLY after LinkedIn via Google exhausted
- ✅ ONLY for specific roles (Dean, CLO, VP Learning)
- ✅ ONLY for confirmed academies with no other contact sources

**Budget Allocation**: 500 API calls (10% of project budget)

**Implementation:**
```json
// ALWAYS use lookup first
{
  "name": "mcp__zoominfo__lookup",
  "arguments": {
    "fieldNames": ["management-levels", "job-functions"]
  }
}

// Then targeted search
{
  "name": "mcp__zoominfo__search_contacts",
  "arguments": {
    "companyName": "Walmart Inc.",
    "jobTitle": "Chief Learning Officer OR Dean OR Vice President Learning",
    "managementLevel": "Vice President,C-Level",
    "perPage": 25
  }
}
```

**Estimated Yield:**
- 50 targeted searches × 2 pages = 100 API calls

---

## Prohibited Use Cases

**❌ DO NOT USE ZOOMINFO FOR:**

### **1. Primary Discovery (Use Free Sources)**
- ❌ Discovering new companies/academies
- ❌ Initial contact discovery
- ❌ Broad industry research
- **Why**: Free sources (awards, LMS vendors, LinkedIn via Google) provide sufficient coverage
- **Alternative**: Use Agents 1-5 with free sources, then enrich with ZoomInfo

### **2. Systematic Processing (Wasteful)**
- ❌ Processing all 8,000 companies in Company_Universe
- ❌ Bulk contact discovery without targeting
- ❌ Exploratory searches without specific criteria
- **Why**: Would consume 80,000+ API calls (1600% of project budget)
- **Alternative**: Target only confirmed academies for validation

### **3. Repeated Operations (Inefficient)**
- ❌ Re-enriching same contacts multiple times
- ❌ Duplicate searches with slightly different parameters
- ❌ Trial-and-error searches without lookup first
- **Why**: Each API call counts against limit, even if data unchanged
- **Alternative**: Cache results in Google Sheets, reference before re-querying

### **4. Low-Value Operations (Poor ROI)**
- ❌ Validating data easily found on company websites
- ❌ Searching for publicly available information
- ❌ Enriching contacts with no academy affiliation
- **Why**: Wastes API calls on data obtainable for free
- **Alternative**: Use Firecrawl for website data, LinkedIn via Google for public profiles

### **5. Unapproved High-Volume Operations**
- ❌ Bulk exports > 1,000 records without approval
- ❌ Multi-page searches (>5 pages) without justification
- ❌ Batch operations exceeding daily limits (>500 calls/day)
- **Why**: Could rapidly exhaust budget or trigger overages
- **Alternative**: Request user approval for any operation >100 API calls

---

## Usage Tracking & Monitoring

### **Processing_Log Sheet Structure**

**Required Columns:**

| Column | Field | Type | Example |
|--------|-------|------|---------|
| A | Timestamp | DateTime | 2025-10-28 14:30:15 |
| B | Agent | String | "ZoomInfo" |
| C | Tool_Name | String | "enrich_contact" |
| D | Records_Affected | Integer | 10 |
| E | API_Calls_Used | Integer | 1 |
| F | Running_Total | Integer | 246 |
| G | Remaining_Budget | Integer | 4754 |
| H | Notes | Text | "Batch enrichment for Agent 5 contacts" |

### **Logging Every ZoomInfo Operation**

```python
def log_zoominfo_usage(tool_name, records_affected, api_calls_used):
    # Get current running total
    last_entry = get_last_zoominfo_entry("Processing_Log")
    running_total = last_entry["running_total"] if last_entry else 0

    # Calculate new totals
    new_running_total = running_total + api_calls_used
    remaining_budget = 5000 - new_running_total

    # Log to Processing_Log
    add_rows(spreadsheet_id, "Processing_Log", [[
        datetime.now().isoformat(),
        "ZoomInfo",
        tool_name,
        records_affected,
        api_calls_used,
        new_running_total,
        remaining_budget,
        f"Operation completed successfully"
    ]])

    # Alert if approaching limits
    if new_running_total >= 4000:
        ALERT(f"⚠️ CRITICAL: {new_running_total}/5000 API calls used (80%+)")
        ALERT("Review remaining operations with user IMMEDIATELY")
    elif new_running_total >= 3500:
        WARN(f"⚠️ WARNING: {new_running_total}/5000 API calls used (70%)")
        WARN("Monitor usage closely")

    return new_running_total, remaining_budget
```

### **Daily Monitoring Checklist**

**Every Day (if ZoomInfo used):**

1. ✅ Open Processing_Log sheet
2. ✅ Filter for Agent = "ZoomInfo"
3. ✅ Check last Running_Total value
4. ✅ Verify daily usage < 500 API calls
5. ✅ Calculate percentage: (running_total / 5000) × 100%
6. ✅ Update project status document

**Green Status (0-60%):**
- Running Total: 0-3,000 API calls
- Action: Continue normal operations

**Yellow Status (60-80%):**
- Running Total: 3,000-4,000 API calls
- Action: Review remaining operations, prioritize critical enrichments only

**Red Status (80-90%):**
- Running Total: 4,000-4,500 API calls
- Action: STOP non-essential operations, user approval required for all further calls

**Critical Status (90-100%):**
- Running Total: 4,500-5,000 API calls
- Action: EMERGENCY STOP, no further calls without explicit user authorization

### **Weekly Monitoring Checklist**

**Every Monday (or start of work week):**

1. ✅ Log into ZoomInfo Admin Portal
   - URL: https://admin.zoominfo.com/#/api-and-webhooks
2. ✅ Check "API Usage" dashboard
   - Total API calls this month
   - Remaining monthly recurring credits
   - Remaining bulk data credits
3. ✅ Compare admin portal vs Processing_Log
   - Verify running totals match (±5% acceptable)
4. ✅ Calculate weekly usage rate
   - This week's calls / 2,000 (weekly budget)
5. ✅ Project remaining capacity
   - (5,000 - running_total) / weeks_remaining
6. ✅ Document findings in weekly status report

### **Monthly Monitoring Checklist**

**First of Every Month:**

1. ✅ Generate monthly usage report
   - Total API calls used this month
   - Total monthly credits consumed
   - Breakdown by agent/operation type
2. ✅ Review against project budget
   - On track? Ahead? Behind?
   - Adjust future usage if needed
3. ✅ Check admin portal for any alerts/warnings
4. ✅ Verify monthly recurring credits reset (1st of month)
5. ✅ Document findings for stakeholder review

---

## API Call Estimation Guide

### **Tool-Specific API Call Costs**

| Tool | Operation | API Calls | Notes |
|------|-----------|-----------|-------|
| `lookup` | Get field values | 0.1 | Minimal cost, always use first |
| `search_contacts` | 1 page (25 results) | 1 | Each page = 1 call |
| `search_contacts` | 5 pages (125 results) | 5 | Pagination multiplies cost |
| `search_companies` | 1 page (25 results) | 1 | Each page = 1 call |
| `enrich_contact` | Single contact | 1 | Use batch operations instead |
| `enrich_contact` | Batch (10 contacts) | 1 | 10x more efficient |
| `enrich_company` | Single company | 1 | Use batch operations instead |
| `enrich_company` | Batch (10 companies) | 1 | 10x more efficient |

### **Common Operation Cost Examples**

**Scenario 1: Enrich 200 Contacts**
```
Operation: enrich_contact (batch of 10)
Batches: 200 contacts ÷ 10 per batch = 20 batches
API Calls: 20 batches × 1 call per batch = 20 API calls
Percentage of Budget: 20 / 5,000 = 0.4%
```

**Scenario 2: Search for CLOs at 50 Companies**
```
Operation: search_contacts with specific criteria
Pages per search: 2 (limited results expected)
API Calls: 50 searches × 2 pages = 100 API calls
Percentage of Budget: 100 / 5,000 = 2%
```

**Scenario 3: Validate 500 Companies**
```
Operation: enrich_company (batch of 10)
Batches: 500 companies ÷ 10 per batch = 50 batches
API Calls: 50 batches × 1 call per batch = 50 API calls
Percentage of Budget: 50 / 5,000 = 1%
```

**Scenario 4: Discovery Search (Multi-Page)**
```
Operation: search_contacts (exploratory)
Pages: 10 pages × 25 results per page = 250 results
API Calls: 10 API calls
Percentage of Budget: 10 / 5,000 = 0.2%
Warning: Multiply by number of searches!
```

### **Cost Optimization Strategies**

**1. Batch Everything Possible**
```
❌ BAD: 100 individual enrich_contact calls = 100 API calls
✅ GOOD: 10 batch enrich_contact calls (10 contacts each) = 10 API calls
Savings: 90 API calls (90% reduction)
```

**2. Use Specific Queries (Reduce Pagination)**
```
❌ BAD: Broad search requiring 10+ pages = 10+ API calls
✅ GOOD: Targeted search with filters, 1-2 pages = 1-2 API calls
Savings: 8+ API calls per search
```

**3. Lookup Before Search (Prevent Failed Queries)**
```
❌ BAD: Search with invalid parameters, retry = 2+ API calls
✅ GOOD: Lookup first, search with valid values = 1 API call
Savings: 1+ API calls per search
```

**4. Cache Results in Google Sheets**
```
❌ BAD: Re-query same contact 3 times = 3 API calls
✅ GOOD: Query once, cache in Contacts_Master, reference later = 1 API call
Savings: 2 API calls per duplicate query
```

---

## Batch Operations Best Practices

### **Batch Size Recommendations**

**Maximum Batch Sizes (MCP Server Limits):**
- `enrich_contact`: 10 contacts per batch
- `enrich_company`: 10 companies per batch

**Optimal Batching Strategy:**

```python
def batch_enrich_contacts(contact_list):
    """
    Efficiently enrich contacts in batches of 10
    """
    batch_size = 10
    total_contacts = len(contact_list)
    num_batches = (total_contacts + batch_size - 1) // batch_size

    print(f"Enriching {total_contacts} contacts in {num_batches} batches")
    print(f"Estimated API calls: {num_batches}")

    # Pre-flight check
    running_total = get_running_total("Processing_Log")
    if running_total + num_batches > 5000:
        print("❌ ABORT: Insufficient budget")
        return

    enriched_contacts = []
    for i in range(0, total_contacts, batch_size):
        batch = contact_list[i:i+batch_size]

        # Enrich batch
        result = enrich_contact({"contacts": batch})
        enriched_contacts.extend(result)

        # Log usage
        log_zoominfo_usage("enrich_contact", len(batch), 1)

        # Rate limiting: small delay between batches
        time.sleep(1)

    return enriched_contacts
```

### **Batch Priority Ranking**

When budget is limited, prioritize batches in this order:

**Priority 1: High-Value Contacts**
- CLO, Dean, VP Learning roles
- Confirmed academy affiliations
- Missing critical data (email, phone)

**Priority 2: Confirmed Academies**
- Companies in Consolidated_Academies
- HIGH or CONFIRMED confidence scores
- Missing validation data

**Priority 3: Medium-Value Contacts**
- Director, Manager L&D roles
- Probable academy affiliations
- Partial contact information

**Priority 4: Company Validation (If Budget Allows)**
- MEDIUM confidence academies
- Tech stack verification
- Employee count validation

### **Error Handling in Batch Operations**

```python
def safe_batch_enrichment(items, enrich_function):
    """
    Batch enrichment with error handling and rollback
    """
    batch_size = 10
    successful = []
    failed = []
    api_calls_used = 0

    for i in range(0, len(items), batch_size):
        batch = items[i:i+batch_size]

        try:
            # Pre-flight check
            running_total = get_running_total("Processing_Log")
            if running_total + 1 > 5000:
                print(f"⚠️ Budget limit reached. Stopping at {len(successful)} items.")
                break

            # Enrich batch
            result = enrich_function({"contacts": batch})
            successful.extend(result)
            api_calls_used += 1

            # Log success
            log_zoominfo_usage(enrich_function.__name__, len(batch), 1)

        except Exception as e:
            print(f"❌ Batch {i//batch_size + 1} failed: {e}")
            failed.extend(batch)
            # Still log the failed API call
            api_calls_used += 1
            log_zoominfo_usage(enrich_function.__name__, 0, 1, notes=f"FAILED: {e}")

    return {
        "successful": successful,
        "failed": failed,
        "api_calls_used": api_calls_used
    }
```

---

## Emergency Procedures

### **Emergency Stop Triggers**

**Trigger 1: Budget Threshold Exceeded (>80%)**
```
Threshold: 4,000 API calls used (80% of 5,000)
Action:
1. STOP all non-critical ZoomInfo operations immediately
2. Alert user via Processing_Log + direct notification
3. Review remaining operations with user
4. Get explicit approval for each additional operation
5. Document decision to continue or halt
```

**Trigger 2: Unexpected High Usage**
```
Condition: >500 API calls used in single session
Action:
1. PAUSE all agents using ZoomInfo
2. Review Processing_Log for anomalies
3. Investigate potential runaway operations
4. Verify running total accuracy
5. Resume only after verification
```

**Trigger 3: API Rate Limiting Detected**
```
Condition: ZoomInfo API returns rate limit error
Action:
1. STOP all ZoomInfo operations immediately
2. Wait 1 hour before retry
3. Log incident in Processing_Log
4. Review usage patterns to prevent recurrence
5. Consider reducing batch sizes or frequency
```

**Trigger 4: Budget Near Exhaustion (>90%)**
```
Threshold: 4,500 API calls used (90% of 5,000)
Action:
1. EMERGENCY STOP - no further calls without explicit user authorization
2. Generate emergency budget report
3. Present to user with recommendations:
   - Complete project with free sources only
   - Request additional budget allocation
   - Postpone remaining enrichment
4. Document all decisions
```

### **Emergency Contact Procedure**

**If ANY of the following occur:**
- Running total exceeds 4,000 API calls
- Unexpected spike in API usage (>500 calls in 1 hour)
- ZoomInfo API rate limiting or errors
- Approaching annual limit (unlikely but critical)

**Immediate Actions:**
1. **STOP** all ZoomInfo MCP operations
2. **LOG** emergency stop in Processing_Log
3. **ALERT** user with summary:
   - Current running total
   - Trigger condition
   - Recommended actions
4. **WAIT** for user approval before resuming

### **Budget Exhaustion Fallback Plan**

**If 5,000 API Call Limit Reached:**

1. **Complete project using 100% free sources:**
   - Awards programs (Agent 1)
   - LMS vendor pages (Agent 2)
   - Website discovery (Agent 3)
   - Press releases (Agent 4)
   - LinkedIn via Google (Agent 5)

2. **Document shortfall in execution summary:**
   - Records enriched vs. planned
   - Data gaps identified
   - Recommendations for future enrichment

3. **Post-project enrichment options:**
   - Wait for next month (new budget cycle)
   - Manual enrichment via ZoomInfo web interface
   - Alternative data sources (Apollo.io, Clearbit, etc.)

---

## Weekly/Monthly Reporting

### **Weekly Status Report Template**

```markdown
# ZoomInfo Usage Report - Week of [Date]

## Summary
- **Total API Calls This Week**: X
- **Running Total (Project)**: Y / 5,000 (Z%)
- **Remaining Budget**: 5,000 - Y = W calls
- **Status**: [GREEN/YELLOW/RED]

## Usage Breakdown
| Agent | Tool | Calls | Records |
|-------|------|-------|---------|
| Agent 1 | enrich_company | 10 | 100 |
| Agent 5 | enrich_contact | 25 | 250 |
| Orchestrator | search_contacts | 15 | 75 |
| **TOTAL** | **-** | **50** | **425** |

## Operations Completed
1. Enriched 250 contacts from Agent 5 discoveries
2. Validated 100 companies in Consolidated_Academies
3. Tech stack discovery for 75 high-priority academies

## Budget Health
- ✅ Daily average: 7 calls/day (target: <50/day)
- ✅ Weekly usage: 50 calls (target: <500/week)
- ✅ Projected completion: 500 total calls (10% of budget)

## Alerts/Issues
- None this week

## Next Week Plan
- Enrich remaining 150 high-priority contacts (15 calls)
- Validate final 200 companies (20 calls)
- Complete tech stack discovery (50 calls)
- **Estimated next week usage**: 85 calls
```

### **Monthly Status Report Template**

```markdown
# ZoomInfo Usage Report - [Month Year]

## Executive Summary
- **Total API Calls This Month**: X
- **Project Running Total**: Y / 5,000 (Z%)
- **Monthly Recurring Credits Used**: A / 12,000 (B%)
- **Annual Capacity Remaining**: 2,500,000 - Y = W calls
- **Overall Status**: [GREEN/YELLOW/RED]

## Monthly Usage by Agent
| Agent | Operations | API Calls | Records Affected |
|-------|------------|-----------|------------------|
| Agent 1 | 15 | 45 | 450 |
| Agent 2 | 10 | 30 | 300 |
| Agent 5 | 50 | 200 | 2000 |
| Orchestrator | 25 | 125 | 1250 |
| **TOTAL** | **100** | **400** | **4000** |

## Key Achievements
1. ✅ Enriched 2,000 contacts with email/phone
2. ✅ Validated 450 companies for ICP qualification
3. ✅ Identified 300 LMS/VILT tech stacks
4. ✅ Stayed well within budget (8% of project allocation)

## Budget Analysis
- **Project Budget**: 5,000 API calls
- **Used This Month**: 400 calls (8%)
- **Running Total**: 1,200 calls (24%)
- **Remaining**: 3,800 calls (76%)
- **Burn Rate**: ~150 calls/week
- **Projected Completion**: ~1,800 total calls (36% of budget)

## Compliance
- ✅ No overage risk detected
- ✅ Daily limits respected (<500 calls/day)
- ✅ Weekly limits respected (<2,000 calls/week)
- ✅ All operations logged in Processing_Log
- ✅ Pre-flight checks completed for all operations

## Admin Portal Verification
- **Portal API Calls**: 1,205
- **Processing_Log Total**: 1,200
- **Variance**: 5 calls (0.4%) - within acceptable range
- **Monthly Credits Remaining**: 11,200 / 12,000 (93%)

## Recommendations
1. Continue current usage patterns (well within budget)
2. No changes needed to agent operations
3. Consider increasing enrichment scope if desired (budget allows)
4. Project on track for < 2,000 total API calls (excellent efficiency)

## Next Month Forecast
- **Estimated Remaining Operations**: 600 API calls
- **Projected Total Usage**: 1,800 calls (36% of project budget)
- **Buffer Remaining**: 3,200 calls (64% unused)
- **Recommendation**: Maintain current pace, no concerns

---

**Report Generated**: [Date]
**Compiled By**: Claude Code Automation
**Next Report Due**: [Date + 1 month]
```

### **Annual Capacity Planning**

**Current Project vs. Annual Capacity:**

```
Corporate Academy Discovery System:
- Projected Usage: 1,800 API calls
- Project Budget: 5,000 API calls
- Percentage of Annual Capacity: 0.072% (1,800 / 2,500,000)

Remaining Annual Capacity:
- Total: 2,500,000 API calls
- This Project: -1,800 calls
- Remaining: 2,498,200 calls (99.928% of annual capacity)

Capacity for Future Projects:
- Could run 1,388 similar projects this year (2,498,200 / 1,800)
- Or 27 projects at max budget (2,498,200 / 5,000 / 12 months)
- Zero overage risk for foreseeable usage patterns
```

---

## Appendix: Quick Reference

### **Critical Numbers**

| Metric | Value | Notes |
|--------|-------|-------|
| Annual API Limit | 2,500,000 | Fair Use Policy |
| Monthly Recurring Credits | 12,000 | Resets 1st of month |
| Bulk Data Credits | 100,000 | One-time, contract lifetime |
| Project API Budget | 5,000 | Corporate Academy Discovery |
| Daily API Limit | 500 | Project guardrail |
| Weekly API Limit | 2,000 | Project guardrail |
| Alert Threshold | 4,000 | 80% of project budget |
| Emergency Stop | 4,500 | 90% of project budget |
| Overage Cost | $10,000 | Per 1M excess API calls |

### **Admin Portal Links**

- **Dashboard**: https://admin.zoominfo.com/#/dashboard
- **API & Webhooks**: https://admin.zoominfo.com/#/api-and-webhooks
- **Usage Reports**: https://admin.zoominfo.com/#/reports/usage
- **User Management**: https://admin.zoominfo.com/#/users

### **Contract Reference**

- **File**: `/Users/russellteter/Desktop/20252026 Zoominfo.pdf`
- **Key Pages**:
  - Page 1-2: Subscription details
  - Page 5: Fair Use Policy (API limits)
  - Page 7-8: Credit definitions
  - Page 8: Platform limits

### **Project Files**

- **Main Instructions**: `CLAUDE.md` (lines 516-788)
- **This Guide**: `docs/ZoomInfo_Usage_Guide.md`
- **Usage Log**: Google Sheets → Processing_Log sheet
- **Contract**: `/Users/russellteter/Desktop/20252026 Zoominfo.pdf`

---

**Document End**

**For Questions or Updates:**
- Refer to ZoomInfo admin portal for real-time usage
- Check Processing_Log for project-specific tracking
- Review contract PDF for authoritative limits
- Consult Claude Code for implementation guidance

**Version History:**
- v1.0 (2025-10-28): Initial comprehensive documentation
