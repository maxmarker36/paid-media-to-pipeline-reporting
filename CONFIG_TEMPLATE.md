# Configuration Template — Paid Media to Pipeline Reporting

Fill out this template to configure the skill for your business. The skill will walk you through this interactively on first use, but you can also pre-fill it here.

---

## 1. Company Context

```yaml
company_name: ""
industry: ""                        # e.g., Solar, Home Services, Insurance, SaaS, Real Estate
business_model: ""                  # e.g., "Lead gen for sales team", "Direct to consumer with sales follow-up"
reporting_audience: ""              # e.g., "VP of Marketing", "CEO", "Client (agency model)"
technical_detail_level: "business"  # Options: business (minimal jargon) | marketing (standard marketing terms) | technical (full detail)
```

---

## 2. Funnel Stages

Define your pipeline stages from first touch to closed customer. Minimum 2 stages, maximum 6.

```yaml
funnel_stages:
  - label: "Lead"                           # What you call Stage 1
    definition: "Form fill or inbound inquiry"  # How you define this stage
    crm_stage_ids: []                        # CRM stage IDs or names that map here
  - label: "Appointment"                     # What you call Stage 2
    definition: "Scheduled demo or consultation"
    crm_stage_ids: []
  - label: "Qualified Appointment"           # What you call Stage 3
    definition: "Met with prospect, confirmed fit"
    crm_stage_ids: []
  - label: "Customer"                        # What you call Stage 4
    definition: "Signed contract or completed purchase"
    crm_stage_ids: []
```

---

## 3. Cost Benchmarks

For each funnel stage, set a target (ideal) and ceiling (maximum acceptable).

```yaml
benchmarks:
  cost_per_lead:
    target: null          # e.g., 41
    ceiling: null         # e.g., 43
  cost_per_appointment:
    target: null          # e.g., 250
    ceiling: null         # e.g., 400
  cost_per_qualified_appointment:
    target: null          # e.g., 650
    ceiling: null         # e.g., 1000
  cost_per_customer:
    target: null          # e.g., 1500
    ceiling: null         # e.g., 2500
```

---

## 4. Budget Decision Rules

Define when to scale spend up or pull back. The default uses a 2-consecutive-period trigger.

```yaml
budget_rules:
  primary_metric: "cost_per_lead"
  consecutive_periods_required: 2         # How many periods before triggering action
  reporting_period: "weekly"              # Options: daily | weekly | biweekly | monthly

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
      periods: 1                          # Immediate action on ceiling breach

  # Override rules: downstream metrics can override CPL signal
  override_increase:
    - metric: "cost_per_appointment"
      condition: "below_target"
      periods: 2
      note: "May increase spend even if CPL is above target"

  override_decrease:
    - metric: "cost_per_appointment"
      condition: "above_ceiling"
      periods: 2
      note: "Pull back spend even if CPL is below target"
```

---

## 5. Data Sources

### Ad Platform Data

```yaml
ad_platforms:
  - platform: ""                    # e.g., META, Google Ads, LinkedIn, TikTok
    data_source: ""                 # Options: windsor_ai | csv_upload | manual_input | other_mcp
    # If windsor_ai:
    windsor_connector: ""           # e.g., "facebook", "google_ads"
    windsor_account_id: ""          # Your account ID
    lead_conversion_field: ""       # e.g., "actions_offsite_conversion_fb_pixel_lead"
    # If csv_upload:
    csv_column_mapping:             # Map your CSV columns to standard fields
      date: ""
      campaign_name: ""
      spend: ""
      leads: ""
      clicks: ""
      impressions: ""
```

### CRM Pipeline Data

```yaml
crm:
  platform: ""                      # Options: hubspot | salesforce | close | pipedrive | other
  pipeline_id: ""                   # Your pipeline ID or name

  # Attribution: how you track which ad platform drove the lead
  attribution_field: ""             # e.g., "paid_attribution" on contact object
  attribution_object: ""            # Where the field lives: "contact" | "deal" | "custom_object"
  attribution_values:               # Map your attribution values to platform names
    meta: ""                        # e.g., "META", "Facebook", "fb"
    google: ""                      # e.g., "Google", "Google Ads", "adwords"
    # Add more as needed

  # Segmentation field (optional): how you segment deals geographically or by product
  segment_field: ""                 # e.g., "geo_market", "product_line", "region"
  segment_object: ""                # Where the field lives: "contact" | "deal"
  segment_values: {}                # Map raw values to display labels, e.g., {"Northern California": "NORCAL"}
```

---

## 6. Campaign Taxonomy

Define how your campaigns are grouped and classified.

```yaml
campaign_groups:
  - name: ""                        # Display name, e.g., "META Brand Search"
    match_rule: ""                  # How to identify in campaign names, e.g., "contains 'Brand'"
    funnel_category: ""             # Options: bottom_funnel | mid_funnel | top_funnel
  - name: ""
    match_rule: ""
    funnel_category: ""

# Campaign name strings to ALWAYS exclude from reporting
exclusion_rules:
  - ""                              # e.g., "test", "internal", "sandbox"
```

---

## 7. Segmentation (Optional)

```yaml
segmentation:
  enabled: false                    # Set to true if you segment campaigns
  dimension: ""                     # e.g., "region", "product_line", "brand_vs_nonbrand"
  segments:
    - label: ""                     # Display label, e.g., "West Coast"
      match_rule: ""                # How to identify in campaign names
    - label: ""
      match_rule: ""

  # Reporting views to generate
  views:
    combined: true                  # Always generate a rolled-up view
    per_segment: true               # Generate one view per segment
```

---

## 8. Data Quality Rules (Optional)

```yaml
data_quality:
  # Test/junk record patterns to exclude
  exclude_patterns:
    - ""                            # e.g., "test", "sandbox", "demo"

  # Field completeness checks
  required_fields_on_deals:
    - ""                            # e.g., "dealstage", "attribution"

  # Attribution quality threshold
  max_unknown_attribution_pct: 20   # Flag if Unknown attribution exceeds this %

  # Lead count discrepancy threshold
  max_lead_discrepancy_pct: 10      # Flag if ad platform vs. CRM lead count differs by more than this %
```

---

## Notes

- This config is read by the skill during setup. You can also answer these questions interactively when the skill first runs.
- Fields marked `null` or `""` will be prompted during the interactive setup.
- You can update this config anytime by re-running the setup flow or editing this file directly.
