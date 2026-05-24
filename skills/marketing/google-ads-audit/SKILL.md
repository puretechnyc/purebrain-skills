---
name: google-ads-audit
description: Weekly Google Ads campaign health check. Pull data via API, analyze performance, generate actionable optimization recommendations with copy-paste fixes.
version: 1.0.0
category: marketing
author: PureBrain
---

# Google Ads Audit & Optimization

## When to Use

- Weekly campaign health checks
- When reviewing a client's Google Ads performance
- After applying optimizations (verification pass)
- When onboarding a new Google Ads client

## Tools Required

- Google Ads API access (via `google-ads` Python client)
- Your Google Ads customer ID(s)

---

## PHASE 1: DATA PULL (What to collect)

### CRITICAL RULE: Cross-reference ALL levels

Settings exist at account, campaign, ad group, AND criterion levels. NEVER report something as "missing" without checking all levels. Present findings as "API shows X at [level] -- verify in UI" when uncertain.

### 1.1 Campaign Settings (ALL campaigns)

```sql
SELECT campaign.name, campaign.status, campaign.advertising_channel_type,
    campaign.bidding_strategy_type,
    campaign.maximize_conversions.target_cpa_micros,
    campaign.maximize_conversion_value.target_roas,
    campaign.geo_target_type_setting.positive_geo_target_type
FROM campaign ORDER BY campaign.status, campaign.name
```

### 1.2 Campaign Performance (30 days)

```sql
SELECT campaign.name, campaign.status,
    metrics.impressions, metrics.clicks, metrics.cost_micros,
    metrics.conversions, metrics.conversions_value
FROM campaign
WHERE campaign.status = 'ENABLED'
    AND segments.date DURING LAST_30_DAYS
ORDER BY metrics.cost_micros DESC
```

### 1.3 Keywords with Quality Scores

```sql
SELECT campaign.name, ad_group.name,
    ad_group_criterion.keyword.text,
    ad_group_criterion.keyword.match_type,
    ad_group_criterion.quality_info.quality_score,
    ad_group_criterion.status, ad_group_criterion.negative,
    metrics.impressions, metrics.clicks, metrics.cost_micros,
    metrics.conversions
FROM keyword_view
WHERE campaign.status = 'ENABLED'
    AND segments.date DURING LAST_30_DAYS
ORDER BY metrics.cost_micros DESC
```

### 1.4 Demographics (Gender, Age, Income)

Run THREE separate queries for gender_view, age_range_view, and income_range_view. Pull bid_modifier, negative status, and performance metrics (impressions, clicks, cost, conversions) for each.

### Demographic ID Reference

- Age: 503001=18-24, 503002=25-34, 503003=35-44, 503004=45-54, 503005=55-64, 503006=65+, 503999=Unknown
- Gender: 10=Male, 11=Female, 20=Unknown
- Income: 510000=Top 10%, 510001=11-20%, 510002=21-30%, 510003=31-40%, 510004=41-50%, 510005=Lower 50%, 510006=Unknown

### 1.5 Search Terms (top 50 by spend)

```sql
SELECT campaign.name, search_term_view.search_term,
    metrics.impressions, metrics.clicks, metrics.cost_micros, metrics.conversions
FROM search_term_view
WHERE segments.date DURING LAST_30_DAYS AND campaign.status = 'ENABLED'
ORDER BY metrics.cost_micros DESC LIMIT 50
```

### 1.6 Conversion Actions

```sql
SELECT conversion_action.name, conversion_action.type, conversion_action.status,
    conversion_action.category, conversion_action.counting_type
FROM conversion_action WHERE conversion_action.status = 'ENABLED'
```

### 1.7 Ad Copy

```sql
SELECT campaign.name, ad_group.name,
    ad_group_ad.ad.responsive_search_ad.headlines,
    ad_group_ad.ad.responsive_search_ad.descriptions,
    ad_group_ad.ad.final_urls, ad_group_ad.ad_strength,
    metrics.impressions, metrics.clicks, metrics.conversions
FROM ad_group_ad
WHERE campaign.status = 'ENABLED' AND ad_group_ad.status = 'ENABLED'
    AND segments.date DURING LAST_30_DAYS
```

---

## PHASE 2: ANALYSIS (Decision Framework)

### 2.1 Campaign Triage

Classify each campaign by CPA performance:
- **GREEN**: CPA below client target or below account average
- **YELLOW**: CPA 1-2x account average
- **RED**: CPA >2x account average or <3 conversions in 30 days

### 2.2 Keyword Triage

For each keyword with spend:
- **SCALE** (green): Converting with CPA below campaign average -- increase bids or add similar keywords
- **MONITOR** (yellow): Converting but CPA above average -- watch for 2 more weeks
- **PAUSE** (red): >$50 spend with 0 conversions -- pause immediately
- **QS FIX** (orange): QS 1-3 with any spend -- fix ad relevance or pause

### 2.3 Demographic Analysis

For each demographic segment:
- Calculate CPA per segment
- Compare to account average
- Segments >2x average CPA with <3 conversions -- recommend exclusion
- Segments >1.5x average -- recommend negative bid modifier (-20% to -30%)
- Segments <0.5x average -- recommend positive bid modifier (+15% to +25%)
- ALWAYS note: "verify current exclusions in UI before recommending"

### 2.4 Search Term Analysis

For each search term with spend:
- Converting: keep, consider adding as exact match keyword
- >$20 spend, 0 conversions: add as negative keyword
- Wrong intent (parts, other brands, wrong geography): add as negative
- Foreign language queries: add as negative

### 2.5 Budget Analysis

For each campaign:
- Calculate: actual spend / (daily budget x 30) = utilization %
- <30% utilization: campaign needs more keywords or broader targeting
- >100% utilization: campaign is budget-constrained, consider increasing
- Reallocate budget from RED campaigns to GREEN campaigns

---

## PHASE 3: RECOMMENDATIONS (Output Format)

### Always provide in this format:

**STEP-BY-STEP FIXES** (numbered, specific, copy-pasteable):
1. Campaign name > Setting > exact value to set
2. Keyword name > action (pause/add/modify)
3. Demographic > exclusion or bid modifier with exact percentage

**KEYWORD LISTS** (copy-paste ready):
- Keywords to pause (with spend and conversion data)
- Keywords to add (modeled from best performers)
- Negative keywords to add

**BUDGET REALLOCATION TABLE:**
| From | To | Amount | Rationale |
|------|----|--------|-----------|

### Rules for recommendations:
1. NEVER recommend something that's already applied (check all levels first)
2. Present demographic findings with actual performance data, not just settings
3. Include conversion value in all CPA calculations when values are assigned
4. Always verify match types before recommending "broad match" fixes
5. Note which campaigns use audience exclusions vs bid modifiers -- they're different strategies

---

## PHASE 4: VERIFICATION (After Changes Applied)

1. Wait 5 minutes for Google Ads to propagate changes
2. Re-pull campaign settings, keyword status, and demographic settings
3. Compare before/after
4. Format as verification table:

| Step | Expected | API Shows | Status |
|------|----------|-----------|--------|
| Campaign budget | $10/day | $X/day | pass/fail |

5. Report any discrepancies

---

## COMMON MISTAKES TO AVOID

1. **Don't report ad-group-level settings as "missing" when checking campaign level**
2. **Don't assume broad match from search term data** -- phrase match also triggers broad queries
3. **Don't recommend demographic changes without performance data** -- exclusions without data are guesses
4. **Don't use stale audit files** -- always pull fresh API data
5. **Always decode demographic IDs** (503001=18-24, not raw numbers)

---

Built by [PureBrain](https://purebrain.ai) -- AI-powered marketing operations.

Want the full suite of 183+ production-tested skills? [Start your trial ->](https://purebrain.ai/pricing)
