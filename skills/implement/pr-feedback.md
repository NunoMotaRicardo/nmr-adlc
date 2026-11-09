# PR feedback loop

Drive every open review comment on the PR to a resolved state: each one ends fixed-and-replied or declined-and-replied, and every thread is marked resolved. You run this in the main thread. Code and tests still belong to the coder subagent (`coder.md`). The user merges.

## Inputs
The PR number and its branch. Two comment sources feed this loop:
- **Bot reviewers** — the logins listed under `review.bots` in `.sdlc/sdlc-config.yml`. A commenting bot that isn't listed → triage it anyway and offer to add it to the config.
- **Findings published by `/pr-review`** — inline comments from the two-axis review, already on the PR as review comments.

Both arrive as PR review comments, so one pass covers them.

## 1. Load the PR and its open comments
```bash
gh pr view <N> --json title,body,url,state,headRefName,baseRefName
gh api repos/<owner>/<repo>/pulls/<N>/comments \
  --jq '.[] | select(.in_reply_to_id == null) | {id, user: .user.login, path, line, body}'
```
Top-level comments need triage; skip ones already replied to.

## 2. Get onto the PR's code
Use the issue's existing worktree (`implement.worktree_path`) if there is one; otherwise `git fetch origin pull/<N>/head:pr-<N>` and check it out. Only ever add commits — never force-push, amend, or rebase the branch.

## 3. Verify each finding against the code
Take nothing on the reviewer's word. For each comment:
- Read the referenced file and enough context around it — calling code, the domain model, the relevant `specs/*.md` contract — to independently confirm the described failure is real.
- Check whether it names a genuinely new defect or an already-accepted pattern used elsewhere in the codebase (a sibling method's convention); that changes the right response.
- Classify it: **confirmed bug (blocking)**, **confirmed non-blocking** (style, minor robustness, a nitpick), **confirmed but out of scope or unreachable**, or **not a bug** (the reviewer misread the code or spec).

Where a finding is ambiguous and you can't verify it confidently, say so in the reply and ask the user before resolving that thread.

**The classification decides the board.** At least one confirmed bug, or a non-blocking finding you'll fix inline → the branch needs a coder round, so board move the issue to `in_progress` (per SKILL.md's board-move rules; skipped if `github.project` is absent) and say so. Everything declined, deferred, or already tracked → no code changes, so it stays at `in_review`; the remaining steps are replies and threads. A review's findings are claims until this step; this is the first point that knows which ones are real.

## 4. Fix the confirmed bugs — coder subagent
Skip this step when triage confirmed nothing that needs code; go straight to the replies.

Otherwise batch the blocking findings (plus any non-blocking ones small enough to fix without scope creep) and spawn the coder subagent per `coder.md`, giving it the findings, the worktree path, and the issue number. Its brief already binds it to minimal changes, a regression test per fix, and the four quality gates.

Beyond its brief, require of this round:
- A regression test per fix, named for the behavior it pins (`test_<behavior>_persists_into_<field>`), failing before and passing after.
- **The docs the fix touches, updated in the same change** — a `specs/*.md` contract or invariant, or a `wiki/architecture/` `Current State` section, that the fix contradicts. Watch the cross-cutting specs and architecture docs no single issue owns (layering rules, a cross-repo contract, the umbrella's system-overview doc); those are the ones that drift.

Then commit describing what was fixed and why (not "address PR comments") and push to the PR's branch.

## 5. Decide the non-fixes
For everything out of scope, unreachable, or not a bug, write the reasoning in one paragraph: what the risk actually is, why it doesn't apply now (no code path reaches it, it matches an established pattern, it would deviate from convention for no current benefit), and what would make it worth revisiting.

Confirmed non-blocking findings get that same scrutiny — low severity doesn't make a real issue stop mattering. Before deciding each one, check whether it's already covered: `gh issue list --search "<keyword>"`, the PR description's carry-forward notes, and the `wiki/` planning docs. Already tracked → say so in the reply instead of proposing a duplicate. Real, untracked, and not worth fixing in this PR → carry it into step 8.

## 6. Reply to every comment
Inline on each thread, never as a new top-level PR comment:
```bash
gh api repos/<owner>/<repo>/pulls/<N>/comments/<comment_id>/replies -f body="<reply>"
```
Fixed → the commit SHA, what changed, the regression test added. Declined → the paragraph from step 5.

## 7. Resolve the threads
Replies don't auto-resolve, and a thread gets resolved only after it has been replied to. Match each thread by its first comment's `databaseId`:
```bash
gh api graphql -f query='
query { repository(owner: "<owner>", name: "<repo>") { pullRequest(number: <N>) {
  reviewThreads(first: 20) { nodes { id isResolved comments(first: 1) { nodes { databaseId body } } } }
} } }'
gh api graphql -f query='mutation { resolveReviewThread(input: {threadId: "<thread_id>"}) { thread { isResolved } } }'
```

## 8. Offer to track what's left
Collect every item representing real, actionable work this PR doesn't cover and no existing issue or doc does: declined-but-real findings, the unfixed non-blocking ones from step 5, and any carry-forward note already in the PR description. **Ask the user** in one batched question covering all of them, flagging which you found already tracked so they aren't re-deciding something settled. On a yes, open each as its own issue — referencing the PR and comment it came from, labelled per the repo's conventions (`gh issue list --json labels,milestone` shows what's in use; untracked tech debt usually takes a component label and no milestone), and carrying a concrete fix sketch plus acceptance criteria rather than a restatement of the finding.

## 9. Resolve merge conflicts
Check if there are merge conflicts in the PR. If yes, resolve them.


## 10. Board and report
If step 3 moved the issue to `in_progress`, every thread now being resolved earns it back: board move to `in_review` (per SKILL.md's board-move rules, including its scope-error handling). If it never left, leave it.

Report: findings fixed vs. declined vs. non-blocking, the commits pushed, threads resolved, which findings turned out to be tracked already, any issues opened with links, and the issue's status. Then stop — the user reviews and merges.

Stay scoped to the comments in play; things no reviewer raised belong to a different issue.
