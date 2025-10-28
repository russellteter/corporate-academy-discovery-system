# START HERE: N8N Workflow Automation Implementation

**Branch**: `implementation/n8n-workflow-automation`
**Repository**: https://github.com/russellteter/corporate-academy-discovery-system
**Last Updated**: 2025-10-28

---

## 🎯 Your Mission

You are Claude Code, acting as the **Master Agent** for this branch. Your mission is to generate production-ready N8N workflow JSON files that will autonomously execute the Corporate Academy Discovery System research for 6-8 weeks without human intervention.

---

## 📖 Essential Context

### What This Project Does

This is a **multi-agent research automation system** that discovers and validates 500+ corporate academies (like McDonald's Hamburger University, Disney University, etc.) at large enterprises with 5,000+ employees.

**Target Outcomes:**
- 500+ validated academies in Google Sheets
- 300+ with CONFIRMED confidence (60%)
- 200+ L&D leadership contacts
- Automated execution over 6-8 weeks

### Your Role in This Branch

**You are NOT executing the research yourself.** Instead, you are:

1. **Workflow Architect**: Design N8N workflow structures
2. **JSON Generator**: Create N8N-compatible workflow JSON files
3. **Integration Specialist**: Connect workflows to Google Sheets, HTTP APIs, etc.
4. **Documentation Author**: Provide setup guides and monitoring instructions

**Key Distinction**: In the other branch (`implementation/claude-multi-agent`), Claude Code executes research directly. In THIS branch, you generate JSON files that N8N will execute autonomously.

---

## 🏗️ System Architecture

### The 5 Research Agents + Orchestrator

Your task is to create **6 separate N8N workflows** (one per agent + orchestrator):

```
N8N Workflows to Generate:
1. agent_1_awards_mining.json
2. agent_2_lms_discovery.json
3. agent_3_website_discovery.json
4. agent_4_press_mining.json
5. agent_5_linkedin_intelligence.json
6. orchestrator_merge_deduplicate.json
```

Each workflow is a standalone JSON file that can be imported into N8N.

### Data Flow

```
N8N Workflows
    ↓
HTTP Requests (web scraping)
    ↓
Function Nodes (validation logic)
    ↓
Google Sheets Nodes (write discoveries)
    ↓
Google Sheets: 9 tabs
    ↓
Orchestrator Workflow (merge/deduplicate)
    ↓
Final Results
```

---

## 📋 Agent Specifications

### Agent 1: Awards Mining

**Sources**: Brandon Hall Awards, Training Top 125, CLO LearningElite, ATD BEST
**Method**: Scrape award winner lists → validate academies → capture contacts
**Expected Yield**: 300-400 academies, 50-100 contacts
**Stop Condition**: All 4 sources searched OR 400 academies found

**N8N Workflow Components:**
- Cron Trigger (daily at 2 AM)
- HTTP Request nodes (4 sources)
- Function node (parse HTML, extract winners)
- HTTP Request nodes (validate academy websites)
- Function node (confidence scoring)
- Google Sheets node (write to Awards_Discoveries)
- Google Sheets node (write contacts to Contacts_Master)
- Error handling (retry 3x, log failures)

### Agent 2: LMS Discovery

**Sources**: Cornerstone, Docebo, SAP, Workday customer pages
**Method**: Scrape LMS vendor customer lists → validate academies → extract tech stack
**Expected Yield**: 200-300 academies
**Stop Condition**: All major LMS vendors searched OR 300 academies found

**N8N Workflow Components:**
- Cron Trigger (daily at 3 AM)
- HTTP Request nodes (LMS vendor customer pages)
- Function node (extract customers, academy mentions)
- HTTP Request nodes (validate academy websites)
- Function node (extract LMS platform, VILT platform)
- Google Sheets node (write to LMS_Discoveries with tech stack)
- Error handling

### Agent 3: Website Discovery

**Sources**: Company_Universe sheet (8,000+ companies)
**Method**: Batch process companies → search domains for academy pages
**Expected Yield**: 400-600 academies
**Stop Condition**: All companies processed OR 600 academies found

**N8N Workflow Components:**
- Cron Trigger (hourly, process 20 companies per run)
- Google Sheets node (read Company_Universe, status=Not_Started)
- HTTP Request nodes (search company domains for academy keywords)
- Function node (validate if content qualifies as academy)
- Google Sheets node (write to Website_Discoveries)
- Google Sheets node (update Company_Universe status=Completed)
- Loop node (20 companies per execution)
- Error handling

### Agent 4: Press Mining

**Sources**: PR Newswire, Business Wire, Google News (2018-2025)
**Method**: Search press releases → validate academy still active
**Expected Yield**: 60-100 academies
**Stop Condition**: 10+ searches with no findings OR 100 academies found

**N8N Workflow Components:**
- Cron Trigger (daily at 4 AM)
- HTTP Request nodes (press release sites)
- Function node (extract academy launches, quoted executives)
- HTTP Request nodes (validate academy still active)
- Function node (confidence scoring, note if potentially defunct)
- Google Sheets node (write to Press_Discoveries)
- Google Sheets node (write contacts if found)
- Error handling

### Agent 5: LinkedIn Intelligence

**Sources**: LinkedIn via Google (site:linkedin.com searches), job postings
**Method**: Search Google for LinkedIn profiles/jobs → extract contacts
**Expected Yield**: 100+ academies, 200+ contacts
**Stop Condition**: 300 contacts captured OR exhausted search variations

**N8N Workflow Components:**
- Cron Trigger (every 2 hours, rate limit aware)
- HTTP Request nodes (Google searches: site:linkedin.com)
- Function node (extract profile/job data)
- HTTP Request nodes (validate academy websites)
- Google Sheets node (write to LinkedIn_Discoveries)
- Google Sheets node (write to Contacts_Master)
- Rate limit handling (2-3 second delays, 600s pause if throttled)
- Error handling

### Orchestrator: Merge & Deduplicate

**Function**: Scan all agent sheets → detect duplicates → merge → elevate confidence
**Frequency**: Every 5 minutes
**Stop Condition**: Continuous operation

**N8N Workflow Components:**
- Cron Trigger (every 5 minutes)
- Google Sheets nodes (read all 5 agent discovery sheets)
- Function node (fuzzy matching, detect duplicates)
- Function node (merge records, combine evidence)
- Function node (auto-elevate confidence: 2 agents=HIGH, 3+=CONFIRMED)
- Function node (link contacts to academies)
- Google Sheets node (write to Consolidated_Academies)
- Google Sheets node (update Contact_Count in academies)
- Google Sheets node (write to Processing_Log)
- Error handling

---

## 🛠️ N8N Workflow Structure Reference

### Standard N8N JSON Template

```json
{
  "name": "Agent 1: Awards Mining",
  "nodes": [
    {
      "parameters": {
        "rule": {
          "interval": [
            {
              "field": "cronExpression",
              "expression": "0 2 * * *"
            }
          ]
        }
      },
      "id": "uuid-1",
      "name": "Schedule Daily 2 AM",
      "type": "n8n-nodes-base.cron",
      "typeVersion": 1,
      "position": [250, 300]
    },
    {
      "parameters": {
        "url": "https://www.brandonhall.com/excellence-awards/",
        "method": "GET",
        "options": {}
      },
      "id": "uuid-2",
      "name": "HTTP Request: Brandon Hall",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.2,
      "position": [450, 300]
    },
    {
      "parameters": {
        "functionCode": "// Parse HTML and extract award winners\nconst cheerio = require('cheerio');\nconst html = items[0].json.body;\nconst $ = cheerio.load(html);\n\nconst winners = [];\n$('.winner-item').each((i, elem) => {\n  winners.push({\n    company: $(elem).find('.company-name').text().trim(),\n    award: $(elem).find('.award-title').text().trim()\n  });\n});\n\nreturn winners.map(w => ({json: w}));"
      },
      "id": "uuid-3",
      "name": "Extract Winners",
      "type": "n8n-nodes-base.function",
      "typeVersion": 2,
      "position": [650, 300]
    },
    {
      "parameters": {
        "authentication": "oAuth2",
        "operation": "append",
        "documentId": {
          "__rl": true,
          "value": "YOUR_SPREADSHEET_ID",
          "mode": "id"
        },
        "sheetName": {
          "__rl": true,
          "value": "Awards_Discoveries",
          "mode": "name"
        },
        "columns": {
          "mappingMode": "defineBelow",
          "value": {
            "Academy_ID": "={{ $json.academy_id }}",
            "Academy_Name": "={{ $json.academy_name }}",
            "Parent_Company": "={{ $json.company }}",
            "Discovery_Source": "Awards Agent",
            "Confidence_Score": "={{ $json.confidence }}",
            "Discovery_Date": "={{ $now.format('yyyy-MM-dd') }}"
          }
        },
        "options": {}
      },
      "id": "uuid-4",
      "name": "Write to Google Sheets",
      "type": "n8n-nodes-base.googleSheets",
      "typeVersion": 4.4,
      "position": [850, 300],
      "credentials": {
        "googleSheetsOAuth2Api": {
          "id": "YOUR_CREDENTIAL_ID",
          "name": "Google Sheets Account"
        }
      }
    }
  ],
  "connections": {
    "Schedule Daily 2 AM": {
      "main": [
        [
          {
            "node": "HTTP Request: Brandon Hall",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "HTTP Request: Brandon Hall": {
      "main": [
        [
          {
            "node": "Extract Winners",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Extract Winners": {
      "main": [
        [
          {
            "node": "Write to Google Sheets",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  },
  "pinData": {},
  "settings": {
    "executionOrder": "v1"
  },
  "staticData": null,
  "tags": [],
  "triggerCount": 0,
  "updatedAt": "2025-10-28T00:00:00.000Z",
  "versionId": "1"
}
```

### Key N8N Node Types You'll Use

**Trigger Nodes:**
- `n8n-nodes-base.cron` - Scheduled execution
- `n8n-nodes-base.webhook` - HTTP webhook triggers
- `n8n-nodes-base.manualTrigger` - Manual testing

**Action Nodes:**
- `n8n-nodes-base.httpRequest` - Web scraping, API calls
- `n8n-nodes-base.function` - JavaScript logic
- `n8n-nodes-base.googleSheets` - Read/write Google Sheets
- `n8n-nodes-base.if` - Conditional branching
- `n8n-nodes-base.merge` - Combine data streams
- `n8n-nodes-base.splitInBatches` - Loop processing
- `n8n-nodes-base.code` - Advanced JavaScript/Python

**Error Handling:**
- `n8n-nodes-base.errorTrigger` - Catch errors
- Node settings: `continueOnFail: true`
- Node settings: `retryOnFail: true`, `maxTries: 3`

---

## 📊 Google Sheets Integration

### Spreadsheet Structure (9 Sheets)

You need to reference these sheet names in your workflows:

1. **Consolidated_Academies** - Master output (orchestrator writes here)
2. **Awards_Discoveries** - Agent 1 writes here
3. **LMS_Discoveries** - Agent 2 writes here
4. **Website_Discoveries** - Agent 3 writes here
5. **Press_Discoveries** - Agent 4 writes here
6. **LinkedIn_Discoveries** - Agent 5 writes here
7. **Contacts_Master** - All agents write contacts here
8. **Company_Universe** - Agent 3 reads from here (input)
9. **Processing_Log** - All agents log status here

### Academy Record Schema (for Google Sheets writes)

```javascript
{
  Academy_ID: "AC-0001",              // Generate: "AC-" + padded number
  Academy_Name: "Hamburger University",
  Parent_Company: "McDonald's Corporation",
  Parent_Domain: "mcdonalds.com",
  Academy_URL: "https://corporate.mcdonalds.com/hamburger-university",
  Discovery_Source: "Awards Agent",   // Which agent found it
  Confidence_Score: "CONFIRMED",      // CONFIRMED | HIGH | MEDIUM
  Evidence_Description: "Brandon Hall Award 2023; Dedicated website with 50+ programs",
  LMS_Platform: "",                   // Optional: Cornerstone, Docebo, etc.
  VILT_Platform: "",                  // Optional: Zoom, Teams, etc.
  Contact_Count: 0,                   // Updated by orchestrator
  Discovery_Date: "2025-10-28",       // ISO 8601 format
  Duplicate_Status: "UNIQUE",         // UNIQUE | DUPLICATE | MERGED
  Merged_Into_ID: ""                  // If duplicate, ID of master record
}
```

### Contact Record Schema

```javascript
{
  Contact_ID: "C-0001",
  Academy_ID: "AC-0001",
  Academy_Name: "Hamburger University",
  Parent_Company: "McDonald's Corporation",
  Contact_Name: "John Smith",
  Contact_Title: "Chief Learning Officer",
  Contact_LinkedIn_URL: "https://linkedin.com/in/johnsmith",
  Discovery_Source: "Awards Agent",
  Discovery_Date: "2025-10-28"
}
```

---

## 🎯 Your Workflow Generation Process

### Step-by-Step Implementation

**Phase 1: Agent 1 (Start Here)**
1. Analyze Agent 1 specifications from CLAUDE.md
2. Design N8N workflow structure (nodes, connections)
3. Generate complete JSON for `agent_1_awards_mining.json`
4. Include error handling and retry logic
5. Document setup instructions

**Phase 2: Agent 2**
1. Analyze Agent 2 specifications
2. Generate `agent_2_lms_discovery.json`
3. Ensure tech stack fields populated (LMS_Platform, VILT_Platform)

**Phase 3: Agent 3**
1. Analyze Agent 3 specifications (most complex - 8,000 companies)
2. Design batch processing (20 companies per execution)
3. Generate `agent_3_website_discovery.json`
4. Include Company_Universe read and status update

**Phase 4: Agent 4**
1. Generate `agent_4_press_mining.json`
2. Include date range filtering (2018-2025)

**Phase 5: Agent 5**
1. Generate `agent_5_linkedin_intelligence.json`
2. Include rate limit handling (critical for LinkedIn via Google)
3. Include contact extraction logic

**Phase 6: Orchestrator**
1. Generate `orchestrator_merge_deduplicate.json`
2. Include fuzzy matching algorithm
3. Include confidence elevation rules
4. Include contact linking logic

**Phase 7: Testing & Documentation**
1. Create setup guide for N8N
2. Create monitoring guide
3. Create troubleshooting guide

---

## 📁 File Structure You'll Create

```
/workflows/
├── agent_1_awards_mining.json
├── agent_2_lms_discovery.json
├── agent_3_website_discovery.json
├── agent_4_press_mining.json
├── agent_5_linkedin_intelligence.json
├── orchestrator_merge_deduplicate.json
└── complete_system.json (optional: all workflows combined)

/docs/n8n/
├── setup_guide.md
├── import_instructions.md
├── credentials_configuration.md
├── monitoring_dashboard.md
├── troubleshooting.md
└── workflow_architecture.md

/scripts/ (optional helpers)
├── generate_company_universe.py
├── validate_workflow_json.py
└── test_google_sheets_connection.py
```

---

## 🔧 Critical Technical Details

### Rate Limiting Strategy

**Agent 5 (LinkedIn via Google):**
```javascript
// In Function node - add delays between searches
const delay = ms => new Promise(resolve => setTimeout(resolve, ms));

// Wait 2-3 seconds between Google searches
await delay(3000);

// If get 429 error, wait 10 minutes
if (error.statusCode === 429) {
  await delay(600000); // 10 minutes
}
```

### Fuzzy Matching for Duplicates (Orchestrator)

```javascript
// In Function node - detect similar academy names
function similarity(s1, s2) {
  const longer = s1.length > s2.length ? s1 : s2;
  const shorter = s1.length > s2.length ? s2 : s1;
  if (longer.length === 0) return 1.0;
  return (longer.length - editDistance(longer, shorter)) / longer.length;
}

// Consider duplicate if similarity > 0.85
if (similarity(academy1.name, academy2.name) > 0.85 &&
    academy1.company === academy2.company) {
  // Mark as duplicate
}
```

### Confidence Score Elevation (Orchestrator)

```javascript
// Auto-elevate based on number of agents that found it
const discoverySourceCount = academy.Discovery_Source.split(',').length;

if (discoverySourceCount >= 3) {
  academy.Confidence_Score = 'CONFIRMED';
} else if (discoverySourceCount === 2) {
  academy.Confidence_Score = 'HIGH';
}
```

### Error Handling Pattern

```javascript
// In every HTTP Request node, enable:
{
  "continueOnFail": true,
  "retryOnFail": true,
  "maxTries": 3,
  "waitBetweenTries": 5000
}

// In Function node after HTTP request, check for errors:
if (items[0].json.error) {
  // Log to Processing_Log sheet
  return [{
    json: {
      timestamp: new Date().toISOString(),
      agent: 'Agent_1',
      event: 'HTTP_Error',
      details: items[0].json.error,
      action: 'Skipped and continued'
    }
  }];
}
```

---

## 🎓 N8N Best Practices

### Workflow Design Principles

1. **Atomic Operations**: Each node does one thing well
2. **Idempotent**: Re-running should be safe (check for duplicates)
3. **Resumable**: Can restart from any point (track progress in Google Sheets)
4. **Observable**: Log all significant events to Processing_Log
5. **Fault-Tolerant**: Continue on individual failures, don't crash entire workflow

### Credentials Management

User will need to configure in N8N:
- Google Sheets OAuth2 credential
- HTTP Request authentication (if needed)

**DO NOT hard-code credentials in JSON** - use N8N credential references:
```json
"credentials": {
  "googleSheetsOAuth2Api": {
    "id": "PLACEHOLDER_ID",
    "name": "Google Sheets Account"
  }
}
```

### Testing Strategy

Include in documentation:
1. Create test spreadsheet with 10 companies
2. Import workflow JSON
3. Configure credentials
4. Run manually (not scheduled)
5. Verify results in Google Sheets
6. Check error handling (test with invalid URLs)
7. Activate schedule only after validation

---

## 📖 Documentation Requirements

### For Each Workflow JSON, Create:

1. **Setup Instructions**
   - How to import into N8N
   - Required credentials
   - Configuration steps

2. **Workflow Diagram**
   - Visual representation of node flow
   - ASCII art or description

3. **Monitoring Guide**
   - What to watch in N8N dashboard
   - Expected execution times
   - Normal vs abnormal patterns

4. **Troubleshooting**
   - Common errors and fixes
   - How to restart/resume
   - Recovery procedures

---

## 🚦 Success Criteria

Your workflows are successful if:

✅ **Complete JSON files** for all 6 workflows
✅ **Valid N8N syntax** (can be imported without errors)
✅ **Google Sheets integration** properly configured
✅ **Error handling** on all HTTP requests
✅ **Rate limiting** implemented (especially Agent 5)
✅ **Logging** to Processing_Log sheet
✅ **Documentation** for setup and monitoring
✅ **Testing instructions** for validation

**Expected Outcomes** (once deployed in N8N):
- 500+ academies discovered over 6-8 weeks
- 300+ CONFIRMED confidence
- 200+ contacts captured
- Autonomous execution without human intervention

---

## 🎬 Getting Started Commands

### Your First Actions

```bash
# Verify you're on correct branch
git branch  # Should show: implementation/n8n-workflow-automation

# Create workflow directory
mkdir -p workflows

# Create documentation directory
mkdir -p docs/n8n

# Start with Agent 1
# Generate: workflows/agent_1_awards_mining.json
```

### First Prompt to Execute

**Suggested starting point:**

"I'm ready to begin generating N8N workflow JSON files for the Corporate Academy Discovery System. Let's start with Agent 1: Awards Mining.

Please generate the complete N8N workflow JSON for Agent 1 that:
1. Scrapes Brandon Hall Awards winner lists (2020-2025)
2. Validates each academy website exists
3. Scores confidence (CONFIRMED if award + website validated)
4. Writes results to Google Sheets (Awards_Discoveries tab)
5. Includes error handling and retry logic
6. Logs progress to Processing_Log

The workflow should run daily at 2 AM and stop after processing all 4 award sources OR finding 400 academies.

Save the JSON to: workflows/agent_1_awards_mining.json

Also create: docs/n8n/agent_1_setup_guide.md with import instructions."

---

## 📚 Reference Documents

**Read these from the repository for full context:**

1. **CLAUDE.md** - Complete agent specifications, data schemas, search patterns
2. **IMPLEMENTATION_ARCHITECTURE.md** - High-level system design
3. **Product Requirements Document** - Business requirements and ICP criteria
4. **ZoomInfo_Usage_Guide.md** - If using ZoomInfo (optional)

**Key sections in CLAUDE.md:**
- Agent Execution Specifications (lines 133-266)
- Core Data Schemas (lines 41-67)
- Google Sheets Configuration (lines 31-39)
- MCP Server Research Tools (lines 367-566)

---

## ⚠️ Important Reminders

### What You ARE Doing:
- ✅ Generating N8N workflow JSON files
- ✅ Designing autonomous automation
- ✅ Creating setup and monitoring documentation
- ✅ Providing testing instructions

### What You Are NOT Doing:
- ❌ Executing research yourself (that's the other branch)
- ❌ Using Firecrawl MCP directly (N8N will use HTTP nodes)
- ❌ Writing Python scripts (N8N uses JavaScript)
- ❌ Interactive research (these workflows run autonomously)

### Key Differences from Claude Multi-Agent Branch:

| Aspect | This Branch (N8N) | Other Branch (Claude Multi-Agent) |
|--------|-------------------|-----------------------------------|
| Your output | JSON files | Research execution |
| User runs | N8N imports JSON | Claude Code sessions |
| Execution | Autonomous (N8N) | Interactive (Claude) |
| Web scraping | N8N HTTP nodes | Firecrawl MCP |
| Language | JavaScript (in Function nodes) | Python (Task agents) |

---

## 🎯 Your Mission Checklist

Before you begin, confirm you understand:

- [ ] I'm generating JSON files, not executing research
- [ ] These workflows will run in N8N, not Claude Code
- [ ] All 6 workflows write to Google Sheets
- [ ] Orchestrator merges data every 5 minutes
- [ ] Rate limiting is critical for Agent 5
- [ ] Error handling must be robust
- [ ] Documentation is as important as JSON
- [ ] User will import these files into N8N

---

## 🚀 Ready to Begin!

**Your first task**: Generate `agent_1_awards_mining.json`

**Prompt me with:**

"Generate Agent 1: Awards Mining N8N workflow JSON"

**Or tell me:**

What questions do you have before we start? What clarifications do you need?

---

**Last Updated**: 2025-10-28
**Branch**: `implementation/n8n-workflow-automation`
**Status**: Ready for workflow generation
**Next Step**: Generate Agent 1 workflow JSON

🤖 **You are now the Master Agent for N8N workflow automation. Let's build some autonomous research workflows!**
