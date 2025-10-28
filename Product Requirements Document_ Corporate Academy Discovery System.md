# **Product Requirements Document: Corporate Academy Discovery System**

**Version:** 1.0  
 **Date:** 2025-10-28  
 **Target Platform:** Claude Code  
 **Estimated Effort:** 6-8 weeks (automated execution)

---

## **1\. EXECUTIVE SUMMARY**

### **1.1 Objective**

Build an automated multi-agent research system to discover and validate 500+ corporate academies/universities/institutes at companies with 5,000+ employees, capturing academy names, parent companies, evidence of existence, technology stacks, and L\&D leadership contacts.

### **1.2 Success Criteria**

* **Minimum:** 500 validated corporate academies/universities with CONFIRMED or HIGH confidence  
* **Target Confidence Distribution:** 60% CONFIRMED (300+), 30% HIGH (150+), 10% MEDIUM (50+)  
* **Contact Discovery:** 200+ L\&D leadership contacts (names, titles, LinkedIn URLs)  
* **Tech Stack Identification:** 40%+ with LMS platform, 30%+ with VILT platform identified  
* **Timeline:** 6-8 weeks of automated execution

### **1.3 Scope**

**In Scope:**

* Discovery of branded learning entities (academies, universities, institutes)  
* Validation of academy existence through multiple sources  
* Opportunistic capture of technology stack signals (LMS, VILT platforms)  
* Contact discovery for L\&D leadership when readily available  
* US-focused companies with 5,000+ employees

**Out of Scope:**

* Full VILT validation (deferred to Phase 2\)  
* Deep enrichment of firmographic data  
* International companies (except where naturally discovered)  
* Individual training programs without academy branding

---

## **2\. SYSTEM ARCHITECTURE**

### **2.1 Multi-Agent Structure**

The system consists of 5 specialized research agents operating in parallel, coordinated by an orchestrator:

```
┌─────────────────────────────────────────────────────────┐
│                    ORCHESTRATOR                         │
│         (Monitors, Merges, Validates)                   │
└────────────────────┬────────────────────────────────────┘
                     │
     ┌───────────────┼───────────────┐
     │               │               │
┌────▼────┐    ┌────▼────┐    ┌────▼────┐
│ AGENT 1 │    │ AGENT 2 │    │ AGENT 3 │
│ Awards  │    │   LMS   │    │ Website │
│ Mining  │    │ Reverse │    │Discovery│
└────┬────┘    └────┬────┘    └────┬────┘
     │               │               │
     │         ┌─────▼─────┐        │
     │         │  AGENT 4  │        │
     │         │   Press   │        │
     │         │  Mining   │        │
     │         └─────┬─────┘        │
     │               │               │
     │         ┌─────▼─────┐        │
     │         │  AGENT 5  │        │
     │         │ LinkedIn  │        │
     └─────────┴─────┬─────┴────────┘
                     │
            ┌────────▼─────────┐
            │  GOOGLE SHEETS   │
            │  (Data Store)    │
            └──────────────────┘
```

### **2.2 Data Storage: Google Sheets**

**Primary Workbook:** `Corporate_Academy_Discovery_Master`

**Required Sheets:**

1. `Consolidated_Academies` \- Master output of merged, validated academies  
2. `Awards_Discoveries` \- Agent 1 raw output  
3. `LMS_Discoveries` \- Agent 2 raw output  
4. `Website_Discoveries` \- Agent 3 raw output  
5. `Press_Discoveries` \- Agent 4 raw output  
6. `LinkedIn_Discoveries` \- Agent 5 raw output  
7. `Contacts_Master` \- All L\&D contacts discovered  
8. `Company_Universe` \- Input list of companies to research  
9. `Processing_Log` \- System events and agent status

---

## **3\. DATA SCHEMA**

### **3.1 Academy Record Schema**

**Used in: Consolidated\_Academies and all agent discovery sheets**

| Field Name | Type | Required | Description | Example |
| ----- | ----- | ----- | ----- | ----- |
| Academy\_ID | String | Yes | Unique identifier | AC-0001 |
| Academy\_Name | String | Yes | Exact branded name | Hamburger University |
| Parent\_Company | String | Yes | Parent corporation | McDonald's Corporation |
| Parent\_Domain | String | Yes | Company domain | mcdonalds.com |
| Academy\_URL | URL | No | Direct link to academy | corporate.mcdonalds.com/hamburger-university |
| Discovery\_Source | String | Yes | Source(s) of discovery | Awards Agent, LMS Agent |
| Confidence\_Score | Enum | Yes | CONFIRMED / HIGH / MEDIUM | CONFIRMED |
| Evidence\_Description | Text | Yes | Summary of all evidence | Brandon Hall Award 2023; Dedicated website with program catalog |
| Primary\_Industry | String | No | Industry sector | Quick Service Restaurant |
| Employee\_Count\_Range | String | No | Size range | 50000+ |
| LMS\_Platform | String | No | Learning management system | Cornerstone OnDemand |
| VILT\_Platform | String | No | Virtual classroom platform | Zoom |
| Contact\_Count | Integer | No | \# of contacts found | 3 |
| Discovery\_Date | Date | Yes | Date first discovered | 2025-10-28 |
| Merge\_Status | Enum | No | PENDING / MERGED / UNIQUE | MERGED |
| Notes | Text | No | Additional observations | Franchise-focused model |

### **3.2 Contact Record Schema**

**Used in: Contacts\_Master**

| Field Name | Type | Required | Description | Example |
| ----- | ----- | ----- | ----- | ----- |
| Contact\_ID | String | Yes | Unique identifier | C-0001 |
| Academy\_ID | String | Yes | Links to academy | AC-0001 |
| Academy\_Name | String | Yes | For reference | Hamburger University |
| Parent\_Company | String | Yes | For reference | McDonald's Corporation |
| Contact\_Name | String | Yes | Full name | Jane Smith |
| Contact\_Title | String | Yes | Job title | Dean of Hamburger University |
| Contact\_LinkedIn\_URL | URL | No | LinkedIn profile | linkedin.com/in/janesmith |
| Contact\_Email | String | No | Email address | jane.smith@mcdonalds.com |
| Discovery\_Source | String | Yes | Which agent found | LinkedIn Agent |
| Discovery\_Date | Date | Yes | Date found | 2025-10-28 |

### **3.3 Company Universe Schema**

**Used in: Company\_Universe (input data)**

| Field Name | Type | Required | Description | Example |
| ----- | ----- | ----- | ----- | ----- |
| Company\_Name | String | Yes | Official name | Walmart Inc. |
| Domain | String | Yes | Primary domain | walmart.com |
| Industry | String | No | Sector | Retail |
| Employee\_Count | String | No | Size | 50000+ |
| Status | Enum | Yes | Not\_Started / In\_Progress / Completed | Not\_Started |
| Date\_Added | Date | Yes | When added | 2025-10-28 |

### **3.4 Processing Log Schema**

**Used in: Processing\_Log**

| Field Name | Type | Required | Description | Example |
| ----- | ----- | ----- | ----- | ----- |
| Timestamp | DateTime | Yes | When event occurred | 2025-10-28 14:32:18 |
| Event\_Type | String | Yes | Type of event | Batch\_Complete |
| Details | Text | Yes | Event description | 20 companies processed, 5 academies found |
| Agent | String | Yes | Which agent | Agent\_3 |

---

## **4\. AGENT SPECIFICATIONS**

### **4.1 Agent 1: Awards Mining Specialist**

**Purpose:** Discover corporate academies through L\&D industry award winners

**Priority:** HIGH (highest confidence discoveries)

**Target Sources:**

* Brandon Hall Group Excellence Awards (brandonhall.com/excellenceawards)  
* Training Magazine Top 125 (trainingmag.com/training-top-125)  
* CLO LearningElite (chieflearningofficer.com/learningelite)  
* ATD BEST Awards (td.org/atd-best-awards)

**Search Methodology:**

1. **Navigate to award source websites**

2. **Search for relevant categories:** "Best Corporate University", "Best Corporate Academy", "Best Learning Program"

3. **Extract award winners (2020-2025):**

   * Company name  
   * Award category and year  
   * Any academy/university/institute name mentioned  
4. **Validate academy existence:**

```
Google Search: site:[company-domain].com ("corporate university" OR "corporate academy" OR "learning institute")
```

   * Look for dedicated website, subdomain, or substantial section  
   * Confirm exact academy NAME (branding)  
5. **Data Capture:**

   * All fields per Academy Record Schema  
   * Discovery\_Source: "\[Award Name\] \[Year\] \+ Website Validation"  
   * Confidence\_Score: CONFIRMED (if award \+ dedicated website) or HIGH (if award \+ limited web presence)  
6. **Contact Capture:**

   * Extract any L\&D leader names/titles mentioned in award articles  
   * Search LinkedIn: "\[Name\] \[Company\]" → capture profile URL  
   * Write to Contacts\_Master sheet  
7. **Output:** Write immediately to `Awards_Discoveries` sheet

**Expected Yield:** 100-150 academies

**Stop Condition:** All 4 award sources searched OR 150 academies found

---

### **4.2 Agent 2: LMS Reverse Engineering Specialist**

**Purpose:** Identify academies by finding companies using enterprise LMS platforms

**Priority:** HIGH (strong technology correlation)

**Target Sources:**

* Cornerstone OnDemand customers (cornerstoneondemand.com/customers)  
* Docebo customer stories (docebo.com/customers)  
* SAP SuccessFactors Learning (search "SAP SuccessFactors Learning customer")  
* Workday Learning (workday.com/customers)  
* G2 LMS reviews (g2.com/categories/learning-management-systems)

**Search Methodology:**

1. **Navigate to LMS vendor customer pages**

2. **Filter/prioritize:** Companies with 5,000+ employees

3. **Look for case studies mentioning:**

   * "Corporate university" / "Academy" / "Learning institute"  
   * "Programmatic training" / "Leadership development"  
4. **Validate academy for each LMS customer:**

```
Google Search: site:[company-domain].com ("corporate university" OR "corporate academy" OR "learning institute")
```

   * Verify academy website exists  
   * Confirm exact academy NAME  
5. **Data Capture:**

   * All fields per Academy Record Schema  
   * Discovery\_Source: "\[LMS Vendor\] case study \+ Website Validation"  
   * **LMS\_Platform:** \[Vendor Name\] ← MANDATORY FIELD  
   * Confidence\_Score: HIGH (case study \+ website) or MEDIUM (case study only)  
6. **Opportunistic Tech Stack:**

   * Read case study for mentions of: Zoom, Microsoft Teams, Adobe Connect, Webex  
   * If found, populate VILT\_Platform field  
7. **Contact Capture:**

   * Extract names/titles quoted in case studies  
   * Search LinkedIn → capture URLs  
   * Write to Contacts\_Master  
8. **Output:** Write immediately to `LMS_Discoveries` sheet

**Expected Yield:** 200-300 academies

**Stop Condition:** All major LMS vendors searched OR 300 academies found

---

### **4.3 Agent 3: Systematic Website Discovery Specialist**

**Purpose:** Process large company universe (8,000+ companies with 5K+ employees)

**Priority:** CRITICAL (highest volume source)

**Input Source:** Read from `Company_Universe` sheet

**Search Methodology:**

1. **Read Company\_Universe sheet, process in batches of 20 companies**

2. **For each company, execute search patterns:**

    **Pattern A:**

```
site:[company-domain].com ("corporate university" OR "corporate academy" OR "learning institute")
```

3.   
   **Pattern B:**

```
site:[company-domain].com (inurl:university OR inurl:academy OR inurl:institute) -customer -product
```

4.   
   **Pattern C:**

```
"[Company Name] University" OR "[Company Name] Academy" OR "[Company Name] Institute"
```

5.   
   **If academy found:**

   * Navigate to academy page  
   * Confirm: Branded learning entity (not product/customer training)  
   * Extract exact academy NAME  
   * Check for: Programs, curriculum, schedules, leadership  
6. **Data Capture:**

   * All fields per Academy Record Schema  
   * Discovery\_Source: "Website Reconnaissance"  
   * Confidence\_Score:  
     * CONFIRMED: Dedicated subdomain OR multi-page site  
     * HIGH: Dedicated section with programs  
     * MEDIUM: Mentioned but limited detail  
   * Evidence\_Description: Describe website structure and content found  
7. **Opportunistic Tech Stack:**

   * Check website footer, login pages for "Powered by \[LMS\]"  
   * Look for Zoom/Teams mentions  
   * Populate LMS\_Platform and VILT\_Platform if visible  
8. **Contact Discovery:**

   * Check academy site for "About" or "Leadership" pages  
   * Extract names, titles, emails if visible  
   * Search LinkedIn for each → capture URLs  
   * Write to Contacts\_Master  
9. **Output:**

   * Write to `Website_Discoveries` sheet immediately  
   * Update `Company_Universe` Status to "Completed"

**Time Budget:** 3-5 minutes per company maximum

**Expected Yield:** 400-600 academies from 8,000-10,000 companies

**Stop Condition:** All companies in Company\_Universe processed OR 600 academies found

---

### **4.4 Agent 4: Press Release Mining Specialist**

**Purpose:** Find academy launches and announcements through press archives

**Priority:** MEDIUM (captures emerging academies)

**Target Sources:**

* PR Newswire (prnewswire.com)  
* Business Wire (businesswire.com)  
* Company newsrooms  
* Google News (2018-2025)

**Search Methodology:**

1. **Execute search patterns:**

    **Search 1:**

```
site:prnewswire.com ("launches corporate university" OR "launches corporate academy" OR "launches learning institute")
```

2.   
   **Search 2:**

```
site:businesswire.com ("corporate university" OR "corporate academy") AND ("announces" OR "unveils" OR "launches")
```

3.   
   **Search 3:**

```
"virtual training" AND "academy" AND "program" 2020..2025
```

4.   
   **For each press release:**

   * Extract: Company name, academy name, launch date, program details  
   * Validate academy still exists:

```
site:[company-domain].com "[Academy Name]"
```

   * Determine if active or defunct  
5. **Data Capture:**

   * All fields per Academy Record Schema  
   * Discovery\_Source: "Press Release \[Date\] \+ Validation"  
   * Confidence\_Score:  
     * HIGH: Press release \+ current website active  
     * MEDIUM: Press release \+ limited web presence  
     * LOW: Press release but appears defunct  
   * Academy\_Launch\_Date: \[Date from press release\] if mentioned  
6. **Contact Capture:**

   * Press releases often include media contacts and quoted executives  
   * Extract all names, titles, emails, phone numbers  
   * Search LinkedIn for executives → capture URLs  
   * Write to Contacts\_Master  
7. **Opportunistic Tech Stack:**

   * If press mentions "powered by \[LMS\]" or platforms → capture  
8. **Output:** Write to `Press_Discoveries` sheet immediately

**Expected Yield:** 60-100 academies

**Stop Condition:** Diminishing returns (10+ searches with no new findings) OR 100 academies found

---

### **4.5 Agent 5: LinkedIn Intelligence Specialist**

**Purpose:** Discover academies through employee profiles and job postings; PRIMARY contact discovery

**Priority:** HIGH (best source for contacts)

**Search Methodology:**

1. **LinkedIn Profile Searches:**

    **Search 1:**

```
"Chief Learning Officer" [Company Name] site:linkedin.com/in
```

2.   
   **Search 2:**

```
"Dean" AND ("University" OR "Academy" OR "Institute") [Company Name] site:linkedin.com/in
```

3.   
   **Search 3:**

```
"Director of Learning" OR "VP Learning" [Company Name] site:linkedin.com/in
```

4.   
   **Search 4:**

```
"[Company] University" OR "[Company] Academy" site:linkedin.com/in
```

5.   
   **LinkedIn Job Searches:**

    **Search 1:**

```
site:linkedin.com/jobs "corporate university" OR "corporate academy"
```

6.   
   **Search 2:**

```
site:careers.[company-domain].com "university" OR "academy" OR "institute"
```

7.   
   **For each profile found:**

   * Extract: Name, title, academy name (if in profile), LinkedIn URL, email (if visible)  
   * Count profiles mentioning same academy (confidence indicator)  
8. **Validate academy:**

```
site:[company-domain].com "[Academy Name]"
```

9.   
   **Data Capture \- ACADEMY:**

   * All fields per Academy Record Schema  
   * Discovery\_Source: "LinkedIn \- \[X\] employee profiles mention academy"  
   * Confidence\_Score:  
     * HIGH: 3+ employees mention same academy  
     * MEDIUM: 1-2 employees mention academy  
   * Write to `LinkedIn_Discoveries` sheet  
10. **Data Capture \- CONTACTS (CRITICAL):**

    * For EVERY profile found, write to Contacts\_Master:  
      * All fields per Contact Record Schema  
      * Even if academy validation uncertain, capture the contact  
    * This is the primary mission of this agent  
11. **Job Posting Analysis:**

    * Extract company, job title, academy mentions, tech platforms  
    * Follow same validation steps

**Rate Limiting:** If LinkedIn blocks searches, wait 1 hour then resume

**Expected Yield:** 150-250 academies, 200+ contacts

**Stop Condition:** 500 contacts captured OR significant rate limiting

---

## **5\. ORCHESTRATOR SPECIFICATION**

**Purpose:** Monitor agent outputs, detect duplicates, merge records, auto-elevate confidence

**Execution Model:** Continuous monitoring loop

### **5.1 Monitoring Loop (Every 5 Minutes)**

**Step 1: Scan for New Records**

* Check row count in each agent sheet (Awards, LMS, Website, Press, LinkedIn)  
* Identify new rows since last scan

**Step 2: Duplicate Detection**

* Compare Academy\_Name across all agent sheets  
* Use fuzzy matching for variations:  
  * "McDonald's Hamburger University" \= "Hamburger University"  
  * "Visa University" \= "University of Visa"  
  * Case-insensitive matching  
* Flag duplicates for merging

**Step 3: Merge Process**

When duplicate detected:

1. **Create consolidated record in `Consolidated_Academies`:**

   * Academy\_ID: Generate unique (AC-0001, AC-0002, sequential)  
   * Academy\_Name: Use most complete name from all sources  
   * Parent\_Company: Use most complete version  
   * Academy\_URL: Use most complete/valid URL (prefer dedicated sites)  
   * Discovery\_Source: Combine all (e.g., "Awards Agent, LMS Agent, Website Agent")  
   * Evidence\_Description: MERGE all evidence from all agent records  
   * LMS\_Platform: Take from any agent that found it  
   * VILT\_Platform: Take from any agent that found it  
   * Contact\_Count: Count matching records in Contacts\_Master  
2. **Apply Confidence Boosting:**

   * If 2 agents found same academy: Upgrade MEDIUM → HIGH  
   * If 3+ agents found same academy: Upgrade HIGH → CONFIRMED  
   * If award recognition \+ 2 other sources: Auto-CONFIRMED  
3. **Update original records:**

   * Set Merge\_Status \= "MERGED" in agent sheets  
   * Add Academy\_ID reference  
4. **Log merge:**

   * Processing\_Log: `Timestamp | Merge_Complete | Academy [Name] merged from [X] sources | Orchestrator`

**Step 4: Auto-Elevation Rules**

For non-duplicate MEDIUM confidence records, upgrade to HIGH if ANY of:

**Rule 1: Multiple Evidence Sources**

* Has 3+ of: Website mention, LinkedIn profiles (2+), Job posting, Press mention, Fortune 1000 parent  
* AND parent company has 10,000+ employees

**Rule 2: Strong LinkedIn Signal**

* 3+ employee profiles mention same academy name

**Rule 3: Press \+ Website**

* Press release exists \+ academy website currently active

**Rule 4: Tech Stack \+ Website**

* LMS or VILT platform identified \+ website evidence

When auto-elevating:

* Update Confidence\_Score in original sheet  
* Add to Evidence\_Description: "Auto-elevated: \[Rule \#\]"  
* Log: `Timestamp | Auto_Elevation | Academy [Name] elevated via Rule [X] | Orchestrator`

**Step 5: Contact Linkage**

* For each academy in Consolidated\_Academies  
* Query Contacts\_Master for matching Parent\_Company  
* Update Contact records with Academy\_ID  
* Update academy Contact\_Count

**Step 6: Dashboard Statistics**

Calculate and log:

* Total academies in Consolidated\_Academies  
* By confidence: CONFIRMED: \[X\], HIGH: \[Y\], MEDIUM: \[Z\]  
* Tech stack known: LMS: \[X\], VILT: \[Y\]  
* Total contacts in Contacts\_Master  
* Companies processed from Company\_Universe

### **5.2 Error Handling**

**If merge fails:**

* Log error in Processing\_Log with details  
* Flag record with Notes: "MANUAL REVIEW REQUIRED"  
* Continue processing

**If spreadsheet write fails:**

* Retry up to 3 times  
* If still fails, log error  
* Continue processing other records

**If uncertain about auto-elevation:**

* Leave as MEDIUM, do not force upgrade  
* Add note explaining uncertainty

### **5.3 Logging**

**Every 30 minutes:**

* Write heartbeat: `Timestamp | Orchestrator_Heartbeat | Status: Running | Academies: [X] | Contacts: [Y] | Orchestrator`

**Stop Condition:**

* All agents report complete (via Processing\_Log)  
* AND no new records detected for 1 hour

---

## **6\. TERMINOLOGY REQUIREMENTS**

### **6.1 Equal Treatment of Terms**

All agents MUST search for all three terms with equal priority:

✅ **"Corporate Academy"** / **"\[Company\] Academy"** ✅ **"Corporate University"** / **"\[Company\] University"**  
 ✅ **"Learning Institute"** / **"\[Company\] Institute"**

### **6.2 Standard Search Pattern Template**

```
("corporate university" OR "corporate academy" OR "learning institute")
OR
("[Company] University" OR "[Company] Academy" OR "[Company] Institute")
OR
(inurl:university OR inurl:academy OR inurl:institute)
```

### **6.3 Naming Convention**

* Record the EXACT name as it appears on the academy website  
* Do not abbreviate or modify branding  
* If multiple variations exist, use the primary branded name

**Examples:**

* ✅ "Hamburger University" (not "McDonald's Academy")  
* ✅ "Deloitte University" (not "Deloitte Learning Institute")  
* ✅ "Visa University" (not "University of Visa")

---

## **7\. IMPLEMENTATION INSTRUCTIONS**

### **7.1 Pre-Execution Setup**

**Step 1: Create Google Sheet**

1. Create new Google Sheet: `Corporate_Academy_Discovery_Master`  
2. Create 9 tabs with exact names: `Consolidated_Academies`, `Awards_Discoveries`, `LMS_Discoveries`, `Website_Discoveries`, `Press_Discoveries`, `LinkedIn_Discoveries`, `Contacts_Master`, `Company_Universe`, `Processing_Log`  
3. Add column headers per Data Schema (Section 3\)  
4. Share sheet with edit access for Claude Code

**Step 2: Populate Company Universe**

1. Open `Company_Universe` sheet  
2. Populate with companies to research:  
   * Fortune 1000 list  
   * ZoomInfo/Apollo search: US companies, 5,000+ employees  
   * Industry-specific lists (QSR, Banking, Tech, Retail, etc.)  
   * Target: 8,000-10,000 companies  
3. Fill columns: Company\_Name, Domain, Industry, Employee\_Count, Status (set all to "Not\_Started"), Date\_Added

**Step 3: Configure Sheet Access**

1. Get Google Sheet ID from URL  
2. Ensure Claude Code has API access to write/read

### **7.2 Execution Sequence**

**Phase 1: Launch All Agents (Parallel Execution)**

Claude Code should spawn parallel processes for each agent:

1. **Agent 1:** Execute Awards Mining (Section 4.1)  
2. **Agent 2:** Execute LMS Reverse Engineering (Section 4.2)  
3. **Agent 3:** Execute Website Discovery (Section 4.3)  
4. **Agent 4:** Execute Press Mining (Section 4.4)  
5. **Agent 5:** Execute LinkedIn Intelligence (Section 4.5)  
6. **Orchestrator:** Execute monitoring loop (Section 5\)

**Phase 2: Monitor Progress**

* Check `Consolidated_Academies` for growing list of merged records  
* Check `Processing_Log` for agent status and errors  
* Monitor for error codes (Section 8\)

**Phase 3: Agent Completion**

* Agents stop when reaching their individual stop conditions  
* Orchestrator continues monitoring until 1 hour of no new records  
* Final heartbeat log indicates completion

**Phase 4: Final Validation**

After all agents complete:

1. **URL Validation:** Test 100% of Academy\_URLs for HTTP 200 response  
2. **Duplicate Scan:** Final fuzzy match check for similar names  
3. **Contact Linkage:** Verify all contacts have valid Academy\_ID  
4. **Spot Check:** Manually review 10% random sample (50 records):  
   * Verify URLs load  
   * Confirm academy names accurate  
   * Check confidence scores appropriate

**Phase 5: Export**

1. Export `Consolidated_Academies` → CSV for CRM import  
2. Export `Contacts_Master` → CSV for CRM contact import  
3. Archive Google Sheet for reference

---

## **8\. ERROR CODES & HANDLING**

### **8.1 Standard Error Codes**

All agents log errors to `Processing_Log` using these codes:

| Code | Description | Agent Action | Orchestrator Action |
| ----- | ----- | ----- | ----- |
| E001 | Website 404/not found | Skip, note in Evidence | Flag for review |
| E002 | Rate limit hit | Wait 1 hour, retry | Monitor retry success |
| E003 | Unable to parse academy name | Use best guess \+ flag | Add to review queue |
| E004 | Spreadsheet write failed | Retry 3x | Alert if persists |
| E005 | Search returned zero results | Move to next search | None |
| E006 | Academy appears defunct | Mark LOW confidence | Flag for exclusion |

### **8.2 Logging Format**

```
Timestamp | ERROR | [Error Code] - [Description] | [Agent Name]
```

**Example:**

```
2025-10-28 14:32:18 | ERROR | E002 - LinkedIn rate limit reached | Agent_5
```

### **8.3 Critical Errors (Stop Execution)**

* E004 (spreadsheet write) fails after 3 retries → Halt agent, alert  
* Google Sheets API access denied → Halt all agents  
* Company\_Universe sheet missing or corrupted → Halt Agent 3

---

## **9\. QUALITY ASSURANCE**

### **9.1 Real-Time Quality Indicators**

**System is functioning correctly if:**

✅ Each agent sheet shows 50+ rows within first week ✅ Consolidated\_Academies shows records with Discovery\_Source listing 2+ agents ✅ Confidence\_Score distribution: \~60% CONFIRMED/HIGH, \~40% MEDIUM ✅ Contacts\_Master accumulates 100+ contacts in first week ✅ Processing\_Log shows regular "Batch\_Complete" entries from all agents ✅ No persistent E004 errors in Processing\_Log

### **9.2 Data Quality Checks (Automated)**

**Orchestrator performs these checks:**

1. **Mandatory Field Validation:**

   * Academy\_Name, Parent\_Company, Discovery\_Source, Confidence\_Score always populated  
   * Flag records with missing mandatory fields  
2. **URL Format Validation:**

   * Academy\_URL starts with http:// or https://  
   * Domain matches or relates to Parent\_Domain  
3. **Confidence Logic Validation:**

   * CONFIRMED records have 3+ evidence sources  
   * HIGH records have 2+ evidence sources  
   * MEDIUM records have 1+ evidence source  
4. **Contact Linkage Validation:**

   * All Contact records have valid Academy\_ID  
   * Academy\_Name in contact matches academy record

### **9.3 Success Metrics Dashboard**

Track these metrics in real-time (display in Processing\_Log or separate tab):

**Volume Metrics:**

* Total Academies: \[Target: 500+\]  
* CONFIRMED: \[Target: 300+, 60%\]  
* HIGH: \[Target: 150+, 30%\]  
* MEDIUM: \[Target: 50+, 10%\]

**Discovery Source Distribution:**

* Cross-validated (2+ agents): \[Target: 40%+\]  
* Single source: \[Target: 60%\]

**Enrichment Metrics:**

* LMS Platform Known: \[Target: 40%+\]  
* VILT Platform Known: \[Target: 30%+\]  
* Contact Count: \[Target: 200+\]  
* Contacts with Email: \[Target: 20%+\]

**Processing Progress:**

* Companies Researched: \[X\] / \[8,000-10,000\]  
* Estimated Completion: \[Date based on velocity\]

---

## **10\. EXECUTION TIMELINE**

### **10.1 Expected Velocity**

**Agent Processing Speeds:**

| Agent | Time Required | Expected Yield |
| ----- | ----- | ----- |
| Agent 1 (Awards) | 40-60 hours | 100-150 academies |
| Agent 2 (LMS) | 60-80 hours | 200-300 academies |
| Agent 3 (Website) | 400-500 hours\* | 400-600 academies |
| Agent 4 (Press) | 30-50 hours | 60-100 academies |
| Agent 5 (LinkedIn) | 80-120 hours | 150-250 academies \+ 200+ contacts |

\*Can be parallelized across multiple instances

### **10.2 Parallel Execution Timeline**

**Week 1-2:**

* All agents launch simultaneously  
* Agents 1, 2, 4, 5 complete dedicated searches  
* Agent 3 starts Fortune 1000 processing  
* Orchestrator begins merging

**Week 3-4:**

* Agent 3 continues company universe  
* Cross-validation boosts confidence scores  
* MEDIUM records auto-elevate to HIGH

**Week 5-6:**

* Agent 3 completes full universe  
* Diminishing returns across all agents  
* Focus shifts to quality validation

**Week 7-8:**

* Final validation and spot-checking  
* Export preparation  
* Documentation

**Total Timeline:** 6-8 weeks (can compress to 4-5 weeks with parallelization)

---

## **11\. DELIVERABLES**

### **11.1 Primary Deliverable**

**Google Sheet: `Corporate_Academy_Discovery_Master`**

With populated tabs:

* `Consolidated_Academies`: 500+ validated academy records  
* `Contacts_Master`: 200+ L\&D leadership contacts  
* `Processing_Log`: Complete audit trail

### **11.2 Export Files**

**File 1: `corporate_academies_export.csv`**

* All records from Consolidated\_Academies  
* Formatted for CRM import  
* Includes all fields per Academy Record Schema

**File 2: `contacts_export.csv`**

* All records from Contacts\_Master  
* Formatted for CRM contact import  
* Linked to academies via Academy\_ID

### **11.3 Documentation**

**File 3: `execution_summary.md`**

* Final statistics (total academies, confidence distribution, etc.)  
* Key findings and insights  
* Notable academies discovered  
* Recommendations for Phase 2 enrichment

---

## **12\. APPENDICES**

### **12.1 Confidence Score Definitions**

**CONFIRMED (60% target):**

* 3+ independent sources validate academy exists  
* Examples: Award recognition \+ LMS case study \+ Dedicated website  
* Or: Website \+ LinkedIn (5+ employees) \+ Job postings \+ Press release

**HIGH (30% target):**

* 2 strong sources validate academy  
* Examples: LMS case study \+ Website validation  
* Or: Award winner \+ Website mention \+ LinkedIn profiles

**MEDIUM (10% target):**

* 1-2 weaker sources suggest academy exists  
* Requires auto-elevation or manual review  
* Examples: LinkedIn mentions (1-2 profiles) \+ company meets size threshold

### **12.2 Search Operator Reference**

**Google Search Operators:**

```
site:[domain].com - Search specific domain
inurl:[text] - URL contains text
intitle:[text] - Page title contains text
"exact phrase" - Exact match
OR - Boolean OR
- (minus) - Exclude term
.. (two periods) - Date range (e.g., 2020..2025)
```

**Example Combined Search:**

```
site:mcdonalds.com ("corporate university" OR "academy") -customer -product
```

### **12.3 Common Academy Name Patterns**

**Corporate Structure Names:**

* "\[Company\] University"  
* "\[Company\] Academy"  
* "\[Company\] Learning Institute"  
* "University of \[Company\]"

**Descriptive Names:**

* "\[Company\] Leadership Institute"  
* "\[Company\] Learning Center"  
* "\[Company\] Training Academy"

**Unique Branded Names:**

* "Hamburger University" (McDonald's)  
* "Apple University" (Apple)  
* "Pixar University" (Pixar)

### **12.4 Technology Platform Reference**

**Common LMS Platforms:**

* Cornerstone OnDemand  
* Docebo  
* SAP SuccessFactors Learning  
* Workday Learning  
* Degreed  
* EdCast  
* 360Learning  
* Absorb LMS

**Common VILT Platforms:**

* Zoom (Webinar, Meetings)  
* Microsoft Teams  
* Adobe Connect  
* Webex Training  
* GoToTraining

---

## **13\. EXECUTION CHECKLIST**

### **13.1 Pre-Launch Checklist**

* \[ \] Google Sheet created with all 9 tabs  
* \[ \] Column headers added to all sheets per schema  
* \[ \] Company\_Universe populated with 8,000+ companies  
* \[ \] Google Sheets API access configured  
* \[ \] Claude Code has edit permissions on sheet

### **13.2 Agent Launch Checklist**

* \[ \] Agent 1 (Awards) process started  
* \[ \] Agent 2 (LMS) process started  
* \[ \] Agent 3 (Website) process started  
* \[ \] Agent 4 (Press) process started  
* \[ \] Agent 5 (LinkedIn) process started  
* \[ \] Orchestrator monitoring loop started

### **13.3 Quality Monitoring Checklist**

* \[ \] Each agent sheet showing new rows within 24 hours  
* \[ \] Processing\_Log showing regular heartbeat entries  
* \[ \] No persistent E004 errors  
* \[ \] Consolidated\_Academies accumulating merged records  
* \[ \] Contact\_Count fields updating in academy records

### **13.4 Completion Checklist**

* \[ \] All agents logged completion in Processing\_Log  
* \[ \] Final academy count: 500+  
* \[ \] Confidence distribution: \~60% CONFIRMED, \~30% HIGH  
* \[ \] URL validation completed (100% of records)  
* \[ \] Spot-check validation completed (10% sample)  
* \[ \] Export files generated (CSV format)  
* \[ \] Execution summary documented

---

**END OF DOCUMENT**

