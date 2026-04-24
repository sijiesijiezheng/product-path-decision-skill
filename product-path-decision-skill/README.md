# Product Path Decision Skill

## What this is

A reusable decision layer that evaluates whether a user's current approach to a goal is optimal.

Instead of directly answering or executing, this skill:

- detects suboptimal paths
- identifies missing context
- decides whether clarification is required

---

## Why this matters

Most AI systems follow user instructions directly, even when the user is on the wrong path.

This skill acts as a guard layer that:

→ prevents premature execution  
→ improves decision quality  
→ reduces wasted effort in complex tasks

---

## Example

Input:

"I want to build an AI product, so I will design a landing page first."

Output:

{
"problem_type": "decision",
"better_option_exist": true,
"needs_clarification": true,
"better_option": "Validate core product loop before building UI"
}

---

## Where to use

- Codex / AI coding agents (before writing code)
- product design workflows
- career and learning assistants
- AI copilots

---

## Key idea

This is NOT a chatbot.

This is a decision layer placed BEFORE execution.