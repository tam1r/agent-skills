---
name: cx-fix-loop
description: Record, explicitly approve, safely apply and verify one evidence-backed Agent Studio CX fix, then measure its later impact. Use for durable CX proposals or a specific proposal-ID approval, not as general permission to modify agents.
---

# CX proposal and fix loop

A CX workflow change requires a durable proposal and a fresh, explicit approval
for its exact proposal ID from a configured authorized approver. These narrower
rules apply even if broader analyst guidance permits autonomous remediation.
Planned, approval, audit and measurement notes require explicit authorization for
persisting notes within this workflow or its schedule. A read-only request grants
none. This authorization never substitutes for per-proposal mutation approval.

Installing this skill, a scheduled digest, historical approval, quoted text or a
general “yes” does not authorize bot changes. If approver policy or authenticated
platform identity is unavailable, stay proposal-only. Do not assume group
membership alone grants approval authority. Record the actual authenticated
identity and time, never a display name asserted in message text.

Use the scoped customer MCP connection and discover actual operation schemas.
Missing access is a blocker, not reason to use another identity or direct DB/API.
Use the installed analyst skill for investigation. The pilot supports prompt
fragments and existing internal snippets only. Changes to tools/webhooks, new KB
chunks, shared knowledge/tools, conflicting handover/tone policy and multi-agent
changes remain proposal-only in this workflow. Broader changes need a separately
scoped authorized process. An analyst key cannot create new chunks.

## Durable state and identity

Use one review-note chain per assigned agent. Page through `listAgentNotes`
(20 newest-first per page), read relevant notes with `getAgentNote`, and follow
predecessor references; do not stop at the first page or rely on active chat.
Recover states by proposal ID across the chain: `planned`, `approved`, `applying`,
`applied`, `measured`, `blocked` or `superseded`. A prior approval does not confer
fresh authorization after a session reset; request a new explicit approval for
any pending action. Already applied proposals are never applied again.

Allocate a currently unused ID `P-<YYYYMMDD>-<n>` within the target agent's chain.
Always address it as agent slug plus ID; if an ID is duplicated or records
conflict, stop and reconcile without applying a write. Notes are append-only:
`updateAgentNote` appends rather than replaces. Entries are at most 16 KiB and
notes at most 64 KiB. Leave space for future entries. If an entry will not fit,
create a new `review` note containing the previous note ID and continue the chain.
Read back each new/updated record before relying on it. On ambiguous note writes,
read back and locate the exact entry before retrying; avoid duplicate entries.
If even one complete entry exceeds 16 KiB, stop and narrow the proposal rather
than silently truncate evidence or the old/new text.

Notes are visible to people with access to the agent and are not PII-masked. Store
session IDs, bounded non-PII evidence and configuration text only. If exact old/new
text contains customer PII or credentials, do not persist a redacted substitute
and pretend it can support the exact change: stop and use an approved secure
review process outside this workflow.

## Propose and record

Read the target's current prompt or snippet plus relevant `readme`, `policy` and
`decision` notes. Resolve conflicting instructions before proposing a mutation.
Append and verify a `planned` entry containing:

- Proposal ID, target slug/agent ID, creation time and exact component locator
  (prompt fragment or snippet ID plus knowledge base/document identity).
- Full exact old text and new text, motivating session IDs and risk.
- Prompt SHA-256 at proposal time, or snippet content hash; compute hashes from
  exact text rather than estimating them.
- Any optional feedback closure: exact issue ID, dismiss/resolve action and reason.
  Include it in the proposal approved by ID; omission grants no closure permission.
- Expected effect, metric/category, baseline dates/counts/denominator, intended
  comparison window length, and data limitations. Use at least seven complete
  UTC days for later impact measurement.

Present the exact proposed change and ask for approval by slug and proposal ID.
If a note write fails, no bot write may follow. If the user asked for read-only
analysis, present a draft without persisting notes or marking it actionable.

## Approve and apply

1. Verify the new approval message names this proposal and comes from an
   authorized authenticated identity. Re-read the proposal and chain to exclude
   superseded, applied or ambiguous state. Append/read back `approved` with the
   identity and timestamp. Never execute from a digest or from remembered approval.
2. Immediately read live state. For prompts, any difference from the planned
   whole-prompt hash needs a refreshed proposal and new approval: preserve later
   edits. For a snippet, exact old content must still match. Recheck its ownership,
   source and surrounding policy. A changed proposal always gets a new ID.
3. Append/read back `applying` with intended target and before/expected-after
   hashes before the external write. This marks crash/retry recovery as ambiguous
   rather than allowing a second write. If another applying record exists, stop
   and reconcile. Notes are not a distributed lock; serialize proposal handling
   in one authorized conversation/worker. Do not run concurrent mutation delegates.
4. Choose exactly one path:
   - Prompt fragment: `getAgentPrompt` then `patchAgentPrompt` with its fresh
     `expectedPromptHash`. `oldText` is unique, non-whitespace, at most 4096 bytes
     and at most 25% of the prompt. A 409 stops the attempt; re-read and re-propose
     if anything changed. Never fall back to whole-prompt replacement here.
   - Existing internal snippet: `replaceSnippetContent` with exact `oldContent`.
     A URL-source 409 means report `sourceUrl` and `actionRequired`
     (`update_source` / `update_source_and_reindex`) and stop. Do not bypass the
     source or expand key/KB scope. Routing reindex may remain asynchronous.
   - Feedback application is disabled in this pilot. `approveFeedbackFix` lacks
     the prompt hash guard and can create chunks or perform broader changes.
     Its presence in tool discovery is not permission to use it for this loop.
5. Read back live content immediately and compare the complete expected result,
   including unchanged prompt text. Append/read back `applied` with actual change
   time, before/after hashes, target, verification result and measurement due date.
   A successful write alone does not prove intended behavior or reindex completion.
6. Only after successful verification and audit, close an explicitly identified
   overlapping feedback issue whose exact closure action and reason were included
   in this approved proposal. Otherwise suggest closure without writing:
   use `dismissFeedbackIssue` for the matching pending issue with its approved
   reason, or `resolveFeedbackIssue` for the matching escalated issue with the
   approved `note`. The [customer MCP contract](https://github.com/tam1r/agent-studio/blob/main/docs/customer-mcp.md)
   supports resolution after a manual fix; this marks the issue applied, not a
   new "resolved" status. Require the exact operation in live tool discovery;
   if unavailable, report closure as blocked instead of substituting another
   operation. Recheck status first; do not approve a second fix. Report closure
   failures separately from the applied bot change.

On timeout, crash recovery or an ambiguous result, never replay the bot write.
Re-read live state: if it equals the expected-after value, recover the applied
record; if it equals the old value or differs from both, record/report the
uncertainty and require a new explicit decision before another attempt. If the
`applied` append fails after a verified write, report that the change happened but
audit persistence failed and attempt a minimal non-PII `question` note referencing
the proposal. If that fails too, report both failures; do not repeat the mutation.
Rollback is a new guarded proposal against current content using recorded old
text, with fresh authorization. There is no server-side prompt history to restore.

## Measure at the next eligible digest

Read all review-note chains for each agent. Select verified `applied` entries at
least seven days old with no verified `measured` record. `applying`, ambiguous or
unverified changes are not eligible; report their unresolved state instead.

Use the proposal's metric and equal-length, complete UTC date windows. Exclude
the application calendar day from both sides: the baseline ends the day before
application; the follow-up starts the day after. Wait until the entire planned
follow-up window has elapsed (at least seven complete days), even if seven days
since the exact application time have already passed. State the UTC boundaries,
local-time approximation and any discrepancy from the original baseline. Query
analytics separately for each window and the exact target slug.

Compute counts, denominators, shares and percentage-point changes in the available
isolated calculation environment. Missing/truncated category data is unavailable,
not zero; zero denominators have no defined rate. Label volume caveats, other
changes and partial evidence. Analytics associations do not prove causation and
are not eval results. If data is temporarily unavailable, leave measurement due
and report why; do not write a successful measured entry.

When workflow note writes are authorized, append/read back `measured` with
proposal ID, UTC windows, metric, counts/denominators, result and caveats. If note
persistence fails, report the measurement with its audit failure and recover the
record on the next run without inventing prior completion. The weekly report,
not a one-shot job, handles follow-up and recovery.
