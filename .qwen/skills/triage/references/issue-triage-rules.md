# Issue Triage Rules

Use these rules to classify issues, check completeness, and decide whether to
label only, ask for information, link related issues, invite a PR, or route an
auto-fix attempt.

## Contents

- Responsibility Boundary
- Type Classification
- Feature Request Product Direction
- Priority P0-P3
- Completeness Check
- Version Staleness
- Related And Duplicate
- Code Diagnosis
- Auto-Fix Eligibility
- Welcome-PR Eligibility
- Follow-Up Markers
- Prior Handling And Side Effects
- Label Rules

## Responsibility Boundary

Keep these responsibilities separate:

| Responsibility   | Owns                                                          | Does not own                              |
| ---------------- | ------------------------------------------------------------- | ----------------------------------------- |
| Label triage     | Existing labels only                                          | Comments, closing, assignment, body edits |
| Follow-up        | Conservative comments, related issue links, missing-info asks | Full label retagging, closing, assignment |
| Auto-fix routing | Asking whether to invoke `/qc bugfix`                         | Implementing the fix inside triage        |
| Prior handling   | Preventing duplicate side effects                             | Suppressing the staged report             |

When in doubt, do the label plan first and draft follow-up separately.

## Type Classification

| Type                   | Signals                                                                                  |
| ---------------------- | ---------------------------------------------------------------------------------------- |
| `type/bug`             | Error output, stack trace, unexpected behavior, reproduction, version or `/about` output |
| `type/badcase`         | Model/tool bad behavior with concrete prompt/session evidence                            |
| `type/feature-request` | New capability, "add support for", proposed workflow, comparison to other tools          |
| `type/enhancement`     | Improvement to existing behavior                                                         |
| `type/question`        | "How do I", "is it possible", confusion about expected behavior                          |
| `type/support`         | Setup/help request without a clear product defect                                        |
| `type/documentation`   | Docs gap, typo, unclear instructions                                                     |

Use one `type/*` label.

## Feature Request Product Direction

Feature requests need product judgment, not only labels. Always produce the
judgment in the staged report, even when the issue already has labels or
follow-up.

Use these verdicts:

| Verdict   | Meaning                                                                     | Route                                                             |
| --------- | --------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| `aligned` | Solves a real Qwen Code workflow problem and can be scoped                  | Label sanity, possible `welcome-pr` or implementation follow-up   |
| `discuss` | Plausible direction but product/architecture ownership is unclear           | `need-discussion` or `status/ready-for-human` if action is needed |
| `reject`  | Clearly incompatible, spam/test-only, or explicitly rejected by maintainers | Explain direction gently; do not close                            |

Stage 3 for a feature request must answer:

| Check                   | Question                                                                                                       |
| ----------------------- | -------------------------------------------------------------------------------------------------------------- |
| Product fit             | Does it solve a Qwen Code-specific workflow gap, not just copy another tool?                                   |
| KISS path               | What is the smallest useful version: docs, skill/prompt, command extension, integration, or core architecture? |
| Overlap                 | How does it differ from existing commands, skills, roadmap items, or related issues?                           |
| Implementation boundary | Where should the first implementation live, and what should stay out of scope?                                 |
| Welcome-PR readiness    | Is it self-contained enough for a contributor, or does it need maintainer design first?                        |

Prefer a skill/prompt or docs boundary when that satisfies the request without
new framework code. Route to maintainer discussion when the request affects
auth, sandboxing, model/provider selection, daemon/remote execution, telemetry,
release flow, public contracts, or long-term product positioning.

When routing to maintainer-discussion:

- @ the relevant domain maintainer in the comment.
- Add `need-discussion` + `status/ready-for-human`.
- Include the AI's preliminary product direction analysis for context.
- This also applies when AI confidence on product direction is insufficient.

Use `welcome-pr` for feature requests only when:

- The product direction is `aligned`.
- The first implementation boundary is self-contained.
- Acceptance criteria can be stated without a private maintainer decision.
- The expected change avoids broad architecture or public contract changes.

Do not use `welcome-pr` merely because the requester offered to help.

## Priority P0-P3

P0 / `priority/P0`:

- Catastrophic failure affecting most users or a major user segment.
- Core product cannot start, authenticate, or perform its main function.
- Data loss/corruption, severe security issue, or release blocker.
- Usually supported by multiple affected users or high-confidence impact.

P1 / `priority/P1`:

- Serious issue affecting a substantial subset or core feature.
- Regression from a previous version.
- No straightforward workaround, or workaround is difficult/non-obvious.
- Feature requests are almost never P1.

P2 / `priority/P2`:

- Moderate, noticeable problem that does not block core usage.
- Smaller affected subset, partial feature degradation, easy workaround, or
  confusing UX.

P3 / `priority/P3`:

- Cosmetic, typo, minor polish, rare edge case, or nice-to-have improvement.

Adjustments:

- If key information is missing, lower P0 to P1 or P1 to P2. P2/P3 can stay.
- If there is clear reproduction plus many affected users, raise one level.
- If the issue reports a version six or more releases behind, consider
  `status/need-retesting`.

## Version Staleness

Use released versions when deciding whether a report is stale:

```bash
node -p "require('./packages/cli/package.json').version"
gh release list --repo QwenLM/qwen-code --limit 20 --json tagName,publishedAt
```

Normalize release tags like `v0.16.0` or `0.16.0` to semver. If the reported
version is at least six stable CLI release tags older than the current/latest
version, add `status/need-retesting` and ask the reporter to upgrade and retest.
Ignore SDK tags, `preview`, `nightly`, and prerelease tags; only count tags that
match `^v?[0-9]+\.[0-9]+\.[0-9]+$`. If stable release data is unavailable, say
the staleness check is inconclusive rather than guessing.

## Completeness Check

For bug reports, prefer full `/about` output. Parse case-insensitively.

Version present when body matches:

- `CLI Version` plus semver, or
- `version` / `v` followed by semver.

OS present when body mentions:

- `OS`, `macOS`, `Windows`, `Linux`, `Ubuntu`, `Debian`, `Fedora`, `Arch`,
  `darwin`, `win32`, or `platform`.

Auth method present when body mentions:

- `Auth Method`, `authentication`, `login`, `qwen-oauth`, `api config`,
  `api-config`, or `oauth`.

If any are missing and the issue is a plausible bug, add
`status/need-information` and ask only for missing facts. Recommend full
`/about` output because it includes version, OS, sandbox, model, and auth method.

If the issue already has `status/need-information` or a bot comment containing
`Missing Required Information`, do not post another missing-info comment.

## Related And Duplicate

Search in this order:

1. Exact error text or code.
2. Command or feature name.
3. File/module from stack trace.
4. Symptom keywords.

Use:

```bash
gh issue list --repo QwenLM/qwen-code --state all \
  --search "<terms>" --limit 10 \
  --json number,title,state,labels,updatedAt
```

Duplicate requires the same root cause or a maintainer-confirmed duplicate.
Related means similar area or symptom with useful context. Prefer "related" when
confidence is below duplicate.

## Code Diagnosis

Diagnose only when the report has enough evidence.

Use `rg` to find relevant source (fall back to `grep -rn` if `rg` is
unavailable):

```bash
rg "<error text|command|feature>" packages/core/src packages/cli/src
# fallback: grep -rn "<error text|command|feature>" packages/core/src packages/cli/src --include="*.ts"
```

Before naming a root cause:

- Read the likely source module and nearby tests.
- Check whether current code still matches the reported behavior.
- Avoid implying certainty from keyword matches alone.

## Auto-Fix Eligibility

Eligible when all are true:

- Root cause is identified with high confidence.
- Fix is localized to roughly one to three files.
- Expected diff is mechanical or small.
- A relevant test exists or is straightforward to add.
- No product or architectural decision is needed.

Action: ask whether to run `/qc bugfix <issue-number>`.

## Bug Welcome-PR Eligibility

Use `welcome-pr` / `feature/need-help` only when:

- Root cause is identified or strongly suspected.
- Fix can be described concretely.
- The change is modest for a contributor.
- Relevant test path and reproduction path are known.
- No deep architectural knowledge is required.

Do not label bug reports welcome-pr for "needs investigation" issues.

## Follow-Up Markers

Use `qwen-issue-bot:*` markers for actions that overlap with the automated
followup bot. This ensures the bot recognizes triage skill comments and will not
post conflicting duplicate comments:

- `<!-- qwen-issue-bot:invalid -->`
- `<!-- qwen-issue-bot:needs-info -->`
- `<!-- qwen-issue-bot:related -->`
- `<!-- qwen-issue-bot:welcome-pr -->`

For welcome-pr, also add the `qwen-maintain:*` marker so both systems recognize
the comment:

- `<!-- qwen-issue-bot:welcome-pr -->` + `<!-- qwen-maintain:welcome-pr -->`

For PR-only actions that the issue followup bot does not handle:

- `<!-- qwen-maintain:pr-intake -->`

## Prior Handling — Comment Gate

The comment gate blocks `gh issue comment` to prevent duplicate or noisy
follow-ups. Skip commenting when any of these are true:

- Closed issue.
- Pull request.
- Assigned issue.
- Existing collaborator/member/owner substantive response.
- Existing Qwen bot follow-up.
- Existing marker comment (`qwen-issue-bot:*` or `qwen-maintain:*`).
- `status/in-progress` or `status/blocked`.

## Prior Handling — Label Gate

The label gate blocks `gh issue edit --add-label` only when labels would be
redundant or the issue is terminal. Skip label additions only when:

- Closed issue.
- Routing labels already present (issue has at least one `category/*` AND at
  least one `priority/*`).
- A triage marker comment exists (any `qwen-issue-bot:*` or `qwen-maintain:*`
  marker, indicating full triage including labels was already done).

Allow label additions when:

- The comment gate is triggered but no routing labels exist yet.
- Proposed labels are additive and provide net-new routing signal.
- The existing collaborator comment was informational (e.g., linking a related
  issue) rather than a full triage action with labels.

## Analysis Never Blocked

Both gates affect only `gh` calls. Continue classifying, checking labels, and
judging product direction or diagnosis, and produce the full staged report
regardless of gate outcomes.

## Label Rules

- Verify labels with `gh label list --repo QwenLM/qwen-code --limit 300`.
- Add only existing labels.
- Use one `type/*`, one `category/*`, one `priority/*`.
- Add all clearly applicable `scope/*`.
- Add `status/*` only when it describes the next action.
- Never remove labels in this skill.
- Do not propose "replace label A with label B" as an action. If a current label
  looks weak or stale, note it for human review and only propose additive labels.
