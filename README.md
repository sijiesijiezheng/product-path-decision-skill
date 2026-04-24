# Product Path Decision Skill

A reusable pre-execution decision skill for chat-based AI assistants.

It checks whether a user's proposed approach should be executed directly, clarified first, or redirected to a better option before the AI generates a final answer.

---

## Problem

When people talk to AI assistants, they often give the AI both:

1. a goal they want to achieve
2. a path they assume is the right way to reach that goal

For example:

```text
I want to quickly improve my resume wording so I can apply immediately.
```

or:

```text
I want to validate my AI product quickly, so I plan to spend a week designing a beautiful landing page first.
```

A normal AI assistant may directly follow the user's instruction.

However, the user's proposed path may not be the most effective way to achieve the real goal.

This often happens because users are limited by their own knowledge boundary. They may only know one visible action they can ask the AI to perform, even if that action is not the best next step.

For example, a non-technical user may want to build an AI product. Because they do not fully understand product development, frontend implementation, backend deployment, or validation strategy, they may ask the AI to start with a page design or a visible feature. If the AI simply follows the request, the project may move further away from the real goal.

The issue is not that AI never corrects users.

Sometimes it does.

But this correction is inconsistent and probabilistic.

Sometimes the AI questions the user's path; sometimes it simply executes.

This skill turns that unstable behavior into a structured decision step.

---

## What this skill does

This skill adds a decision layer before answer generation.

It evaluates:

- what the user is trying to achieve
- what approach the user is currently considering
- whether the current approach should be executed directly
- whether a clearly better option exists
- whether clarification is needed before proceeding

Instead of answering immediately, it returns structured JSON.

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

The goal is not to make the AI argumentative.

The goal is to make the AI pause when the user's proposed path may be wrong.

---

## Why this matters

Many AI assistants are optimized to be helpful and compliant.

That means they often execute the user's surface-level request even when the request may not be the best path toward the user's actual goal.

This skill is designed to prevent premature execution.

It helps AI systems move from:

```text
"Sure, I will do that."
```

to:

```text
"Before doing that, let's check whether this is the right path."
```

This is especially useful in complex or unfamiliar tasks where the user may not know what to ask for.

---

## When to use this skill

Use this skill when the user expresses:

- a goal
- a current approach
- a plan that may not be optimal
- a request where the AI should decide whether to execute, clarify, or redirect first

Common trigger patterns:

```text
I want to achieve X, so I plan to do Y.
Should I do X by doing Y?
I want to quickly do X, can you help me do Y?
I plan to solve X by doing Y.
I want to build X, so I will start with Y.
```

---

## When not to use this skill

Do not use this skill as the final answer for:

- factual questions
- translation requests
- summarization tasks
- direct execution tasks
- purely emotional inputs without a clear approach

These should be classified as `execution` or `direction`.

---

## Problem types

| Type | Meaning |
|---|---|
| `decision` | The user has a goal and a current approach |
| `direction` | The user has a goal or confusion but no clear approach |
| `execution` | The user asks for a direct task or factual output |

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

## Example 1: Resume wording

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

Why this matters:

A normal AI assistant may directly rewrite the resume wording.

This skill first checks whether wording is really the bottleneck.

---

## Example 2: Product-building path

Input:

```text
I want to validate my AI product quickly, so I plan to spend a week designing a beautiful landing page first.
```

Expected decision:

```json
{
  "problem_type": "decision",
  "better_option_exist": true,
  "needs_clarification": false,
  "current_approach": "Spend a week designing a beautiful landing page first",
  "better_option": "Validate the core input-output loop before investing in landing page design"
}
```

Why this matters:

For product validation, a polished landing page may not be the first bottleneck.

The better path may be to test whether the core user input, model reasoning, and output quality can work reliably first.

---

## Example 3: Direct execution

Input:

```text
What’s the weather in New York today?
```

Output:

```json
{
  "problem_type": "execution",
  "goal_identifiable": false,
  "approach_identifiable": false,
  "is_applicable": false,
  "better_option_exist": false,
  "assumption_risk": "low",
  "needs_clarification": false,
  "user_goal": "",
  "current_approach": "",
  "better_option": "",
  "reason": ""
}
```

Why this matters:

The skill should not over-analyze direct factual requests.

It should classify them as execution tasks and skip decision analysis.

---

## Compared with a native AI assistant

| Capability | Native AI assistant | With this skill |
|---|---|---|
| Directly follows user request | Usually yes | Only after decision check |
| Checks whether the current approach is optimal | Inconsistent | Yes |
| Separates decision / direction / execution | Usually no | Yes |
| Detects when clarification is needed | Inconsistent | Yes |
| Returns structured decision output | No | Yes |
| Prevents premature execution | Inconsistent | Yes |

---

## Suitable use cases

This skill is designed for chat-based AI systems where the user may propose a suboptimal path.

Suitable scenarios include:

- AI customer service: detect when the user's requested path is not the most useful service path
- Enterprise chatbot: guide employees toward the right workflow instead of simply drafting messages
- Learning assistant: correct inefficient learning plans before execution
- Career or resume assistant: identify when wording changes are insufficient without stronger content
- Product-building coach: prevent premature execution such as building UI before validating the core loop
- Agent workflow router: decide whether to answer, clarify, or redirect before downstream execution

---

## Not intended for

This is not a content-generation skill.

It does not directly create:

- diagrams
- slides
- resumes
- code
- final user-facing answers

It is a decision-control skill.

Its value is in improving the step before an AI assistant answers or executes.

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

## Current status

This repository contains a lightweight skill prototype.

It includes:

- a reusable `SKILL.md`
- structured output schema
- representative test cases
- validation documentation
- baseline comparison screenshots

Future extensions may include:

- Custom GPT version
- API endpoint
- workflow node integration
- enterprise chatbot integration
- coding-agent pre-execution routing

---

## Key idea

This is not a chatbot.

It is a pre-execution decision layer that helps AI systems avoid blindly following a user's first proposed path.
