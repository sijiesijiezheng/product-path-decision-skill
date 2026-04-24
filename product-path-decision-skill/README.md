# Product Path Decision Skill

A reusable decision layer that checks whether a user's current approach is optimal before an AI system executes the request.

This skill is designed for AI agents, coding assistants, and workflow systems that should not blindly follow a user's first proposed path.

---

## What this is

Most AI systems respond directly to the user's instruction.

However, in complex tasks, the user's chosen path may not be optimal.

This skill adds a pre-execution decision layer that evaluates:

- what the user is trying to achieve
- what approach the user is currently considering
- whether a clearly better option exists
- whether clarification is needed before proceeding

It returns structured JSON instead of directly answering the user.

---

## Why this matters

Without a decision layer, an AI assistant may immediately execute a weak or premature path.

Example:

```text
I want to validate my AI product quickly, so I plan to spend a week designing a beautiful landing page first.
```

A normal assistant may start helping with the landing page.

This skill first checks whether that path is optimal.

In this case, the better path is likely:

```text
Validate the core input-output loop before investing in landing page design.
```

---

## Core idea

Traditional flow:

```text
User input → Direct answer
```

Decision Skill flow:

```text
User input → Decision check → Then decide whether to answer, clarify, or redirect
```

---

## Output schema

The skill returns JSON:

```json
{
  "problem_type": "decision | direction | execution",
  "goal_identifiable": true,
  "approach_identifiable": true,
  "is_applicable": true,
  "better_option_exist": true,
  "assumption_risk": "high | low",
  "needs_clarification": true,
  "user_goal": "...",
  "current_approach": "...",
  "better_option": "...",
  "reason": "..."
}
```

---

## Problem types

| Type | Meaning |
|---|---|
| `decision` | The user has a goal and a current approach |
| `direction` | The user has a goal or confusion but no clear approach |
| `execution` | The user asks for a direct task or factual output |

---

## Example

Input:

```text
I want to quickly improve my resume wording so I can apply immediately.
```

Output:

```json
{
  "problem_type": "decision",
  "goal_identifiable": true,
  "approach_identifiable": true,
  "is_applicable": true,
  "better_option_exist": true,
  "assumption_risk": "high",
  "needs_clarification": true,
  "user_goal": "Quickly improve resume to apply for jobs immediately",
  "current_approach": "Improve resume wording",
  "better_option": "First strengthen and clarify key experiences and achievements, then refine wording",
  "reason": "Improving wording alone may not significantly improve outcomes if the underlying content is weak, but this depends on the current resume quality."
}
```

---

## How to use

### 1. As an Agent / Codex skill

Use `SKILL.md` as the instruction file for the decision layer.

Before executing a user request, ask the agent to run this skill first:

```text
Use the Product Path Decision Skill to evaluate this plan:

"I want to build an AI product, so I plan to design a beautiful landing page first."
```

The agent should return JSON only.

---

### 2. As a workflow decision node

This skill can be placed before answer generation:

```text
User input
→ Decision Skill
→ decision / direction / execution routing
→ answer, clarify, or execute
```

---

## Repository structure

```text
product-path-decision-skill/
  README.md
  SKILL.md
  validation.md
  examples/
    test_cases.json
  assets/
    baseline-deepseek-resume-01.png
    baseline-deepseek-resume-02.png
    baseline-deepseek-resume-03.png
    baseline-deepseek-resume-04.png
    final-skill-resume-clarification.png
    final-skill-weather-execution.png
```

---

## Validation

Validation covers three areas:

1. Baseline comparison without the skill
2. Problem type classification
3. Clarification stability across similar cases

See:

```text
validation.md
examples/test_cases.json
```

The validation shows that the skill can:

- classify decision / direction / execution inputs
- prevent direct execution for factual queries
- detect when a better path may exist
- identify when clarification is needed before giving advice

---

## Suitable use cases

- AI coding agents before writing code
- product-building workflows
- resume / career decision support
- learning strategy correction
- workflow routing before answer generation

---

## Not intended for

- direct factual answering
- translation-only tasks
- emotional support
- open-ended exploration without a current approach

---

## Key idea

This is not a chatbot.

It is a pre-execution decision layer that helps AI systems avoid blindly following a user's first proposed path.
