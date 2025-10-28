# ZoomInfo MCP - Quick Reference Card

**Last Updated**: 2025-10-28 (Generous Limits)
**Contract Period**: July 10, 2025 - July 9, 2026
**Project Budget**: 250,000 API calls (10% of annual capacity)

---

## 🚨 Critical Numbers

| Limit Type | Value | Notes |
|------------|-------|-------|
| **Annual API Limit** | 2,500,000 calls | Fair Use Policy |
| **Overage Cost** | $10,000 per 1M excess | CRITICAL RISK (unlikely) |
| **Project Budget** | 250,000 calls | 10% of annual |
| **Daily Limit** | 5,000 calls | Safety guardrail |
| **Weekly Limit** | 25,000 calls | Safety guardrail |
| **Alert Threshold** | 200,000 calls | 80% warning |
| **Emergency Review** | 225,000 calls | 90% review point |
| **Monthly Credits** | 12,000 | Resets 1st of month |

---

## ✅ Pre-Flight Checklist (MANDATORY)

**Before EVERY ZoomInfo operation:**

```
[ ] 1. Check Processing_Log running total < 250,000
[ ] 2. Calculate planned API calls: [records] ÷ 10
[ ] 3. Verify: running_total + planned < 250,000
[ ] 4. Confirm approach aligned with tier priorities
[ ] 5. Use lookup tool FIRST (if search operation)
```

---

## 🎯 Approved Use Cases

### **Tier 1: Contact Enrichment - Agent 5 (50% budget = 125,000 calls)**
- ✅ PRIMARY: Mass contact enrichment (5,000-20,000 contacts)
- ✅ Targeted L&D leadership discovery (CLO, VP Learning, Dean)
- ✅ Batch operations (10 contacts per call)
- ✅ Example: 10,000 contacts = 1,000 API calls

### **Tier 2: Targeted Contact Discovery - Agent 5 (Included in Tier 1)**
- ✅ Direct ZoomInfo searches for specific L&D roles
- ✅ Search for CLOs at Fortune 500 companies
- ✅ Example: 1,000 searches × 2 pages = 2,000 API calls

### **Tier 3: Tech Stack Discovery - Agent 2 (20% budget = 50,000 calls)**
- ✅ Comprehensive LMS/VILT platform identification
- ✅ Identify tech stack for all confirmed academies
- ✅ Example: 2,000 companies = 2,000 API calls

### **Tier 4: Company Validation - Orchestrator (20% budget = 50,000 calls)**
- ✅ Batch validate all Consolidated_Academies
- ✅ Verify employee counts, revenue, industry
- ✅ Example: 5,000 companies = 500 API calls (batch)

### **Tier 5: Awards Winner Validation - Agent 1 (10% budget = 25,000 calls)**
- ✅ Enrich award winner company data
- ✅ Validate companies from Brandon Hall, Training Top 125, etc.
- ✅ Example: 2,500 companies = 250 API calls (batch)

---

## ❌ Prohibited Use Cases

**NEVER use ZoomInfo for:**
- ❌ Re-enriching same records (duplicate work)
- ❌ Operations without logging to Processing_Log

---

## 📊 API Call Costs

| Tool | Operation | API Calls |
|------|-----------|-----------|
| `lookup` | Field values | 0.1 |
| `search_contacts` | 1 page (25 results) | 1 |
| `search_companies` | 1 page (25 results) | 1 |
| `enrich_contact` | **Single** | 1 |
| `enrich_contact` | **Batch (10)** | 1 ✅ |
| `enrich_company` | **Single** | 1 |
| `enrich_company` | **Batch (10)** | 1 ✅ |

**Always batch for 10x efficiency!**

---

## 🔍 Quick Workflow Pattern

```python
# 1. CHECK BUDGET
running_total = get_last_entry("Processing_Log")
if running_total + planned_calls > 250000:
    ABORT("Insufficient budget")

# 2. USE LOOKUP FIRST
lookup(fieldNames=["management-levels", "industries"])

# 3. BATCH OPERATION (10 records per call)
enrich_contact({
    "contacts": [10 contacts with valid IDs]
})

# 4. LOG IMMEDIATELY
add_rows("Processing_Log", [[
    timestamp, "ZoomInfo", tool_name,
    records_affected, api_calls_used,
    running_total, remaining_budget, notes
]])

# 5. CHECK ALERTS
if running_total >= 200000:
    ALERT("Review usage with user!")
```

---

## 🚦 Status Thresholds

| Usage | Status | Action |
|-------|--------|--------|
| 0-150,000 calls (0-60%) | 🟢 GREEN | Continue normal operations |
| 150,000-200,000 calls (60-80%) | 🟡 YELLOW | Monitor closely, prioritize |
| 200,000-225,000 calls (80-90%) | 🔴 RED | ALERT user, review remaining |
| 225,000+ calls (90-100%) | 🚨 CRITICAL | STOP, require approval |

---

## 📝 Processing_Log Format

| Column | Field | Example |
|--------|-------|---------|
| A | Timestamp | 2025-10-28T14:30:15 |
| B | Agent_Name | "ZoomInfo" |
| C | Tool_Name | "enrich_contact" |
| D | Records_Affected | 10 |
| E | API_Calls_Used | 1 |
| F | Running_Total | 12,450 |
| G | Remaining_Budget | 237,550 |
| H | Operation_Notes | "Batch 1/20: Agent 5 contacts" |

---

## 🎛️ Agent-Specific Budgets

| Agent | Approved Budget | Primary Use Case |
|-------|----------------|------------------|
| **Agent 1** | 25,000 calls (10%) | Award winner validation |
| **Agent 2** | 50,000 calls (20%) | Tech stack discovery |
| **Agent 5** | 125,000 calls (50%) | Contact enrichment |
| **Orchestrator** | 50,000 calls (20%) | Batch validation |
| **TOTAL** | **250,000 calls** | **100% allocated** |

---

## ⚡ Quick Commands

**Check Current Usage:**
```bash
# Read last ZoomInfo entry in Processing_Log
running_total = [Last value in Column F]
remaining = 250000 - running_total
percentage = (running_total / 250000) * 100
```

**Calculate Batch Cost:**
```bash
# For contact/company enrichment
batches = CEILING(records / 10)
api_calls = batches * 1
```

**Verify Budget:**
```bash
if running_total + planned_calls <= 250000:
    PROCEED
elif running_total + planned_calls <= 225000:
    REVIEW_WITH_USER
else:
    ABORT
```

---

## 📞 Emergency Contacts

**Admin Portal**: https://admin.zoominfo.com/#/api-and-webhooks
**Contract PDF**: `/Users/russellteter/Desktop/20252026 Zoominfo.pdf`
**Full Guide**: `docs/ZoomInfo_Usage_Guide.md`
**Tracking Template**: `docs/ZoomInfo_Tracking_Template.md`
**Project Instructions**: `CLAUDE.md` (lines 609-875)

---

## 🆘 Emergency Procedures

**If running_total >= 200,000 calls:**
1. 🛑 PAUSE non-essential ZoomInfo operations
2. 📊 Generate usage report
3. 👤 Alert user immediately
4. ⏸️ Review remaining operations and priorities

**If ZoomInfo API errors occur:**
1. 🛑 STOP operations
2. 📝 Log error to Processing_Log
3. ⏰ Wait 1 hour before retry
4. 🔍 Investigate cause

**If budget exhausted (250,000 calls):**
1. 🛑 HARD STOP - no further calls
2. ✅ Complete remaining work with free sources
3. 📄 Document usage in execution summary
4. 💡 Recommend additional budget for next phase if needed

---

## 💡 Best Practices

1. **Always lookup before search** - Prevents wasted API calls
2. **Batch everything** - 10x more efficient than individual calls
3. **Free sources first** - ZoomInfo for enrichment and mass operations
4. **Log immediately** - Don't buffer, write to Processing_Log after each operation
5. **Monitor daily** - Check running total, stay under 5,000 calls/day
6. **Verify weekly** - Compare Processing_Log vs admin portal

---

## 📈 Project Cost Projection

**Conservative (Recommended):**
- Contact enrichment: 50,000 calls (10,000 contacts batch)
- Company validation: 25,000 calls (2,500 companies batch)
- Tech stack discovery: 25,000 calls (2,500 companies)
- Targeted searches: 25,000 calls (comprehensive)
- **TOTAL: ~125,000 calls (50% of budget)**

**Maximum Safe:**
- Contact enrichment: 100,000 calls (20,000 contacts batch)
- Company validation: 50,000 calls (5,000 companies batch)
- Tech stack discovery: 40,000 calls (4,000 companies)
- Targeted searches: 40,000 calls (deep research)
- **TOTAL: ~230,000 calls (92% of budget)**

**SUCCESS**: Project can use up to 250,000 calls (10% of annual)

---

## 🔗 Related Documentation

- **Full Usage Guide**: `docs/ZoomInfo_Usage_Guide.md` (comprehensive 15-page guide)
- **Tracking Template**: `docs/ZoomInfo_Tracking_Template.md` (Google Sheets setup)
- **Project Instructions**: `CLAUDE.md` (integrated agent specifications)
- **Contract Reference**: `/Users/russellteter/Desktop/20252026 Zoominfo.pdf`

---

**Remember**: This is a SAFETY SYSTEM, not a RESTRICTION.
You have 2.5M annual calls - this project uses 10% maximum.
The guardrails prevent ACCIDENTS, not normal usage.

✅ **Current Status**: READY FOR USE
🔒 **Authentication**: Complete via `/mcp` command
🛡️ **Guardrails**: ACTIVE and ENFORCED
💡 **Budget**: Generous limits for comprehensive operations
