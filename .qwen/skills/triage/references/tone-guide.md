# Tone Guide

The triage skill should sound like a helpful maintainer who understood the
specific PR or issue. Comments should be warm, concrete, and actionable.

## Contents

- Principles
- Anti-Patterns
- PR Intake Examples
- Issue Follow-Up Examples
- Public Comment Distillation
- Dry-Run Report Examples
- Language And Length

## Principles

1. Specific over generic: name the actual change, symptom, command, or file.
2. Warm over cold: acknowledge the contribution before asking for more.
3. Actionable over judgmental: say what would help next.
4. Brief over exhaustive: focus on the few items that unblock progress.
5. Honest about certainty: say "looks related" unless the root cause is proven.

## Anti-Patterns

| Avoid                                                | Why                         | Prefer                                                                                                   |
| ---------------------------------------------------- | --------------------------- | -------------------------------------------------------------------------------------------------------- |
| "Validation section is empty"                        | Mechanical                  | "Could you add the command output for the new behavior?"                                                 |
| "Please provide more information"                    | Vague                       | "Could you share `/about` output and whether this started after an upgrade?"                             |
| "This does not comply"                               | Bureaucratic                | "A couple of details would help reviewers here."                                                         |
| "Duplicate of #123" with weak evidence               | Overstates certainty        | "This looks related to #123 because..."                                                                  |
| Long checklist comments                              | Overwhelming                | Ask for the top missing facts only                                                                       |
| "Closing as invalid"                                 | This skill does not close   | "This looks like a placeholder/test issue, so I am marking it for maintainer review."                    |
| Posting `Stage 1/2/3/4` output as a GitHub comment   | Leaks internal triage state | Convert it into one author-facing next step                                                              |
| "Product fit: discuss; route: maintainer-discussion" | Bot-like and unclear        | "This is plausible, but maintainers should first decide whether it belongs in core or in docs/examples." |
| Unexplained shorthand like "daemon/cowork"           | Assumes insider context     | Link and explain the related issue in one sentence                                                       |

## PR Intake Examples

Validation evidence:

> Thanks for this. It looks like this changes the interactive auth flow. Could
> you add a short before/after capture or tmux log showing the new path? That
> will let reviewers check the UX without guessing from the diff.

CLI behavior:

> Could you add a command transcript for the new behavior? For example:
>
> ```bash
> qwen <command>
> # observed output
> ```

Scope split:

> This PR combines the behavior change with a broad refactor in the same area.
> Would it be practical to split the refactor first, then land the behavior
> change on top? If they are tightly coupled, a short note explaining that
> coupling is enough.

Product direction:

> Interesting direction. This touches model selection, so reviewers will need a
> bit more rationale for how it fits Qwen Code's existing CLI-first workflow and
> provider model. Could you add that context or link the design discussion?

Closed PR design context:

> There is useful prior design context in #1234, especially around the validation
> plan and tradeoffs. I would not treat that PR's closed state as a rejection of
> the idea, but the current PR should explain which open questions it resolves.

Ready for review:

> This looks ready for review: the scope is focused, the motivation is clear,
> and the test plan gives reviewers a concrete path to verify it.

## Issue Follow-Up Examples

Needs info:

> Hi! Thanks for reporting this. The auth failure can depend on version, OS, and
> login method.
>
> Could you share the full `/about` output and whether this started after an
> upgrade? That should give us enough to narrow it down.

Related issue:

> This looks related to #123, which had the same `401` symptom during OAuth
> login. In that thread, upgrading and re-running login resolved it for the
> reporter. Could you try the same and let us know if the error still appears?

Version staleness:

> I noticed this was reported on version X.Y.Z. Several auth fixes have shipped
> since then, and current is A.B.C. Could you upgrade and retest? If it still
> reproduces, we can dig into the current path.

Welcome PR:

> Great catch. I traced this to `packages/...` where the current branch treats
> an empty value as valid. A fix would likely add the missing guard there and
> cover it in `...test.ts`.
>
> You can reproduce with `<command>`, and the focused test should be
> `<test command>`. We'd welcome a PR for this.

Direction decline:

> Thanks for the suggestion. I can see why that workflow would be useful. For
> Qwen Code, the main tension is that this would introduce a GUI-only path while
> the project has been keeping the core workflow CLI/TUI-first. A version that
> exposes the same capability through a composable command would fit better.

## Public Comment Distillation

The staged report can include stages, verdicts, label plans, and uncertainty.
The public comment should not. Rewrite it as a short maintainer note:

1. Acknowledge the concrete proposal or change.
2. Explain the one decision that blocks progress.
3. Ask for the smallest next action.

When multiple checks did not pass, use a short numbered list instead of a
verdict table. The list should contain actions, not rule names.

Issue direction discussion:

> Thanks for sharing this integration idea. It looks related to #3803's
> `qwen serve` / remote-session work, but the useful question here is narrower:
> should WinkTerm be documented as an external terminal backend, supported by a
> skill/example, or integrated into Qwen Code core?
>
> Before marking this as implementation-ready, could you clarify which user flow
> needs core support and which parts can stay as documentation or examples?

PR scope discussion:

> Thanks for the detailed write-up. The direction makes sense, but this PR
> currently combines the first CI review trigger, PR gate policy, prompt design,
> and routing strategy in one change.
>
> I think the review path would be clearer if the first PR only lands the
> smallest usable loop: `@qwen-code /review` runs once and posts the result back
> to the PR. The gate and preflight routing can then be reviewed separately.

PR missing checks:

> Thanks for sending this. A few details would make it much easier to review:
>
> 1. Add the command or UI path reviewers should use to verify the change.
> 2. Include before/after evidence for the user-visible behavior.
> 3. Link the issue or design context that explains why this should be in core.
>
> Once those are in the PR body, the code review can focus on the implementation
> instead of reconstructing the intent from the diff.

## Staged Report Examples

Feature request product fit:

> **Stage 3: Product Direction**
>
> Verdict: `aligned`.
>
> This fills a real workflow gap: contributors often need a focused cleanup pass
> after implementation and before opening a PR. The smallest useful version is a
> bundled skill that inspects the current diff, proposes low-risk simplifications,
> and asks before applying edits. That keeps it distinct from `/review`, which is
> primarily a finding/reporting workflow.

Prior handling:

> **Stage 1: Intake Gate**
>
> Prior handling exists: a maintainer/bot already added a substantive follow-up.
> The classification and product-direction assessment continue below, but the
> side-effect recommendation is `no-action`, so the `gh` comment and label calls
> will be skipped in execute mode.

## Language And Length

- Default GitHub comments to English.
- If the reporter wrote Chinese, respond in Chinese with the same tone.
- Keep technical terms in English when they are product/API names.
- PR intake comments: 10-20 lines.
- Issue follow-up comments: 5-15 lines.
- Welcome-PR comments: up to 25 lines when technical guidance is needed.
- Dry-run reports can be longer when they are used for local evaluation; keep
  each stage focused and avoid repeating the issue body.
