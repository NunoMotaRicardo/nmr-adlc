---
name: research
description: >
  Investigate a question against primary sources and return cited findings. Use when an
  answer turns on an external fact — library semantics, an API's real behavior, a spec, a
  data format — or when a pm plan work item's next step is research.
---

# Research

Answer one question from **primary sources**, and cite every claim. What you return goes to `/pm`, which records it in the work item's detail file — you write no files yourself.

Spin the reading up as a **background agent** so the user keeps working while it reads.

## The question

One question, stated in a sentence before any reading starts. A research pass with a vague question returns a literature review nobody asked for. If the request carries several questions, name them and research the one that unblocks the work item; report the rest as still open.

## Primary sources only

Follow every claim back to the source that owns it:
- **Library or API behavior** → its source code and official docs, at the version this repo pins in `pyproject.toml`/`uv.lock`. A blog post describing yfinance's behavior is not evidence of yfinance's behavior.
- **A data format or provider's semantics** → the provider's own documentation, plus a real sample pulled from the actual store or endpoint.
- **This repo's behavior** → the code, its `specs/*.md` contracts (root-level, code-bearing repos only), and git history. Not a document describing the code — the code.
- **Standards and specs** → the spec text itself.

A claim you can only support from a secondary write-up gets reported **as** secondhand, flagged, with the primary source you couldn't reach. That flag is the finding — a confident claim resting on a blog post is how a plan matures on a false fact.

## Report

Return, per question:
- **The answer** — what's actually true, stated plainly.
- **The evidence** — the source for each claim: URL, file path and line, version, or the command whose output you're quoting. A claim without a source doesn't ship.
- **What it means for the work item** — how the answer changes the item's shape, scope, or next step. This is what `/pm` acts on.
- **What stayed open** — the questions the sources didn't settle, and what would settle them.
