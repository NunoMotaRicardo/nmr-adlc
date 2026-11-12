# UI prototype

Generate **several radically different variations** of one surface, switchable from a floating bar. The user flips between them, picks one — or steals pieces from each — and the rest is thrown away.

Signals this is the branch: *"What should this page look like?"* · *"I want to see options for this dashboard before committing."* · any time the alternative is the user spending a day choosing between three vague mockups in their head.

If the question is about logic or state rather than appearance — wrong branch, use [LOGIC.md](LOGIC.md).

## Two sub-shapes — strongly prefer A

Variants are far easier to judge **butting up against the rest of the app** — real header, real data, real density. A standalone route is a vacuum where every variant looks fine.

**Sub-shape A — inside an existing surface (default).** The route exists. Variants render on that same route, gated by a `?variant=` search param; existing data fetching, params, and auth all stay, and only the rendering swaps. Something with no page yet that would naturally live *inside* one — a new dashboard section, a new card — is still sub-shape A: mount the variants in the host.

**Sub-shape B — a new route (last resort).** Only when the thing genuinely has no existing surface to live in. Follow the project's routing convention rather than inventing a top-level structure, and put `prototype` in the path so it's unmistakable. Before committing to B, check once more that there's really no page this could be embedded in — an empty route hides the design problems a populated one exposes.

The floating bar is identical in both.

## 1. State the question and pick N
Default to **3** variants; past 5 they stop being radically different and become noise. Write the plan in one line at the top of the file: *"Three variants of the symbol detail view, switchable via `?variant=`, on the existing route."*

## 2. Generate radically different variants
Each variant answers to the surface's purpose, the data it can reach, and the project's existing styling system. Give each a clear name — `VariantA`, `VariantB`, `VariantC`.

They must be **structurally** different: different layout, different information hierarchy, different primary affordance. Three tweaked card grids is wallpaper, not a prototype. Two drafts coming out similar → redo one under an explicit constraint ("no card grid").

## 3. Wire the switcher
One switcher on the route reads the param, renders the matching variant, and mounts the floating bar beneath it. Sub-shape A keeps all existing data fetching above the switcher, so only the rendered subtree changes.

## 4. Build the floating bar
Fixed at bottom-centre: left arrow (previous, wrapping), the current variant key and its name (`B — sidebar layout`), right arrow (next, wrapping).

- Arrows update the URL param through the framework's router, so a variant is shareable and survives reload.
- `←`/`→` cycle too — passing through when an `input`, `textarea`, or `contenteditable` has focus.
- Visually distinct from the page — high-contrast pill, shadow — so it reads as scaffolding rather than part of the design being judged.
- Free to stay unguarded: this branch never merges, so the bar can't reach users.

Put it in one shared component both sub-shapes reuse.

## 5. Hand it over
Surface the URL and the variant keys. The user flips through when they get to it, and the useful feedback is usually *"the header from B with the sidebar from C"* — that's the actual design they want.

## 6. Capture the answer
Once a variant wins, hand which and why to `/pm`, and record the chosen design as a dated decision record in the target repo's `<pm_docs>/architecture/adr/` via the same skill — the layout, the information hierarchy, and what the losing variants got wrong. Running unattended, leave it in `NOTES.md` on the branch first.

Then the whole worktree goes, winner included. The real page is built later from that decision record, through `/implement`; nothing here is promoted.

## Anti-patterns
- **Variants differing only in colour or copy.** Real variants disagree about structure.
- **Sharing too much between variants.** A shared header is fine; a shared layout defeats the point.
- **Wiring variants to real mutations.** Read-only, or a stub — the question is what it should look like.
- **Promoting the winning variant.** It was written under prototype constraints and it leaves with the branch. The decision record is what survives, and the real page is written from that.
