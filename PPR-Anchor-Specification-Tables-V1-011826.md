# PPR ANCHOR SPECIFICATION TABLES - BATCH 1
## Matching Logic for Personal Pattern Recognition

**Date Created:** January 18, 2026  
**Purpose:** Define anchor conditions for PPR pattern matching across Q10A, Q10B, Q11, Q12  
**For:** Kismet (Platform Innovation Leader) - Implementation Reference  
**Status:** Ready for Implementation

---

## OVERVIEW

### The Problem

Each question has 8 pre-written PPR patterns, but users can make 72-540 unique selection combinations depending on question structure. We need matching logic to assign the closest PPR pattern when exact matches don't exist.

### The Solution: Anchor-Based Matching

Each pattern has:
- **Primary Anchor:** Must-match condition(s) - pattern only shows if these match
- **Secondary Criteria:** Nice-to-have matches - used for tiebreakers
- **Fallback Role:** Which unmatched combinations this pattern catches

### Matching Priority Order

1. **CP1 (Position)** - Most defines user's fundamental stance
2. **CP3 (Change Factor)** - Most defines what would move them
3. **CP2 (Concerns)** - Most variable, used for tiebreakers

### Implementation Logic

```python
def get_ppr_pattern(cp1, cp2_list, cp3, question):
    patterns = QUESTION_PATTERNS[question]
    
    # Step 1: Try exact CP1 + CP3 anchor match
    for pattern in patterns:
        if matches_anchor(cp1, cp3, pattern):
            return pattern
    
    # Step 2: Try CP1-only match with best CP2 fit
    cp1_matches = [p for p in patterns if p.cp1_anchor == cp1]
    if cp1_matches:
        return best_cp2_fit(cp1_matches, cp2_list)
    
    # Step 3: Return default pattern for CP1 value
    return default_pattern_for_cp1(cp1, question)
```

---

## OPTION REFERENCE TABLES

### Q10A Options

| Checkpoint | Option | C1 Text |
|------------|--------|---------|
| CP1 | 1 | Life extension is very important regardless of function |
| CP1 | 2 | Staying alive somewhat important, depends on situation |
| CP1 | 3 | Avoid aggressive intervention if function has declined |
| CP2 | 4 | Worried doctors might undervalue my life with disability |
| CP2 | 5 | Uncertain what life with physical limitations is like |
| CP2 | 6 | Worried about becoming a burden to loved ones |
| CP2 | 7 | Have seen others struggle with physical limitations |
| CP3 | 8 | Meeting people with disabilities living meaningful lives |
| CP3 | 9 | Having my team's support to coordinate medical advocacy |
| CP3 | 10 | Learning more about interventions and their outcomes |
| CP3 | 11 | Understanding disability doesn't mean low quality of life |

**Selection Types:**
- CP1: Single-select (1 of 3)
- CP2: Multi-select (up to 2 of 4)
- CP3: Single-select (1 of 4)

**Total Unique Combinations:** 3 x 10 x 4 = **120**

---

### Q10B Options

| Checkpoint | Option | C1 Text |
|------------|--------|---------|
| CP1 | 1 | Life extension very important regardless of mental capacity |
| CP1 | 2 | Physical preservation less important than mental capacity |
| CP1 | 3 | Balance based on specific mental capacities retained |
| CP2 | 4 | Fear of being aware but unable to communicate needs |
| CP2 | 5 | Concern about being kept alive in late-stage dementia |
| CP2 | 6 | Fear of becoming unrecognizable to myself or loved ones |
| CP2 | 7 | Fear of existing without recognition or awareness |
| CP3 | 8 | Learning about quality of life for people with dementia |
| CP3 | 9 | Knowing my family wants me present even without recognition |
| CP3 | 10 | Understanding medical advances in dementia treatment |
| CP3 | 11 | Better understanding of value beyond cognitive ability |

**Selection Types:**
- CP1: Single-select (1 of 3)
- CP2: Multi-select (up to 2 of 4)
- CP3: Single-select (1 of 4)

**Total Unique Combinations:** 3 x 10 x 4 = **120**

---

### Q11 Options

| Checkpoint | Option | C1 Text |
|------------|--------|---------|
| CP1 | 1 | Very important - comfort is my highest priority |
| CP1 | 2 | Somewhat important - balanced with other priorities |
| CP1 | 3 | Not important - willing to accept pain for other goals |
| CP2 | 4 | Provider fears about hastening death limit pain relief |
| CP2 | 5 | My faith views pain and suffering differently |
| CP2 | 6 | Fear sedation means losing awareness |
| CP2 | 7 | Physical pain is only part of total suffering |
| CP2 | 8 | Aware that race/ethnicity affects pain treatment |
| CP2 | 9 | Cultural communication style affects pain assessment |
| CP3 | 10 | Learning opioids don't actually hasten death |
| CP3 | 11 | Understanding pain includes emotional and social suffering |
| CP3 | 12 | Observing good care in others |
| CP3 | 13 | Learning accepting treatment doesn't violate my values |
| CP3 | 14 | Providers trained in my cultural pain expressions |
| CP3 | 15 | Knowing how to advocate effectively for pain control |

**Selection Types:**
- CP1: Single-select (1 of 3)
- CP2: Multi-select (up to 2 of 6)
- CP3: Single-select (1 of 6)

**Total Unique Combinations:** 3 x 21 x 6 = **378**

---

### Q12 Options

| Checkpoint | Option | C1 Text |
|------------|--------|---------|
| CP1 | 1 | Very important - maintaining independence is my priority |
| CP1 | 2 | Somewhat important - depends on what help is needed |
| CP1 | 3 | Not important - willing to receive help from others |
| CP2 | 4 | Worried about burdening family with hours of care needed |
| CP2 | 5 | Fear of losing sense of self if I can't do things myself |
| CP2 | 6 | Concern about quality of help - bad care worse than no care |
| CP2 | 7 | My independence values conflict with family expectations |
| CP2 | 8 | Independence impossible due to finances, resources, access |
| CP2 | 9 | Worried medical team won't honor independence preferences |
| CP3 | 10 | Understanding interdependence as human reality, not failure |
| CP3 | 11 | Seeing someone keep dignity despite needing major help |
| CP3 | 12 | Recognizing barriers are structural, not personal failures |
| CP3 | 13 | Accessing culturally appropriate care that honors my values |
| CP3 | 14 | Learning refusing help burdens family more than accepting |
| CP3 | 15 | Finding resources that enable independence when I choose it |

**Selection Types:**
- CP1: Single-select (1 of 3)
- CP2: Multi-select (up to 2 of 6)
- CP3: Single-select (1 of 6)

**Total Unique Combinations:** 3 x 21 x 6 = **378**

---

## Q10A ANCHOR SPECIFICATION

### Pattern Summary

| # | Pattern Name | CP1 Anchor | CP3 Anchor | CP2 Tiebreaker |
|---|--------------|------------|------------|----------------|
| 1 | Informed Conditional Thinking | 2 | 10 | 4 preferred |
| 2 | Absolute Conviction + Relational Concern | 1 | 9 | 6 preferred |
| 3 | Function-Focused + Knowledge Gaps | 3 | 10 | 5 preferred |
| 4 | Trauma-Informed Reconsidering | 2 | 11 | 7 preferred |
| 5 | Determined Autonomy + Systemic Awareness | 1 | 8 | 4 preferred |
| 6 | Values-Driven + Relationship Concern | 3 | 9 | 6 preferred |
| 7 | Epistemic Humility | 2 | 11 | 5 preferred |
| 8 | Conviction + Information-Seeking | 1 | 10 | 7 preferred |

### Detailed Anchor Specifications

#### Pattern 1: Informed Conditional Thinking
**Anchor:** CP1 = 2 AND CP3 = 10  
**Secondary:** CP2 includes 4 (provider bias)  
**Fallback for:** All CP1=2, CP3=10 combinations regardless of CP2  
**Theme:** Protecting autonomy through knowledge

#### Pattern 2: Absolute Conviction + Relational Concern
**Anchor:** CP1 = 1 AND CP3 = 9  
**Secondary:** CP2 includes 6 (burden fear)  
**Fallback for:** All CP1=1, CP3=9 combinations regardless of CP2  
**Theme:** Seeking infrastructure to match values

#### Pattern 3: Function-Focused + Knowledge Gaps
**Anchor:** CP1 = 3 AND CP3 = 10  
**Secondary:** CP2 includes 5 (uncertainty)  
**Fallback for:** All CP1=3, CP3=10 combinations regardless of CP2  
**Theme:** Grounding fears in concrete reality

#### Pattern 4: Trauma-Informed Reconsidering
**Anchor:** CP1 = 2 AND CP3 = 11  
**Secondary:** CP2 includes 7 (witnessed struggle)  
**Fallback for:** All CP1=2, CP3=11 combinations regardless of CP2  
**Theme:** Distinguishing suffering from societal failure

#### Pattern 5: Determined Autonomy + Systemic Awareness
**Anchor:** CP1 = 1 AND CP3 = 8  
**Secondary:** CP2 includes 4 (provider bias)  
**Fallback for:** All CP1=1, CP3=8 combinations regardless of CP2  
**Theme:** Preparing for advocacy

#### Pattern 6: Values-Driven + Relationship Concern
**Anchor:** CP1 = 3 AND CP3 = 9  
**Secondary:** CP2 includes 6 (burden fear)  
**Fallback for:** All CP1=3, CP3=9 combinations regardless of CP2  
**Theme:** Alignment between individual and collective

#### Pattern 7: Epistemic Humility
**Anchor:** CP1 = 2 AND CP3 = 11  
**Secondary:** CP2 includes 5 (uncertainty)  
**Fallback for:** CP1=2, CP3=11 when CP2 doesn't include 7  
**Note:** Shares anchor with Pattern 4; use CP2 to differentiate  
**Theme:** Creating space for learning

#### Pattern 8: Conviction + Information-Seeking
**Anchor:** CP1 = 1 AND CP3 = 10  
**Secondary:** CP2 includes 7 (witnessed struggle)  
**Fallback for:** All CP1=1, CP3=10 combinations regardless of CP2  
**Theme:** Seeking reality check for firm values

### Q10A Matching Matrix

| CP1 | CP3 | Assigned Pattern | Notes |
|-----|-----|------------------|-------|
| 1 | 8 | Pattern 5 | Disability meeting |
| 1 | 9 | Pattern 2 | Team support |
| 1 | 10 | Pattern 8 | Medical education |
| 1 | 11 | Pattern 5* | Fallback to closest |
| 2 | 8 | Pattern 4* | Fallback to closest |
| 2 | 9 | Pattern 2* | Fallback to closest |
| 2 | 10 | Pattern 1 | Education need |
| 2 | 11 | Pattern 4 or 7 | Use CP2 to differentiate |
| 3 | 8 | Pattern 3* | Fallback to closest |
| 3 | 9 | Pattern 6 | Team support |
| 3 | 10 | Pattern 3 | Medical education |
| 3 | 11 | Pattern 6* | Fallback to closest |

*Fallback assignments when no exact anchor match exists

### Q10A CP2 Tiebreaker Logic

When CP1 + CP3 anchor matches multiple patterns (e.g., Pattern 4 vs 7):

```
IF CP2 includes 7 (witnessed struggle):
    return Pattern 4
ELSE IF CP2 includes 5 (uncertainty):
    return Pattern 7
ELSE:
    return Pattern 4 (default for CP1=2, CP3=11)
```

---

## Q10B ANCHOR SPECIFICATION

### Pattern Summary

| # | Pattern Name | CP1 Anchor | CP3 Anchor | CP2 Tiebreaker |
|---|--------------|------------|------------|----------------|
| 1 | Firm Conviction + Evidence-Seeking | 1 | 8 | 4 preferred |
| 2 | Individual vs Relational Values | 2 | 9 | 6 preferred |
| 3 | Threshold-Focused + Hope | 3 | 10 | 5 preferred |
| 4 | Philosophical Courage | 1 | 11 | 7 preferred |
| 5 | Cognitive to Relational Shift | 2 | 9 | 4 preferred |
| 6 | Epistemically Humble Planning | 3 | 8 | 4+6 preferred |
| 7 | Openness to Paradigm Shift | 2 | 11 | 7 preferred |
| 8 | Layered Hope | 1 | 10 | 6 preferred |

### Detailed Anchor Specifications

#### Pattern 1: Firm Conviction + Evidence-Seeking
**Anchor:** CP1 = 1 AND CP3 = 8  
**Secondary:** CP2 includes 4 (communication fear)  
**Fallback for:** All CP1=1, CP3=8 combinations  
**Theme:** Grounding absolute values in empirical reality

#### Pattern 2: Individual vs Relational Values
**Anchor:** CP1 = 2 AND CP3 = 9  
**Secondary:** CP2 includes 6 (identity fear)  
**Fallback for:** All CP1=2, CP3=9 combinations  
**Theme:** Weighing self-determination against relational meaning

#### Pattern 3: Threshold-Focused + Hope
**Anchor:** CP1 = 3 AND CP3 = 10  
**Secondary:** CP2 includes 5 (late-stage fear)  
**Fallback for:** All CP1=3, CP3=10 combinations  
**Theme:** Contingent planning with medical hope

#### Pattern 4: Philosophical Courage
**Anchor:** CP1 = 1 AND CP3 = 11  
**Secondary:** CP2 includes 7 (no awareness fear)  
**Fallback for:** All CP1=1, CP3=11 combinations  
**Theme:** Seeking deepest justifications

#### Pattern 5: Cognitive to Relational Shift
**Anchor:** CP1 = 2 AND CP3 = 9  
**Secondary:** CP2 includes 4 (communication fear)  
**Note:** Shares anchor with Pattern 2; use CP2 to differentiate  
**Theme:** Discovering values are about bonds

#### Pattern 6: Epistemically Humble Planning
**Anchor:** CP1 = 3 AND CP3 = 8  
**Secondary:** CP2 includes 4 AND 6 (multi-select)  
**Fallback for:** All CP1=3, CP3=8 combinations  
**Theme:** Acknowledging possible wrongness

#### Pattern 7: Openness to Paradigm Shift
**Anchor:** CP1 = 2 AND CP3 = 11  
**Secondary:** CP2 includes 7 (no awareness fear)  
**Fallback for:** All CP1=2, CP3=11 combinations  
**Theme:** Holding position while remaining curious

#### Pattern 8: Layered Hope
**Anchor:** CP1 = 1 AND CP3 = 10  
**Secondary:** CP2 includes 6 (identity fear)  
**Fallback for:** All CP1=1, CP3=10 combinations  
**Theme:** Accepting outcome while hoping for better

### Q10B Matching Matrix

| CP1 | CP3 | Assigned Pattern | Notes |
|-----|-----|------------------|-------|
| 1 | 8 | Pattern 1 | QOL research |
| 1 | 9 | Pattern 2* | Fallback to closest |
| 1 | 10 | Pattern 8 | Medical advances |
| 1 | 11 | Pattern 4 | Value beyond cognition |
| 2 | 8 | Pattern 1* | Fallback to closest |
| 2 | 9 | Pattern 2 or 5 | Use CP2 to differentiate |
| 2 | 10 | Pattern 3* | Fallback to closest |
| 2 | 11 | Pattern 7 | Value beyond cognition |
| 3 | 8 | Pattern 6 | QOL research |
| 3 | 9 | Pattern 2* | Fallback to closest |
| 3 | 10 | Pattern 3 | Medical advances |
| 3 | 11 | Pattern 7* | Fallback to closest |

### Q10B CP2 Tiebreaker Logic

When CP1=2, CP3=9 (Pattern 2 vs 5):

```
IF CP2 includes 4 (communication fear):
    return Pattern 5
ELSE IF CP2 includes 6 (identity fear):
    return Pattern 2
ELSE:
    return Pattern 2 (default)
```

---

## Q11 ANCHOR SPECIFICATION

### Pattern Summary

| # | Pattern Name | CP1 Anchor | CP3 Anchor | CP2 Tiebreaker |
|---|--------------|------------|------------|----------------|
| 1 | Clear Priority + Provider Barrier | 1 | 10 | 4 preferred |
| 2 | Comfort vs Consciousness Tension | 1 | 12 | 6 preferred |
| 3 | Balance + Spiritual Integration | 2 | 13 | 7 preferred |
| 4 | Values-Driven + Cultural Need | 3 | 14 | 5 preferred |
| 5 | Priority + Disparity Awareness | 1 | 15 | 8 preferred |
| 6 | Balance + Consciousness Fear | 2 | 10 | 6 preferred |
| 7 | Goals Over Comfort + Holistic View | 3 | 12 | 7 preferred |
| 8 | Priority + Communication Gap | 1 | 15 | 9 preferred |

### Detailed Anchor Specifications

#### Pattern 1: Clear Priority + Provider Barrier
**Anchor:** CP1 = 1 AND CP3 = 10  
**Secondary:** CP2 includes 4 (provider fear)  
**Fallback for:** All CP1=1, CP3=10 combinations  
**Theme:** Fighting for evidence-based care

#### Pattern 2: Comfort vs Consciousness Tension
**Anchor:** CP1 = 1 AND CP3 = 12  
**Secondary:** CP2 includes 6 (sedation fear)  
**Fallback for:** All CP1=1, CP3=12 combinations  
**Theme:** Needing models, not just statistics

#### Pattern 3: Balance + Spiritual Integration
**Anchor:** CP1 = 2 AND CP3 = 13  
**Secondary:** CP2 includes 7 (total pain)  
**Fallback for:** All CP1=2, CP3=13 combinations  
**Theme:** Bridging medical and spiritual care

#### Pattern 4: Values-Driven + Cultural Need
**Anchor:** CP1 = 3 AND CP3 = 14  
**Secondary:** CP2 includes 5 (faith perspective)  
**Fallback for:** All CP1=3, CP3=14 combinations  
**Theme:** Chosen endurance vs invisible suffering

#### Pattern 5: Priority + Disparity Awareness
**Anchor:** CP1 = 1 AND CP3 = 15  
**Secondary:** CP2 includes 8 (racial disparity)  
**Note:** Shares CP3 anchor with Pattern 8; use CP2 to differentiate  
**Theme:** Navigating failing systems

#### Pattern 6: Balance + Consciousness Fear
**Anchor:** CP1 = 2 AND CP3 = 10  
**Secondary:** CP2 includes 6 (sedation fear)  
**Fallback for:** All CP1=2, CP3=10 combinations  
**Theme:** Dying present, not just dying later

#### Pattern 7: Goals Over Comfort + Holistic View
**Anchor:** CP1 = 3 AND CP3 = 12  
**Secondary:** CP2 includes 7 (total pain)  
**Fallback for:** All CP1=3, CP3=12 combinations  
**Theme:** Defining what care means for you

#### Pattern 8: Priority + Communication Gap
**Anchor:** CP1 = 1 AND CP3 = 15  
**Secondary:** CP2 includes 9 (cultural expression)  
**Note:** Shares CP3 anchor with Pattern 5; use CP2 to differentiate  
**Theme:** Being seen, not performing

### Q11 Matching Matrix

| CP1 | CP3 | Assigned Pattern | Notes |
|-----|-----|------------------|-------|
| 1 | 10 | Pattern 1 | Opioid evidence |
| 1 | 11 | Pattern 1* | Fallback |
| 1 | 12 | Pattern 2 | Witnessing |
| 1 | 13 | Pattern 3* | Fallback |
| 1 | 14 | Pattern 4* | Fallback |
| 1 | 15 | Pattern 5 or 8 | Use CP2 to differentiate |
| 2 | 10 | Pattern 6 | Opioid evidence |
| 2 | 11 | Pattern 6* | Fallback |
| 2 | 12 | Pattern 2* | Fallback |
| 2 | 13 | Pattern 3 | Spiritual permission |
| 2 | 14 | Pattern 4* | Fallback |
| 2 | 15 | Pattern 5* | Fallback |
| 3 | 10 | Pattern 6* | Fallback |
| 3 | 11 | Pattern 7* | Fallback |
| 3 | 12 | Pattern 7 | Witnessing |
| 3 | 13 | Pattern 3* | Fallback |
| 3 | 14 | Pattern 4 | Cultural training |
| 3 | 15 | Pattern 5* | Fallback |

### Q11 CP2 Tiebreaker Logic

When CP1=1, CP3=15 (Pattern 5 vs 8):

```
IF CP2 includes 8 (racial disparity):
    return Pattern 5
ELSE IF CP2 includes 9 (cultural expression):
    return Pattern 8
ELSE:
    return Pattern 5 (default for advocacy tools)
```

---

## Q12 ANCHOR SPECIFICATION

### Pattern Summary

| # | Pattern Name | CP1 Anchor | CP3 Anchor | CP2 Tiebreaker |
|---|--------------|------------|------------|----------------|
| 1 | Independence Maximalist | 1 | 15 | 5 preferred |
| 2 | Family Protector | 1 OR 2 | 14 | 4 preferred |
| 3 | Structural Realist | 3 | 12 | 8 preferred |
| 4 | Quality Focused | 2 OR 3 | 11 | 6 preferred |
| 5 | Cultural Navigator | ANY | 13 | 7 required |
| 6 | Voice Protector | 1 | 10 OR 12 | 9 preferred |
| 7 | Interdependence Seeker | 3 | 10 | 4 OR 5 preferred |
| 8 | Open Explorer | 2 | ANY | ANY moderate |

### Detailed Anchor Specifications

#### Pattern 1: Independence Maximalist
**Anchor:** CP1 = 1 AND CP3 = 15  
**Secondary:** CP2 includes 5 (identity loss)  
**Fallback for:** All CP1=1, CP3=15 combinations  
**Theme:** Coherent maximalist values

#### Pattern 2: Family Protector
**Anchor:** (CP1 = 1 OR CP1 = 2) AND CP3 = 14  
**Secondary:** CP2 includes 4 (burden fear)  
**Fallback for:** All CP3=14 combinations except CP1=3  
**Theme:** Care ethics in action

#### Pattern 3: Structural Realist
**Anchor:** CP1 = 3 AND CP3 = 12  
**Secondary:** CP2 includes 8 (structural barriers)  
**Fallback for:** All CP1=3, CP3=12 combinations  
**Theme:** System failure, not personal failure

#### Pattern 4: Quality Focused
**Anchor:** (CP1 = 2 OR CP1 = 3) AND CP3 = 11  
**Secondary:** CP2 includes 6 (quality concern)  
**Fallback for:** All CP3=11 combinations except CP1=1  
**Theme:** Bad care is the fear, not dependence

#### Pattern 5: Cultural Navigator
**Anchor:** CP3 = 13 AND CP2 includes 7  
**Secondary:** CP1 = ANY  
**Note:** CP2 is part of anchor, not just tiebreaker  
**Fallback for:** All CP3=13 with CP2=7 combinations  
**Theme:** Living between worlds

#### Pattern 6: Voice Protector
**Anchor:** CP1 = 1 AND (CP3 = 10 OR CP3 = 12)  
**Secondary:** CP2 includes 9 (preferences not honored)  
**Fallback for:** CP1=1 with CP3=10 or CP3=12  
**Theme:** Protecting future self

#### Pattern 7: Interdependence Seeker
**Anchor:** CP1 = 3 AND CP3 = 10  
**Secondary:** CP2 includes 4 OR 5 (burden or identity)  
**Fallback for:** All CP1=3, CP3=10 combinations  
**Theme:** Movement toward interdependence philosophy

#### Pattern 8: Open Explorer
**Anchor:** CP1 = 2  
**Secondary:** Any CP2, Any CP3  
**Note:** Catch-all for moderate position  
**Fallback for:** All CP1=2 combinations without stronger matches  
**Theme:** Appropriate humility about uncertain future

### Q12 Matching Matrix

| CP1 | CP3 | Assigned Pattern | Notes |
|-----|-----|------------------|-------|
| 1 | 10 | Pattern 6 | Interdependence + voice concern |
| 1 | 11 | Pattern 4* | Fallback |
| 1 | 12 | Pattern 6 | Structural + voice concern |
| 1 | 13 | Pattern 5 if CP2=7, else Pattern 1* | Cultural check |
| 1 | 14 | Pattern 2 | Family desire |
| 1 | 15 | Pattern 1 | Resources |
| 2 | 10 | Pattern 8 | Open explorer default |
| 2 | 11 | Pattern 4 | Quality focused |
| 2 | 12 | Pattern 8 | Open explorer default |
| 2 | 13 | Pattern 5 if CP2=7, else Pattern 8 | Cultural check |
| 2 | 14 | Pattern 2 | Family desire |
| 2 | 15 | Pattern 8 | Open explorer default |
| 3 | 10 | Pattern 7 | Interdependence seeker |
| 3 | 11 | Pattern 4 | Quality focused |
| 3 | 12 | Pattern 3 | Structural realist |
| 3 | 13 | Pattern 5 if CP2=7, else Pattern 3* | Cultural check |
| 3 | 14 | Pattern 2* | Fallback |
| 3 | 15 | Pattern 3* | Fallback |

### Q12 Special Logic

Pattern 5 (Cultural Navigator) has unique logic because CP2 is part of the anchor:

```
IF CP3 = 13 AND CP2 includes 7 (cultural conflict):
    return Pattern 5
ELSE:
    # Fall through to normal matching
```

Pattern 8 (Open Explorer) serves as catch-all for CP1=2:

```
IF CP1 = 2 AND no other pattern matched:
    return Pattern 8
```

---

## IMPLEMENTATION PSEUDOCODE

### Master Matching Function

```python
def get_ppr_pattern(question, cp1, cp2_list, cp3):
    """
    Returns the appropriate PPR pattern for user selections.
    
    Args:
        question: str - "Q10A", "Q10B", "Q11", or "Q12"
        cp1: int - Selected CP1 option number
        cp2_list: list[int] - Selected CP2 option number(s)
        cp3: int - Selected CP3 option number
    
    Returns:
        dict with pattern_id and pattern_text
    """
    
    # Load question-specific patterns
    patterns = PATTERN_CONFIG[question]
    
    # Step 1: Check for exact anchor matches
    for pattern in patterns:
        if matches_primary_anchor(pattern, cp1, cp2_list, cp3):
            return pattern
    
    # Step 2: Check for CP1+CP3 matches with CP2 tiebreaker
    candidates = [p for p in patterns 
                  if matches_cp1_cp3(p, cp1, cp3)]
    if candidates:
        return best_cp2_match(candidates, cp2_list)
    
    # Step 3: Check for CP1-only matches
    candidates = [p for p in patterns 
                  if p.cp1_anchor == cp1 or p.cp1_anchor == "ANY"]
    if candidates:
        return first_match(candidates)
    
    # Step 4: Return default pattern
    return get_default_pattern(question, cp1)


def matches_primary_anchor(pattern, cp1, cp2_list, cp3):
    """Check if all primary anchor conditions are met."""
    
    # Check CP1
    if pattern.cp1_anchor != "ANY":
        if isinstance(pattern.cp1_anchor, list):
            if cp1 not in pattern.cp1_anchor:
                return False
        elif cp1 != pattern.cp1_anchor:
            return False
    
    # Check CP3
    if pattern.cp3_anchor != "ANY":
        if isinstance(pattern.cp3_anchor, list):
            if cp3 not in pattern.cp3_anchor:
                return False
        elif cp3 != pattern.cp3_anchor:
            return False
    
    # Check CP2 if it's part of anchor (Pattern 5 in Q12)
    if pattern.cp2_required:
        if not any(opt in cp2_list for opt in pattern.cp2_required):
            return False
    
    return True


def best_cp2_match(candidates, cp2_list):
    """Among candidates, return pattern with best CP2 fit."""
    
    scored = []
    for pattern in candidates:
        score = 0
        for preferred in pattern.cp2_preferred:
            if preferred in cp2_list:
                score += 1
        scored.append((score, pattern))
    
    scored.sort(key=lambda x: x[0], reverse=True)
    return scored[0][1]
```

---

## TESTING CHECKLIST

### For Each Question, Verify:

- [ ] All exact anchor combinations return correct pattern
- [ ] CP2 tiebreaker logic differentiates correctly
- [ ] Fallback assignments make thematic sense
- [ ] No combination returns null/error
- [ ] Default patterns exist for each CP1 value

### Edge Cases to Test:

| Scenario | Expected Behavior |
|----------|-------------------|
| CP2 empty (single select error) | Return pattern based on CP1+CP3 only |
| CP2 contains all options | Return pattern with highest CP2 match count |
| Multiple patterns share exact anchor | Use CP2 tiebreaker consistently |
| User selects unusual combination | Return closest thematic match |

---

## VERSION HISTORY

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | January 18, 2026 | Initial specification for all Batch 1 questions |

---

**END OF PPR ANCHOR SPECIFICATION**

**Status:** Ready for Kismet Implementation  
**Coverage:** Q10A, Q10B, Q11, Q12 (all Batch 1)  
**Total Patterns:** 32 (8 per question)  
**Total Combinations Served:** 996 (120 + 120 + 378 + 378)
