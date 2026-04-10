# Paid Media to Pipeline Reporting

A Claude skill for performance marketers who run paid media into a CRM sales pipeline and need full-funnel reporting from ad spend through closed customer.

---

## What This Does

If you run paid ads (META, Google, LinkedIn, TikTok, etc.) that generate leads, and those leads enter a sales pipeline where a team works them toward a closed deal — this skill connects those two worlds.

Most reporting stops at Cost per Lead. This skill goes further:

**Spend → Leads → Appointments → Qualified Appointments → Customers**

It pulls data from your ad platforms and your CRM, blends them into one table, and tells you what's actually driving revenue — not just what's driving cheap clicks.

It also enforces budget rules: when to scale spend, when to pull back, and when downstream metrics should override your CPL signal.

---

## Who This Is For

- **Performance marketers** managing paid acquisition for businesses with a sales pipeline
- **Industries:** Solar, home services, insurance, real estate, SaaS, legal, financial advisory — any business where paid leads get worked by a sales team
- **NOT for:** Ecommerce, direct-to-consumer without a sales pipeline, or brand awareness campaigns without conversion tracking

---

## What Is a Claude Skill?

If you're new to Claude skills — a skill is a set of instructions that tells Claude how to perform a specific workflow. Think of it as a custom playbook that Claude follows when you ask it to do something.

Instead of explaining your reporting process from scratch every time, the skill encodes your funnel stages, benchmarks, data sources, and formatting preferences so Claude can run your reports consistently.

### How Skills Work

1. You install the skill in your Claude environment (instructions below)
2. When you ask Claude something that matches the skill's purpose (e.g., "run my weekly paid media report"), Claude automatically activates the skill
3. The skill guides Claude through the exact data pull, analysis, and formatting process
4. You get consistent, presentation-ready output every time

---

## Installation

### For Claude.ai (Projects)

1. Create a new Project in Claude.ai (or open an existing one)
2. Go to Project Settings → Skills
3. Click "Add Skill"
4. Copy the contents of `SKILL.md` from this repo and paste it into the skill editor
5. Save

### For Claude Code

1. Create the skill directory:
```bash
mkdir -p ~/.claude/skills/paid-media-to-pipeline-reporting
```

2. Copy the skill file:
```bash
cp SKILL.md ~/.claude/skills/paid-media-to-pipeline-reporting/SKILL.md
```

3. Optionally copy the config template and examples:
```bash
cp CONFIG_TEMPLATE.md ~/.claude/skills/paid-media-to-pipeline-reporting/
cp -r examples/ ~/.claude/skills/paid-media-to-pipeline-reporting/examples/
```

4. Restart Claude Code. The skill will be detected automatically.

### For Claude API / SDK

Include the contents of `SKILL.md` in your system prompt when making API calls related to paid media reporting.

---

## First-Time Setup

When you first use the skill, it will ask you a series of setup questions:

### Tier 1 — Required (gets you running)

1. **What are your funnel stages?** — The skill ships with a default: Lead → Appointment → Qualified Appointment → Customer. You can rename these to match your business (e.g., Lead → Demo → SQL → Closed Won).

2. **What are your cost benchmarks?** — For each stage, set a target (ideal cost) and a ceiling (maximum acceptable cost). These power the budget rules.

3. **What ad platforms do you use?** — META, Google, LinkedIn, TikTok, etc.

4. **Where does your data live?** — Options: Windsor.ai (MCP connector), CSV uploads, manual input, or another MCP connector.

5. **What CRM do you use?** — HubSpot, Salesforce, Close, Pipedrive, etc. The skill needs to know your pipeline ID, stage mappings, and attribution fields.

### Tier 2 — Optional (unlocks advanced features)

These are prompted when relevant, not during initial setup:

- **Budget decision rules** — When to scale spend, when to pull back, override logic
- **Campaign taxonomy** — How to group and classify campaigns by name patterns
- **Geographic or custom segmentation** — Regional, product line, or other breakdowns
- **Data quality rules** — Test record exclusions, attribution quality thresholds

You can also pre-fill the configuration by editing `CONFIG_TEMPLATE.md` directly. See `examples/EXAMPLE_CONFIG_SOLAR.md` for a completed example.

---

## How to Use It

Once configured, you can request reports in three modes:

### Weekly Summary
```
"Run my weekly paid media summary for last week"
```
Returns 3-5 key insights, a blended performance table, budget rule flags, and comparisons vs. prior week and benchmarks.

### Monthly Report
```
"Generate my monthly paid media report for March"
```
Returns full blended tables, pipeline breakdown by campaign group, What's Working / What's Not / Opportunities / Risks / Next Steps, and MoM comparisons.

### Quarterly Review
```
"Pull together a Q1 quarterly review"
```
Returns 3-month trend analysis, strategic recommendations, platform performance comparison, and annual target tracking.

### Ad Hoc Analysis
```
"Why did our CPL spike last week?"
"Compare META vs. Google performance this month"
"What's our appointment rate trend over the last 6 weeks?"
```
The skill uses its diagnostic framework to investigate performance issues and identify root causes.

---

## What's in This Repo

```
paid-media-to-pipeline-reporting/
├── SKILL.md                              # Core skill file (install this)
├── CONFIG_TEMPLATE.md                    # Blank config for your business
├── README.md                             # You're reading it
└── examples/
    └── EXAMPLE_CONFIG_SOLAR.md           # Completed example (fictional solar company)
```

---

## Data Source Requirements

The skill is **connector-agnostic** — it doesn't require a specific data tool. It supports:

| Data Source | How It Works |
|-------------|-------------|
| **Windsor.ai MCP** | Pulls data directly via MCP connector. Requires Windsor.ai account and connected ad platform. |
| **CSV Upload** | You export from your ad platform and upload. Skill specifies required column format. |
| **Manual Input** | You paste data into a template. Good for quick analysis or platforms without exports. |
| **Other MCP** | Any MCP connector that exposes ad platform data. Configure during setup. |

For CRM data, the skill currently has built-in support for **HubSpot** queries. For Salesforce, Pipedrive, Close, or others, the skill adapts its query structure to whatever MCP or export format is available.

---

## Key Concepts

### The Two-Signal Budget System

Most performance marketers make budget decisions on CPL alone. This skill uses two signals:

- **Primary signal:** Cost per Lead (or whatever your Stage 1 metric is)
- **Override signal:** A downstream metric (Cost per Appointment, Cost per Customer, etc.)

The override catches a common trap: CPL looks great, but the leads are junk and never convert. Or CPL looks high, but those expensive leads close at a much higher rate. The downstream signal corrects for lead quality.

### Funnel Stage Flexibility

The skill doesn't force you into a specific funnel. You define your stages, and the skill adapts all tables, metrics, budget rules, and diagnostics to your definitions. A four-stage funnel and a two-stage funnel both work.

### Diagnostic Framework

When something goes wrong, the skill follows a structured diagnosis: front-end (ad platform) causes first, then back-end (CRM/pipeline) causes. This prevents the common mistake of blaming ads for what's actually a sales team problem, or vice versa.

---

## Contributing

If you use this skill and find ways to improve it — better diagnostic patterns, additional CRM support, new data source integrations — pull requests are welcome.

---

## License

MIT
