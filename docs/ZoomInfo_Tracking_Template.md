# ZoomInfo Usage Tracking Template

**Purpose**: Define the structure for tracking ZoomInfo API usage in the Processing_Log sheet of Google Sheets.

---

## Processing_Log Sheet - ZoomInfo Entries

### **Column Structure**

| Column | Field Name | Data Type | Width | Description |
|--------|------------|-----------|-------|-------------|
| A | Timestamp | DateTime | 150px | ISO 8601 format: 2025-10-28T14:30:15 |
| B | Agent_Name | String | 100px | Always "ZoomInfo" for API calls |
| C | Tool_Name | String | 150px | MCP tool used (lookup, search_contacts, enrich_contact, etc.) |
| D | Records_Affected | Integer | 80px | Number of contacts/companies processed |
| E | API_Calls_Used | Integer | 80px | Estimated API calls for this operation |
| F | Running_Total | Integer | 100px | Cumulative API calls for entire project |
| G | Remaining_Budget | Integer | 120px | 5,000 - Running_Total |
| H | Operation_Notes | Text | 300px | Description, batch info, or error messages |

### **Sample Data**

```
A: 2025-10-28T09:15:30 | B: ZoomInfo | C: lookup | D: 0 | E: 0 | F: 0 | G: 5000 | H: Initial lookup for management-levels
A: 2025-10-28T09:20:45 | B: ZoomInfo | C: enrich_contact | D: 10 | E: 1 | F: 1 | G: 4999 | H: Batch 1/20: Enrich Agent 5 contacts
A: 2025-10-28T09:25:12 | B: ZoomInfo | C: enrich_contact | D: 10 | E: 1 | F: 2 | G: 4998 | H: Batch 2/20: Enrich Agent 5 contacts
A: 2025-10-28T10:00:00 | B: ZoomInfo | C: search_contacts | D: 25 | E: 1 | F: 3 | G: 4997 | H: Search CLOs at Fortune 500 tech companies
A: 2025-10-28T14:30:22 | B: ZoomInfo | C: enrich_company | D: 10 | E: 1 | F: 4 | G: 4996 | H: Validate company data for Consolidated_Academies
```

---

## Google Sheets Formula Helpers

### **Cell Formulas for Tracking**

**Running Total (Column F):**
```
=IF(B2="ZoomInfo", IF(ROW()=2, E2, F1+E2), "")
```
- Only calculates for ZoomInfo entries
- First ZoomInfo row: uses API_Calls_Used directly
- Subsequent rows: adds to previous running total

**Remaining Budget (Column G):**
```
=IF(B2="ZoomInfo", 5000-F2, "")
```
- Subtracts running total from project budget (5,000)

**Conditional Formatting Rules:**

**1. Green Status (Remaining_Budget > 1,000):**
- Range: G2:G1000
- Format: Light green background (#D9EAD3)
- Condition: `=$G2 > 1000`

**2. Yellow Status (Remaining_Budget 200-1,000):**
- Range: G2:G1000
- Format: Yellow background (#FFF2CC)
- Condition: `=AND($G2 >= 200, $G2 <= 1000)`

**3. Red Status (Remaining_Budget < 200):**
- Range: G2:G1000
- Format: Red background (#F4CCCC)
- Condition: `=$G2 < 200`

**4. Alert on High Usage (API_Calls_Used > 50):**
- Range: E2:E1000
- Format: Orange background (#FCE5CD)
- Condition: `=AND($B2="ZoomInfo", $E2 > 50)`

---

## Dashboard Summary (Separate Sheet Tab)

### **ZoomInfo_Dashboard Sheet Structure**

**Key Metrics Section:**

| Metric | Formula | Display Format |
|--------|---------|----------------|
| **Total API Calls Used** | `=SUMIF(Processing_Log!B:B,"ZoomInfo",Processing_Log!E:E)` | 1,234 calls |
| **Project Budget** | `5000` | 5,000 calls |
| **Remaining Budget** | `=5000-[Total API Calls]` | 3,766 calls |
| **Budget Used (%)** | `=[Total API Calls]/5000*100` | 24.7% |
| **Status** | `=IF([Budget Used]<60,"GREEN",IF([Budget Used]<80,"YELLOW","RED"))` | GREEN |

**Usage by Tool:**

| Tool Name | Operations | API Calls | Records |
|-----------|------------|-----------|---------|
| lookup | `=COUNTIFS(Processing_Log!B:B,"ZoomInfo",Processing_Log!C:C,"lookup")` | `=SUMIFS(Processing_Log!E:E,Processing_Log!B:B,"ZoomInfo",Processing_Log!C:C,"lookup")` | `=SUMIFS(Processing_Log!D:D,Processing_Log!B:B,"ZoomInfo",Processing_Log!C:C,"lookup")` |
| search_contacts | ... | ... | ... |
| search_companies | ... | ... | ... |
| enrich_contact | ... | ... | ... |
| enrich_company | ... | ... | ... |
| **TOTAL** | `=SUM(...)` | `=SUM(...)` | `=SUM(...)` |

**Usage by Agent:**

```
=QUERY(Processing_Log!A:H,
  "SELECT B, COUNT(C), SUM(E), SUM(D)
   WHERE B = 'ZoomInfo'
   GROUP BY B
   LABEL COUNT(C) 'Operations', SUM(E) 'API Calls', SUM(D) 'Records'")
```

**Daily Usage Chart:**

- X-Axis: Date (from Timestamp column)
- Y-Axis: Cumulative API Calls (Running_Total column)
- Chart Type: Line chart
- Trend line: Show projection to 6-8 weeks

**Budget Alert Box:**

```
=IF([Budget Used]>=90,
  "🚨 EMERGENCY: " & [Remaining Budget] & " calls remaining!",
  IF([Budget Used]>=80,
    "⚠️ WARNING: " & [Remaining Budget] & " calls remaining",
    "✅ HEALTHY: " & [Remaining Budget] & " calls remaining"))
```

---

## Pre-Flight Check Template

**Before ANY ZoomInfo operation, use this checklist:**

### **1. Budget Verification**

```
Current Status:
- Running Total: [Get from last Processing_Log entry, Column F]
- Remaining Budget: [Get from last Processing_Log entry, Column G]
- Status: [Calculate: <60% = GREEN, 60-80% = YELLOW, >80% = RED]

Planned Operation:
- Tool: [lookup / search_contacts / enrich_contact / etc.]
- Estimated API Calls: [Calculate based on API Call Estimation Guide]
- Records to Process: [Number of contacts/companies]

Budget Check:
- After Operation Total: [Running Total] + [Estimated Calls] = ?
- Remaining After: 5000 - [After Operation Total] = ?
- Percentage Used: [After Operation Total] / 5000 * 100 = ?%

DECISION:
[ ] PROCEED - Budget sufficient, operation approved
[ ] REVIEW - Approaching 80%, discuss with user
[ ] ABORT - Insufficient budget, use free sources
```

### **2. Free Source Check**

```
Before using ZoomInfo, verify free sources exhausted:

For Contact Enrichment:
[ ] Checked LinkedIn via Google (site:linkedin.com/in)
[ ] Checked awards programs (Brandon Hall, Training Top 125, etc.)
[ ] Checked press releases (PR Newswire, Business Wire)
[ ] Checked company website careers/leadership pages

For Company Validation:
[ ] Checked company website About/Leadership pages
[ ] Checked LinkedIn company page
[ ] Checked Crunchbase/public databases
[ ] Checked SEC filings (if public company)

For Tech Stack Discovery:
[ ] Checked LMS vendor customer pages
[ ] Checked company job postings (tech requirements)
[ ] Checked LinkedIn job descriptions
[ ] Checked company press releases

DECISION:
[ ] FREE SOURCES EXHAUSTED - ZoomInfo approved
[ ] FREE SOURCES AVAILABLE - Use those first
```

### **3. Operation Logging Template**

```python
# After ZoomInfo operation completes
operation_log = {
    "timestamp": "2025-10-28T14:30:15",
    "agent_name": "ZoomInfo",
    "tool_name": "enrich_contact",  # or search_contacts, enrich_company, etc.
    "records_affected": 10,          # Number of contacts/companies
    "api_calls_used": 1,             # Actual API calls consumed
    "running_total": 246,            # Previous total + this operation
    "remaining_budget": 4754,        # 5000 - running_total
    "operation_notes": "Batch 1/20: Enriched Agent 5 LinkedIn contacts with email/phone"
}

# Write to Processing_Log sheet
add_rows(spreadsheet_id, "Processing_Log", [[
    operation_log["timestamp"],
    operation_log["agent_name"],
    operation_log["tool_name"],
    operation_log["records_affected"],
    operation_log["api_calls_used"],
    operation_log["running_total"],
    operation_log["remaining_budget"],
    operation_log["operation_notes"]
]])

# Alert if approaching limits
if operation_log["running_total"] >= 4000:
    print("🚨 CRITICAL: 80%+ of budget used - Review with user!")
elif operation_log["running_total"] >= 3500:
    print("⚠️ WARNING: 70%+ of budget used - Monitor closely")
```

---

## Weekly Reporting Template

**Copy this template for weekly status reports:**

```markdown
# ZoomInfo Usage Report - Week of [Start Date] to [End Date]

## Quick Stats
- **This Week's Usage**: [X] API calls
- **Running Total**: [Y] / 5,000 calls ([Z]%)
- **Remaining Budget**: [W] calls
- **Status**: [GREEN/YELLOW/RED]

## Usage Breakdown
| Date | Tool | Operations | API Calls | Records |
|------|------|------------|-----------|---------|
| Mon | enrich_contact | 5 | 5 | 50 |
| Tue | search_contacts | 3 | 8 | 75 |
| Wed | enrich_company | 2 | 2 | 20 |
| Thu | enrich_contact | 8 | 8 | 80 |
| Fri | lookup + search | 5 | 10 | 125 |
| **TOTAL** | **-** | **23** | **33** | **350** |

## Top Operations This Week
1. Enriched 225 contacts from Agent 5 (23 API calls)
2. Validated 50 companies in Consolidated_Academies (5 API calls)
3. Targeted searches for 10 hard-to-find CLOs (5 API calls)

## Budget Health Check
✅ Daily avg: 6.6 calls/day (target: <71/day for 6-week project)
✅ Weekly usage: 33 calls (target: <500/week)
✅ Projected: ~200 total calls at current pace (4% of budget)

## Compliance
- [x] All operations logged in Processing_Log
- [x] Pre-flight checks completed
- [x] No overage alerts
- [x] Within daily/weekly limits

## Next Week Plan
- Enrich remaining 100 high-priority contacts (10 calls)
- Complete company validation for CONFIRMED academies (15 calls)
- Tech stack discovery for top 50 prospects (20 calls)
- **Estimated next week**: 45 calls
```

---

## Monthly Reporting Template

**Copy this template for monthly status reports:**

```markdown
# ZoomInfo Usage Report - [Month Year]

## Executive Summary
- **Monthly API Calls**: [X] calls
- **Project Running Total**: [Y] / 5,000 calls ([Z]%)
- **Remaining Budget**: [W] calls
- **Monthly Credits Used**: [A] / 12,000 ([B]%)
- **Status**: [GREEN/YELLOW/RED]

## Usage by Agent
| Agent | Operations | API Calls | Records | Top Use Case |
|-------|------------|-----------|---------|--------------|
| Agent 1 | 10 | 25 | 250 | Company validation |
| Agent 2 | 5 | 15 | 150 | Tech stack discovery |
| Agent 5 | 40 | 120 | 1200 | Contact enrichment |
| Orchestrator | 15 | 40 | 400 | Batch validation |
| **TOTAL** | **70** | **200** | **2000** | **-** |

## Key Achievements This Month
1. ✅ Enriched 1,200 contacts with verified email/phone
2. ✅ Validated 400 companies against ICP criteria
3. ✅ Identified 150 LMS/VILT platform installations
4. ✅ Maintained <5% budget utilization

## Budget Analysis
- **Budget Allocated**: 5,000 API calls
- **Used This Month**: 200 calls (4%)
- **Running Total**: 600 calls (12%)
- **Remaining**: 4,400 calls (88%)
- **Burn Rate**: ~50 calls/week
- **Projected Total**: ~1,200 calls (24% of budget)

## Admin Portal Verification
- **Portal Total**: 605 API calls
- **Processing_Log Total**: 600 API calls
- **Variance**: 5 calls (0.8%) ✅ Acceptable
- **Monthly Credits Remaining**: 11,500 / 12,000 (96%)

## Recommendations
1. Current usage pace is excellent - well within budget
2. Sufficient capacity for additional enrichment if needed
3. Project on track for <25% budget utilization
4. No concerns or adjustments needed

## Next Month Forecast
- **Remaining Operations**: ~150-200 API calls
- **Projected Total**: ~1,200 calls (24% budget)
- **Buffer**: 3,800 calls (76% unused)
- **Risk Level**: NONE

---
**Report Date**: [Date]
**Next Review**: [Date + 1 month]
```

---

## Alert Templates

### **80% Budget Alert (4,000 calls used)**

```
🚨 ZOOMINFO BUDGET ALERT - IMMEDIATE ACTION REQUIRED

Status: RED - 80% of project budget consumed
Current Usage: 4,000 / 5,000 API calls
Remaining: 1,000 calls

ACTIONS REQUIRED:
1. STOP all non-essential ZoomInfo operations
2. Review remaining operations with user
3. Prioritize only critical enrichments
4. Consider completing project with free sources
5. User approval required for all further calls

Log Entry: [Timestamp] - [Agent] - [Tool] - [Details]
```

### **90% Budget Alert (4,500 calls used)**

```
🚨🚨 EMERGENCY STOP - ZOOMINFO BUDGET CRITICAL 🚨🚨

Status: CRITICAL - 90% of project budget consumed
Current Usage: 4,500 / 5,000 API calls
Remaining: 500 calls

EMERGENCY ACTIONS:
1. FULL STOP on all ZoomInfo operations
2. No further calls without explicit user authorization
3. Generate final usage report
4. Recommend fallback to 100% free sources
5. User decision required: Continue or halt

Log Entry: [Timestamp] - [Agent] - [Tool] - [Details]
```

### **Unexpected Spike Alert (>100 calls in 1 hour)**

```
⚠️ ZOOMINFO USAGE SPIKE DETECTED

Condition: >100 API calls in last 1 hour
Time Window: [Start Time] to [End Time]
Calls Detected: [X] calls
Running Total: [Y] / 5,000 calls

INVESTIGATION REQUIRED:
1. Pause all agents using ZoomInfo
2. Review Processing_Log for this time window
3. Identify operation causing spike
4. Verify running total accuracy
5. Resume only after verification complete

Possible Causes:
- Runaway batch operation
- Pagination without limits
- Duplicate operations
- Error retry loop
```

---

## Implementation Checklist

**For Claude Code operators, complete this checklist:**

### **Setup (One-Time)**

- [ ] Create Processing_Log sheet in Google Sheets
- [ ] Add columns A-H with headers as specified
- [ ] Set up conditional formatting rules
- [ ] Create ZoomInfo_Dashboard sheet
- [ ] Add formulas for key metrics
- [ ] Set up usage charts
- [ ] Verify formulas calculating correctly
- [ ] Document spreadsheet ID in CLAUDE.md

### **Before Each ZoomInfo Operation**

- [ ] Read last entry in Processing_Log
- [ ] Calculate current running total
- [ ] Estimate API calls for planned operation
- [ ] Verify budget sufficient (total + planned < 5,000)
- [ ] Check free sources exhausted
- [ ] Use lookup tool first (if search operation)
- [ ] Proceed with operation

### **After Each ZoomInfo Operation**

- [ ] Log operation to Processing_Log immediately
- [ ] Calculate new running total
- [ ] Update remaining budget
- [ ] Check for alerts (>4,000 calls)
- [ ] Verify formulas updated correctly

### **Weekly Reviews**

- [ ] Review Processing_Log for this week
- [ ] Calculate weekly usage total
- [ ] Check against weekly limit (<2,000 calls)
- [ ] Log into admin portal for verification
- [ ] Generate weekly status report
- [ ] Document any issues or alerts

### **Monthly Reviews**

- [ ] Generate monthly usage report
- [ ] Compare admin portal vs. Processing_Log
- [ ] Calculate burn rate and projections
- [ ] Review budget health
- [ ] Document recommendations
- [ ] Update project timeline if needed

---

**Template End**

**Usage Notes:**
- Copy these templates as needed for tracking and reporting
- Customize formulas based on actual spreadsheet structure
- Adjust alert thresholds if project budget changes
- Maintain consistent logging for accurate tracking
- Reference ZoomInfo_Usage_Guide.md for detailed procedures

**Files:**
- This Template: `docs/ZoomInfo_Tracking_Template.md`
- Full Guide: `docs/ZoomInfo_Usage_Guide.md`
- Project Instructions: `CLAUDE.md` (lines 516-788)
