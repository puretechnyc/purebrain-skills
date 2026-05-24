---
name: expert-council
description: Dynamic Expert Council QA system. Selects domain-relevant expert personas to review deliverables before final delivery. 3 tiers with build/review mode toggle. Works with any AI assistant.
version: 1.0.0
category: operations
author: PureBrain
---

# Dynamic Expert Council QA

Run your work through a simulated expert review panel before publishing. Select domain-relevant expert personas, get scored feedback, and identify improvements you would have missed.

## Modes

### BUILD MODE (default)
- Council is OFF
- Fast iteration, no review gates
- Creative freedom during design/development
- Council only activates when explicitly triggered

### REVIEW MODE (triggered)
- Activated by: "run it through the council" or "ready for review" or "final review"
- Runs tiered QA on the deliverable
- Returns scored feedback + recommendations
- Deactivates after review completes (back to build mode)

## Tiers

### TIER 1 -- Quick Check (1 expert, ~30 seconds)
- **Trigger**: Internal docs, messages, quick updates
- **Expert**: Brand/product insider only
- **Output**: Brand voice check, accuracy flag, go/no-go

### TIER 2 -- Standard Review (2-3 experts, ~1-2 min)
- **Trigger**: Social posts, emails, blog drafts, content
- **Experts**: 2 domain-relevant + brand insider
- **Output**: Per-expert strengths/weaknesses/improvements, synthesis

### TIER 3 -- Full Council (5 experts + cross-review, ~3-5 min)
- **Trigger**: Homepage launches, pricing pages, investor materials, major campaigns
- **Experts**: 4 domain-relevant + brand insider
- **Process**: Independent review > cross-review > chairman synthesis
- **Output**: Full QA report, scored improvements, go/no-go with conditions

## Auto-Trigger Rules (optional)

These deliverable types auto-trigger the council BEFORE publishing/sending:
- Email campaigns to subscriber list > Tier 2
- Homepage deploy to production > Tier 3
- Social posts > Tier 1
- Client deliverables > Tier 2
- Investor materials > Tier 3
- Pricing/offer changes > Tier 3

Design iterations, drafts, internal docs > NEVER auto-trigger.

## Expert Selection

1. Identify the deliverable TYPE (email, website, copy, social, video, proposal, etc.)
2. Load your expert roster (see recommended panel below)
3. Filter experts by matching domain tags
4. Select top 2-4 based on tier level
5. Always include your brand/product insider

### Recommended Expert Panels

| Review Type | Recommended Panel | When to Use |
|-------------|-------------------|-------------|
| Landing Page Copy | Alex Hormozi, Russell Brunson, David Ogilvy | Any conversion page |
| Brand & Positioning | Seth Godin, April Dunford, Marty Neumeier | Messaging, category creation |
| Social Media | Gary Vaynerchuk, Ann Handley, Joe Pulizzi | Social posts, engagement |
| Sales Messaging | Chris Voss, Aaron Ross, Jill Konrath | Emails, objection handling |
| Email Marketing | Chase Dimond, Russell Brunson, Ann Handley | Sequences, newsletters |
| Product Strategy | Steve Jobs, Jeff Bezos, Peter Thiel | Product decisions, roadmap |
| Pricing & Offers | Alex Hormozi, Hermann Simon, Russell Brunson | Pricing pages, offers |
| SEO & Content | Neil Patel, Rand Fishkin, Eli Schwartz | SEO strategy, content marketing |

## Review Process (per expert)

Each expert persona receives:
- The deliverable content
- Their persona/framework description
- 3 questions to answer:
  1. What works well? (specific elements)
  2. What needs improvement? (specific issues with suggested fixes)
  3. Score 1-10: How ready is this for the target audience?

## Cross-Review (Tier 3 only)

After all experts submit:
- Each expert sees the others' reviews
- Each responds: "Strongest point from another reviewer?" and "What did we ALL miss?"
- Chairman synthesizes into final report

## Output Format

```
EXPERT COUNCIL REVIEW
Tier: [1/2/3]
Asset: [description]
Experts: [names]

[Expert 1 Name] (Score: X/10)
+ [strength]
- [improvement needed]
Suggestion: [specific fix]

[Expert 2 Name] (Score: X/10)
...

SYNTHESIS:
Average Score: X/10
Top 3 Improvements:
1. [most impactful fix]
2. [second fix]
3. [third fix]
Recommendation: [GO / GO WITH CHANGES / HOLD]
```

## How to Use with Claude Code

```
You: "Review this landing page copy through the expert council at Tier 2"

Claude: [Adopts Hormozi persona, reviews, scores]
        [Adopts Brunson persona, reviews, scores]
        [Synthesizes both reviews into actionable improvements]
```

---

Built by [PureBrain](https://purebrain.ai) -- AI-powered marketing operations.

Want the full suite of 183+ production-tested skills? [Start your trial ->](https://purebrain.ai/pricing)
