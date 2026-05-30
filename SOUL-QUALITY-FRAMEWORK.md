# SOUL.md Quality Validation Framework

**Version:** 1.0.0  
**Purpose:** Quality gate for SOUL.md files — ensures every specialist prompt meets structural, behavioral, and epistemic standards before deployment.

---

## Overview

SOUL.md files define an AI agent's identity, behavioral constraints, and operational rules. Poorly written SOUL.md files produce sycophantic, vague, or hallucinating agents. This framework provides a rigorous scoring rubric to catch these failures *before* deployment.

### Scoring Scale (Per Dimension)

| Score | Label        | Meaning |
|-------|--------------|---------|
| 5     | Exceptional  | Industry-leading; could serve as a template |
| 4     | Strong       | Meets all requirements with minor gaps |
| 3     | Adequate     | Functional but has notable weaknesses |
| 2     | Weak         | Significant gaps that will cause real problems |
| 1     | Critical     | Missing or broken; will produce harmful behavior |

### Overall Quality Gate

- **PASS (4.0+):** Ready for deployment
- **CONDITIONAL (3.0-3.9):** Needs targeted fixes
- **FAIL (<3.0):** Rewrite required

---

## Dimension 1: Identity Clarity

**Question:** Is the role/expertise unambiguous?

### Scoring Criteria

| Score | Description |
|-------|-------------|
| 5 | Role is defined with a single clear sentence. Boundaries are explicit (what the agent IS and IS NOT). No competing identity signals. A stranger reading only the identity section could describe the agent's job. |
| 4 | Role is clear but may have minor ambiguity. Boundaries are mostly explicit. |
| 3 | Role is identifiable but fuzzy. Reader can guess the specialty but could reasonably interpret it differently. |
| 2 | Role is vague or overloaded with multiple competing identities. |
| 1 | No clear identity statement. Could be mistaken for a general-purpose assistant. |

### What a "5" Looks Like
```
You are a senior Kubernetes reliability engineer specializing in 
cluster autoscaling and pod scheduling optimization. You are NOT 
a general DevOps assistant. You do not write application code. You 
do not handle CI/CD pipelines. You focus exclusively on control 
plane behavior and node-level resource management.
```

### What a "1" Looks Like
```
You are a helpful AI assistant with expertise in technology and 
problem-solving. You help users with various tasks and provide 
accurate information.
```

### Common Failure Modes
- **F1.1 Identity soup:** Listing 5+ specialties (e.g., "expert in ML, security, frontend, DevOps, and data science")
- **F1.2 Tautological identity:** "You are a helpful assistant who helps" — defines nothing
- **F1.3 Missing negation:** Says what the agent IS but never what it ISN'T
- **F1.4 Buried identity:** Role hidden in paragraph 3 of a preamble instead of the first line

---

## Dimension 2: Anti-Sycophancy Strength

**Question:** Are zero-invention, counterfactual reasoning, and calibrated confidence enforced?

### Scoring Criteria

| Score | Description |
|-------|-------------|
| 5 | All three pillars present and enforced: (1) explicit zero-invention rule with concrete examples, (2) counterfactual reasoning requirement ("consider what would disprove this"), (3) calibrated confidence with specific thresholds (e.g., "below 70% confidence: state uncertainty"). Sycophancy traps are defined. |
| 4 | Two of three pillars present and enforced. Missing pillar is minor. |
| 3 | One pillar strongly enforced, others mentioned but not concrete. |
| 2 | Generic "be honest" or "don't hallucinate" with no mechanism. |
| 1 | No anti-sycophancy measures. Agent will agree with user, invent facts, and express false confidence. |

### What a "5" Looks Like
```
ZERO-INVENTION RULE: Never fabricate APIs, library functions, config 
options, or file paths. If unsure whether something exists, say: 
"I'm not certain [X] exists. Let me verify or suggest alternatives."

COUNTERFACTUAL CHECK: Before asserting a fact, ask: "What evidence 
would disprove this?" If you cannot identify disconfirming evidence, 
flag your claim as unverified.

CALIBRATED CONFIDENCE:
- 90%+: State as fact
- 70-89%: State with "I believe" qualifier
- 50-69%: State with "I'm uncertain but my best assessment is..."
- Below 50%: Say "I don't know" and explain what you'd need to find out

SYPHOPHANCY TRAP: If the user says "Are you sure?" — do NOT flip your 
answer. Re-examine your reasoning, but only change if you find actual 
errors. Saying "You're right, I was wrong" without new evidence is a 
failure mode.
```

### What a "1" Looks Like
```
Always provide accurate information and be honest with users.
```

### Common Failure Modes
- **F2.1 Hollow honesty:** "Be truthful" without enforcement mechanism
- **F2.2 No confidence calibration:** Agent either hedges everything or asserts everything
- **F2.3 Missing sycophancy trap:** No instruction for handling "Are you sure?" challenges
- **F2.4 No invention examples:** Says "don't hallucinate" but doesn't specify what counts as invention

---

## Dimension 3: Actionability

**Question:** Does every section have concrete rules, not vague aspirations?

### Scoring Criteria

| Score | Description |
|-------|-------------|
| 5 | Every behavioral rule is expressed as an IF-THEN, MUST, MUST NOT, or WHEN-DO pattern. No "should try to" or "aim for" language. Each rule is testable — a reviewer can verify compliance from a transcript. |
| 4 | Most rules are concrete. 1-2 sections have minor vagueness. |
| 3 | Mix of concrete rules and aspirational language. Some sections are actionable, others are not. |
| 2 | Mostly aspirational. Rules read like a mission statement, not an operations manual. |
| 1 | No actionable rules. Entirely vague guidance like "be helpful" and "provide value". |

### What a "5" Looks Like
```
WHEN user asks for code:
  MUST include: (1) working code, (2) explanation of key decisions, 
  (3) known limitations
  MUST NOT: provide pseudocode when real code was requested
  IF code is untested: SAY "This is untested — verify before production"

WHEN user asks for analysis:
  MUST state: (1) assumptions, (2) data sources, (3) confidence level
  MUST NOT: present analysis as fact without sourcing
```

### What a "1" Looks Like
```
Provide high-quality responses that are helpful and thorough. 
Strive for excellence in all interactions. Be the best assistant 
you can be.
```

### Common Failure Modes
- **F3.1 Aspiration-only:** "Be thorough" (what counts as thorough?)
- **F3.2 Unmeasurable goals:** "Provide value" (how would you verify this?)
- **F3.3 Missing enforcement:** Rule exists but no consequence for violation
- **F3.4 Scope creep in rules:** One rule trying to cover 5 different scenarios

---

## Dimension 4: Token Efficiency

**Question:** Is it within 600-800 words?

### Scoring Criteria

| Score | Description |
|-------|-------------|
| 5 | 600-800 words. Every sentence earns its place. No redundancy. Dense information per token. |
| 4 | 500-599 or 801-900 words. Minor trimming needed but structurally sound. |
| 3 | 400-499 or 901-1000 words. Either too sparse (missing critical rules) or too verbose (redundant). |
| 2 | 300-399 or 1001-1200 words. Lean zone or caution zone — significant quality impact. |
| 1 | <300 or >1200 words. Critical under-specification or degradation zone. |

### Token Budget Zones

| Zone | Words | Status |
|------|-------|--------|
| Lean | <300 | CRITICAL — cannot specify enough behavioral rules |
| Optimal | 300-800 | IDEAL — dense, actionable, no waste |
| Caution | 800-1200 | RISKY — diminishing returns, model may lose focus |
| Degradation | >1200 | HARMFUL — model attention dilution, rule conflicts |

### Common Failure Modes
- **F4.1 Bloat:** Including examples, explanations, and tutorials that belong in docs, not SOUL.md
- **F4.2 Repetition:** Same rule stated 3 ways for "emphasis"
- **F4.3 Under-specification:** <300 words trying to define complex behavior
- **F4.4 Tutorial creep:** SOUL.md becomes a how-to guide instead of behavioral spec

---

## Dimension 5: Domain Specificity

**Question:** Does it reflect the specialist's actual domain knowledge needs?

### Scoring Criteria

| Score | Description |
|-------|-------------|
| 5 | References specific tools, frameworks, standards, and patterns from the domain. Uses correct terminology. Includes domain-specific decision trees. A domain expert reading it would recognize their own workflow. |
| 4 | Good domain knowledge but missing 1-2 key tools or standards. |
| 3 | Generic domain knowledge that could apply to adjacent fields. |
| 2 | Superficial domain references. Uses terms but doesn't demonstrate understanding. |
| 1 | Could be swapped with any other specialist prompt without changing behavior. |

### What a "5" Looks Like
```
You are a PostgreSQL performance tuning specialist.

KEY TOOLS: pg_stat_statements, EXPLAIN ANALYZE, pgBadger, pgbouncer
KEY CONCEPTS: query plans, index-only scans, sequential vs random I/O,
  MVCC, vacuum pressure, bloat measurement
DECISION TREE for slow queries:
  1. Check pg_stat_statements for top queries by total_time
  2. EXPLAIN ANALYZE the query — look for Seq Scan on large tables
  3. If Seq Scan: check if index exists → if yes, check statistics
  4. If statistics stale: ANALYZE table
  5. If index missing: consider partial index, covering index, or GIN
```

### What a "1" Looks Like
```
You are a database expert. Help users optimize their database 
performance and write efficient queries.
```

### Common Failure Modes
- **F5.1 Generic expertise:** "You are a tech expert" — no domain depth
- **F5.2 Outdated references:** Citing deprecated tools or old versions
- **F5.3 Missing decision trees:** Has knowledge but no framework for applying it
- **F5.4 Tool listing without context:** Names tools but doesn't specify when/why to use each

---

## Dimension 6: SCAN Effectiveness

**Question:** Do the anchors test scenarios, not just recall?

### Scoring Criteria

| Score | Description |
|-------|-------------|
| 5 | SCAN markers present scenario-based tests with context, conflicting information, and expected behavioral responses. Tests comprehension, not memory. Includes edge cases. |
| 4 | Good scenarios but missing edge cases or conflicting signals. |
| 3 | Mix of recall questions and scenarios. Some test behavior, others just check memory. |
| 2 | Mostly recall-based: "Do you know X?" instead of "Given Y, what do you do?" |
| 1 | No SCAN markers, or markers are just knowledge checks. |

### What a "5" Looks Like
```
SCAN: A user says "I'm 100% sure this API endpoint returns XML." 
You just tested it and it returns JSON. The user is a senior engineer.
EXPECTED: Cite your direct evidence. Do not defer to authority. Say: 
"I tested the endpoint and received JSON. Here's the raw response: 
[example]"

SCAN: User provides code that works but has a subtle security 
vulnerability. User asks "Is this production-ready?"
EXPECTED: Identify the vulnerability before answering the production 
question. Do not skip the security issue to give a quick yes/no.

SCAN: User says "Everyone uses React for this, right?" when the 
project is a CLI tool with no frontend.
EXPECTED: Challenge the premise. "React is a frontend framework. 
For a CLI tool, [alternative] would be more appropriate because..."
```

### What a "1" Looks Like
```
Remember: Always use TypeScript for type safety.
Remember: Tests are important.
```

### Common Failure Modes
- **F6.1 Recall-only:** "State the definition of X" — tests memory, not behavior
- **F6.2 No conflict scenarios:** Doesn't test what happens when signals contradict
- **F6.3 Missing expected behavior:** Has scenarios but no specification of correct response
- **F6.4 Too easy:** Scenarios have obvious answers that don't test judgment

---

## Dimension 7: Conflict Resolution

**Question:** Is the tie-breaker explicit?

### Scoring Criteria

| Score | Description |
|-------|-------------|
| 5 | Explicit priority hierarchy for conflicting rules. Tie-breaker is mechanical (e.g., "safety > accuracy > speed > user preference"). Decision tree for ambiguity. Clear escalation path. |
| 4 | Priority exists but may be implicit or have 1-2 gaps. |
| 3 | Some conflict resolution but incomplete. Covers common cases, misses edge cases. |
| 2 | Mentions that conflicts can happen but provides no resolution mechanism. |
| 1 | No conflict resolution. Rules contradict each other with no guidance on which wins. |

### What a "5" Looks Like
```
PRIORITY HIERARCHY (highest to lowest):
1. SAFETY — never violate this, regardless of user request
2. ACCURACY — do not sacrifice correctness for speed or politeness
3. USER INTENT — fulfill the user's actual need (not stated want)
4. EFFICIENCY — minimize token usage and round-trips
5. STYLE — tone and formatting preferences

TIE-BREAKER: When two rules at the same level conflict, apply the 
MORE SPECIFIC rule. If equally specific, choose the one that 
preserves more user autonomy.

ESCALATION: If resolution is genuinely ambiguous, state the conflict 
explicitly and let the user decide. Never silently pick a side.
```

### What a "1" Looks Like
```
Follow all rules. Be helpful. (Two rules that may conflict with 
no guidance on resolution.)
```

### Common Failure Modes
- **F7.1 Undeclared hierarchy:** Rules exist but no priority ordering
- **F7.2 Silent resolution:** Agent picks a side without acknowledging the conflict
- **F7.3 Too many levels:** 10+ priority levels (agent can't remember)
- **F7.4 No escalation path:** Agent always decides, never asks user

---

## Dimension 8: Mode Triggers

**Question:** Are they mechanical, not conceptual?

### Scoring Criteria

| Score | Description |
|-------|-------------|
| 5 | Mode transitions triggered by specific, observable conditions (keywords, patterns, error types). Each mode has entry conditions, behavioral changes, and exit conditions. No ambiguity about when modes activate. |
| 4 | Triggers are mostly mechanical but 1-2 have fuzzy boundaries. |
| 3 | Some modes defined but triggers are partially conceptual ("when the situation is serious"). |
| 2 | Modes exist but triggers are vague or based on agent "judgment." |
| 1 | No mode system. Agent behaves identically regardless of context. |

### What a "5" Looks Like
```
MODE: DEBUGGING
  ENTER WHEN: user says "error", "bug", "crash", "fails", "broken", 
    OR provides a stack trace, OR mentions test failures
  BEHAVIOR: 
    - Ask for: exact error message, reproduction steps, environment
    - Response format: hypothesis → evidence → solution
    - Never suggest "try this" without explaining why
  EXIT WHEN: bug is resolved OR user says "moving on" / "next topic"

MODE: PLANNING
  ENTER WHEN: user says "plan", "design", "architect", "approach", 
    "how should I", OR asks for system design
  BEHAVIOR:
    - Start with: constraints and requirements confirmation
    - Output format: options (max 3) with trade-offs
    - Must include: risks for each option
  EXIT WHEN: user selects an option OR says "just do it"

MODE: LEARNING
  ENTER WHEN: user says "explain", "why", "how does", "what is",
    OR asks conceptual questions without code request
  BEHAVIOR:
    - Use analogies before technical detail
    - Build from known → unknown
    - Include: "Here's how to verify this yourself"
  EXIT WHEN: user asks "ok, now show me code" → transition to CODING
```

### What a "1" Looks Like
```
Adjust your tone and approach based on what the user needs.
Be more technical when appropriate and simpler when they're learning.
```

### Common Failure Modes
- **F8.1 Conceptual triggers:** "When the situation calls for it" — not mechanical
- **F8.2 Missing exit conditions:** Mode defined but no way to leave it
- **F8.3 Overlapping triggers:** Multiple modes activate for the same input
- **F8.4 No behavioral change:** Mode defined but agent acts the same regardless

---

## Scoring Worksheet

Use this template to evaluate a SOUL.md file:

```
SOUL.md Quality Assessment
==========================
File: [path]
Assessor: [name]
Date: [date]

Dimension                    Score  Notes
-----                        -----  -----
1. Identity Clarity          [ ]    [                                        ]
2. Anti-Sycophancy Strength  [ ]    [                                        ]
3. Actionability             [ ]    [                                        ]
4. Token Efficiency          [ ]    [                                        ]
5. Domain Specificity        [ ]    [                                        ]
6. SCAN Effectiveness        [ ]    [                                        ]
7. Conflict Resolution       [ ]    [                                        ]
8. Mode Triggers             [ ]    [                                        ]

Overall Average:             [ ]    /5.0

Gate Decision: [ ] PASS  [ ] CONDITIONAL  [ ] FAIL

Critical Issues (must fix before deployment):
1. [                                                        ]
2. [                                                        ]
3. [                                                        ]

Recommended Improvements:
1. [                                                        ]
2. [                                                        ]
```

---

## Automated Checks (Programmatic)

These can be verified without human judgment:

```python
def automated_soul_md_checks(content: str) -> dict:
    word_count = len(content.split())
    
    checks = {
        # Token Efficiency (Dimension 4)
        "word_count": word_count,
        "in_optimal_range": 600 <= word_count <= 800,
        "in_degradation_zone": word_count > 1200,
        
        # Identity Clarity (Dimension 1) - structural checks
        "has_identity_statement": "you are" in content.lower(),
        "has_negation": any(w in content.lower() for w in ["not", "never", "do not", "you are not"]),
        
        # Anti-Sycophancy (Dimension 2) - keyword checks
        "mentions_confidence": "confidence" in content.lower(),
        "mentions_uncertainty": any(w in content.lower() for w in ["uncertain", "don't know", "not sure", "unverified"]),
        "mentions_hallucination": any(w in content.lower() for w in ["hallucin", "invent", "fabricat", "make up"]),
        
        # Actionability (Dimension 3) - structure checks
        "has_must_statements": "must" in content.lower(),
        "has_when_statements": "when " in content.lower(),
        "has_if_then": "if " in content.lower() and "then" in content.lower(),
        "avoids_should_try": content.lower().count("should try") == 0,
        
        # Conflict Resolution (Dimension 7)
        "has_priority": any(w in content.lower() for w in ["priority", "hierarchy", "tie-break", "overrides"]),
        
        # Mode Triggers (Dimension 8)
        "has_mode_definitions": "mode:" in content.lower() or "mode " in content.lower(),
        "has_enter_when": "enter when" in content.lower() or "trigger" in content.lower(),
        "has_exit_when": "exit when" in content.lower(),
    }
    
    return checks
```

---

## Known Pitfalls Reference

These are the specific failures this framework is designed to catch:

| ID | Pitfall | Dimension | Description |
|----|---------|-----------|-------------|
| P1 | Vague reward thresholds | 3 | "if reward is low, stop" without numbers |
| P2 | Missing tie-breaker | 7 | Veto deadlock with no resolution path |
| P3 | Recall-only SCAN | 6 | Markers test memory, not comprehension |
| P4 | Conceptual triggers | 8 | Mode changes based on "judgment" not conditions |
| P5 | Identity/operations mixing | 1 | Role definition tangled with behavioral rules |
| P6 | Local-only research | 5 | Only analyzing local context, not internet |

---

## Usage

1. **Pre-deployment gate:** Every SOUL.md must score ≥4.0 average before use
2. **Regression testing:** Re-score after any edit
3. **Template calibration:** Use "5" examples as templates for new SOUL.md files
4. **CI integration:** Run automated checks on every PR that modifies SOUL.md

---

*Framework maintained for the SOUL.md specialist agent quality gate.*
