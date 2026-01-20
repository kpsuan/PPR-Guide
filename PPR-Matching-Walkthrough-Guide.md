# PPR Anchor-Based Matching System - Walkthrough Guide

## Overview

The PPR (Personal Pattern Recognition) system matches user selections across 3 checkpoints to one of 8 predefined patterns per question. Each pattern provides personalized insight text based on the user's choices.

---

## How It Works

### The 3 Checkpoints

Each question has 3 checkpoints of choices:

| Checkpoint | Name | Selection Type | Purpose |
|------------|------|----------------|---------|
| **CP1** (Checkpoint 1) | Position | Single select | User's primary stance/priority |
| **CP2** (Checkpoint 2) | Concerns | Multi-select (2-3 options) | User's fears/concerns |
| **CP3** (Checkpoint 3) | Change Factor | Single select | What might change their mind |

### Matching Priority

The algorithm uses this priority order: **CP1 > CP3 > CP2**

1. **CP1 (Primary Anchor)**: First filter - narrows to patterns matching user's position
2. **CP3 (Secondary Anchor)**: Second filter - further narrows based on change factor
3. **CP2 (Tiebreaker)**: Only used when multiple patterns share CP1+CP3

---

## The Matching Algorithm

```
Step 1: Filter by CP1
        └── Find all patterns where user's CP1 is in cp1_anchors

Step 2: Filter by CP3
        └── From Step 1 results, find patterns where user's CP3 is in cp3_anchors

Step 3: Check cp2_required (if any)
        └── Filter out patterns where user doesn't have required CP2 selections

Step 4: If single match → Return it (match_type: "primary")

Step 5: If multiple matches → Apply CP2 tiebreaker
        └── Score each pattern by how many cp2_preferred options user selected
        └── Highest score wins
        └── If tie, use priority field (higher wins)
        └── Return winner (match_type: "tiebreaker")

Step 6: If no match → Check fallback patterns
        └── Find patterns with is_fallback=True AND user's CP1 in fallback_for_cp1
        └── Return fallback (match_type: "fallback")

Step 7: If still no match → Return no_match
```

---

## Q10A Example Walkthrough

### Option Mapping for Q10A

**Checkpoint 1 (CP1) - Position:**
| Option # | Text |
|----------|------|
| 1 | Very important - I want maximum intervention |
| 2 | Somewhat important - depends on situation |
| 3 | Avoid aggressive intervention |

**Checkpoint 2 (CP2) - Concerns:**
| Option # | Text |
|----------|------|
| 4 | Provider bias against disability |
| 5 | Uncertain about future quality of life |
| 6 | Fear of burdening family |
| 7 | Witnessed others struggle |

**Checkpoint 3 (CP3) - Change Factor:**
| Option # | Text |
|----------|------|
| 8 | Meeting someone with disability |
| 9 | Coordinated team support |
| 10 | Comprehensive medical education |
| 11 | Understanding disability paradox |

### The 8 Patterns for Q10A

| Pattern | Name | CP1 Anchor | CP3 Anchor | CP2 Preferred |
|---------|------|------------|------------|---------------|
| 1 | Conditional + Provider Bias + Education | [2] | [10] | [4] |
| 2 | Absolute + Burden Fear + Team Support | [1] | [9] | [6] |
| 3 | Function-Focused + Uncertainty + Education | [3] | [10] | [5] |
| 4 | Conditional + Witnessed + Disability Understanding | [2] | [11] | [7] |
| 5 | Absolute + Provider Bias + Disability Meeting | [1] | [8] | [4] |
| 6 | Function-Focused + Burden + Team Support | [3] | [9] | [6] |
| 7 | Conditional + Uncertainty + Disability Understanding | [2] | [11] | [5] |
| 8 | Absolute + Witnessed + Medical Education | [1] | [10] | [7] |

**Note:** Patterns 4 and 7 share the same CP1+CP3 anchors (2, 11). They use CP2 tiebreaker:
- Pattern 4 prefers [7] (witnessed struggle) - has priority=1
- Pattern 7 prefers [5] (uncertainty) - has priority=0

---

## Test Cases

### Test 1: Primary Match (Unique CP1+CP3)

**Selections:**
- Checkpoint 1: Option 2 (Somewhat important)
- Checkpoint 2: Options 4 and 6
- Checkpoint 3: Option 10 (Medical education)

**Algorithm:**
1. CP1=2 → Patterns 1, 4, 7 match
2. CP3=10 → Only Pattern 1 matches
3. Single match → Return Pattern 1

**Result:** Pattern 1 - "Conditional Position + Provider Bias + Education Need"
**Match Type:** primary

---

### Test 2: Tiebreaker (Shared CP1+CP3)

**Selections:**
- Checkpoint 1: Option 2 (Somewhat important)
- Checkpoint 2: Options 5 and 7
- Checkpoint 3: Option 11 (Disability understanding)

**Algorithm:**
1. CP1=2 → Patterns 1, 4, 7 match
2. CP3=11 → Patterns 4 and 7 match
3. Multiple matches → Apply CP2 tiebreaker
   - Pattern 4 prefers [7] → User has 7 → Score: 1
   - Pattern 7 prefers [5] → User has 5 → Score: 1
4. Tie! Check priority
   - Pattern 4 priority=1, Pattern 7 priority=0
5. Pattern 4 wins

**Result:** Pattern 4 - "Conditional Position + Witnessed Struggle + Disability Understanding"
**Match Type:** tiebreaker

---

### Test 3: CP2 Determines Winner

**Selections:**
- Checkpoint 1: Option 2
- Checkpoint 2: Option 7 only (witnessed struggle)
- Checkpoint 3: Option 11

**Algorithm:**
1. CP1=2, CP3=11 → Patterns 4 and 7 match
2. CP2 tiebreaker:
   - Pattern 4 prefers [7] → User has 7 → Score: 1
   - Pattern 7 prefers [5] → User doesn't have 5 → Score: 0
3. Pattern 4 wins by score

**Result:** Pattern 4
**Match Type:** tiebreaker

---

## Complete Test Suite

### How to Test in Frontend

1. Go to the question (Q10A, Q10B, Q11, or Q12)
2. Select options as specified in each test case
3. Complete all 3 checkpoints and reach the Summary page
4. Check the PPR text displayed matches the expected pattern

### How to Test via API (curl)

```bash
curl -X POST http://localhost:8000/api/v1/content/questions/{QUESTION_ID}/ppr/match/ \
  -H "Content-Type: application/json" \
  -d '{"cp1_selection": X, "cp2_selections": [X, Y], "cp3_selection": Z}'
```

---

## Q10A Test Cases

### Test Q10A-1: Primary Match (Unique Anchor)
| Checkpoint | Select | Option Text |
|------------|--------|-------------|
| CP1 | Option 2 | Somewhat important |
| CP2 | Options 4, 6 | Provider bias + Burden |
| CP3 | Option 10 | Medical education |

**Expected:** Pattern 1 - "Conditional Position + Provider Bias + Education Need"
**Match Type:** `primary`
**Why:** Only Pattern 1 has CP1=2 AND CP3=10

```bash
curl -X POST http://localhost:8000/api/v1/content/questions/Q10A/ppr/match/ \
  -H "Content-Type: application/json" \
  -d '{"cp1_selection": 2, "cp2_selections": [4, 6], "cp3_selection": 10}'
```

---

### Test Q10A-2: Tiebreaker - Both CP2 Match (Priority Wins)
| Checkpoint | Select | Option Text |
|------------|--------|-------------|
| CP1 | Option 2 | Somewhat important |
| CP2 | Options 5, 7 | Uncertainty + Witnessed |
| CP3 | Option 11 | Disability understanding |

**Expected:** Pattern 4 - "Conditional Position + Witnessed Struggle + Disability Understanding"
**Match Type:** `tiebreaker`
**Why:** Both Pattern 4 and 7 share CP1=2, CP3=11. User selected both preferred options [5,7]. Scores tie (1 each), so Pattern 4 wins by priority (1 > 0).

```bash
curl -X POST http://localhost:8000/api/v1/content/questions/Q10A/ppr/match/ \
  -H "Content-Type: application/json" \
  -d '{"cp1_selection": 2, "cp2_selections": [5, 7], "cp3_selection": 11}'
```

---

### Test Q10A-3: Tiebreaker - CP2 Score Determines Winner
| Checkpoint | Select | Option Text |
|------------|--------|-------------|
| CP1 | Option 2 | Somewhat important |
| CP2 | Option 5 only | Uncertainty |
| CP3 | Option 11 | Disability understanding |

**Expected:** Pattern 7 - "Conditional Position + Uncertainty + Disability Understanding"
**Match Type:** `tiebreaker`
**Why:** Pattern 7 prefers [5], Pattern 4 prefers [7]. User only has [5], so Pattern 7 scores 1, Pattern 4 scores 0.

```bash
curl -X POST http://localhost:8000/api/v1/content/questions/Q10A/ppr/match/ \
  -H "Content-Type: application/json" \
  -d '{"cp1_selection": 2, "cp2_selections": [5], "cp3_selection": 11}'
```

---

### Test Q10A-4: Tiebreaker - Other CP2 Wins
| Checkpoint | Select | Option Text |
|------------|--------|-------------|
| CP1 | Option 2 | Somewhat important |
| CP2 | Option 7 only | Witnessed struggle |
| CP3 | Option 11 | Disability understanding |

**Expected:** Pattern 4 - "Conditional Position + Witnessed Struggle + Disability Understanding"
**Match Type:** `tiebreaker`
**Why:** Pattern 4 prefers [7], user has [7]. Pattern 4 scores 1, Pattern 7 scores 0.

```bash
curl -X POST http://localhost:8000/api/v1/content/questions/Q10A/ppr/match/ \
  -H "Content-Type: application/json" \
  -d '{"cp1_selection": 2, "cp2_selections": [7], "cp3_selection": 11}'
```

---

### Test Q10A-5: CP1=1 Primary Match
| Checkpoint | Select | Option Text |
|------------|--------|-------------|
| CP1 | Option 1 | Very important |
| CP2 | Options 4, 5 | Provider bias + Uncertainty |
| CP3 | Option 9 | Team support |

**Expected:** Pattern 2 - "Absolute Position + Burden Fear + Team Support Need"
**Match Type:** `primary`
**Why:** Only Pattern 2 has CP1=1 AND CP3=9

```bash
curl -X POST http://localhost:8000/api/v1/content/questions/Q10A/ppr/match/ \
  -H "Content-Type: application/json" \
  -d '{"cp1_selection": 1, "cp2_selections": [4, 5], "cp3_selection": 9}'
```

---

### Test Q10A-6: CP1=3 Primary Match
| Checkpoint | Select | Option Text |
|------------|--------|-------------|
| CP1 | Option 3 | Avoid aggressive intervention |
| CP2 | Options 6, 7 | Burden + Witnessed |
| CP3 | Option 9 | Team support |

**Expected:** Pattern 6 - "Function-Focused + Burden Fear + Team Support"
**Match Type:** `primary`

```bash
curl -X POST http://localhost:8000/api/v1/content/questions/Q10A/ppr/match/ \
  -H "Content-Type: application/json" \
  -d '{"cp1_selection": 3, "cp2_selections": [6, 7], "cp3_selection": 9}'
```

---

### Test Q10A-7: All 8 Patterns Coverage

| Test | CP1 | CP2 | CP3 | Expected Pattern |
|------|-----|-----|-----|--------------------|
| 7a | 2 | [4] | 10 | Pattern 1 |
| 7b | 1 | [6] | 9 | Pattern 2 |
| 7c | 3 | [5] | 10 | Pattern 3 |
| 7d | 2 | [7] | 11 | Pattern 4 |
| 7e | 1 | [4] | 8 | Pattern 5 |
| 7f | 3 | [6] | 9 | Pattern 6 |
| 7g | 2 | [5] | 11 | Pattern 7 |
| 7h | 1 | [7] | 10 | Pattern 8 |

---

## Q10B Test Cases

### Test Q10B-1: Primary Match
| Checkpoint | Select | Option Text |
|------------|--------|-------------|
| CP1 | Option 1 | Absolute position |
| CP2 | Options 4, 5 | Communication + Late-stage fear |
| CP3 | Option 8 | QOL research |

**Expected:** Pattern 1 - "Absolute + Communication Fear + QOL Research"
**Match Type:** `primary`

```bash
curl -X POST http://localhost:8000/api/v1/content/questions/Q10B/ppr/match/ \
  -H "Content-Type: application/json" \
  -d '{"cp1_selection": 1, "cp2_selections": [4, 5], "cp3_selection": 8}'
```

---

### Test Q10B-2: Tiebreaker (Pattern 2 vs 5)
| Checkpoint | Select | Option Text |
|------------|--------|-------------|
| CP1 | Option 2 | Conditional |
| CP2 | Options 4, 6 | Communication + Identity fear |
| CP3 | Option 9 | Family presence |

**Expected:** Pattern 2 - "Conditional + Identity Fear + Family Presence"
**Match Type:** `tiebreaker`
**Why:** Both Pattern 2 and 5 share CP1=2, CP3=9. User has both [4,6]. Pattern 2 prefers [6] (score 1), Pattern 5 prefers [4] (score 1). Tie! Pattern 2 wins by priority (1 > 0).

```bash
curl -X POST http://localhost:8000/api/v1/content/questions/Q10B/ppr/match/ \
  -H "Content-Type: application/json" \
  -d '{"cp1_selection": 2, "cp2_selections": [4, 6], "cp3_selection": 9}'
```

---

### Test Q10B-3: Pattern 5 Wins by Score
| Checkpoint | Select | Option Text |
|------------|--------|-------------|
| CP1 | Option 2 | Conditional |
| CP2 | Option 4 only | Communication fear |
| CP3 | Option 9 | Family presence |

**Expected:** Pattern 5 - "Capacity Priority + Communication Fear + Family Presence"
**Match Type:** `tiebreaker`
**Why:** Pattern 5 prefers [4], user has [4]. Pattern 2 prefers [6], user doesn't have it. Pattern 5 scores 1, Pattern 2 scores 0.

```bash
curl -X POST http://localhost:8000/api/v1/content/questions/Q10B/ppr/match/ \
  -H "Content-Type: application/json" \
  -d '{"cp1_selection": 2, "cp2_selections": [4], "cp3_selection": 9}'
```

---

### Test Q10B-4: Multi-Preferred Pattern (Pattern 6)
| Checkpoint | Select | Option Text |
|------------|--------|-------------|
| CP1 | Option 3 | Balance |
| CP2 | Options 4, 6 | Communication + Identity fear |
| CP3 | Option 8 | QOL research |

**Expected:** Pattern 6 - "Balance + Multiple Fears + QOL Research"
**Match Type:** `primary`
**Why:** Pattern 6 has cp2_preferred=[4,6]. User has both, scoring 2.

```bash
curl -X POST http://localhost:8000/api/v1/content/questions/Q10B/ppr/match/ \
  -H "Content-Type: application/json" \
  -d '{"cp1_selection": 3, "cp2_selections": [4, 6], "cp3_selection": 8}'
```

---

## Q11 Test Cases

### Test Q11-1: Primary Match
| Checkpoint | Select | Option Text |
|------------|--------|-------------|
| CP1 | Option 1 | Comfort priority |
| CP2 | Options 4, 6 | Provider fear + Consciousness |
| CP3 | Option 10 | Evidence mind-changer |

**Expected:** Pattern 1 - "Comfort Priority + Provider Fear Concern + Evidence Mind-Changer"
**Match Type:** `primary`

```bash
curl -X POST http://localhost:8000/api/v1/content/questions/Q11/ppr/match/ \
  -H "Content-Type: application/json" \
  -d '{"cp1_selection": 1, "cp2_selections": [4, 6], "cp3_selection": 10}'
```

---

### Test Q11-2: Tiebreaker (Pattern 5 vs 8)
| Checkpoint | Select | Option Text |
|------------|--------|-------------|
| CP1 | Option 1 | Comfort priority |
| CP2 | Options 8, 9 | Disparity + Cultural expression |
| CP3 | Option 15 | Advocacy tools |

**Expected:** Pattern 5 - "Comfort Priority + Disparity Awareness + Advocacy Tools Mind-Changer"
**Match Type:** `tiebreaker`
**Why:** Both Pattern 5 and 8 share CP1=1, CP3=15. Both score 1. Pattern 5 wins by priority (1 > 0).

```bash
curl -X POST http://localhost:8000/api/v1/content/questions/Q11/ppr/match/ \
  -H "Content-Type: application/json" \
  -d '{"cp1_selection": 1, "cp2_selections": [8, 9], "cp3_selection": 15}'
```

---

### Test Q11-3: Pattern 8 Wins by Score
| Checkpoint | Select | Option Text |
|------------|--------|-------------|
| CP1 | Option 1 | Comfort priority |
| CP2 | Option 9 only | Cultural expression |
| CP3 | Option 15 | Advocacy tools |

**Expected:** Pattern 8 - "Comfort Priority + Cultural Expression Concern + Advocacy Tools Mind-Changer"
**Match Type:** `tiebreaker`
**Why:** Pattern 8 prefers [9], user has [9] → score 1. Pattern 5 prefers [8], user doesn't have → score 0.

```bash
curl -X POST http://localhost:8000/api/v1/content/questions/Q11/ppr/match/ \
  -H "Content-Type: application/json" \
  -d '{"cp1_selection": 1, "cp2_selections": [9], "cp3_selection": 15}'
```

---

## Q12 Test Cases

### Test Q12-1: Primary Match
| Checkpoint | Select | Option Text |
|------------|--------|-------------|
| CP1 | Option 2 | Somewhat important |
| CP2 | Options 5, 6 | Identity loss + Quality of care |
| CP3 | Option 11 | Witnessing dignity |

**Expected:** Pattern 4 - "Quality Focused"
**Match Type:** `primary`

```bash
curl -X POST http://localhost:8000/api/v1/content/questions/Q12/ppr/match/ \
  -H "Content-Type: application/json" \
  -d '{"cp1_selection": 2, "cp2_selections": [5, 6], "cp3_selection": 11}'
```

---

### Test Q12-2: Independence Maximalist
| Checkpoint | Select | Option Text |
|------------|--------|-------------|
| CP1 | Option 1 | Very important |
| CP2 | Option 5 | Identity loss |
| CP3 | Option 15 | Resource awareness |

**Expected:** Pattern 1 - "Independence Maximalist"
**Match Type:** `primary`

```bash
curl -X POST http://localhost:8000/api/v1/content/questions/Q12/ppr/match/ \
  -H "Content-Type: application/json" \
  -d '{"cp1_selection": 1, "cp2_selections": [5], "cp3_selection": 15}'
```

---

### Test Q12-3: Structural Realist
| Checkpoint | Select | Option Text |
|------------|--------|-------------|
| CP1 | Option 3 | Not important |
| CP2 | Option 8 | Structural barriers |
| CP3 | Option 12 | Structural understanding |

**Expected:** Pattern 3 - "Structural Realist"
**Match Type:** `primary`

```bash
curl -X POST http://localhost:8000/api/v1/content/questions/Q12/ppr/match/ \
  -H "Content-Type: application/json" \
  -d '{"cp1_selection": 3, "cp2_selections": [8], "cp3_selection": 12}'
```

---

## Edge Cases

### Edge Case 1: No CP2 Preferred Match (Tiebreaker defaults to priority)
| Checkpoint | Select | Option Text |
|------------|--------|-------------|
| CP1 | Option 2 | Somewhat important |
| CP2 | Option 6 only | Burden (not preferred by either 4 or 7) |
| CP3 | Option 11 | Disability understanding |

**Expected:** Pattern 4 (higher priority wins when no CP2 preference matches)
**Match Type:** `tiebreaker`

```bash
curl -X POST http://localhost:8000/api/v1/content/questions/Q10A/ppr/match/ \
  -H "Content-Type: application/json" \
  -d '{"cp1_selection": 2, "cp2_selections": [6], "cp3_selection": 11}'
```

---

### Edge Case 2: Multiple CP2 Selections (3 options)
| Checkpoint | Select | Option Text |
|------------|--------|-------------|
| CP1 | Option 2 | Somewhat important |
| CP2 | Options 4, 5, 7 | Provider bias + Uncertainty + Witnessed |
| CP3 | Option 11 | Disability understanding |

**Expected:** Pattern 4 (has 7) or Pattern 7 (has 5) - both score 1, Pattern 4 wins by priority
**Match Type:** `tiebreaker`

```bash
curl -X POST http://localhost:8000/api/v1/content/questions/Q10A/ppr/match/ \
  -H "Content-Type: application/json" \
  -d '{"cp1_selection": 2, "cp2_selections": [4, 5, 7], "cp3_selection": 11}'
```

---

### Edge Case 3: Single CP2 Selection
| Checkpoint | Select | Option Text |
|------------|--------|-------------|
| CP1 | Option 1 | Very important |
| CP2 | Option 4 only | Provider bias |
| CP3 | Option 8 | Disability meeting |

**Expected:** Pattern 5 - "Absolute Position + Provider Bias + Disability Meeting"
**Match Type:** `primary`

```bash
curl -X POST http://localhost:8000/api/v1/content/questions/Q10A/ppr/match/ \
  -H "Content-Type: application/json" \
  -d '{"cp1_selection": 1, "cp2_selections": [4], "cp3_selection": 8}'
```

---

## Summary: All Pattern Coverage

### Q10A Patterns
| Pattern | CP1 | CP3 | CP2 Preferred | Test Command |
|---------|-----|-----|---------------|--------------|
| 1 Informed Conditional Thinking | 2 | 10 | 4 | `{"cp1_selection": 2, "cp2_selections": [4], "cp3_selection": 10}` |
| 2 Absolute Conviction + Relational Concern | 1 | 9 | 6 | `{"cp1_selection": 1, "cp2_selections": [6], "cp3_selection": 9}` |
| 3 Function-Focused + Knowledge Gaps | 3 | 10 | 5 | `{"cp1_selection": 3, "cp2_selections": [5], "cp3_selection": 10}` |
| 4 Trauma-Informed Reconsidering | 2 | 11 | 7 | `{"cp1_selection": 2, "cp2_selections": [7], "cp3_selection": 11}` |
| 5 Determined Autonomy + Systemic Awareness | 1 | 8 | 4 | `{"cp1_selection": 1, "cp2_selections": [4], "cp3_selection": 8}` |
| 6 Values-Driven + Relationship Concern | 3 | 9 | 6 | `{"cp1_selection": 3, "cp2_selections": [6], "cp3_selection": 9}` |
| 7 Epistemic Humility | 2 | 11 | 5 | `{"cp1_selection": 2, "cp2_selections": [5], "cp3_selection": 11}` |
| 8 Conviction + Information-Seeking | 1 | 10 | 7 | `{"cp1_selection": 1, "cp2_selections": [7], "cp3_selection": 10}` |

### Q10B Patterns
| Pattern | CP1 | CP3 | CP2 Preferred | Test Command |
|---------|-----|-----|---------------|--------------|
| 1 Firm Conviction + Evidence-Seeking | 1 | 8 | 4 | `{"cp1_selection": 1, "cp2_selections": [4], "cp3_selection": 8}` |
| 2 Individual vs Relational Values | 2 | 9 | 6 | `{"cp1_selection": 2, "cp2_selections": [6], "cp3_selection": 9}` |
| 3 Threshold-Focused + Hope | 3 | 10 | 5 | `{"cp1_selection": 3, "cp2_selections": [5], "cp3_selection": 10}` |
| 4 Philosophical Courage | 1 | 11 | 7 | `{"cp1_selection": 1, "cp2_selections": [7], "cp3_selection": 11}` |
| 5 Cognitive to Relational Shift | 2 | 9 | 4 | `{"cp1_selection": 2, "cp2_selections": [4], "cp3_selection": 9}` |
| 6 Epistemically Humble Planning | 3 | 8 | 4+6 | `{"cp1_selection": 3, "cp2_selections": [4, 6], "cp3_selection": 8}` |
| 7 Openness to Paradigm Shift | 2 | 11 | 7 | `{"cp1_selection": 2, "cp2_selections": [7], "cp3_selection": 11}` |
| 8 Layered Hope | 1 | 10 | 6 | `{"cp1_selection": 1, "cp2_selections": [6], "cp3_selection": 10}` |

### Q11 Patterns
| Pattern | CP1 | CP3 | CP2 Preferred | Test Command |
|---------|-----|-----|---------------|--------------|
| 1 Clear Priority + Provider Barrier | 1 | 10 | 4 | `{"cp1_selection": 1, "cp2_selections": [4], "cp3_selection": 10}` |
| 2 Comfort vs Consciousness Tension | 1 | 12 | 6 | `{"cp1_selection": 1, "cp2_selections": [6], "cp3_selection": 12}` |
| 3 Balance + Spiritual Integration | 2 | 13 | 7 | `{"cp1_selection": 2, "cp2_selections": [7], "cp3_selection": 13}` |
| 4 Values-Driven + Cultural Need | 3 | 14 | 5 | `{"cp1_selection": 3, "cp2_selections": [5], "cp3_selection": 14}` |
| 5 Priority + Disparity Awareness | 1 | 15 | 8 | `{"cp1_selection": 1, "cp2_selections": [8], "cp3_selection": 15}` |
| 6 Balance + Consciousness Fear | 2 | 10 | 6 | `{"cp1_selection": 2, "cp2_selections": [6], "cp3_selection": 10}` |
| 7 Goals Over Comfort + Holistic View | 3 | 12 | 7 | `{"cp1_selection": 3, "cp2_selections": [7], "cp3_selection": 12}` |
| 8 Priority + Communication Gap | 1 | 15 | 9 | `{"cp1_selection": 1, "cp2_selections": [9], "cp3_selection": 15}` |

### Q12 Patterns
| Pattern | CP1 | CP3 | CP2 Note | Test Command |
|---------|-----|-----|----------|--------------|
| 1 Independence Maximalist | 1 | 15 | 5 preferred | `{"cp1_selection": 1, "cp2_selections": [5], "cp3_selection": 15}` |
| 2 Family Protector | 1 OR 2 | 14 | 4 preferred | `{"cp1_selection": 1, "cp2_selections": [4], "cp3_selection": 14}` |
| 3 Structural Realist | 3 | 12 | 8 preferred | `{"cp1_selection": 3, "cp2_selections": [8], "cp3_selection": 12}` |
| 4 Quality Focused | 2 OR 3 | 11 | 6 preferred | `{"cp1_selection": 2, "cp2_selections": [6], "cp3_selection": 11}` |
| 5 Cultural Navigator | ANY | 13 | 7 REQUIRED | `{"cp1_selection": 2, "cp2_selections": [7], "cp3_selection": 13}` |
| 6 Voice Protector | 1 | 10 OR 12 | 9 preferred | `{"cp1_selection": 1, "cp2_selections": [9], "cp3_selection": 10}` |
| 7 Interdependence Seeker | 3 | 10 | 4 OR 5 preferred | `{"cp1_selection": 3, "cp2_selections": [4], "cp3_selection": 10}` |
| 8 Open Explorer | 2 | ANY | catch-all | `{"cp1_selection": 2, "cp2_selections": [5], "cp3_selection": 11}` |

---


## Quick Reference: All Questions

### Q10A
- CP1: 1=Very important, 2=Somewhat, 3=Avoid intervention
- CP2: 4=Provider bias, 5=Uncertainty, 6=Burden, 7=Witnessed
- CP3: 8=Disability meeting, 9=Team support, 10=Education, 11=Disability paradox

### Q10B
- CP1: 1=Absolute, 2=Conditional, 3=Balance
- CP2: 4=Communication fear, 5=Late-stage fear, 6=Identity fear, 7=No awareness
- CP3: 8=QOL research, 9=Family presence, 10=Medical advances, 11=Value beyond cognition

### Q11
- CP1: 1=Comfort priority, 2=Balance, 3=Goals over comfort
- CP2: 4=Provider fear, 5=Faith, 6=Consciousness, 7=Total pain, 8=Disparity, 9=Cultural
- CP3: 10=Evidence, 12=Witnessing, 13=Spiritual, 14=Cultural adaptation, 15=Advocacy

### Q12
- CP1: 1=Very important, 2=Somewhat, 3=Not important
- CP2: 4=Burden, 5=Identity loss, 6=Quality of care, 7=Cultural conflict, 8=Structural, 9=Preferences
- CP3: 10=Interdependence, 11=Witnessing dignity, 12=Structural understanding, 13=Cultural competence, 14=Family, 15=Resources
