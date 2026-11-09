---
name: brainstorm
description: >
  Relentless one-question-at-a-time interview that takes a diffuse idea to a set of resolved
  work items sharing one goal. Use when the user wants to open or shape a plan, stress-test a
  design or decision, asks to be grilled, or a pm plan work item's next step is a brainstorm.
---

# Brainstorm

Interview the user **relentlessly** until the shared understanding is confirmed and the idea has resolved into work items. You walk one decision branch at a time; you write no files. What you return goes to `/pm`, which records it.

## Ground rules

- **One question at a time.** Give your recommended answer first, then ask the user to confirm, correct, or choose an alternative. A question that offers no recommendation makes the user do your thinking.
- **Look before you ask.** Check whether the answer already sits in nearby code, `wiki/`, the repo's `specs/` (root-level, code-bearing repos only), or the plan's own detail files. Ask the user where to look only when the context isn't discoverable.
- **Relentless means every branch.** Continue until the user has confirmed each open branch — including the ones that surfaced mid-session. A branch left unasked becomes a surprise during implementation.

## Two entry points

**Opening a plan** — the idea is diffuse and the goal is what you're establishing. Drive toward the goal first, then toward the work items that serve it.

**Advancing one work item** — `/pm` sends you a single item whose shape, scope, or goal isn't settled, plus its detail file. Stay inside that item; a question that belongs to a sibling item goes back in the report rather than getting resolved here.

## Resolving into work items

Most sessions resolve into one coherent plan. Some expose independent findings that can be picked up, scheduled, or delegated separately. **A finding is its own work item when it can be handed to someone carrying no context on the others.** Forcing those into one narrative is what makes a plan unworkable later.

For each work item, capture:
- the finding or open question;
- why it's independent;
- the dependencies it has or creates, named by work-item title;
- its concrete **next step** — brainstorm, research, prototype, diagnose, decide, write-document, create-issues, or implement — with the rationale when the choice isn't self-evident.

Pick the next step by what the item actually lacks. An item waiting on an external fact takes research, not another interview.

## Completion

Stop when every decision branch is confirmed and you can state either one coherent plan or a set of independently actionable work items, each with its next step. Report that to `/pm` — the summary, the work items, and any question you deliberately left open with the reason it's still open.
