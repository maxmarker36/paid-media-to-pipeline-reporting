# Example Configuration — SunPeak Solar (Fictional)

This is a completed example showing how a solar installation company might configure the skill. Use this as a reference when filling out your own `CONFIG_TEMPLATE.md`.

---

## 1. Company Context

```yaml
company_name: "SunPeak Solar"
industry: "Solar / Home Services"
business_model: "Lead gen for in-house sales team. Paid ads drive homeowner inquiries, sales reps qualify and close."
reporting_audience: "VP of Marketing — business-focused, wants headline insights and budget recommendations, not deep platform mechanics"
technical_detail_level: "business"
```

---

## 2. Funnel Stages

```yaml
funnel_stages:
  - label: "Lead"
    definition: "Homeowner submits form or calls from ad"
    crm_stage_ids: ["new_lead", "contacted"]
  - label: "Appointment"
    definition: "In-home consultation scheduled"
    crm_stage_ids: ["appointment_set", "appointment_completed"]
  - label: "Qualified Appointment"
    definition: "Roof qualifies for solar, homeowner received proposal"
    crm_stage_ids: ["proposal_sent"]
  - label: "Customer"
    definition: "Contract signed"
    crm_stage_ids: ["contract_signed", "installation_scheduled"]
```

---

## 3. Cost Benchmarks

```yaml
benchmarks:
  cost_per_lead:
    target: 45
    ceiling: 55
  cost_per_appointment:
    target: 180
    ceiling: 250
  cost_per_qualified_appointment:
    target: 400
    ceiling: 600
  cost_per_customer:
    target: 1200
    ceiling: 1800
```

---

## 4. Budget Decision Rules

```yaml
budget_rules:
  primary_metric: "cost_per_lead"
  consecutive_periods_required: 2
  reporting_period: "weekly"

  increase_spend_when:
    - metric: "cost_per_lead"
      condition: "below_target"
      periods: 2

  decrease_spend_when:
    - metric: "cost_per_lead"
      condition: "above_target"
      periods: 2
    - metric: "cost_per_lead"
      condition: "above_ceiling"
      periods: 1

  override_increase:
    - metric: "cost_per_customer"
      condition: "below_target"
      periods: 2
      note: "If we're closing customers cheaply, we can tolerate higher CPL"

  override_decrease:
    - metric: "cost_per_customer"
      condition: "above_ceiling"
      periods: 2
      note: "If customer acquisition cost is too high, pull back even if CPL looks good"
```

---

## 5. Data Sources

### Ad Platform Data

```yaml
ad_platforms:
  - platform: "META"
    data_source: "windsor_ai"
    windsor_connector: "facebook"
    windsor_account_id: "9876543210"
    lead_conversion_field: "actions_offsite_conversion_fb_pixel_lead"
  - platform: "Google Ads"
    data_source: "csv_upload"
    csv_column_mapping:
      date: "Day"
      campaign_name: "Campaign"
      spend: "Cost"
      leads: "Conversions"
      clicks: "Clicks"
      impressions: "Impr."
```

### CRM Pipeline Data

```yaml
crm:
  platform: "hubspot"
  pipeline_id: "solar_sales_pipeline"

  attribution_field: "original_source_detail"
  attribution_object: "contact"
  attribution_values:
    meta: "Facebook Paid"
    google: "Google Paid"

  segment_field: "service_region"
  segment_object: "deal"
  segment_values:
    "Phoenix Metro": "PHX"
    "Tucson": "TUC"
    "Flagstaff": "FLAG"
```

---

## 6. Campaign Taxonomy

```yaml
campaign_groups:
  - name: "META Solar Install"
    match_rule: "contains 'Solar_Install'"
    funnel_category: "bottom_funnel"
  - name: "META Solar Savings"
    match_rule: "contains 'Solar_Savings'"
    funnel_category: "mid_funnel"
  - name: "Google Brand Search"
    match_rule: "contains 'Brand'"
    funnel_category: "bottom_funnel"
  - name: "Google Non-Brand Search"
    match_rule: "contains 'NonBrand'"
    funnel_category: "bottom_funnel"

exclusion_rules:
  - "test"
  - "internal"
  - "employee"
```

---

## 7. Segmentation

```yaml
segmentation:
  enabled: true
  dimension: "region"
  segments:
    - label: "PHX"
      match_rule: "contains 'PHX'"
    - label: "TUC"
      match_rule: "contains 'TUC'"
    - label: "FLAG"
      match_rule: "contains 'FLAG'"

  views:
    combined: true
    per_segment: true
```

---

## 8. Data Quality Rules

```yaml
data_quality:
  exclude_patterns:
    - "test"
    - "sandbox"
    - "internal"

  required_fields_on_deals:
    - "dealstage"
    - "service_region"

  max_unknown_attribution_pct: 15
  max_lead_discrepancy_pct: 10
```

---

## What This Produces

With this config, the skill generates reports like:

**Combined View (All Regions)**

| Campaign Group | Spend | Leads | CPL | Appts | Appt Rate | CPA | Proposals | CPQA | Contracts | CPCU |
|---------------|-------|-------|-----|-------|-----------|-----|-----------|------|-----------|------|
| META Solar Install | $8,200 | 178 | $46 | 22 | 12.4% | $373 | 9 | $911 | 4 | $2,050 |
| META Solar Savings | $3,100 | 142 | $22 | 6 | 4.2% | $517 | 2 | $1,550 | 1 | $3,100 |
| Google Brand | $2,400 | 38 | $63 | 12 | 31.6% | $200 | 8 | $300 | 5 | $480 |
| Google Non-Brand | $4,800 | 74 | $65 | 9 | 12.2% | $533 | 3 | $1,600 | 1 | $4,800 |
| **Totals** | **$18,500** | **432** | **$43** | **49** | **11.3%** | **$378** | **22** | **$841** | **11** | **$1,682** |

**PHX Region View** — same structure, filtered to Phoenix campaigns only.

**Budget Rule Flags:**
- ⚠️ META Solar Savings: CPA ($517) exceeds appointment ceiling ($250) — monitor for one more week before pulling back
- ✅ Google Brand: Customer acquisition cost ($480) well below $1,200 target — candidate for spend increase
