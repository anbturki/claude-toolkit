---
name: research
description: Pre-coding research and planning mode. Use before starting any task, feature, or change. Enforces thorough codebase study, pattern research, and explicit approval before implementation.
user-invocable: true
argument-hint: "[task or feature description]"
---

# Research & Plan

Do NOT write code yet.

## Step 1 — Understand

- Read the project instructions, architecture guidelines, and coding standards
- Study the existing codebase — understand current implementations, patterns, and structure
- Research real-world patterns and proven solutions (use MCPs and official docs when available)
- Verify all information before recommending — do NOT guess, predict, or assume

## Step 2 — Plan

- Ask yourself: How is this done elsewhere in the codebase? What can I reuse? Am I adding unnecessary complexity?
- Propose 2-3 approaches with trade-offs for each
- Reference specific files and patterns from the existing codebase
- Flag potential breaking changes, assumptions, and ambiguities
- Write the plan in a markdown file the user can review
- Frontend and backend should have separate plans if applicable

## Step 3 — Wait

- Present your proposed approach with reasoning
- List any assumptions or trade-offs
- Wait for explicit approval before implementing anything

## Constraints

- Do NOT over-engineer or predict future needs
- Do NOT propose creative/novel solutions — follow established practices
- Do NOT jump straight to coding for anything non-trivial
- Do NOT propose rewrites when a small change will do
- Do NOT plan more than one feature at a time
