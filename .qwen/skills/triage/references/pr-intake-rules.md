# PR Intake Rules

Use these rules to decide whether a PR is ready for human review. Intake is not
deep code review and should not approve, merge, or request changes.

## Contents

- Sources Of Truth
- Product Fit
- Historical Design Context
- Body Completeness
- Author Validation
- Scope And Size
- Deep Review Handoff
- Label Taxonomy

## Sources Of Truth

- Current body structure: `.github/pull_request_template.md`
- Product direction: `.qwen/review-rules.md` if present, plus
  `docs/developers/roadmap.md`
- Historical context: linked issues, closed PR discussions, and markdown design
  docs mentioned by the PR or maintainer
- Size thresholds: PR gate design, 800 warning / 1500 strong split suggestion
- Labels: live repository labels from `gh label list --repo QwenLM/qwen-code
--limit 300`

## Product Fit

Aligned PRs usually:

- Extend the CLI/TUI-first developer workflow.
- Reuse the composable tool model, slash-command model, or existing repository
  conventions.
- Improve reliability, performance, developer experience, docs, tests, CI, or
  a user-reported bug.
- Are adjacent to roadmap items or existing public contracts.

Needs maintainer discussion:

- Core agent behavior, tool permissions, authentication, model selection,
  sandboxing, telemetry, release flow, public CLI/SDK contracts.
- Large or architectural changes without a linked issue or design rationale.
- "Tool X has this" arguments without Qwen Code-specific fit.
- Rewrites where an incremental extension looks possible.

Reject only when the direction is clearly incompatible, spam/test-only, or a
maintainer explicitly recorded that the same direction should not be pursued.
Prefer "needs discussion" when evidence is weak.

## Historical Design Context

Closed PRs are design history, not automatic negative precedent. Some closed PRs
contain useful markdown design notes, implementation sketches, test plans, or
tradeoff analysis that should still inform current intake.

When using closed PRs:

- Extract reusable rationale, constraints, terminology, and validation ideas.
- Note unresolved questions separately from rejected directions.
- Treat "closed because the plan was incomplete" as context to clarify the new
  PR, not as evidence the idea is bad.
- Treat prior discussion as a reject signal only when maintainers clearly said
  the product should not take that direction.
- Prefer phrasing like "there is useful prior design context in #NNNN" over
  "this was rejected before."

## Body Completeness

Current template sections:

- `## What this PR does`
- `## Why it's needed`
- `## Reviewer Test Plan`
- `### How to verify`
- `### Evidence (Before & After)`
- `### Tested on`
- `## Risk & Scope`
- `## Linked Issues`

Legacy PRs may use:

- `## Summary`
- `## Validation`
- `## Scope / Risk`
- `## Linked Issues / Bugs`

Treat either structure as acceptable if the content is meaningful. Strip HTML
comments, empty headings, placeholder bullets, empty test tables, and boilerplate
before judging completeness.

Complete means:

- The motivation and user-facing or maintainer-facing value are understandable.
- The reviewer test plan says what to verify and what result to expect.
- Risk/scope names the main risk, out-of-scope validation, or says N/A with a
  plausible reason.
- Linked issues are present when the PR claims to fix a bug or implement a
  tracked request. Independent small improvements may be acceptable without one.

Incomplete triggers:

- Empty body or only template placeholders.
- "Tested locally" without commands, outputs, screenshots, logs, or before/after.
- Risk/scope left blank for behavior, auth, sandbox, model, release, telemetry,
  or workflow changes.

When body completeness fails, stop at intake. Do not inspect implementation
correctness deeply to compensate for missing motivation, validation, or risk
context. The useful maintainer response is a clear request for the missing
author-provided context.

## Author Validation

Judge only evidence supplied by the PR author in the PR body or the author's
top-level PR comments. Do not run tests to manufacture evidence.

Good evidence:

- Exact commands and observed output.
- Reproduction before and verification after for bug fixes.
- Screenshots, GIFs, videos, tmux logs, or before/after captures for TUI or
  interactive workflows.
- Logs, JSON traces, or workflow run links.
- Benchmarks for performance claims.

Expected evidence by PR type:

| PR type            | Expected evidence                                      |
| ------------------ | ------------------------------------------------------ |
| TUI or interactive | Screenshot, recording, tmux log, or before/after       |
| CLI behavior       | Command transcript with observed output                |
| Bug fix            | Reproduction plus fixed behavior                       |
| API/SDK            | Test output or usage example                           |
| Performance        | Before/after numbers                                   |
| CI/workflow        | Workflow run link or local actionlint/smoke output     |
| Pure refactor      | Relevant tests/typecheck and why behavior is unchanged |
| Docs only          | N/A is acceptable if the doc surface is reviewed       |

## Scope And Size

Meaningful changed lines exclude lockfiles, generated files, snapshots, and
schema artifacts:

- `package-lock.json`, `pnpm-lock.yaml`
- `*.generated.*`, `*.snap`
- `schemas/*.schema.json`

Thresholds:

- `< 800`: normal.
- `800-1500`: warn and suggest a split if concerns are separable.
- `> 1500`: strongly suggest splitting unless the PR is cohesive.

Split signals:

- Refactor plus feature.
- Dependency or formatting churn plus behavior changes.
- Multiple unrelated packages or user workflows.
- "While I was here" cleanup.

`oversized-ok` means a maintainer explicitly accepts reviewability risk. It does
not make the PR more correct.

## Deep Review Handoff

Only hand off to code review after the PR is directionally reviewable:

- Product fit is `aligned`, or `discuss` has an explicit maintainer route.
- Body completeness is `complete`.
- Author validation is `present`, or the missing evidence is intentionally not
  applicable and that is credible.
- Scope is `focused`, or an oversized PR has maintainer acknowledgement.

Do not run deep code review as a substitute for missing PR context. Deep review
should evaluate correctness, security, performance, tests, and implementation
quality after intake is already meaningful.

## Label Taxonomy

Only use existing labels. Verify before suggesting.

Common families:

- Type: `type/bug`, `type/badcase`, `type/documentation`,
  `type/enhancement`, `type/feature-request`, `type/question`,
  `type/support`
- Priority: `priority/P0`, `priority/P1`, `priority/P2`, `priority/P3`
- Status: `status/need-information`, `status/waiting-for-feedback`,
  `status/ready-for-human`, `status/in-review`, `status/need-retesting`
- Category: `category/cli`, `category/core`, `category/ui`,
  `category/authentication`, `category/tools`, `category/configuration`,
  `category/integration`, `category/platform`, `category/performance`,
  `category/security`, `category/telemetry`, `category/development`
- Scope: add all applicable existing `scope/*` labels.
- Roadmap: add an existing `roadmap/*` only when clearly applicable.

Verdict mapping:

| Verdict                          | Label plan                                  |
| -------------------------------- | ------------------------------------------- |
| Product Fit = `discuss`          | `need-discussion`, `status/ready-for-human` |
| Product Fit = `reject`           | `TBD`, `status/ready-for-human`             |
| Body Completeness = `incomplete` | `status/need-information`                   |
| Scope = `oversized-acknowledged` | `oversized-ok` only if maintainer confirmed |
| Author Validation = `missing`    | `status/waiting-for-feedback`               |
| All pass                         | `status/in-review`                          |
