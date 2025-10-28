# Implementation Architecture

**Project**: Corporate Academy Discovery System
**Repository**: https://github.com/russellteter/corporate-academy-discovery-system
**Version**: 1.0
**Last Updated**: 2025-10-28

---

## Overview

This project has **TWO distinct implementation paths**, each in its own Git branch. Both implementations use the same project requirements, documentation, and research methodology, but execute the work differently.

```
main (documentation & requirements)
├── implementation/claude-multi-agent (Path 1: Claude Code Executes)
└── implementation/n8n-workflow-automation (Path 2: N8N JSON Generation)
```

---

## Branch Architecture

### `main` Branch (Documentation Hub)

**Purpose**: Stable documentation, requirements, and project specifications

**Contents**:
- `CLAUDE.md` - Project instructions and agent specifications
- PRDs and validation reports
- Data schemas and methodologies
- Research guidelines and ICP criteria

**Usage**: Source of truth for both implementations

**Updates**: Only update when requirements/documentation change for BOTH paths

---

## Path 1: Claude Multi-Agent Implementation

### Branch: `implementation/claude-multi-agent`

**Implementation Strategy**: Claude Code acts as orchestrator and executes all research agents directly

**Execution Model**: Interactive, session-based with Claude Code

**Architecture**:

```
Claude Code Session
├── Agent 1: Awards Mining (Claude executes directly)
├── Agent 2: LMS Discovery (Claude executes directly)
├── Agent 3: Website Discovery (Claude executes directly)
├── Agent 4: Press Mining (Claude executes directly)
├── Agent 5: LinkedIn Intelligence (Claude executes directly)
└── Orchestrator: (Claude coordinates and merges)
```

**How It Works**:

1. **User starts Claude Code session**
2. **Claude reads project instructions** from CLAUDE.md
3. **Claude spawns Task agents** for each research agent (1-5)
4. **Agents execute in parallel** using MCP servers:
   - Firecrawl for web scraping
   - Google Sheets for data persistence
   - Sequential for complex analysis
   - Memory for cross-session state
5. **Claude orchestrates**: Merges duplicates, elevates confidence, links contacts
6. **User monitors progress**: Real-time updates in Claude Code
7. **Results written to Google Sheets**: Live data accumulation

**Key Characteristics**:

✅ **Interactive**: User can guide, ask questions, adjust strategy
✅ **Flexible**: Claude adapts to findings, changes approach mid-stream
✅ **Transparent**: See reasoning, evidence, decision-making
✅ **Session-based**: Can pause/resume, checkpoint progress
✅ **MCP-powered**: Direct access to Firecrawl, Google Sheets, ZoomInfo
✅ **Context-aware**: Claude maintains understanding across agents

**Technology Stack**:

- **Orchestrator**: Claude Code with Task agents
- **Web Scraping**: Firecrawl MCP server
- **Data Storage**: Google Sheets MCP server
- **Analysis**: Sequential Thinking MCP
- **State Management**: Memory MCP
- **Optional Acceleration**: ZoomInfo MCP

**Execution Pattern**:

```python
# Pseudo-code of how Claude executes this

# Session Start
user: "Start Agent 1: Awards Mining"

claude:
  1. Read CLAUDE.md agent specifications
  2. Task(subagent_type="Explore", prompt="Find Brandon Hall Awards 2020-2025")
  3. firecrawl_search("Brandon Hall Excellence Awards 2023")
  4. firecrawl_scrape(award_page_url)
  5. Extract winners, validate academies
  6. mcp__google-sheets__add_rows(Awards_Discoveries, data)
  7. Report progress to user
  8. Continue until 400 academies or sources exhausted
```

**Advantages**:

- Real-time human oversight and guidance
- Adaptive strategy based on findings
- Can pivot when hitting roadblocks
- Transparent reasoning and evidence
- Interactive debugging and refinement
- No external infrastructure needed

**Disadvantages**:

- Requires active Claude Code session
- Manual triggering of agents
- Context window management needed
- Not fully autonomous (requires user presence)

**Ideal For**:

- Exploratory research with unknown variables
- Projects needing human judgment calls
- Situations requiring strategy adjustments
- Learning how the system works
- Testing and validation phases

---

## Path 2: N8N Workflow Automation

### Branch: `implementation/n8n-workflow-automation`

**Implementation Strategy**: Claude Code generates N8N workflow JSON that runs autonomously

**Execution Model**: Autonomous, scheduled workflows running in N8N

**Architecture**:

```
N8N Workflow (Single JSON File)
├── Workflow: Agent 1 - Awards Mining
├── Workflow: Agent 2 - LMS Discovery
├── Workflow: Agent 3 - Website Discovery
├── Workflow: Agent 4 - Press Mining
├── Workflow: Agent 5 - LinkedIn Intelligence
├── Workflow: Orchestrator - Merge & Deduplicate
└── Workflow: Dashboard - Status & Alerts
```

**How It Works**:

1. **Claude Code generates N8N JSON** workflow definition
2. **User imports JSON into N8N** (local or cloud instance)
3. **N8N executes workflows autonomously**:
   - Scheduled triggers (daily batches)
   - HTTP node for web scraping
   - Google Sheets node for data writes
   - Function nodes for logic/validation
   - Merge nodes for deduplication
4. **Workflows run unattended** for 6-8 weeks
5. **Results accumulate in Google Sheets** automatically
6. **Email/Slack alerts** on milestones or errors

**Key Characteristics**:

✅ **Autonomous**: Runs without human intervention
✅ **Scheduled**: Daily/hourly execution on timers
✅ **Scalable**: Can run 24/7, process thousands of companies
✅ **Reproducible**: Same workflow, consistent results
✅ **Production-ready**: Built for long-running operations
✅ **Error handling**: Retry logic, fallbacks, alerts

**Technology Stack**:

- **Orchestrator**: N8N workflow engine
- **Web Scraping**: N8N HTTP Request node + Cheerio
- **Data Storage**: N8N Google Sheets node
- **Logic**: N8N Function nodes (JavaScript)
- **Scheduling**: N8N Cron triggers
- **Notifications**: N8N Email/Slack nodes

**N8N Workflow Structure**:

```json
{
  "name": "Agent 1: Awards Mining",
  "nodes": [
    {
      "name": "Schedule Trigger",
      "type": "n8n-nodes-base.cron",
      "parameters": {"rule": "0 2 * * *"}  // Daily 2 AM
    },
    {
      "name": "HTTP Request: Brandon Hall",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "url": "https://www.brandonhall.com/awards/",
        "method": "GET"
      }
    },
    {
      "name": "Extract Winners",
      "type": "n8n-nodes-base.function",
      "parameters": {
        "functionCode": "// Parse HTML, extract companies"
      }
    },
    {
      "name": "Validate Academy",
      "type": "n8n-nodes-base.httpRequest"
    },
    {
      "name": "Write to Google Sheets",
      "type": "n8n-nodes-base.googleSheets",
      "parameters": {
        "operation": "append",
        "sheetId": "Awards_Discoveries"
      }
    }
  ]
}
```

**Advantages**:

- Fully autonomous execution
- Runs 24/7 without human presence
- Consistent, reproducible results
- Built-in error handling and retries
- Scalable to thousands of operations
- Production deployment ready
- Can pause/resume workflows
- Visual workflow editor (N8N UI)

**Disadvantages**:

- Less flexible than interactive Claude
- Requires N8N infrastructure (self-hosted or cloud)
- Fixed logic - harder to adapt mid-execution
- Debugging requires examining workflow logs
- Initial setup more complex
- No real-time human judgment

**Ideal For**:

- Production deployments
- Long-running autonomous research
- Scheduled batch processing
- Projects with clear, fixed methodology
- Situations requiring 24/7 execution
- Teams wanting hands-off automation

---

## Implementation Comparison

| Aspect | Claude Multi-Agent | N8N Workflow Automation |
|--------|-------------------|------------------------|
| **Execution** | Interactive, session-based | Autonomous, scheduled |
| **Human Involvement** | Active guidance | Set-and-forget |
| **Flexibility** | High - adapt on the fly | Low - fixed workflow |
| **Scalability** | Limited by session length | Unlimited - runs 24/7 |
| **Debugging** | Real-time, transparent | Log analysis |
| **Infrastructure** | None (Claude Code only) | N8N instance required |
| **Best For** | Exploration, testing | Production, scale |
| **Timeline** | 6-8 weeks (manual triggers) | 6-8 weeks (automatic) |
| **Complexity** | Low (no coding) | Medium (JSON workflow) |
| **Reproducibility** | Variable (human-guided) | High (fixed workflow) |

---

## Branch Workflow Guidelines

### Working on Claude Multi-Agent Implementation

```bash
# Switch to branch
git checkout implementation/claude-multi-agent

# Make changes (add agent logic, scripts, etc.)
# Commit frequently
git add agents/awards_mining.py
git commit -m "feat: implement Agent 1 awards mining logic"

# Push to GitHub
git push

# When ready to test
# Just run the agents directly in Claude Code session
```

### Working on N8N Workflow Implementation

```bash
# Switch to branch
git checkout implementation/n8n-workflow-automation

# Make changes (modify N8N JSON workflow)
# Commit workflow versions
git add workflows/agent_1_awards_mining.json
git commit -m "feat: add Agent 1 N8N workflow definition"

# Push to GitHub
git push

# When ready to test
# Import JSON into N8N instance
# Activate workflow
# Monitor execution
```

---

## Development Workflow

### Shared Updates (Apply to Both Branches)

If you make a change that applies to **both** implementations (e.g., updated ICP criteria, new data schema):

```bash
# Update in main branch
git checkout main
# Edit CLAUDE.md or documentation
git add CLAUDE.md
git commit -m "docs: update ICP criteria for 7,500+ employees"
git push

# Merge into both implementation branches
git checkout implementation/claude-multi-agent
git merge main
git push

git checkout implementation/n8n-workflow-automation
git merge main
git push
```

### Independent Updates (Branch-Specific)

Changes specific to one implementation stay in that branch only.

**Example**: Adding Claude-specific agent logic
```bash
git checkout implementation/claude-multi-agent
# Add agent code
git commit -m "feat: add interactive validation for Agent 3"
# This stays ONLY in claude-multi-agent branch
```

**Example**: Adding N8N error handling
```bash
git checkout implementation/n8n-workflow-automation
# Modify workflow JSON
git commit -m "feat: add retry logic to Agent 2 workflow"
# This stays ONLY in n8n-workflow-automation branch
```

---

## Deliverables by Branch

### Claude Multi-Agent Branch Deliverables

**In Repository**:
- `/agents/` - Python scripts for each agent (optional, for complex logic)
- `/scripts/` - Helper utilities for data validation
- `/docs/execution_logs/` - Session logs and findings

**External** (not in Git):
- Google Sheets with research data
- Execution notes in Claude Code
- Progress checkpoints in Memory MCP

### N8N Workflow Branch Deliverables

**In Repository**:
- `/workflows/agent_1_awards_mining.json` - N8N workflow definition
- `/workflows/agent_2_lms_discovery.json`
- `/workflows/agent_3_website_discovery.json`
- `/workflows/agent_4_press_mining.json`
- `/workflows/agent_5_linkedin_intelligence.json`
- `/workflows/orchestrator_merge.json`
- `/workflows/complete_system.json` - Combined workflow
- `/docs/n8n_setup_guide.md` - Installation and configuration
- `/docs/n8n_monitoring.md` - How to monitor workflows

**External** (not in Git):
- N8N instance (self-hosted or cloud)
- Google Sheets with research data
- Webhook endpoints for triggers
- Email/Slack notification channels

---

## Testing Strategy

### Claude Multi-Agent Testing

```bash
git checkout implementation/claude-multi-agent

# Test Agent 1 interactively in Claude Code
user: "Run Agent 1 on first 10 companies"
claude: [executes awards mining agent]

# Verify results in Google Sheets
# Iterate and refine approach
# Document findings

# Commit improvements
git commit -m "feat: improve Agent 1 confidence scoring"
```

### N8N Workflow Testing

```bash
git checkout implementation/n8n-workflow-automation

# Import workflow JSON into N8N test instance
# Run with small dataset (10 companies)
# Check execution logs in N8N UI
# Verify results in Google Sheets

# Fix issues in JSON
git commit -m "fix: correct Agent 2 HTTP request timeout"

# Re-import and test again
```

---

## Success Criteria

Both implementations should achieve the same outcomes:

✅ **500+ validated academies** in Consolidated_Academies sheet
✅ **300+ CONFIRMED confidence** (60%)
✅ **200+ L&D contacts** in Contacts_Master sheet
✅ **40%+ LMS platform identification**
✅ **Execution timeline**: 6-8 weeks
✅ **Data quality**: No duplicates, proper confidence scoring

**Unique to Claude Multi-Agent**:
- ✅ Real-time user feedback incorporated
- ✅ Strategy adjustments documented
- ✅ Transparent reasoning captured

**Unique to N8N Workflow**:
- ✅ Autonomous 24/7 execution
- ✅ Reproducible workflow (can re-run)
- ✅ Production-ready deployment

---

## Next Steps

### To Start Claude Multi-Agent Implementation:

```bash
git checkout implementation/claude-multi-agent

# Claude Code will:
# 1. Read CLAUDE.md for agent specs
# 2. Create /agents/ directory structure
# 3. Implement Agent 1 logic
# 4. Test with first 50 companies
# 5. Iterate based on findings
```

### To Start N8N Workflow Implementation:

```bash
git checkout implementation/n8n-workflow-automation

# Claude Code will:
# 1. Analyze CLAUDE.md agent specifications
# 2. Generate N8N workflow JSON for Agent 1
# 3. Create complete workflow definitions
# 4. Provide N8N import instructions
# 5. Document testing procedures
```

---

## Questions & Clarifications

**Q: Can I switch between implementations mid-project?**
A: Yes! Both branches use the same data store (Google Sheets). You can test one approach, switch branches, try the other.

**Q: Can I use parts of both?**
A: Yes! You could use Claude Multi-Agent for exploration, then generate N8N workflows for production execution.

**Q: Which should I start with?**
A:
- **Claude Multi-Agent**: If you want to understand the research process, need flexibility, want to learn
- **N8N Workflow**: If you want hands-off automation, have N8N experience, need scalability

**Q: Can I merge both branches later?**
A: Not really - they're fundamentally different execution models. But you can maintain both as alternative implementations.

---

**Repository**: https://github.com/russellteter/corporate-academy-discovery-system
**Documentation**: See `CLAUDE.md` for detailed agent specifications
**Status**: Architecture defined, ready for implementation
