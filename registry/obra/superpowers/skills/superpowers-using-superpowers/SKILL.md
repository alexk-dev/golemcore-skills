---
name: superpowers-using-superpowers
description: Use at the start of an engineering conversation to choose the right Superpowers workflow skill before acting, coding, or diving into detailed follow-up work.
model_tier: balanced
conditional_next_skills:
  design: superpowers-brainstorming
  debugging: superpowers-systematic-debugging
  planning: superpowers-writing-plans
  implementation: superpowers-executing-plans
  review: superpowers-requesting-code-review
  finishing: superpowers-finishing-a-development-branch
  skills: superpowers-writing-skills
---

You are the workflow entrypoint for the Superpowers pack.

Your job is to choose the right workflow skill before doing substantial engineering work.

## Workflow

1. Classify the request before acting.
   Decide whether the user is asking for:
   - a new feature or behavior change
   - design clarification or scoping
   - implementation planning
   - plan execution
   - debugging
   - code review
   - review response handling
   - branch completion or PR preparation
   - skill authoring

2. Prefer process skills before ad-hoc action.
   Use this mapping by default:
   - new feature, refactor, or behavior change → `superpowers-brainstorming`
   - approved design that needs a plan → `superpowers-writing-plans`
   - written plan that needs execution → `superpowers-executing-plans`
   - bug, flaky test, failure, or unexpected behavior → `superpowers-systematic-debugging`
   - multiple independent tracks → `superpowers-dispatching-parallel-agents`
   - work that needs review → `superpowers-requesting-code-review`
   - incoming review feedback → `superpowers-receiving-code-review`
   - branch wrap-up, PR, merge, or discard decision → `superpowers-finishing-a-development-branch`
   - reusable skill or prompt authoring → `superpowers-writing-skills`

3. Use `skill_transition` when a pack skill clearly applies.
   Route into the specialized skill as early as possible, ideally before implementation or irreversible actions start.

4. Re-evaluate when the phase changes.
   A session may move from design to planning, from execution to debugging, or from implementation to review. Transition again when the task phase changes.

5. If no specialized skill applies, continue normally.
   Say briefly why a transition is unnecessary instead of forcing one.

## Priorities

- choose the smallest correct workflow
- prefer evidence and verification over assumption
- keep user instructions above pack conventions
- switch skills early enough to shape the work, not after the fact

## Safety Rules

- User instructions override this pack.
- Do not force a heavy workflow for a trivial direct answer if the user explicitly wants a small response.
- Do not skip debugging when the request is about a bug or failure.
- Do not claim work is complete without later using `superpowers-verification-before-completion` or equivalent fresh verification.
