---
name: "Paid Media to Pipeline Reporting"
description: "Full-funnel reporting for performance marketers running paid media into a CRM sales pipeline. Blends ad platform data with pipeline data to produce end-to-end reporting: Spend → Leads → Appointments → Qualified Appointments → Customers. Use for weekly summaries, monthly reports, quarterly reviews, CPL/CPA budget rule enforcement, and any request involving paid media performance tied to pipeline outcomes."
---

# Paid Media to Pipeline Reporting

## WHAT THIS SKILL DOES

This skill is the **single source of truth** for connecting paid media performance to sales pipeline outcomes. It replaces the need to analyze ad data and CRM data separately. Every report blends BOTH data sources into one unified view:

**Ad Platform Data (Top of Funnel):** Spend, Leads, CPL, CTR, CPM, CPC
**CRM Pipeline Data (Bottom of Funnel):** Appointments, Qualified Appointments, Customers, Stage Progression

The final output is always a **blended table** showing the full journey from ad dollar to closed customer, segmented by Platform × Campaign Group × Region (if applicable).

---

## FIRST-TIME SETUP

Before running any reports, this skill needs to understand your business. It will walk you through configuration in two tiers.

### Tier 1: Minimum Viable Config (Required)

The skill MUST ask these questions before producing any output:

**1. What are your funnel stages?**

Default four-stage hierarchy (user can rename, add, or remove):

| Stage | Default Label | Example Alternatives |
|-------|--------------|---------------------|
| Stage 1 | Lead | Form Fill, Inquiry, MQL |
| Stage 2 | Appointment | Demo, Consultation, Estimate, Discovery Call |
| Stage 3 | Qualified Appointment | SQL, Qualified Demo, Signed Proposal, Booked Job |
| Stage 4 | Customer | Closed Won, Contract Signed, Completed Job |

Ask: *"What do you call each stage in your funnel? I'll use these labels throughout all reports."*

**2. What are your cost benchmarks?**

For each funnel stage, ask for:
- **Target benchmark** (ideal cost per stage)
- **Hard ceiling** (maximum acceptable cost — triggers immediate pullback if breached)

Example prompt: *"What's your target Cost per Lead and the maximum you'd accept before pulling back spend? Now same question for Cost per Appointment..."*

Store as:

```
benchmarks:
  cost_per_lead:
    target: $XX
    ceiling: $XX
  cost_per_appointment:
    target: $XX
    ceiling: $XX
  cost_per_qualified_appointment:
    target: $XX
    ceiling: $XX
  cost_per_customer:
    target: $XX
    ceiling: $XX
```

**3. What ad platforms are you running?**

Options: META (Facebook/Instagram), Google Ads, LinkedIn Ads, TikTok Ads, Microsoft Ads, other.

**4. What is your data source?**

Options:
- **Windsor.ai MCP** → Skill will prompt for connector name and account ID
- **CSV upload** → Skill will specify required column format
- **Manual input** → Skill will provide a template for pasting data
- **Other MCP connector** → Skill will ask for connector details

**5. What CRM do you use for pipeline tracking?**

Options: HubSpot, Salesforce, Close, Pipedrive, other.

For each CRM, the skill needs:
- Pipeline name or ID
- How to identify each funnel stage (stage names or IDs)
- What field defines a "qualified" outcome vs. an unqualified one
- Contact/deal attribution fields (how you track which ad platform drove the lead)

### Tier 2: Advanced Config (Optional — prompted when relevant)

These are asked the first time a feature becomes relevant, NOT during initial setup:

**Budget decision rules:**
```
budget_rules:
  increase_spend_trigger:
    metric: cost_per_lead
    condition: below_target
    consecutive_periods: 2
  decrease_spend_trigger:
    metric: cost_per_lead
    condition: above_target
    consecutive_periods: 2
  override_increase:
    metric: cost_per_appointment  # downstream metric can override CPL signal
    condition: below_target
    consecutive_periods: 2
  override_decrease:
    metric: cost_per_appointment  # downstream metric can override CPL signal
    condition: above_ceiling
    consecutive_periods: 2
```

**Campaign taxonomy:**
- How campaigns are grouped (by string matching on campaign names)
- What funnel category each group belongs to (bottom-funnel vs. mid-funnel)
- Exclusion strings (campaigns to ignore in reporting)

**Geographic or custom segmentation:**
- Do you segment by region, product line, brand vs. non-brand, or other?
- How are segments identified in campaign names?

**Reporting audience context:**
- Who receives these reports? (executive, marketing director, client, etc.)
- What level of technical detail is appropriate?

---

## BENCHMARKS & BUDGET RULES

### Budget Decision Framework

The skill enforces a **two-signal system**: the primary signal (typically CPL) and an override signal (a downstream metric like Cost per Appointment or Cost per Customer).

**Default logic (user configures thresholds during setup):**

| Condition | Action |
|-----------|--------|
| CPL below target for 2 consecutive periods | Increase spend |
| CPL above target for 2 consecutive periods | Pull back spend |
| CPL above hard ceiling for ANY period | Pull back spend immediately |
| Downstream cost below target for 2 periods | May increase spend (even if CPL is above target) |
| Downstream cost above ceiling for 2 periods | Pull back spend (even if CPL is below target) |

**Why this matters:** CPL alone is misleading if lead quality is poor. A $20 CPL means nothing if those leads never convert to appointments. The downstream override catches this.

### Red Flags to Monitor (Universal)

- Primary metric exceeds hard ceiling for 2+ periods → immediate action required
- Conversion rate between any two funnel stages drops >20% vs. prior period → investigate
- High "Unknown" or unattributed leads → tracking gap in lead capture
- Lead volume rises but downstream conversion falls → lead quality degradation
- Spend stable but leads declining → algorithm issue, audience saturation, or platform problem

---

## DATA PULL PROCESS

**NEVER take shortcuts. Always execute ALL steps in order.**

### Step 1: Pull Paid Media Data

**If using Windsor.ai MCP:**
```
Windsor.ai:get_data
- connector: "[user_connector]"
- accounts: ["[user_account_id]"]
- fields: ["campaign", "date", "spend", "[lead_conversion_field]", "clicks", "impressions", "ctr", "cpm", "cpc"]
- date_from: "[START_DATE]"
- date_to: "[END_DATE]"
- filter: [appropriate filter for campaign group]
```

Execute for EACH campaign group defined in the user's taxonomy.

**If using CSV upload:**
Expected columns: `date`, `campaign_name`, `spend`, `leads`, `clicks`, `impressions`
Optional columns: `ctr`, `cpm`, `cpc` (calculated if not provided)

**If using manual input:**
Provide this template for the user to fill:
```
| Campaign Group | Spend | Leads | Clicks | Impressions |
|---------------|-------|-------|--------|-------------|
| [Group 1]     | $X    | X     | X      | X           |
| [Group 2]     | $X    | X     | X      | X           |
```

**Post-processing:**
- Filter out campaigns with $0 spend and 0 impressions (inactive)
- Group by campaign taxonomy (Campaign Group × Funnel Category × Segment)
- Flag any data anomalies (missing values, sudden spikes/drops)

### Step 2: Pull Pipeline Data from CRM

**If using HubSpot:**
```
HubSpot:search_crm_objects
- objectType: DEAL
- filterGroups: [{filters: [
    {propertyName: "pipeline", operator: "EQ", value: "[user_pipeline_id]"},
    {propertyName: "createdate", operator: "GTE", value: "[START_DATE]"},
    {propertyName: "createdate", operator: "LT", value: "[END_DATE_+1_DAY]"}
  ]}]
- properties: ["dealname", "dealstage", "createdate", "[user_attribution_fields]", "[user_segment_fields]"]
- limit: 200
```

**If using another CRM:** Adapt the query structure to the available MCP or instruct the user on what data to export.

**If total results exceed limit:** Paginate using the offset/cursor returned.

### Step 3: Classify Pipeline Deals by Funnel Stage

Map each deal to the user's defined funnel stages:

| Deal Stage ID/Name | Maps To |
|-------------------|---------|
| [User's Stage 1 values] | Lead |
| [User's Stage 2 values] | Appointment |
| [User's Stage 3 values] | Qualified Appointment |
| [User's Stage 4 values] | Customer |

Count deals at each stage for the reporting period.

### Step 4: Get Attribution for Pipeline Deals

Pull attribution data to connect pipeline deals back to paid media campaigns.

Attribution fields live where the user specified during setup (contact record, deal record, or custom object). Follow the user's configuration for which field to use and which record to pull it from.

**CRITICAL RULES:**
- Batch lookups to avoid API timeouts
- If attribution is missing → mark as "Unknown" (do NOT skip)
- Track the percentage of Unknown attribution as a data quality metric

### Step 5: Map Pipeline Deals to Campaign Groups

Match each deal to its paid media campaign group using attribution data.

### Step 6: Calculate Blended Metrics

For each campaign group:
- **CPL** = Spend / Leads
- **Cost per Appointment** = Spend / Appointments
- **Cost per Qualified Appointment** = Spend / Qualified Appointments
- **Cost per Customer** = Spend / Customers
- **Stage Conversion Rates** = Stage N count / Stage N-1 count

### Step 7: Flag Data Quality Issues

Always report:
- % of pipeline deals with Unknown/missing attribution
- % of deals missing segmentation data (if segmentation is configured)
- Any test or junk deals detected (flag naming patterns the user specifies)
- Discrepancies between ad platform lead counts and CRM lead counts

---

## OUTPUT FORMAT

### Default Blended Table

If the user has configured funnel stages as Lead → Appointment → Qualified Appointment → Customer:

```
| Campaign Group | Spend | Leads | CPL | Appts | Appt Rate | CPA | Qual Appts | CPQA | Customers | CPC |
|---------------|-------|-------|-----|-------|-----------|-----|-----------|------|-----------|-----|
| [Group 1]     | $X    | X     | $X  | X     | X%        | $X  | X         | $X   | X         | $X  |
| [Group 2]     | $X    | X     | $X  | X     | X%        | $X  | X         | $X   | X         | $X  |
| Totals        | $X    | X     | $X  | X     | X%        | $X  | X         | $X   | X         | $X  |
```

**Column labels adapt to whatever the user named their funnel stages.**

### Segmented Views (If Configured)

If the user segments by region, product line, or other dimension:
- **Combined view** = all segments rolled together by campaign group
- **Segment view** = same table structure filtered to one segment

### Funnel Category Split

If the user has defined funnel categories (e.g., bottom-funnel vs. mid-funnel campaigns):
- Every view MUST split into separate sections by funnel category
- Each category may have different benchmarks

---

## INSIGHT FORMAT

### Headline

A natural, executive-readable sentence summarizing the period and key finding. Should read like a status update. Start with the time period, state what's working, then pivot to the core issue.

Example: "March weekly totals show an efficient $38 CPL across 145 leads, but appointment conversion is lagging at 4.2% versus the 8% target"

### Bullet Format

❏ **Bold lead-in phrase, then a period.** Rest of the bullet is plain text with metric details in parentheses and strategic insight or recommended action. Keep bullets to 2-3 sentences max.

❏ **Second bullet follows the same pattern.** Supporting data point or comparison (vs. benchmark, vs. prior period, vs. other segment) with concluding action or implication.

### Section Structure

Reports follow this structure:
1. **Headline Takeaways** — what leadership should know in 30 seconds
2. **Blended Performance Table** — the numbers
3. **What's Working** — campaigns, audiences, or platforms performing above benchmark
4. **What's Not Working** — underperformers with root cause hypothesis
5. **Opportunities** — actions to scale what's working
6. **Risks** — threats to flag before they become problems
7. **Next Steps** — specific, time-bound recommendations

---

## ANALYSIS MODES

### Weekly Summary (default reporting period: Mon–Sun)
- 3-5 key insights with recommendations
- Blended table for the week
- Flag any budget rule triggers (consecutive-period trends)
- Compare vs. prior week and vs. benchmarks
- Quick pipeline status

### Monthly Report
- Full blended table for the month
- Pipeline breakdown by Campaign Group × Segment (if applicable)
- What's Working / What's Not / Opportunities / Risks / Next Steps
- MoM comparisons
- Budget rule assessment
- Data quality audit (Unknown attribution %, missing fields %)

### Quarterly Review
- 3-month trend analysis
- Strategic recommendations for next quarter
- Platform performance comparison
- Segment comparison (if applicable)
- Seasonal pattern identification
- Target tracking vs. annual goals

---

## DIAGNOSTIC FRAMEWORK

When performance issues arise, follow this diagnostic order:

### 1. Front-End Diagnosis (Ad Platform Data)
- **CTR dropping?** → Creative fatigue, audience saturation, or seasonal decline
- **CPM rising?** → Increased competition, audience too narrow, or platform auction changes
- **CPL rising but CTR stable?** → Landing page issue or conversion tracking problem
- **Leads dropping but spend stable?** → Algorithm issue, campaign structure problem, or budget cap hit

### 2. Back-End Diagnosis (CRM Pipeline Data)
- **Appointment rate dropping?** → Sales team capacity, lead quality decline, or speed-to-lead issue
- **Qualification rate dropping?** → Wrong audience reaching pipeline, or qualification criteria changed
- **Cost per Appointment rising but CPL stable?** → Back-end conversion problem, NOT an acquisition problem
- **High Unknown attribution?** → Tracking gap in lead capture forms or CRM integration

### 3. Root Cause Mapping
| Symptom | Front-End Cause | Back-End Cause |
|---------|----------------|----------------|
| High cost per downstream stage | High CPL driving up costs | Low conversion between stages |
| Low appointment rate | Poor lead quality from ads | Sales team not following up |
| Low qualification rate | Wrong audience targeting | Product-market mismatch |
| High Unknown attribution % | — | Form/CRM tracking broken |

---

## CRITICAL REMINDERS

### Never Take Shortcuts
- **ALWAYS pull both data sources** — ad platform AND CRM pipeline
- **ALWAYS verify attribution** — don't assume; check the actual field values
- **ALWAYS recalculate metrics** — don't trust provided percentages or pre-calculated fields
- **ALWAYS flag data quality issues** — Unknown attribution, missing fields, test records
- **ALWAYS compare** — vs. benchmark, vs. prior period, vs. other segments

### Reporting Standards
- Headlines first: what leadership should know in 30 seconds
- Action-oriented insights — explain the "so what" for every data point
- Compare everything — vs. benchmark, vs. prior period, vs. other segments
- End with clear next steps — budget decisions, creative actions, diagnostic follow-ups

### Data Quality Checks
- Verify lead counts: ad platform leads vs. CRM lead count (flag discrepancies >10%)
- Verify attribution coverage: what % of pipeline deals have known attribution?
- Verify stage progression: deals should flow forward; flag anomalies
- Exclude test/junk records: use naming patterns the user specifies during setup
- Check for duplicates: same person appearing multiple times in pipeline

---

**Version:** 1.0
**Category:** Paid Media Analytics
**Target User:** Performance marketers running paid acquisition into a CRM sales pipeline
