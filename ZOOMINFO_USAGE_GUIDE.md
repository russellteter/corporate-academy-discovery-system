# ZoomInfo MCP Usage Guide - Corporate Academy Discovery System

**Version:** 1.0
**Date:** 2025-10-28
**Scope:** Strategic enrichment layer for hybrid methodology
**Project Budget:** 5,000 API calls (0.2% of annual capacity)

---

## Table of Contents

1. [Quick Reference](#quick-reference)
2. [MCP Tool Documentation](#mcp-tool-documentation)
3. [Agent-Specific Workflows](#agent-specific-workflows)
4. [Batch Processing Templates](#batch-processing-templates)
5. [Pre-Flight Check Implementation](#pre-flight-check-implementation)
6. [Tracking & Logging](#tracking--logging)
7. [Error Handling](#error-handling)
8. [Weekly Reporting](#weekly-reporting)
9. [Emergency Procedures](#emergency-procedures)
10. [Code Examples](#code-examples)

---

## Quick Reference

### Essential Rules

**ALWAYS:**
- ✅ Use lookup before enrichment (may be free)
- ✅ Batch operations (10 records = 1 API call)
- ✅ Log every operation to Processing_Log
- ✅ Run pre-flight check before each operation
- ✅ Exhaust free sources first

**NEVER:**
- ❌ Skip pre-flight checks
- ❌ Enrich individually when batching possible
- ❌ Use ZoomInfo for primary discovery
- ❌ Ignore budget warnings
- ❌ Continue after emergency stop

### Budget Quick Status

| Zone | Range | Action |
|------|-------|--------|
| 🟢 GREEN | 0-3,000 calls | Normal operations |
| 🟡 YELLOW | 3,000-4,000 calls | Monitor closely |
| 🔴 RED | 4,000-4,500 calls | Alert user, require approval |
| 🚨 CRITICAL | 4,500-5,000 calls | Emergency stop |

### Tool Cost Quick Reference

| Tool | Cost | Batch Support | Notes |
|------|------|---------------|-------|
| lookup | 0-1 calls | Yes (10 records) | Try first, may be free |
| enrich_contact | 1 call | Yes (10 records) | Email, phone, title |
| enrich_company | 1 call | Yes (10 records) | Employee count, revenue, tech |
| search_contacts | 1 call | No | Returns up to 100 results |
| search_companies | 1 call | No | Returns up to 100 results |

---

## MCP Tool Documentation

### mcp__zoominfo__lookup

**Purpose:** Check if record exists in ZoomInfo before enrichment

**Cost:** 0 API calls if no match, 1 call if match found

**When to Use:** MANDATORY first step before any enrichment operation

**Parameters:**
```python
{
  "records": [
    {
      "first_name": "John",
      "last_name": "Smith",
      "company_name": "Walmart Inc"
    }
  ]
}
```

**Response:**
```python
{
  "matches": [
    {
      "record_id": "zi_abc123",
      "confidence": 0.95,
      "match_found": true
    }
  ],
  "api_calls_used": 1  # Only if match found
}
```

**Usage Pattern:**
```python
# Step 1: Always lookup first
lookup_results = mcp__zoominfo__lookup(
    records=[{
        "first_name": "John",
        "last_name": "Smith",
        "company_name": "Walmart"
    }]
)

# Step 2: Only enrich if match found
if lookup_results["matches"][0]["match_found"]:
    enrich_results = mcp__zoominfo__enrich_contact(
        record_ids=[lookup_results["matches"][0]["record_id"]]
    )
```

**Best Practices:**
- Always batch lookups (10 records at once)
- Check match confidence (0.8+ recommended)
- Log lookup results even if no match
- Don't enrich without successful lookup

---

### mcp__zoominfo__enrich_contact

**Purpose:** Add email, phone, title details to contact records

**Cost:** 1 API call per batch (up to 10 records)

**When to Use:** After successful lookup, for contacts discovered via free sources

**Parameters:**
```python
{
  "record_ids": ["zi_abc123", "zi_def456", ...],  # Max 10
  "fields": ["email", "phone", "title", "department"]
}
```

**Response:**
```python
{
  "enriched_records": [
    {
      "record_id": "zi_abc123",
      "email": "john.smith@walmart.com",
      "phone": "+1-555-123-4567",
      "title": "Chief Learning Officer",
      "department": "Human Resources"
    }
  ],
  "api_calls_used": 1  # For entire batch
}
```

**Usage Pattern:**
```python
# WRONG: Individual enrichment (10 contacts = 10 API calls)
for contact in contacts:
    enrich_contact(contact)  # DON'T DO THIS

# RIGHT: Batch enrichment (10 contacts = 1 API call)
for batch in chunks(contacts, 10):
    enrich_contact_batch(batch)  # DO THIS
```

**Best Practices:**
- ALWAYS batch (10 records per call)
- Request only needed fields
- Validate record_ids from lookup first
- Log both batch size and API calls used

---

### mcp__zoominfo__enrich_company

**Purpose:** Add employee count, revenue, tech stack to company records

**Cost:** 1 API call per batch (up to 10 records)

**When to Use:** For tech stack discovery (Tier 3) or company validation (Tier 2)

**Parameters:**
```python
{
  "company_ids": ["zi_comp_123", "zi_comp_456", ...],  # Max 10
  "fields": ["employee_count", "revenue", "technologies", "industry"]
}
```

**Response:**
```python
{
  "enriched_companies": [
    {
      "company_id": "zi_comp_123",
      "employee_count": 25000,
      "revenue": "$500M-$1B",
      "technologies": ["Zoom", "Cornerstone", "SAP SuccessFactors"],
      "industry": "Retail"
    }
  ],
  "api_calls_used": 1
}
```

**Usage Pattern:**
```python
# Tech stack discovery workflow
companies_needing_tech_stack = get_companies_without_lms()

for batch in chunks(companies_needing_tech_stack, 10):
    # Pre-flight check
    if not zoominfo_preflight_check("enrich_company", 1):
        break  # Budget limit reached

    # Enrich batch
    results = enrich_company(batch, fields=["technologies"])

    # Log operation
    log_zoominfo_operation("enrich_company", len(batch), 1)
```

**Best Practices:**
- Use for Tier 3 (tech stack) priority
- Batch 10 companies per call
- Request technologies field for VILT platform identification
- Validate employee_count > 7,500 for ICP match

---

### mcp__zoominfo__search_contacts

**Purpose:** Find contacts by search criteria (last resort)

**Cost:** 1 API call per search query

**When to Use:** Tier 4 only - fill gaps after free sources exhausted

**Parameters:**
```python
{
  "criteria": {
    "job_title": "Chief Learning Officer",
    "company_name": "Walmart",
    "location": "United States",
    "employee_count_min": 7500
  },
  "limit": 100  # Max results per query
}
```

**Response:**
```python
{
  "contacts": [
    {
      "first_name": "John",
      "last_name": "Smith",
      "title": "Chief Learning Officer",
      "company": "Walmart Inc",
      "email": "john.smith@walmart.com",
      "phone": "+1-555-123-4567"
    }
  ],
  "total_results": 1,
  "api_calls_used": 1
}
```

**Usage Pattern:**
```python
# ONLY use when free sources fail to find contacts
if linkedin_via_google_returned_zero_results():
    # Pre-flight check (Tier 4 has 500 call budget)
    if not zoominfo_preflight_check("search_contacts", 1):
        return []  # Budget exhausted

    # Search as last resort
    results = search_contacts(
        job_title="Chief Learning Officer",
        company_name=company_name
    )

    # Log operation with justification
    log_zoominfo_operation(
        "search_contacts",
        len(results["contacts"]),
        1,
        justification="LinkedIn via Google returned 0 results"
    )
```

**Best Practices:**
- Use ONLY as last resort (Tier 4 = 10% budget)
- Require explicit justification in log
- Verify free sources truly exhausted
- Limit searches to 10 per week max

---

### mcp__zoominfo__search_companies

**Purpose:** Find companies by search criteria (discovery supplement)

**Cost:** 1 API call per search query

**When to Use:** Tier 4 only - supplement free discovery, not replace

**Parameters:**
```python
{
  "criteria": {
    "industry": "Financial Services",
    "employee_count_min": 7500,
    "technologies": ["Zoom", "Cornerstone"],
    "keywords": "corporate academy OR corporate university"
  },
  "limit": 100
}
```

**Response:**
```python
{
  "companies": [
    {
      "company_name": "JPMorgan Chase",
      "employee_count": 250000,
      "industry": "Banking",
      "technologies": ["Zoom", "Cornerstone", "Docebo"],
      "website": "jpmorganchase.com"
    }
  ],
  "total_results": 1,
  "api_calls_used": 1
}
```

**Usage Pattern:**
```python
# Use to supplement Agent 3 Fortune 1000 sweep
fortune_1000_discoveries = agent_3_results()

# Identify gaps (industries with <50 discoveries)
underrepresented_industries = identify_gaps(fortune_1000_discoveries)

for industry in underrepresented_industries:
    # Pre-flight check
    if not zoominfo_preflight_check("search_companies", 1):
        break

    # Targeted search to fill gaps
    results = search_companies(
        industry=industry,
        employee_count_min=7500,
        keywords="corporate academy"
    )

    log_zoominfo_operation(
        "search_companies",
        len(results["companies"]),
        1,
        justification=f"Fill gap in {industry} (only {gap_count} discoveries)"
    )
```

**Best Practices:**
- Use to fill gaps, not replace free discovery
- Target specific industries with low free discovery yield
- Maximum 10 searches per project
- Document gap justification in every search log

---

## Agent-Specific Workflows

### Agent 1: Award Program Validation

**Budget:** 500 API calls (10% of project)

**Use Case:** Validate award winner company details after free discovery

**Workflow:**

```python
def agent_1_zoominfo_workflow():
    """Agent 1: Validate award winners using ZoomInfo"""

    # Step 1: Free discovery (PRIMARY)
    award_winners = scrape_brandon_hall_winners()  # Free
    award_winners += scrape_training_top_125()     # Free
    award_winners += scrape_clo_learningelite()    # Free

    print(f"Found {len(award_winners)} award winners (free sources)")

    # Step 2: Filter to companies needing validation
    needs_validation = [
        company for company in award_winners
        if company.employee_count is None  # Missing data
    ]

    print(f"{len(needs_validation)} companies need validation")

    # Step 3: Pre-flight check
    estimated_calls = len(needs_validation) / 10  # Batch size
    if not zoominfo_preflight_check("enrich_company", estimated_calls):
        print("Budget limit - skipping validation")
        return award_winners  # Return free data without validation

    # Step 4: Batch validation
    validated = []
    for batch in chunks(needs_validation, 10):
        # Lookup first
        lookup_results = zoominfo_lookup([
            {"company_name": c.name} for c in batch
        ])

        # Enrich matched companies
        matched_ids = [
            r["record_id"] for r in lookup_results["matches"]
            if r["match_found"] and r["confidence"] > 0.8
        ]

        if matched_ids:
            enriched = zoominfo_enrich_company(
                company_ids=matched_ids,
                fields=["employee_count", "industry"]
            )
            validated.extend(enriched["enriched_companies"])

            # Log operation
            log_zoominfo_operation(
                "enrich_company",
                len(matched_ids),
                1,
                agent="Agent_1"
            )

        time.sleep(1)  # Rate limiting courtesy

    print(f"Validated {len(validated)} companies via ZoomInfo")

    # Step 5: Merge free + validated data
    final_results = merge_award_data(award_winners, validated)

    return final_results
```

**Key Points:**
- Free sources are PRIMARY (80-90%)
- ZoomInfo validates missing data only
- Batch size: 10 companies per call
- Maximum 50 companies validated per day
- Budget allocation: 500 API calls total

---

### Agent 2: LMS Customer Validation

**Budget:** 500 API calls (10% of project)

**Use Case:** Verify LMS customer company details

**Workflow:**

```python
def agent_2_zoominfo_workflow():
    """Agent 2: Validate LMS customers using ZoomInfo"""

    # Step 1: Free discovery (PRIMARY)
    lms_customers = []
    lms_customers += scrape_cornerstone_customers()  # Free
    lms_customers += scrape_docebo_customers()       # Free
    lms_customers += scrape_sap_sf_customers()       # Free

    print(f"Found {len(lms_customers)} LMS customers (free sources)")

    # Step 2: Filter companies needing employee count verification
    needs_validation = [
        company for company in lms_customers
        if company.employee_count is None or
           company.employee_count < 7500  # ICP threshold
    ]

    # Step 3: Pre-flight check
    estimated_calls = len(needs_validation) / 10
    if not zoominfo_preflight_check("enrich_company", estimated_calls):
        return lms_customers  # Return without validation

    # Step 4: Batch validation (employee count focus)
    validated = []
    for batch in chunks(needs_validation, 10):
        lookup_results = zoominfo_lookup([
            {"company_name": c.name} for c in batch
        ])

        matched_ids = [
            r["record_id"] for r in lookup_results["matches"]
            if r["match_found"]
        ]

        if matched_ids:
            enriched = zoominfo_enrich_company(
                company_ids=matched_ids,
                fields=["employee_count"]  # Only need employee count
            )

            # Filter: Keep only 7,500+ employees (ICP match)
            icp_matches = [
                c for c in enriched["enriched_companies"]
                if c["employee_count"] >= 7500
            ]

            validated.extend(icp_matches)
            log_zoominfo_operation("enrich_company", len(matched_ids), 1, agent="Agent_2")

    print(f"Validated {len(validated)} ICP matches via ZoomInfo")

    # Step 5: Return only ICP-qualified companies
    final_results = merge_lms_data(lms_customers, validated)
    icp_qualified = [c for c in final_results if c.employee_count >= 7500]

    return icp_qualified
```

**Key Points:**
- Validates employee count for ICP match (7,500+)
- Filters out non-ICP companies early
- Only enriches employee_count field (cost-efficient)
- Maximum 50 companies per day

---

### Agent 5: Contact Enrichment (PRIMARY ZOOMINFO USE)

**Budget:** 2,000 API calls (40% of project - HIGHEST)

**Use Case:** Add email/phone to contacts discovered via free sources

**Workflow:**

```python
def agent_5_zoominfo_workflow(company_name):
    """Agent 5: Enrich contacts with email/phone using ZoomInfo"""

    # Step 1: Free discovery via LinkedIn/Google (PRIMARY)
    contacts = []

    # Search patterns
    searches = [
        f'site:linkedin.com/in "Chief Learning Officer" {company_name}',
        f'site:linkedin.com/in "VP Learning" OR "VP L&D" {company_name}',
        f'site:linkedin.com/in "Director Learning" {company_name}',
        f'site:linkedin.com/in "Head of Academy" {company_name}'
    ]

    for query in searches:
        results = google_search_with_delay(query)  # Free
        profiles = extract_linkedin_profiles(results)
        contacts.extend(profiles)
        time.sleep(random.uniform(2, 4))  # Rate limiting

    print(f"Found {len(contacts)} contacts via LinkedIn/Google (free)")

    # Step 2: De-duplicate
    unique_contacts = deduplicate_by_name_company(contacts)
    print(f"{len(unique_contacts)} unique contacts after dedup")

    # Step 3: Pre-flight check
    estimated_calls = len(unique_contacts) / 10  # Batch size
    if not zoominfo_preflight_check("enrich_contact", estimated_calls):
        print("⚠️ Budget limit - returning contacts without enrichment")
        return unique_contacts  # Return names/titles only

    # Step 4: Batch enrichment (email/phone)
    enriched_contacts = []

    for batch in chunks(unique_contacts, 10):
        # Lookup first (mandatory)
        lookup_results = zoominfo_lookup([
            {
                "first_name": c.first_name,
                "last_name": c.last_name,
                "company_name": c.company
            }
            for c in batch
        ])

        # Filter high-confidence matches
        matched_ids = [
            r["record_id"] for r in lookup_results["matches"]
            if r["match_found"] and r["confidence"] >= 0.85
        ]

        if matched_ids:
            # Enrich batch (1 API call for up to 10 contacts)
            enriched = zoominfo_enrich_contact(
                record_ids=matched_ids,
                fields=["email", "phone", "title"]
            )

            enriched_contacts.extend(enriched["enriched_records"])

            # Log operation
            log_zoominfo_operation(
                "enrich_contact",
                len(matched_ids),
                1,
                agent="Agent_5"
            )

        time.sleep(1)  # Rate limiting courtesy

    print(f"✅ Enriched {len(enriched_contacts)} contacts with email/phone")

    # Step 5: Merge free data with enriched data
    final_contacts = merge_contact_data(unique_contacts, enriched_contacts)

    # Step 6: Quality check
    with_email = [c for c in final_contacts if c.email]
    print(f"📊 {len(with_email)}/{len(final_contacts)} contacts have email ({len(with_email)/len(final_contacts)*100:.1f}%)")

    return final_contacts
```

**Key Points:**
- FREE discovery finds contacts (LinkedIn via Google)
- ZoomInfo ADDS email/phone to existing contacts
- Batch size: 10 contacts per API call
- Maximum 200 contacts per day (20 batches)
- Budget: 2,000 API calls (highest allocation)
- **This is the PRIMARY ZoomInfo use case**

---

### Orchestrator: Tech Stack Discovery

**Budget:** 1,500 API calls (30% of project)

**Use Case:** Identify VILT platforms (Zoom/Teams/other) for targeting

**Workflow:**

```python
def orchestrator_tech_stack_workflow():
    """Orchestrator: Discover tech stacks via ZoomInfo"""

    # Step 1: Get companies without tech stack data
    all_companies = get_all_discovered_companies()

    missing_tech_stack = [
        company for company in all_companies
        if company.vilt_platform is None and
           company.lms_platform is None
    ]

    print(f"{len(missing_tech_stack)} companies missing tech stack data")

    # Step 2: Pre-flight check
    estimated_calls = len(missing_tech_stack) / 10
    if not zoominfo_preflight_check("enrich_company", estimated_calls):
        return all_companies  # Skip tech stack discovery

    # Step 3: Batch tech stack enrichment
    tech_stacks = []

    for batch in chunks(missing_tech_stack, 10):
        lookup_results = zoominfo_lookup([
            {"company_name": c.name} for c in batch
        ])

        matched_ids = [
            r["record_id"] for r in lookup_results["matches"]
            if r["match_found"]
        ]

        if matched_ids:
            # Request only technologies field
            enriched = zoominfo_enrich_company(
                company_ids=matched_ids,
                fields=["technologies"]  # Tech stack only
            )

            tech_stacks.extend(enriched["enriched_companies"])
            log_zoominfo_operation("enrich_company", len(matched_ids), 1, agent="Orchestrator")

    # Step 4: Parse tech stacks for VILT platforms
    for company in tech_stacks:
        technologies = company.get("technologies", [])

        # Identify VILT platform
        if "Zoom" in technologies:
            company.vilt_platform = "Zoom"
        elif "Microsoft Teams" in technologies:
            company.vilt_platform = "Microsoft Teams"
        elif "Adobe Connect" in technologies:
            company.vilt_platform = "Adobe Connect"
        elif "Webex" in technologies:
            company.vilt_platform = "Webex"

        # Identify LMS
        if "Cornerstone" in technologies:
            company.lms_platform = "Cornerstone"
        elif "Docebo" in technologies:
            company.lms_platform = "Docebo"
        elif "SAP SuccessFactors" in technologies:
            company.lms_platform = "SAP SuccessFactors"

    print(f"✅ Discovered tech stacks for {len(tech_stacks)} companies")

    # Step 5: Merge and prioritize
    final_companies = merge_tech_stack_data(all_companies, tech_stacks)

    # Step 6: Prioritization report
    zoom_users = [c for c in final_companies if c.vilt_platform == "Zoom"]
    teams_users = [c for c in final_companies if c.vilt_platform == "Microsoft Teams"]

    print(f"🎯 Targeting Report:")
    print(f"  - Zoom users: {len(zoom_users)} companies (PRIMARY)")
    print(f"  - Teams users: {len(teams_users)} companies (SECONDARY)")

    return final_companies
```

**Key Points:**
- Discovers VILT platforms for targeting
- Identifies Zoom users (primary target)
- Maximum 150 companies per day
- Budget: 1,500 API calls (30% allocation)
- Enables strategic prioritization

---

## Batch Processing Templates

### Template 1: Contact Enrichment Batch

```python
def batch_enrich_contacts(contacts, batch_size=10):
    """
    Enrich contacts in batches with full safety checks

    Args:
        contacts: List of contact dicts with first_name, last_name, company
        batch_size: Records per batch (max 10)

    Returns:
        List of enriched contact dicts with email, phone, title
    """
    enriched_results = []

    # Pre-flight check
    estimated_calls = math.ceil(len(contacts) / batch_size)
    if not zoominfo_preflight_check("enrich_contact", estimated_calls):
        print(f"⚠️ Budget insufficient for {estimated_calls} calls")
        return []

    # Process batches
    for i in range(0, len(contacts), batch_size):
        batch = contacts[i:i+batch_size]

        print(f"Processing batch {i//batch_size + 1} ({len(batch)} contacts)...")

        # Step 1: Lookup
        lookup_results = zoominfo_lookup(batch)

        # Step 2: Filter high-confidence matches
        matched_ids = [
            r["record_id"] for r in lookup_results["matches"]
            if r["match_found"] and r["confidence"] >= 0.80
        ]

        if not matched_ids:
            print(f"  No matches found in batch")
            continue

        # Step 3: Enrich
        enriched = zoominfo_enrich_contact(
            record_ids=matched_ids,
            fields=["email", "phone", "title", "department"]
        )

        enriched_results.extend(enriched["enriched_records"])

        # Step 4: Log
        log_zoominfo_operation(
            tool_name="enrich_contact",
            records_affected=len(matched_ids),
            api_calls_used=1,
            batch_size=len(batch)
        )

        print(f"  ✅ Enriched {len(matched_ids)} contacts (1 API call)")

        # Rate limiting
        time.sleep(1)

    print(f"\n✅ Total enriched: {len(enriched_results)} contacts")
    print(f"✅ Total API calls: {math.ceil(len(contacts) / batch_size)}")

    return enriched_results
```

### Template 2: Company Validation Batch

```python
def batch_validate_companies(companies, batch_size=10):
    """
    Validate companies in batches with employee count check

    Args:
        companies: List of company dicts with name
        batch_size: Records per batch (max 10)

    Returns:
        List of ICP-qualified companies (7,500+ employees)
    """
    validated_results = []

    # Pre-flight check
    estimated_calls = math.ceil(len(companies) / batch_size)
    if not zoominfo_preflight_check("enrich_company", estimated_calls):
        return []

    # Process batches
    for i in range(0, len(companies), batch_size):
        batch = companies[i:i+batch_size]

        # Lookup
        lookup_results = zoominfo_lookup([
            {"company_name": c["name"]} for c in batch
        ])

        matched_ids = [
            r["record_id"] for r in lookup_results["matches"]
            if r["match_found"]
        ]

        if matched_ids:
            # Enrich
            enriched = zoominfo_enrich_company(
                company_ids=matched_ids,
                fields=["employee_count", "industry", "revenue"]
            )

            # Filter ICP matches (7,500+ employees)
            icp_matches = [
                c for c in enriched["enriched_companies"]
                if c["employee_count"] >= 7500
            ]

            validated_results.extend(icp_matches)

            # Log
            log_zoominfo_operation(
                "enrich_company",
                len(matched_ids),
                1,
                icp_matches=len(icp_matches)
            )

            print(f"  ✅ Validated {len(icp_matches)}/{len(matched_ids)} ICP matches")

        time.sleep(1)

    return validated_results
```

### Template 3: Tech Stack Discovery Batch

```python
def batch_discover_tech_stacks(companies, batch_size=10):
    """
    Discover VILT/LMS tech stacks in batches

    Args:
        companies: List of company dicts
        batch_size: Records per batch (max 10)

    Returns:
        List of companies with tech_stack field populated
    """
    tech_stack_results = []

    # Pre-flight check
    estimated_calls = math.ceil(len(companies) / batch_size)
    if not zoominfo_preflight_check("enrich_company", estimated_calls):
        return []

    # Process batches
    for i in range(0, len(companies), batch_size):
        batch = companies[i:i+batch_size]

        # Lookup
        lookup_results = zoominfo_lookup([
            {"company_name": c["name"]} for c in batch
        ])

        matched_ids = [
            r["record_id"] for r in lookup_results["matches"]
            if r["match_found"]
        ]

        if matched_ids:
            # Enrich (technologies only)
            enriched = zoominfo_enrich_company(
                company_ids=matched_ids,
                fields=["technologies"]
            )

            # Parse tech stacks
            for company in enriched["enriched_companies"]:
                tech_list = company.get("technologies", [])

                company["vilt_platform"] = identify_vilt_platform(tech_list)
                company["lms_platform"] = identify_lms_platform(tech_list)

            tech_stack_results.extend(enriched["enriched_companies"])

            # Log
            log_zoominfo_operation("enrich_company", len(matched_ids), 1)

            print(f"  ✅ Discovered tech stacks for {len(matched_ids)} companies")

        time.sleep(1)

    return tech_stack_results


def identify_vilt_platform(tech_list):
    """Parse technology list for VILT platform"""
    if "Zoom" in tech_list:
        return "Zoom"
    elif "Microsoft Teams" in tech_list:
        return "Microsoft Teams"
    elif "Adobe Connect" in tech_list:
        return "Adobe Connect"
    elif "Webex" in tech_list:
        return "Webex"
    else:
        return "Unknown"


def identify_lms_platform(tech_list):
    """Parse technology list for LMS platform"""
    if "Cornerstone" in tech_list:
        return "Cornerstone"
    elif "Docebo" in tech_list:
        return "Docebo"
    elif "SAP SuccessFactors" in tech_list:
        return "SAP SuccessFactors"
    elif "Workday" in tech_list:
        return "Workday Learning"
    else:
        return "Unknown"
```

---

## Pre-Flight Check Implementation

### Complete Pre-Flight Check Function

```python
import time
from datetime import datetime

# Global state
ZOOMINFO_ENABLED = True
EMERGENCY_STOP_TRIGGERED = False

def zoominfo_preflight_check(operation_name, estimated_calls, agent_name="Unknown"):
    """
    Mandatory pre-flight safety check before ANY ZoomInfo operation

    Args:
        operation_name: Tool name (enrich_contact, enrich_company, search_contacts, etc.)
        estimated_calls: Number of API calls operation will consume
        agent_name: Name of agent requesting operation

    Returns:
        bool: True if operation approved, False if denied

    Raises:
        EmergencyStopException: If budget in CRITICAL zone (90%+)
    """

    # Check 1: Emergency stop flag
    if EMERGENCY_STOP_TRIGGERED:
        print("🚨 EMERGENCY STOP ACTIVE - All ZoomInfo operations halted")
        return False

    if not ZOOMINFO_ENABLED:
        print("⚠️ ZoomInfo disabled - skipping operation")
        return False

    # Check 2: Get current budget status
    current_usage = get_total_api_calls_from_log()
    remaining = 5000 - current_usage
    percentage = (current_usage / 5000) * 100

    # Check 3: Calculate post-operation status
    planned_total = current_usage + estimated_calls
    planned_percentage = (planned_total / 5000) * 100

    # Check 4: CRITICAL zone (90%+) - Emergency stop
    if current_usage >= 4500:
        trigger_emergency_stop(
            current_usage=current_usage,
            attempted_operation=operation_name,
            agent=agent_name
        )
        raise EmergencyStopException("🚨 CRITICAL: 90% budget consumed - operations halted")

    # Check 5: Would operation exceed budget?
    if planned_total > 5000:
        print(f"❌ DENIED: Operation would exceed budget")
        print(f"   Current: {current_usage} calls ({percentage:.1f}%)")
        print(f"   Requested: {estimated_calls} calls")
        print(f"   Total: {planned_total}/5,000 (OVER BUDGET)")

        log_to_sheets("Processing_Log", [[
            timestamp(),
            f"ZI_PREFLIGHT_DENIED",
            f"{operation_name} denied: {planned_total}/5000 would exceed budget",
            "🔴 RED"
        ]])

        return False

    # Check 6: RED zone (80-90%) - Require user approval
    if current_usage >= 4000:
        print(f"🔴 RED ZONE: {percentage:.1f}% budget consumed")
        print(f"   Operation: {operation_name} ({estimated_calls} calls)")
        print(f"   ⚠️ REQUIRING USER APPROVAL")

        # Log warning
        log_to_sheets("Processing_Log", [[
            timestamp(),
            "ZI_RED_ZONE_WARNING",
            f"{operation_name} requested in RED zone - user approval required",
            "🔴 RED"
        ]])

        # In automated mode, deny. In manual mode, prompt user.
        # For this project, default to deny in RED zone
        return False

    # Check 7: YELLOW zone (60-80%) - Verify free alternatives exhausted
    if current_usage >= 3000:
        print(f"🟡 YELLOW ZONE: {percentage:.1f}% budget consumed")

        if not verify_free_alternatives_exhausted(operation_name, agent_name):
            print(f"❌ DENIED: Free alternatives not exhausted")
            log_to_sheets("Processing_Log", [[
                timestamp(),
                "ZI_PREFLIGHT_DENIED",
                f"{operation_name} denied: Free alternatives available in YELLOW zone",
                "🟡 YELLOW"
            ]])
            return False

    # Check 8: Verify lookup attempted first (for enrichment operations)
    if operation_name in ["enrich_contact", "enrich_company"]:
        if not verify_lookup_attempted():
            print(f"❌ DENIED: Must attempt lookup before enrichment")
            return False

    # Check 9: Agent-specific budget limits
    agent_usage = get_agent_api_calls(agent_name)
    agent_limit = get_agent_budget_limit(agent_name)

    if agent_usage + estimated_calls > agent_limit:
        print(f"❌ DENIED: {agent_name} would exceed agent budget")
        print(f"   Agent usage: {agent_usage}/{agent_limit} calls")
        print(f"   Requested: {estimated_calls} calls")
        return False

    # ALL CHECKS PASSED - Approve operation
    print(f"✅ APPROVED: {operation_name} ({estimated_calls} calls)")
    print(f"   Current: {current_usage}/5,000 ({percentage:.1f}%)")
    print(f"   After: {planned_total}/5,000 ({planned_percentage:.1f}%)")
    print(f"   Remaining: {remaining - estimated_calls} calls")

    # Log approval
    log_to_sheets("Processing_Log", [[
        timestamp(),
        "ZI_PREFLIGHT_APPROVED",
        f"{operation_name}: {estimated_calls} calls, {remaining - estimated_calls} remaining",
        get_status_level(planned_percentage)
    ]])

    return True


def get_total_api_calls_from_log():
    """Get total ZoomInfo API calls used from Processing_Log"""
    # Read Processing_Log from Google Sheets
    log_data = get_sheet_data("Processing_Log", "A:D")

    total_calls = 0
    for row in log_data:
        if row[1].startswith("ZI_") and "calls" in row[2]:
            # Parse: "50 records, 5 calls, 4995 remaining"
            parts = row[2].split(",")
            for part in parts:
                if "calls" in part and "remaining" not in part:
                    calls = int(part.strip().split()[0])
                    total_calls += calls

    return total_calls


def verify_free_alternatives_exhausted(operation_name, agent_name):
    """Verify free sources were attempted before ZoomInfo"""
    # Check agent-specific logs for free source usage

    if agent_name == "Agent_5":
        # Check if LinkedIn via Google was attempted
        google_searches = count_google_searches_today(agent_name)
        if google_searches < 10:
            print(f"   ⚠️ Only {google_searches} Google searches today - try free sources first")
            return False

    elif agent_name == "Agent_1":
        # Check if award sites were scraped
        award_scrapes = count_award_scrapes_today()
        if award_scrapes < 3:
            print(f"   ⚠️ Award sites not fully scraped - try free sources first")
            return False

    # Default: assume free sources exhausted
    return True


def verify_lookup_attempted():
    """Check if lookup was called recently (within last 5 minutes)"""
    recent_logs = get_recent_logs(minutes=5)

    for log in recent_logs:
        if "ZI_lookup" in log[1]:
            return True

    return False


def get_agent_budget_limit(agent_name):
    """Get budget limit for specific agent"""
    agent_budgets = {
        "Agent_1": 500,      # Award validation
        "Agent_2": 500,      # LMS validation
        "Agent_5": 2000,     # Contact enrichment (HIGHEST)
        "Orchestrator": 1500, # Tech stack discovery
        "Agent_4": 500       # Targeted search (last resort)
    }
    return agent_budgets.get(agent_name, 0)


def get_agent_api_calls(agent_name):
    """Get current API calls used by specific agent"""
    log_data = get_sheet_data("Processing_Log", "A:D")

    agent_calls = 0
    for row in log_data:
        if agent_name in row[2] and row[1].startswith("ZI_"):
            # Parse API calls from log
            parts = row[2].split(",")
            for part in parts:
                if "calls" in part and "remaining" not in part:
                    calls = int(part.strip().split()[0])
                    agent_calls += calls

    return agent_calls


def trigger_emergency_stop(current_usage, attempted_operation, agent):
    """Trigger emergency stop at 90% budget"""
    global ZOOMINFO_ENABLED, EMERGENCY_STOP_TRIGGERED

    ZOOMINFO_ENABLED = False
    EMERGENCY_STOP_TRIGGERED = True

    # Log emergency stop
    log_to_sheets("Processing_Log", [[
        timestamp(),
        "ZI_EMERGENCY_STOP",
        f"90% threshold reached: {current_usage}/5000 calls - HALTING ALL OPERATIONS",
        "🚨 CRITICAL"
    ]])

    # Generate incident report
    generate_incident_report(
        trigger_usage=current_usage,
        attempted_operation=attempted_operation,
        triggering_agent=agent
    )

    # Alert user
    print("\n" + "=" * 80)
    print("🚨 EMERGENCY STOP TRIGGERED 🚨")
    print("=" * 80)
    print(f"Budget: {current_usage}/5,000 API calls (90%+ consumed)")
    print(f"Attempted: {attempted_operation} by {agent}")
    print(f"Status: ALL ZoomInfo operations HALTED")
    print(f"Action: Switched to 100% free sources mode")
    print(f"Next: User approval required to resume ZoomInfo usage")
    print("=" * 80 + "\n")

    # Switch to free-only mode
    switch_to_free_mode()


def switch_to_free_mode():
    """Disable ZoomInfo and use only free sources"""
    print("🔄 Switching to FREE-ONLY mode...")
    print("   - ZoomInfo: DISABLED")
    print("   - LinkedIn via Google: ACTIVE")
    print("   - Award sites: ACTIVE")
    print("   - LMS vendors: ACTIVE")
    print("   - Press releases: ACTIVE")
    print("✅ Free-only mode activated")


def get_status_level(percentage):
    """Get status emoji based on budget percentage"""
    if percentage >= 90:
        return "🚨 CRITICAL"
    elif percentage >= 80:
        return "🔴 RED"
    elif percentage >= 60:
        return "🟡 YELLOW"
    else:
        return "🟢 GREEN"


class EmergencyStopException(Exception):
    """Raised when emergency stop triggered"""
    pass
```

---

## Tracking & Logging

### Log Operation Function

```python
def log_zoominfo_operation(tool_name, records_affected, api_calls_used,
                          agent="Unknown", justification="", **kwargs):
    """
    Log every ZoomInfo operation to Processing_Log

    Args:
        tool_name: Name of ZoomInfo tool (enrich_contact, enrich_company, etc.)
        records_affected: Number of records processed
        api_calls_used: Number of API calls consumed
        agent: Agent name performing operation
        justification: Reason for operation (required for searches)
        **kwargs: Additional metadata (batch_size, icp_matches, etc.)
    """

    # Calculate running totals
    current_total = get_total_api_calls_from_log()
    new_total = current_total + api_calls_used
    remaining = 5000 - new_total
    percentage = (new_total / 5000) * 100

    # Determine status level
    status = get_status_level(percentage)

    # Build details string
    details_parts = [
        f"{records_affected} records",
        f"{api_calls_used} calls",
        f"{remaining} remaining ({percentage:.1f}%)",
        f"Agent: {agent}"
    ]

    # Add optional metadata
    if kwargs.get("batch_size"):
        details_parts.append(f"Batch: {kwargs['batch_size']}")
    if kwargs.get("icp_matches"):
        details_parts.append(f"ICP: {kwargs['icp_matches']}")
    if justification:
        details_parts.append(f"Reason: {justification}")

    details = ", ".join(details_parts)

    # Log to Processing_Log
    add_rows("Processing_Log", [[
        timestamp(),
        f"ZI_{tool_name}",
        details,
        status
    ]])

    # Trigger alerts if entering warning zones
    if percentage >= 80 and percentage < 90:
        if current_total < 4000 and new_total >= 4000:
            alert_user_red_zone(new_total, percentage)

    elif percentage >= 90:
        if current_total < 4500 and new_total >= 4500:
            trigger_emergency_stop(new_total, tool_name, agent)

    print(f"📝 Logged: {tool_name} - {details} [{status}]")


def alert_user_red_zone(usage, percentage):
    """Alert user when RED zone entered"""
    print("\n" + "=" * 60)
    print("🔴 RED ZONE ALERT 🔴")
    print("=" * 60)
    print(f"Budget: {usage}/5,000 API calls ({percentage:.1f}%)")
    print(f"Status: Entering RED zone (80-90%)")
    print(f"Action: Requiring approval for further operations")
    print(f"Remaining: {5000 - usage} calls")
    print("=" * 60 + "\n")

    # Log alert
    log_to_sheets("Processing_Log", [[
        timestamp(),
        "ZI_RED_ZONE_ALERT",
        f"Budget {percentage:.1f}% consumed - approval required for further ops",
        "🔴 RED"
    ]])
```

---

## Weekly Reporting

### Weekly Usage Report Function

```python
def generate_weekly_zoominfo_report():
    """
    Generate comprehensive weekly ZoomInfo usage report

    Run every Monday to track budget consumption and project trends
    """

    # Get usage data
    total_calls = get_total_api_calls_from_log()
    week_calls = get_api_calls_this_week()
    remaining = 5000 - total_calls
    percentage = (total_calls / 5000) * 100

    # Calculate projections
    weeks_elapsed = get_weeks_since_project_start()
    weekly_avg = total_calls / weeks_elapsed if weeks_elapsed > 0 else 0
    weeks_remaining_in_budget = remaining / weekly_avg if weekly_avg > 0 else 999

    # Get agent breakdown
    agent_usage = get_agent_usage_breakdown()

    # Get tool breakdown
    tool_usage = get_tool_usage_breakdown()

    # Get top operations
    top_ops = get_top_operations_this_week()

    # Build report
    report = f"""
📊 ZOOMINFO WEEKLY USAGE REPORT
Week: {get_current_week()} | Date: {datetime.now().strftime('%Y-%m-%d')}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
BUDGET STATUS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
This Week:        {week_calls} API calls
Project Total:    {total_calls} / 5,000 ({percentage:.1f}%)
Remaining:        {remaining} calls
Status:           {get_status_level(percentage)}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PROJECTIONS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Weeks Elapsed:    {weeks_elapsed}
Weekly Average:   {weekly_avg:.0f} calls/week
Budget Lifespan:  {weeks_remaining_in_budget:.1f} weeks remaining

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
AGENT BREAKDOWN
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
{format_agent_usage(agent_usage)}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TOOL BREAKDOWN
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
{format_tool_usage(tool_usage)}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TOP OPERATIONS THIS WEEK
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
{format_top_operations(top_ops)}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
RECOMMENDATIONS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
{generate_recommendations(percentage, weekly_avg, agent_usage)}
"""

    # Log report to Processing_Log
    log_to_sheets("Processing_Log", [[
        timestamp(),
        "ZI_WEEKLY_REPORT",
        f"Week {get_current_week()}: {week_calls} calls this week, {total_calls} total ({percentage:.1f}%)",
        get_status_level(percentage)
    ]])

    # Save detailed report to file
    save_report_to_file(report, f"ZoomInfo_Weekly_Report_Week_{get_current_week()}.txt")

    print(report)

    return report


def format_agent_usage(agent_usage):
    """Format agent usage breakdown"""
    lines = []
    for agent, data in sorted(agent_usage.items(), key=lambda x: x[1]["calls"], reverse=True):
        calls = data["calls"]
        limit = data["limit"]
        pct = (calls / limit * 100) if limit > 0 else 0
        lines.append(f"{agent:20s}: {calls:4d} / {limit:4d} calls ({pct:5.1f}%)")
    return "\n".join(lines)


def format_tool_usage(tool_usage):
    """Format tool usage breakdown"""
    lines = []
    for tool, calls in sorted(tool_usage.items(), key=lambda x: x[1], reverse=True):
        lines.append(f"{tool:20s}: {calls:4d} calls")
    return "\n".join(lines)


def format_top_operations(top_ops):
    """Format top operations this week"""
    lines = []
    for i, op in enumerate(top_ops[:10], 1):
        lines.append(f"{i:2d}. {op['timestamp']} - {op['tool']} - {op['details']}")
    return "\n".join(lines)


def generate_recommendations(percentage, weekly_avg, agent_usage):
    """Generate actionable recommendations"""
    recommendations = []

    if percentage >= 80:
        recommendations.append("🔴 URGENT: Budget in RED zone - switch to free-only mode")
    elif percentage >= 60:
        recommendations.append("🟡 WARNING: Budget in YELLOW zone - prioritize highest-value operations only")
    else:
        recommendations.append("🟢 HEALTHY: Budget on track - continue current usage patterns")

    # Check agent budget compliance
    for agent, data in agent_usage.items():
        if data["calls"] > data["limit"] * 0.9:
            recommendations.append(f"⚠️ {agent} approaching budget limit ({data['calls']}/{data['limit']})")

    # Check weekly burn rate
    projected_total = weekly_avg * 8  # 8-week project
    if projected_total > 5000:
        recommendations.append(f"⚠️ Weekly burn rate ({weekly_avg:.0f} calls/week) exceeds budget - reduce usage")

    return "\n".join(recommendations)
```

---

## Emergency Procedures

### Emergency Response Checklist

**If budget reaches 90% (4,500 API calls):**

1. **IMMEDIATE ACTIONS** (automated):
   - Set `ZOOMINFO_ENABLED = False`
   - Set `EMERGENCY_STOP_TRIGGERED = True`
   - Log emergency stop to Processing_Log
   - Generate incident report
   - Alert user via console output

2. **INCIDENT REPORT GENERATION**:
   ```python
   def generate_incident_report(trigger_usage, attempted_operation, triggering_agent):
       """Generate comprehensive incident report for emergency stop"""

       report = {
           "incident_id": f"ZI_EMERGENCY_{timestamp()}",
           "trigger_time": datetime.now().isoformat(),
           "trigger_usage": trigger_usage,
           "trigger_percentage": (trigger_usage / 5000) * 100,
           "attempted_operation": attempted_operation,
           "triggering_agent": triggering_agent,

           # Usage breakdown
           "agent_usage": get_agent_usage_breakdown(),
           "tool_usage": get_tool_usage_breakdown(),
           "daily_usage_trend": get_daily_usage_trend(),

           # Analysis
           "root_cause_analysis": analyze_root_cause(trigger_usage),
           "top_10_operations": get_top_operations_all_time(),

           # Recommendations
           "recommendations": generate_emergency_recommendations()
       }

       # Save report
       save_json(report, f"incident_reports/emergency_{timestamp()}.json")

       # Log to Processing_Log
       log_to_sheets("Processing_Log", [[
           timestamp(),
           "ZI_INCIDENT_REPORT",
           f"Emergency stop - report generated: {report['incident_id']}",
           "🚨 CRITICAL"
       ]])

       return report
   ```

3. **ROOT CAUSE ANALYSIS**:
   - Which agent consumed most budget?
   - Which operations were most frequent?
   - Were free alternatives properly exhausted?
   - Were batch operations used correctly?
   - Were there any anomalous spikes in usage?

4. **USER NOTIFICATION**:
   - Email/Slack alert with incident report attached
   - Summary of budget consumption
   - Explanation of emergency stop trigger
   - Next steps for resolution

5. **RESUME APPROVAL PROCESS**:
   ```python
   def request_resume_approval():
       """Request user approval to resume ZoomInfo usage"""

       print("\n" + "=" * 80)
       print("ZOOMINFO RESUME APPROVAL REQUEST")
       print("=" * 80)
       print("\nCurrent Status:")
       print(f"  - Budget: {get_total_api_calls_from_log()}/5,000 calls")
       print(f"  - Emergency stop triggered at 90%")
       print(f"  - ZoomInfo operations currently HALTED")
       print(f"  - Free sources: ACTIVE")

       print("\nTo resume ZoomInfo usage:")
       print("  1. Review incident report")
       print("  2. Verify root cause addressed")
       print("  3. Confirm free alternatives exhausted")
       print("  4. Document justification for resume")
       print("  5. Provide explicit written approval")

       approval = input("\nApprove resume? (yes/no): ").strip().lower()

       if approval == "yes":
           justification = input("Justification: ").strip()

           # Log approval
           log_to_sheets("Processing_Log", [[
               timestamp(),
               "ZI_RESUME_APPROVED",
               f"User approved resume: {justification}",
               "🔴 RED"
           ]])

           # Re-enable ZoomInfo (RED zone rules still apply)
           global ZOOMINFO_ENABLED, EMERGENCY_STOP_TRIGGERED
           ZOOMINFO_ENABLED = True
           EMERGENCY_STOP_TRIGGERED = False

           print("✅ ZoomInfo re-enabled (RED zone rules active)")
           return True

       else:
           print("❌ Resume denied - remaining in free-only mode")
           return False
   ```

6. **FREE-ONLY MODE CONTINUATION**:
   - All agents continue with free sources
   - Contact discovery via LinkedIn/Google (no email/phone)
   - Company validation via free sources only
   - Expected output: 200-400 contacts (names-only) vs 250-500 (with email/phone)

---

## Code Examples

### Complete End-to-End Example

```python
#!/usr/bin/env python3
"""
Complete example: Agent 5 contact enrichment with ZoomInfo
Demonstrates full workflow from free discovery to enrichment
"""

import time
import random
from datetime import datetime


def main():
    """Main workflow: Discover and enrich contacts for a company"""

    company_name = "Walmart"

    print(f"\n{'='*80}")
    print(f"CONTACT DISCOVERY & ENRICHMENT: {company_name}")
    print(f"{'='*80}\n")

    # Step 1: Free discovery (LinkedIn via Google)
    print("STEP 1: FREE DISCOVERY (LinkedIn via Google)")
    print("-" * 80)
    contacts = discover_contacts_free(company_name)
    print(f"✅ Found {len(contacts)} contacts via free sources\n")

    # Step 2: Pre-flight check
    print("STEP 2: ZOOMINFO PRE-FLIGHT CHECK")
    print("-" * 80)
    estimated_calls = len(contacts) / 10  # Batch size 10

    if not zoominfo_preflight_check("enrich_contact", estimated_calls, agent_name="Agent_5"):
        print("⚠️ Pre-flight check FAILED - returning contacts without enrichment\n")
        save_contacts_to_sheet(contacts, enriched=False)
        return contacts

    print(f"✅ Pre-flight check PASSED - proceeding with enrichment\n")

    # Step 3: Batch enrichment
    print("STEP 3: BATCH ENRICHMENT (ZoomInfo)")
    print("-" * 80)
    enriched_contacts = enrich_contacts_batch(contacts)
    print(f"✅ Enriched {len(enriched_contacts)} contacts with email/phone\n")

    # Step 4: Quality check
    print("STEP 4: QUALITY CHECK")
    print("-" * 80)
    quality_report = generate_quality_report(contacts, enriched_contacts)
    print(quality_report)

    # Step 5: Save results
    print("\nSTEP 5: SAVE RESULTS")
    print("-" * 80)
    save_contacts_to_sheet(enriched_contacts, enriched=True)
    print(f"✅ Saved {len(enriched_contacts)} enriched contacts to Contacts_Master\n")

    print(f"{'='*80}")
    print("WORKFLOW COMPLETE")
    print(f"{'='*80}\n")

    return enriched_contacts


def discover_contacts_free(company_name):
    """Discover contacts using free sources (LinkedIn via Google)"""

    contacts = []

    # Search patterns
    search_patterns = [
        f'site:linkedin.com/in "Chief Learning Officer" {company_name}',
        f'site:linkedin.com/in "VP Learning" OR "VP L&D" {company_name}',
        f'site:linkedin.com/in "Director Learning" OR "Director L&D" {company_name}',
        f'site:linkedin.com/in "Head of Academy" OR "Academy Director" {company_name}'
    ]

    for pattern in search_patterns:
        print(f"  Searching: {pattern}")

        # Simulate Google search (replace with actual Firecrawl)
        results = google_search_with_delay(pattern)

        # Extract profiles from results
        profiles = extract_linkedin_profiles(results)
        contacts.extend(profiles)

        print(f"    Found: {len(profiles)} profiles")

        # Rate limiting
        time.sleep(random.uniform(2, 4))

    # De-duplicate
    unique_contacts = deduplicate_contacts(contacts)

    print(f"\n  Total unique contacts: {len(unique_contacts)}")

    return unique_contacts


def enrich_contacts_batch(contacts):
    """Enrich contacts in batches using ZoomInfo"""

    enriched_results = []
    batch_size = 10

    num_batches = (len(contacts) + batch_size - 1) // batch_size

    for i in range(0, len(contacts), batch_size):
        batch = contacts[i:i+batch_size]
        batch_num = i // batch_size + 1

        print(f"  Batch {batch_num}/{num_batches}: Processing {len(batch)} contacts...")

        # Step 1: Lookup (mandatory first)
        lookup_results = zoominfo_lookup([
            {
                "first_name": c["first_name"],
                "last_name": c["last_name"],
                "company_name": c["company"]
            }
            for c in batch
        ])

        # Step 2: Filter high-confidence matches
        matched_ids = [
            r["record_id"] for r in lookup_results["matches"]
            if r["match_found"] and r["confidence"] >= 0.80
        ]

        print(f"    Lookup: {len(matched_ids)}/{len(batch)} matches found")

        if not matched_ids:
            print(f"    Skipping batch (no matches)")
            continue

        # Step 3: Enrich batch
        enriched = zoominfo_enrich_contact(
            record_ids=matched_ids,
            fields=["email", "phone", "title", "department"]
        )

        enriched_results.extend(enriched["enriched_records"])

        # Step 4: Log operation
        log_zoominfo_operation(
            tool_name="enrich_contact",
            records_affected=len(matched_ids),
            api_calls_used=1,
            agent="Agent_5",
            batch_size=len(batch)
        )

        print(f"    ✅ Enriched {len(matched_ids)} contacts (1 API call)")

        # Rate limiting
        time.sleep(1)

    return enriched_results


def generate_quality_report(original_contacts, enriched_contacts):
    """Generate quality report comparing original vs enriched"""

    total_original = len(original_contacts)
    total_enriched = len(enriched_contacts)

    with_email = len([c for c in enriched_contacts if c.get("email")])
    with_phone = len([c for c in enriched_contacts if c.get("phone")])
    with_both = len([c for c in enriched_contacts if c.get("email") and c.get("phone")])

    enrichment_rate = (total_enriched / total_original * 100) if total_original > 0 else 0
    email_rate = (with_email / total_enriched * 100) if total_enriched > 0 else 0
    phone_rate = (with_phone / total_enriched * 100) if total_enriched > 0 else 0
    complete_rate = (with_both / total_enriched * 100) if total_enriched > 0 else 0

    report = f"""
Quality Report:
  Original Contacts:     {total_original}
  Enriched Contacts:     {total_enriched} ({enrichment_rate:.1f}%)

  With Email:            {with_email} ({email_rate:.1f}%)
  With Phone:            {with_phone} ({phone_rate:.1f}%)
  With Both:             {with_both} ({complete_rate:.1f}%)

  Actionability:         {complete_rate:.1f}% (contacts with email + phone)
"""

    return report


def save_contacts_to_sheet(contacts, enriched=False):
    """Save contacts to Google Sheets"""

    # Format data for Google Sheets
    rows = []
    for contact in contacts:
        row = [
            contact.get("first_name"),
            contact.get("last_name"),
            contact.get("title"),
            contact.get("company"),
            contact.get("email", ""),
            contact.get("phone", ""),
            contact.get("department", ""),
            "Enriched" if enriched else "Free Discovery",
            datetime.now().strftime("%Y-%m-%d")
        ]
        rows.append(row)

    # Add to Contacts_Master sheet
    add_rows("Contacts_Master", rows)

    print(f"  Saved {len(rows)} contacts to Contacts_Master")


if __name__ == "__main__":
    main()
```

---

**Last Updated:** 2025-10-28
**Version:** 1.0
**Next Review:** After Agent 1 & 2 pilot testing
