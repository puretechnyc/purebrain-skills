---
name: parallel-research
description: Multi-agent parallel research methodology. 3-5 agents investigate different perspectives of a topic simultaneously for comprehensive coverage with minimal overlap. 4x speedup vs sequential research.
version: 1.0.0
category: operations
author: PureBrain
---

# Parallel Research Skill

A multi-perspective research methodology where 3-5 agents (or research threads) simultaneously investigate different angles of a topic. By dividing perspectives (not just workload), you achieve comprehensive coverage with minimal overlap and diverse insights.

## When to Use

**Invoke when**:
- Complex, multi-perspective questions need answering
- Comprehensive topic coverage required quickly
- Limited time for deep research (parallel > sequential)
- Need diverse viewpoints on same topic
- Research question has multiple valid angles

**Do not use when**:
- Single clear domain (a single specialist is more efficient)
- Need sequential depth on one angle
- Research requires dependent discoveries (each step needs prior)
- Simple factual lookup

## Prerequisites

**Roles Needed**:
- **External researcher** -- Industry context, market data, published sources
- **Pattern analyst** -- Structural/architectural patterns
- **Historical researcher** -- Prior art, evolution, context
- **Risk analyst** -- Security, risk, vulnerability angle
- **Synthesizer** -- Consolidates all perspectives

**Context Needed**:
- Clear research question
- Angle assignments (who researches what aspect)
- Synthesis criteria (what makes a good combined answer)

## Procedure

### Step 1: Angle Assignment (~5 minutes)

Divide the research into non-overlapping angles:

1. Decompose question into 3-5 distinct perspectives
2. Assign each perspective to an appropriate researcher
3. Ensure perspectives are complementary, not redundant
4. Brief each researcher on their specific angle

**Example Angles**:
- Technical implementation
- Industry precedent
- Security implications
- User experience impact
- Historical context

**Deliverable**: Angle assignment matrix

---

### Step 2: Parallel Investigation (~30-60 minutes, all run simultaneously)

All researchers work in parallel:

1. Each focuses ONLY on their assigned angle
2. Gather evidence, examples, insights
3. Note confidence levels
4. Flag cross-cutting discoveries for synthesis
5. Produce angle-specific report

**Deliverable**: 3-5 specialist research reports

---

### Step 3: Synthesis (~15-20 minutes)

Consolidate perspectives:

1. Read all angle-specific reports
2. Identify cross-perspective themes
3. Note contradictions or tensions
4. Create unified answer that honors all angles
5. Produce confidence-weighted synthesis

**Deliverable**: Synthesized research findings

---

### Step 4: Quality Check (~5 minutes)

Verify coverage and quality:

1. Calculate overlap percentage (target: <15%)
2. Assess coverage completeness
3. Rate overall quality
4. Document any gaps

**Deliverable**: Quality assessment

---

## Parallelization

**Can run in parallel**:
- Step 2 (the heart of this method) -- All research happens simultaneously
- This is where the 4x speedup comes from

**Must be sequential**:
- Step 1 before 2 (assignments needed)
- Step 2 before 3 (reports needed for synthesis)
- Step 3 before 4 (synthesis needed for quality check)

## Success Indicators

- [ ] 3-5 specialist research reports completed
- [ ] <15% overlap between reports (truly different perspectives)
- [ ] Synthesis addresses all angles coherently
- [ ] Quality rating 8.5+ across all reports
- [ ] Comprehensive coverage achieved (no major gaps)
- [ ] 3-4x faster than sequential research would have been

## Example

**Scenario**: Research "AI agent coordination patterns" across perspectives

```
Step 1 (Assign):
  - External researcher: Industry examples and academic papers
  - Pattern analyst: Structural patterns and architectures
  - Historical researcher: Historical approaches and evolution
  - Risk analyst: Security implications of multi-agent systems

Step 2 (Research) - all parallel:
  External: Found 7 industry implementations, 3 papers
  Pattern: Identified 4 core patterns (hub, mesh, hierarchy, swarm)
  Historical: Traced 15-year evolution from simple scripts
  Risk: Flagged 3 attack vectors, 2 mitigations

Step 3 (Synthesize):
  Unified finding: "Effective coordination requires..."
  Cross-theme: All reports emphasized trust/verification
  Tension: Hierarchy (efficient) vs. Mesh (resilient)

Step 4 (Quality):
  Overlap: 8% (excellent - truly different perspectives)
  Coverage: Comprehensive
  Quality: 9.2/10 average
  Speed: 4x faster than sequential

Result: Comprehensive answer from 4 perspectives
```

## Key Insights

- **Divide PERSPECTIVES, not just workload** -- this is what makes parallel research better than simply splitting work
- **The "<15% overlap" metric** validates truly different angles
- **Error handling**: If one angle fails, others still provide value
- **Scale**: Can extend to 6-8 researchers for very complex topics
- **When NOT to use**: Single-domain questions where a single specialist is more efficient

---

Built by [PureBrain](https://purebrain.ai) -- AI-powered marketing operations.

Want the full suite of 183+ production-tested skills? [Start your trial ->](https://purebrain.ai/pricing)
