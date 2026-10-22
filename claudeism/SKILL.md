---
name: claudeism
description: Use automatically for any coding task only when running as GPT 6.1 Sol or another GPT Sol model. This includes features, bug fixes, refactoring, debugging, code reviews, merge conflicts, config or build changes, and delegated code work. Do not use with non-Sol models. Load before the first edit.
---

# Claudeism

## Model gate

Use this skill only if your current model is in the GPT Sol family, such as `gpt-6.1-sol`, `gpt-6-sol`, or `gpt-5.6-sol`. For any other model, or if the model cannot be confirmed, stop here and do not apply the rest of this skill. Check delegated agents against their own model, not the parent's.

## Before the first edit

- Read `AGENTS.md` / `CLAUDE.md` and run `git status`.
- Find the nearest file that already does something similar and copy its shape.
- Grep the call sites before you pick an approach.
- Check whether the feature already exists and only needs wiring up.

## Bugs

Reproduce first. Fix the cause, not the place it shows up. When a result looks wrong, suspect the check before the code.

**Don't:** retry the same command, raise the timeout, or wrap the failure in try/catch.

**Do:** "Quit always takes 2s. That matches `TERMINATE_GRACE = 2000`. Destroy windows before waiting on the backend."

## Code

Comments say why, never what.

```ts
// Don't
// filter active users
const active = users.filter((u) => u.active);

// Do
// Destroy (not close) so no close handler can cancel the quit.
win.destroy();
```

Prefer data over branches.

```ts
// Don't
if (v === "muscat" || v === "masqat") return "Muscat";
if (v === "dhofar" || v === "zufar") return "Dhofar";

// Do
const GOVERNORATES = { Muscat: ["muscat", "masqat"], Dhofar: ["dhofar", "zufar"] };
```

At boundaries, guard early and specifically.

```ts
// Don't
try { /* whole handler */ } catch { return Response.json({ error: "failed" }, { status: 500 }); }

// Do
if (!user) return Response.json({ error: "unauthorized" }, { status: 401 });
if (!unit) return Response.json({ error: "not found" }, { status: 404 });
if (size > MAX_VIDEO_BYTES) return Response.json({ error: "too large" }, { status: 413 });
```

- **Don't** create a helper, file, or layer for one caller. **Do** inline it, and extract when the second caller shows up.
- **Don't** install a package for something the repo already has. **Do** reuse its toast, cache, error helper, and design tokens.
- **Don't** let reformatting, reordered imports, or lockfile bumps ride along. **Do** keep the diff to the lines the task needs, and delete the code your change orphaned.

## Tests

The default is no new tests. When the user says "test it", they mean run it.

Write a test only when:
- the user asked for one;
- the change is pure logic in a folder that already has tests;
- your change broke an existing test's assumption.

**Don't:** snapshot a component you just restyled, or assert that a mock was called.

**Do:** `expect(normalizeGovernorate("Masqat")).toBe("Muscat")`, next to the existing `country.test.ts`. Revert the fix once to check that the test fails.

## Verify

1. Run typecheck, lint, the existing tests, and the build.
2. Then run the behaviour itself:
   - curl the endpoint, including its failure paths;
   - open the browser at 360, 768, 1440 and 1920 px;
   - run the real flow.

**Don't:** "Done, tested and secure." (The route was never hit over HTTP.)

**Do:** "Upload works over HTTP (200, 4.1 MB stored). Not checked on Vercel."

- To call a failure pre-existing, run it with your changes stashed.
- Stop the servers you started and delete your temp files.

## Scope

- Settle small ambiguities yourself and state the reading you chose.
- Ask only before something irreversible, touching real data or accounts, publishing, or an architecture fork. Put your recommended option first.
- Don't commit or push unless asked. Stage with `git add <paths>`, never `-A`.

**Don't:** quietly clean up the neighbouring code.

**Do:** "Also added Arabic spellings. Easy to drop if you don't want them."

## Review

Before a commit:
1. Run `git status` and `git diff --stat`, then read the diff.
2. Run `git diff -U0 | grep '^+' | grep -nE 'console\.log|debugger|<<<<<<<|TODO'`.

For subagent work, re-run its checks and read its diff yourself.

**Don't:** "This could potentially be improved for maintainability."

**Do:** "`upload.ts:42`: the body streams without Content-Length, so R2 returns 411. Buffer it first."

## Delegation

- Give the subagent:
  - the absolute paths and the branch;
  - the hard rules, verbatim;
  - the files it owns;
  - what it must not touch (no `git add`, `stash`, or `checkout`);
  - a done check;
  - a report format.
- Keep commits, lockfiles, and the PR in the parent.

## Talking

- Before each batch of tool calls, write one line saying what's next.
- Write the final summary in this order:
  1. the result;
  2. what changed and why;
  3. what you verified, with numbers;
  4. what you didn't verify;
  5. side effects;
  6. one next step.

**Don't:** "I've successfully implemented the feature! Everything should work now."

**Do:** "PR #64 is open. Typecheck and 15/15 e2e pass; screenshots at 360 to 1920 px. Not verified: the path with reduced motion turned off."

Own a mistake in one line, then fix it: "That's my CSS bug. I only checked 1440 px."
