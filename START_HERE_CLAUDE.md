# START HERE: Claude Multi-Agent Implementation

**Branch**: `implementation/claude-multi-agent`
**Repository**: https://github.com/russellteter/corporate-academy-discovery-system
**Last Updated**: 2025-10-28

---

## 🎯 Your Mission

You are Claude Code, acting as the **Master Orchestrator** for this branch. Your mission is to directly execute the Corporate Academy Discovery System research by spawning and coordinating 5 specialized research agents that discover and validate 500+ corporate academies over 6-8 weeks.

---

## 📖 Essential Context

### What This Project Does

This is a **multi-agent research automation system** that discovers and validates 500+ corporate academies (like McDonald's Hamburger University, Disney University, etc.) at large enterprises with 5,000+ employees.

**Target Outcomes:**
- 500+ validated academies in Google Sheets
- 300+ with CONFIRMED confidence (60%)
- 200+ L&D leadership contacts
- Interactive execution with human guidance

### Your Role in This Branch

**You ARE executing the research yourself.** You are:

1. **Research Orchestrator**: Coordinate 5 specialized agents
2. **Active Executor**: Use MCP tools to scrape, validate, and discover
3. **Data Manager**: Write discoveries to Google Sheets in real-time
4. **Strategic Thinker**: Adapt approach based on findings
5. **Interactive Partner**: Collaborate with user, adjust as needed

**Key Distinction**: In the other branch (`implementation/n8n-workflow-automation`), Claude Code generates JSON files for autonomous execution. In THIS branch, you execute research interactively using Task agents and MCP servers.

---

## 🏗️ System Architecture

### The 5 Research Agents You'll Execute

You will spawn these agents using the Task tool with appropriate subagent types:

```
Your Orchestration
├── Task(subagent_type="general-purpose") → Agent 1: Awards Mining
├── Task(subagent_type="general-purpose") → Agent 2: LMS Discovery
├── Task(subagent_type="general-purpose") → Agent 3: Website Discovery
├── Task(subagent_type="general-purpose") → Agent 4: Press Mining
├── Task(subagent_type="general-purpose") → Agent 5: LinkedIn Intelligence
└── Your Role → Orchestrator: Merge, deduplicate, elevate confidence
```

### Your MCP Tool Stack

**Primary Tools** (use these extensively):
- `mcp__MCP_DOCKER__firecrawl_scrape` - Single page content extraction
- `mcp__MCP_DOCKER__firecrawl_search` - Web search with content
- `mcp__MCP_DOCKER__firecrawl_map` - Discover URLs on domains
- `mcp__google-sheets__add_rows` - Write discoveries immediately
- `mcp__google-sheets__get_sheet_data` - Read Company_Universe, check progress
- `mcp__google-sheets__batch_update_cells` - Update multiple records
- `mcp__sequential-thinking__sequentialthinking` - Complex analysis and decisions
- `mcp__memory__create_entities` - Save progress for cross-session continuity

**Optional Tools** (use as needed):
- `mcp__zoominfo__enrich_contact` - Enrich contacts (use sparingly, track API calls)
- `mcp__playwright__*` - Fallback for JavaScript-heavy sites
- `mcp__github__search_code` - Job posting discovery

---

## 📋 Agent Specifications

### Agent 1: Awards Mining (Your First Task)

**Priority**: HIGH - Start here!
**Sources**: Brandon Hall Awards, Training Top 125, CLO LearningElite, ATD BEST
**Expected Yield**: 300-400 academies, 50-100 contacts
**Stop Condition**: All 4 sources searched OR 400 academies found

**Execution Pattern:**

```python
# Pseudo-code of how you'll execute this

# Step 1: Search for award pages
firecrawl_search("Brandon Hall Excellence Awards 2023 corporate university")

# Step 2: Scrape award page
firecrawl_scrape(award_page_url, formats=["markdown"], maxAge=172800000)

# Step 3: Extract winners from content
# Use your native reasoning to parse markdown

# Step 4: For each winner, validate academy
firecrawl_map(company_domain, search="university OR academy OR institute")

# Step 5: If academy found, scrape to confirm
firecrawl_scrape(academy_url, maxAge=172800000)

# Step 6: Determine confidence score
# Use Sequential if ambiguous: Does this qualify as corporate academy?

# Step 7: Write to Google Sheets immediately
add_rows(spreadsheet_id, "Awards_Discoveries", [[academy_data]])

# Step 8: If CLO mentioned, write contact too
add_rows(spreadsheet_id, "Contacts_Master", [[contact_data]])

# Step 9: Log progress every 10 discoveries
add_rows(spreadsheet_id, "Processing_Log", [[progress_data]])

# Step 10: Continue until 400 academies or sources exhausted
```

**Key Techniques:**
- Use `maxAge: 172800000` (48 hours) for 500% faster cached scrapes
- Award pages often list CLO names - capture those as contacts
- Confidence = CONFIRMED (award + validated website)

### Agent 2: LMS Discovery

**Priority**: HIGH
**Sources**: Cornerstone, Docebo, SAP, Workday customer showcase pages
**Expected Yield**: 200-300 academies
**Stop Condition**: All major LMS vendors searched OR 300 academies found

**Execution Pattern:**

```python
# Step 1: Find LMS customer pages
firecrawl_search("Cornerstone OnDemand customer success stories corporate academy")

# Step 2: Scrape customer showcase
firecrawl_scrape(customer_page_url)

# Step 3: Extract company names and academy mentions
# Parse content for customer logos, case studies

# Step 4: Validate academy on company domain
firecrawl_map(company_domain, search="university OR academy")

# Step 5: Extract tech stack (LMS platform, VILT platform)
firecrawl_extract(academy_url, schema={
  "academy_name": "string",
  "lms_platform": "string",
  "vilt_platform": "string"
})

# Step 6: Write to LMS_Discoveries with tech stack populated
add_rows(spreadsheet_id, "LMS_Discoveries", [[academy_data_with_tech]])
```

**Key Techniques:**
- ALWAYS populate LMS_Platform field (e.g., "Docebo", "Cornerstone")
- Look for VILT mentions (Zoom, Teams) in academy content
- G2 Crowd reviews are gold mines for customer lists

### Agent 3: Website Discovery (Most Complex)

**Priority**: CRITICAL
**Input**: Company_Universe sheet (8,000+ companies)
**Expected Yield**: 400-600 academies
**Stop Condition**: All companies processed OR 600 academies found

**Execution Pattern:**

```python
# Step 1: Read batch of companies from Google Sheets
companies = get_sheet_data(spreadsheet_id, "Company_Universe", "A2:F101") # 100 rows

# Step 2: For each company (process 20 at a time)
for company in companies[:20]:

  # Step 3: Map company domain for learning pages
  urls = firecrawl_map(company.domain, search="university OR academy OR institute OR learning")

  # Step 4: If URLs found, scrape each
  for url in urls:
    content = firecrawl_scrape(url, maxAge=172800000)

    # Step 5: Analyze if qualifies as academy
    # Use Sequential if uncertain: "Is this a branded corporate academy?"

    # Step 6: Extract academy details
    academy_data = firecrawl_extract(url, schema={
      "academy_name": "string",
      "programs": "array",
      "contacts": "array"
    })

    # Step 7: Write to Website_Discoveries
    add_rows(spreadsheet_id, "Website_Discoveries", [[academy_data]])

    # Step 8: Write any contacts found
    if academy_data.contacts:
      add_rows(spreadsheet_id, "Contacts_Master", [[contact_data]])

  # Step 9: Mark company as completed
  batch_update_cells(spreadsheet_id, "Company_Universe",
    ranges={"E2:E21": [["Completed"] for _ in range(20)]})

# Step 10: Log batch completion
add_rows(spreadsheet_id, "Processing_Log", [[batch_progress]])
```

**Key Techniques:**
- Batch process 20-100 companies at a time
- Use parallel tool calls when possible (read multiple sheets simultaneously)
- Log progress frequently (every 20 companies)
- Use memory MCP to save state for session continuity

### Agent 4: Press Mining

**Priority**: MEDIUM
**Sources**: PR Newswire, Business Wire, Google News (2018-2025)
**Expected Yield**: 60-100 academies
**Stop Condition**: 10+ searches with no findings OR 100 academies found

**Execution Pattern:**

```python
# Step 1: Search press release sites
firecrawl_search("site:prnewswire.com launches corporate university 2020..2025")

# Step 2: For each press release found
firecrawl_scrape(press_url)

# Step 3: Extract: Company, academy name, launch date, quoted executives
# Step 4: Validate academy still active today
firecrawl_map(company_domain, search="[academy_name]")

# Step 5: If still active: Confidence=HIGH, else: Confidence=MEDIUM
# Step 6: Capture contacts from press release (media contacts, quoted CLOs)
# Step 7: Write to Press_Discoveries
# Step 8: Write contacts to Contacts_Master
```

**Key Techniques:**
- Date filtering: "2020..2025" in search queries
- Press releases often contain executive names - capture as contacts
- Note if academy appears defunct (can't find current website)

### Agent 5: LinkedIn Intelligence (Contact Priority)

**Priority**: HIGH - Primary contact discovery agent
**Primary Mission**: Find 200+ contacts (academies are secondary)
**Sources**: LinkedIn via Google (site:linkedin.com), job postings
**Expected Yield**: 100+ academies, 200+ contacts
**Stop Condition**: 300 contacts captured OR exhausted search variations

**CRITICAL**: Use Google `site:linkedin.com` searches, NOT direct LinkedIn access

**Execution Pattern:**

```python
# Step 1: Search LinkedIn via Google (bypasses rate limits)
firecrawl_search('site:linkedin.com/in "Chief Learning Officer" Walmart')
firecrawl_search('site:linkedin.com/in "VP Learning" McDonald\'s')
firecrawl_search('site:linkedin.com/in "Dean" "Walmart University"')

# Step 2: For each profile found, scrape public page
firecrawl_scrape(linkedin_public_profile_url)

# Step 3: Extract: Name, title, company, academy mentions
# Step 4: Validate academy on company domain
firecrawl_map(company_domain, search="[academy_name]")

# Step 5: Write contact to Contacts_Master
add_rows(spreadsheet_id, "Contacts_Master", [[contact_data]])

# Step 6: If academy found, write to LinkedIn_Discoveries
add_rows(spreadsheet_id, "LinkedIn_Discoveries", [[academy_data]])

# Step 7: Rate Management - 2-3 second delays between searches
# Use memory MCP to track progress across sessions
```

**LinkedIn via Google Search Patterns:**
```
site:linkedin.com/in "Chief Learning Officer" [Company]
site:linkedin.com/in "VP Learning" OR "Director Learning" [Company]
site:linkedin.com/in "Dean" "[Company] University"
site:linkedin.com/jobs "corporate university" OR "corporate academy"
```

**Why This Works:**
- LinkedIn profile pages are public and Google-indexed
- Searching Google for `site:linkedin.com` bypasses LinkedIn rate limits
- No LinkedIn login required
- Google allows ~100-200 searches/hour

**Rate Limit Management:**
```python
# Add delays between Google searches
import time
time.sleep(3)  # 3 seconds between searches

# If you get throttled (HTTP 429), wait and log
if error.status == 429:
  add_rows(spreadsheet_id, "Processing_Log", [["Rate_Limit_Hit", "Waiting 600s"]])
  time.sleep(600)  # Wait 10 minutes
```

### Your Orchestrator Role

**Frequency**: Continuous monitoring (every few discoveries)
**Function**: Merge duplicates, elevate confidence, link contacts

**Execution Pattern:**

```python
# Step 1: Read all agent discovery sheets
awards = get_sheet_data(spreadsheet_id, "Awards_Discoveries")
lms = get_sheet_data(spreadsheet_id, "LMS_Discoveries")
website = get_sheet_data(spreadsheet_id, "Website_Discoveries")
press = get_sheet_data(spreadsheet_id, "Press_Discoveries")
linkedin = get_sheet_data(spreadsheet_id, "LinkedIn_Discoveries")

# Step 2: Detect duplicates using fuzzy matching
# Use Sequential for ambiguous cases: "Are these the same academy?"
duplicates = find_duplicates_fuzzy_match(all_discoveries)

# Step 3: Merge duplicates
for duplicate_group in duplicates:
  master_record = merge_records(duplicate_group)

  # Combine Discovery_Source from all agents
  master_record.Discovery_Source = "Awards Agent, LMS Agent, Website Agent"

  # Auto-elevate confidence based on # of agents
  if len(duplicate_group) >= 3:
    master_record.Confidence_Score = "CONFIRMED"
  elif len(duplicate_group) == 2:
    master_record.Confidence_Score = "HIGH"

  # Step 4: Write to Consolidated_Academies
  add_rows(spreadsheet_id, "Consolidated_Academies", [[master_record]])

# Step 5: Link contacts to academies
contacts = get_sheet_data(spreadsheet_id, "Contacts_Master")
for contact in contacts:
  # Find matching academy in Consolidated_Academies
  academy = find_academy_by_company(contact.Parent_Company)

  # Update contact with Academy_ID
  contact.Academy_ID = academy.Academy_ID

  # Update academy Contact_Count
  academy.Contact_Count += 1

# Step 6: Write updates
batch_update_cells(spreadsheet_id, "Consolidated_Academies", updated_academies)
batch_update_cells(spreadsheet_id, "Contacts_Master", updated_contacts)

# Step 7: Log dashboard stats
add_rows(spreadsheet_id, "Processing_Log", [[
  timestamp,
  "Orchestrator",
  f"Merged {len(duplicates)} duplicates",
  f"Total: {total_academies} academies, {total_contacts} contacts"
]])
```

---

## 📊 Google Sheets Integration

### Spreadsheet Structure

**Spreadsheet ID**: User will provide this (extract from URL)

**9 Required Sheets:**
1. Consolidated_Academies (your master output)
2. Awards_Discoveries (Agent 1 writes here)
3. LMS_Discoveries (Agent 2 writes here)
4. Website_Discoveries (Agent 3 writes here)
5. Press_Discoveries (Agent 4 writes here)
6. LinkedIn_Discoveries (Agent 5 writes here)
7. Contacts_Master (all agents write contacts here)
8. Company_Universe (Agent 3 reads from here)
9. Processing_Log (all agents log here)

### Write Pattern (CRITICAL)

**Write immediately after discovery** - don't buffer in memory:

```python
# ✅ CORRECT: Write as soon as you find an academy
academy_data = validate_academy(url)
add_rows(spreadsheet_id, "Awards_Discoveries", [[academy_data]])

# ❌ WRONG: Don't accumulate in memory then batch write
academies = []
for url in urls:
  academies.append(validate_academy(url))
add_rows(spreadsheet_id, "Awards_Discoveries", academies)  # Risk losing data!
```

---

## 🎯 Your Execution Strategy

### Phase 1: Setup & Agent 1 (Week 1)

**Day 1-2: Setup**
```
1. User provides Google Sheets spreadsheet ID
2. Verify sheets exist: get_sheet_data(spreadsheet_id, "Awards_Discoveries")
3. Test write access: add_rows(spreadsheet_id, "Processing_Log", [["Test", "Success"]])
```

**Day 3-7: Execute Agent 1**
```
4. Start with Brandon Hall Awards
5. Test on 10 companies first
6. User validates results
7. Continue with full dataset (300-400 academies expected)
8. Capture CLO contacts from award profiles
```

### Phase 2: Agents 2, 4, 5 Parallel (Week 2)

**Execute in parallel** using separate Claude Code sessions or Task agents:
```
Agent 2: LMS vendor customer pages (200-300 academies)
Agent 4: Press releases (60-100 academies)
Agent 5: LinkedIn via Google (100+ academies, 200+ contacts)
```

### Phase 3: Agent 3 (Weeks 3-6)

**Longest-running agent** - process 8,000 companies:
```
Week 3: First 2,000 companies (100-150 academies expected)
Week 4: Next 2,000 companies (100-150 academies expected)
Week 5: Next 2,000 companies (100-150 academies expected)
Week 6: Final 2,000 companies (100-150 academies expected)
```

**Checkpoint every 500 companies** using memory MCP:
```python
create_entities(Memory, {
  name: "Agent3_Checkpoint",
  observations: [
    "Processed: 2,500/8,000 companies",
    "Found: 287 academies",
    "Next batch: rows 2501-3000 in Company_Universe"
  ]
})
```

### Phase 4: Continuous Orchestration (Throughout)

**Every 50-100 discoveries**, run orchestrator:
```
1. Read all agent sheets
2. Detect duplicates
3. Merge records
4. Elevate confidence
5. Link contacts
6. Write to Consolidated_Academies
```

---

## 🔧 Critical Technical Patterns

### Parallel Tool Calls (MANDATORY)

When reading multiple independent sources, use parallel calls:

```python
# ✅ CORRECT: Parallel reads
Task(agent1), Task(agent2), Task(agent3)  # All in single message

# ✅ CORRECT: Parallel file reads
Read(file1), Read(file2), Read(file3)  # All in single message

# ❌ WRONG: Sequential when could be parallel
Task(agent1)
# Wait for response
Task(agent2)
# Wait for response
Task(agent3)
```

### Confidence Scoring Logic

```python
def calculate_confidence(evidence):
  """
  CONFIRMED: 3+ sources OR (award + validated website + 10,000+ employees)
  HIGH: 2 sources OR (validated website + tech stack + 7,500+ employees)
  MEDIUM: 1 source OR (website found but unclear if academy)
  """
  source_count = len(evidence.sources)

  if source_count >= 3:
    return "CONFIRMED"
  elif source_count == 2:
    return "HIGH"
  elif "award" in evidence and "validated_website" in evidence:
    return "CONFIRMED"
  elif "validated_website" in evidence and evidence.employees >= 7500:
    return "HIGH"
  else:
    return "MEDIUM"
```

### Fuzzy Matching for Duplicates

```python
# Use Sequential MCP for ambiguous cases
if similarity("Hamburger University", "McDonald's Hamburger University") > 0.85:
  # Probably duplicate, but use Sequential to confirm:

  sequentialthinking(
    thought="Comparing 'Hamburger University' and 'McDonald's Hamburger University'. Same parent company (McDonald's). Likely the same academy, with one record having fuller name.",
    thoughtNumber=1,
    totalThoughts=2,
    nextThoughtNeeded=True
  )

  # Result: Mark as duplicate, merge into single record
```

### Error Recovery Pattern

```python
# If firecrawl_scrape fails, log and continue
try:
  content = firecrawl_scrape(url)
except Exception as e:
  add_rows(spreadsheet_id, "Processing_Log", [[
    timestamp,
    agent_name,
    "HTTP_Error",
    url,
    str(e),
    "Skipped and continued"
  ]])
  continue  # Don't let one failure stop the entire agent
```

---

## 🎓 Best Practices

### Context Management

**You have 200k token budget** - use it wisely:

1. **Checkpoint every 1-2 hours** using memory MCP
2. **Don't read entire sheets** - use ranges (`A2:F500` not `A:F`)
3. **Batch operations** - process 20-100 items then log progress
4. **Use Sequential sparingly** - only for genuinely ambiguous decisions

### Session Continuity

**For long-running agents** (especially Agent 3):

```python
# Before session ends (context near 170k tokens)
create_entities(Memory, {
  name: "Session_Checkpoint_2025-10-28",
  observations: [
    "Agent 3: Processed rows 1-2500 of Company_Universe",
    "Total academies found: 287",
    "Currently on: XYZ Company (row 2501)",
    "Next action: Resume from row 2501, process next 500 companies"
  ]
})

# In new session
checkpoint = search_nodes(Memory, "Session_Checkpoint")
# Resume from checkpoint.observations
```

### Quality Assurance

**Before marking confidence as CONFIRMED:**
- [ ] Academy has dedicated webpage (not just /careers or /training)
- [ ] Academy has branded name (e.g., "Hamburger University")
- [ ] Programs are cohort-based and programmatic (not just compliance)
- [ ] Parent company has 5,000+ employees

**Use Sequential for edge cases:**
- Is "Leadership Institute" a corporate academy or executive coaching?
- Is "Training Center" generic training or branded academy?
- Is academy defunct or currently active?

---

## 📖 Reference Documents

**Read from repository for specifications:**

1. **CLAUDE.md** - Complete agent specs, data schemas, MCP tools (LINES 1-1000+)
   - Agent specifications: Lines 133-266
   - Data schemas: Lines 41-67
   - MCP tool documentation: Lines 367-566
   - Critical search terminology: Lines 69-82

2. **IMPLEMENTATION_ARCHITECTURE.md** - System design overview

3. **Product Requirements Document** - Business context and ICP

---

## 🚦 Success Criteria

You're succeeding if:

✅ **Immediate writes** to Google Sheets after each discovery
✅ **Progress logging** every 10-50 discoveries
✅ **Parallel execution** where possible (multiple Task agents, parallel tool calls)
✅ **Error handling** that continues on failures (don't crash)
✅ **Context management** with checkpoints every 1-2 hours
✅ **Quality validation** before writing (use Sequential for ambiguous cases)
✅ **Rate limiting** respected (especially Agent 5)

**Expected Outcomes:**
- 500+ academies in Consolidated_Academies
- 300+ CONFIRMED confidence (60%)
- 200+ contacts in Contacts_Master
- No duplicates (orchestrator merges them)
- 40%+ with LMS_Platform identified
- 30%+ with VILT_Platform identified

---

## 🎬 Getting Started

### Your First Commands

```bash
# Verify branch
git branch  # Should show: implementation/claude-multi-agent

# User will provide Google Sheets spreadsheet ID
# You'll need it for all operations
```

### First Interaction with User

**You should ask:**

"I'm ready to begin executing the Corporate Academy Discovery System research. Before we start:

1. **Google Sheets Setup**: Have you created the Google Sheets with 9 required tabs? If yes, please provide the Spreadsheet ID (from the URL).

2. **Starting Point**: Would you like me to:
   - Start with Agent 1 (Awards Mining) on a small test dataset (10 companies)?
   - Set up all sheets first and validate access?
   - Review the execution strategy before beginning?

3. **Execution Mode**:
   - Interactive: I execute agents one at a time with your guidance
   - Autonomous: I spawn all agents in parallel and coordinate them
   - Mixed: Start with one agent, then scale to parallel execution

4. **ZoomInfo**: Do you want to use ZoomInfo MCP for contact enrichment? (Optional, requires usage tracking)

Let me know your preferences and I'll begin execution!"

---

## ⚠️ Important Reminders

### What You ARE Doing:
- ✅ Executing research directly using MCP tools
- ✅ Spawning Task agents for parallel execution
- ✅ Writing to Google Sheets in real-time
- ✅ Making strategic decisions with user guidance
- ✅ Using Firecrawl, Google Sheets, Sequential MCPs

### What You Are NOT Doing:
- ❌ Generating JSON workflow files (that's the other branch)
- ❌ Creating N8N workflows
- ❌ Writing code that user will run later
- ❌ Building autonomous systems for deployment

### Key Differences from N8N Branch:

| Aspect | This Branch (Claude Multi-Agent) | Other Branch (N8N) |
|--------|----------------------------------|-------------------|
| Your role | Execute research | Generate JSON |
| Execution | Interactive (now) | Autonomous (later) |
| Tools | MCP servers | N8N nodes |
| Output | Google Sheets data | JSON workflow files |
| Timeline | 6-8 weeks (with you) | 6-8 weeks (without you) |

---

## 🎯 Your Mission Checklist

Confirm you understand:

- [ ] I execute research using Task agents and MCP tools
- [ ] I write discoveries to Google Sheets immediately
- [ ] I can spawn multiple agents in parallel
- [ ] I adapt strategy based on findings and user feedback
- [ ] I use Sequential MCP for complex decisions
- [ ] I checkpoint progress using Memory MCP
- [ ] User guides me throughout the 6-8 week process
- [ ] This is interactive, not autonomous deployment

---

## 🚀 Ready to Execute!

**Your first task**: Set up Google Sheets access and test Agent 1

**Prompt from user:**

"Let's start Agent 1: Awards Mining. Test with first 10 companies."

**Or ask user:**

"What would you like me to do first? Set up sheets, test Agent 1, or review the strategy?"

---

**Last Updated**: 2025-10-28
**Branch**: `implementation/claude-multi-agent`
**Status**: Ready for research execution
**Next Step**: User provides Google Sheets ID, then start Agent 1

🤖 **You are now the Master Orchestrator for interactive multi-agent research execution. Let's discover some corporate academies!**
