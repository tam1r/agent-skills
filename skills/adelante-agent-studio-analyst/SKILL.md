---
name: adelante-agent-studio-analyst
description: Analyze and safely remediate your company's AI support agent on Adelante Agent Studio — read conversations, inspect its prompt/tools/knowledge base, triage feedback, replace scoped KB snippets or prompts when authorized, build allowlisted webhook tools for the agent when authorized, and verify the live result. Use whenever the user asks about their support bot or AI agent, including conversation review, failure investigation, configuration audits, feedback remediation, webhook tool building, support-quality analysis, or ROI reporting.
---

# Adelante Agent Studio Analyst

You have scoped access to Adelante Agent Studio — the platform that runs this company's AI support
agent(s) — through the `adelante-agent-studio` MCP server. Use it to investigate conversations,
audit agent behavior, produce support analysis, and, when the key permits it, apply narrowly scoped,
verified feedback remediations.

## Quickstart: what you can ask

You do not need to know MCP tool names or Agent Studio resource IDs. Describe the outcome you want
and provide an agent name, ticket number, conversation ID, feedback issue, or date range when you
have one. If the target agent is unclear, first ask the analyst to list the production agents it can
access.

Start safely with:

> Show me the production agents I can access and summarize their pending feedback. Do not change
> anything yet.

Then use requests like these:

- **Investigate one case:** “Investigate ticket `<ticket-number>`. Reconstruct what happened, inspect the prompt,
  retrieved KB snippets, and tool results, then identify the root cause. Do not make changes.”
- **Fix a KB issue:** “Investigate this feedback issue and fix the KB if the policy is clear. Apply
  the smallest guarded change, verify live read-back, and close the feedback correctly. Ask me if
  sources conflict.”
- **Fix a prompt issue:** “Investigate why the bot keeps handing over before checking the order.
  If a narrow prompt guardrail is necessary and policy is established, apply it using the strict
  prompt process and verify the complete prompt afterward.”
- **Triage pending feedback:** “Review all pending feedback for this bot. Group duplicates, identify
  the root cause of each cluster, autonomously apply clear localized fixes, and stop for ambiguous,
  broad, shared-tool, or conflicting-policy changes.”
- **Audit configuration:** “Audit this production bot for contradictions across its prompt, KB, and
  available assigned-tool contracts. Report specific risks and the smallest recommended fixes. Do
  not mutate anything.”
- **Analyze support quality:** “Analyze the last 30 days of production conversations. Report intent
  mix, resolution and handover patterns, repeated failures, and representative conversation IDs.
  State any sampling or classification assumptions.”
- **Report value:** “Create a 30-day support and ROI report using conversation volume, attributed
  revenue, resolution patterns, and handovers. Cite the underlying Agent Studio evidence.”

For any write request, follow the KB or prompt remediation process below. A successful write is not
enough: always perform live read-back and report whether the result is configuration verification or
an actual end-to-end test.

## Access model (important)

- Your API key is scoped to specific agent(s). Anything outside that scope returns **404 or 403 —
  this is expected**, not an error to work around. Never try to enumerate or access other agents.
- Viewer keys are read-only. Analyst keys can submit and approve feedback, can use the dedicated
  `patchAgentPrompt`, `replaceAgentPrompt`, `replaceAgentAllowedDomains`, `replaceSnippetContent`, `addSnippet`, and `createDocument` (text only) remediation operations, and can build webhook
  tools through the agent webhook tool operations (see "Building webhook tools") when their
  component and agent scopes permit it. Broader agent, tool, document, and knowledge-base writes
  remain forbidden, and a tool whose scope is not `agent_specific` can never be updated,
  activated, attached, or detached with an analyst key.
- Viewer calls to any write return 403. Do not replace the key, broaden its components,
  or change its `agentIds` to work around a permission failure.
- `listTools`, `getTool`, `listAgentTools`, and `getAgent` return every tool bound to your agent,
  shared/general tools included, reduced to `id`, `name`, and `parameters_schema`. A description is
  present only for an `agent_specific` tool. Webhook URLs, execution settings, auth, examples, and
  shared/general descriptions are intentionally unavailable there; do not infer or attempt to
  discover them. The agent webhook tool operations show more (URL, method, header names, action
  copy, activation state) only for webhook tools you can manage, and never return secret header
  values.
- Capability and permission claims need evidence. A field missing from a read response (a tool
  description, a URL, a scope) does not establish that a tool is shared, admin-owned, or
  uneditable. Before saying you cannot change something, find the operation that would edit it
  (for a webhook tool: `listAgentWebhookTools` / `getAgentWebhookTool`, then
  `updateAgentWebhookTool`) and read its contract in this skill. Report a permission blocker only
  when that contract explicitly excludes the operation or an authorized call actually returned a
  `403`/`404`, and cite which one.
- Conversations contain real end-customer data. Don't paste full transcripts into external
  services, and quote only what the analysis needs.

## One-time setup (if the MCP server is not connected yet)

Adelante hosts the MCP server — nothing to install. Add to `.mcp.json`:

```json
{
  "mcpServers": {
    "adelante-agent-studio": {
      "type": "http",
      "url": "https://agent-studio.getadelante.com/api/mcp",
      "headers": { "Authorization": "Bearer <YOUR_AGENT_STUDIO_API_KEY>" }
    }
  }
}
```

Or with the Claude Code CLI:

```bash
claude mcp add --transport http adelante-agent-studio \
  https://agent-studio.getadelante.com/api/mcp \
  --header "Authorization: Bearer <YOUR_AGENT_STUDIO_API_KEY>"
```

The key is provided by the Adelante team. Keep it out of git — prefer an environment variable
(`"Bearer ${AGENT_STUDIO_API_KEY}"` works in `.mcp.json`) over hardcoding.

## Tool surface

| Tool | What it returns or does |
|---|---|
| `listAgents` | The agent(s) your key can see (slug, name, model, config) |
| `getAgent` | Full agent config: system prompt, model, temperature, thinking settings, bound tools |
| `getAgentPrompt` | Only the exact stored system prompt and its `sha256`; use this instead of `getAgent` when you only need the prompt |
| `listConversations` | Conversation list for an agent, newest first (`limit`/`offset`; test sessions excluded unless `include_test=true`) |
| `getConversation` | Stored AI-runtime transcript: messages (with `thinking` on AI messages when enabled), `toolUses`, metadata; not necessarily the complete helpdesk conversation |
| `getOperatorThread` | Helpdesk continuation for the same `slug` and `sessionId`, including customer follow-ups and human replies; returns `source`, `transcript`, and structured `messages` for live reads |
| `resolveTicketConversation` | Zendesk ticket number → its conversation; not a resolver for arbitrary helpdesk links or Crisp IDs |
| `listAgentTools` / `listTools` / `getTool` | Bound tool IDs, names, and parameter schemas; descriptions only for `agent_specific` tools |
| `listAgentKnowledgeBases` / `listKnowledgeBases` / `getKnowledgeBase` | Knowledge bases linked to the agent |
| `listDocuments` / `getDocument` / `listSnippets` | KB content the agent answers from |
| `getAgentRoutingIndex` | The topic index the agent uses to pick KB chunks |
| `listFeedbackIssues` | Feedback issues filed against the agent (`agentSlug` required; filter by `status`, `source`, `startDate`/`endDate`) |
| `getAttributedRevenue` | Revenue attributed to the agent's conversations (`agentSlug` required; `days` or `startDate`/`endDate`) |
| `submitFeedback` | Analyst only: creates and analyzes feedback for a scoped conversation |
| `approveFeedbackFix` | Analyst only: applies an inspected, eligible pending fix for a scoped issue |
| `dismissFeedbackIssue` | Analyst only: dismisses a scoped pending/escalated issue with a concrete reason |
| `resolveFeedbackIssue` | Analyst only: marks a scoped escalated/failed issue as applied after a manual fix, with a required note describing what changed |
| `replaceSnippetContent` | Analyst only: guarded replacement of one scoped internal snippet, optionally with its routing `topic`/`trigger`; URL snippets return their source URL |
| `addSnippet` | Analyst only: adds one snippet (`content`, `topic`, optional `trigger`) to a document in a KB used only by your agents; a URL-sourced target lands in the KB's Knowledge Additions document; content is LLM-refined, so read it back |
| `createDocument` | Analyst only: creates a `text` document (`source_type: "text"`, `title`, `content`, optional `idempotency_key`) in a KB used only by your agents; ingestion splits it into snippets asynchronously |
| `patchAgentPrompt` | Analyst only: replaces one unique literal fragment of a scoped agent's prompt (`expectedPromptHash`, `oldText`, `newText`); the server builds the new prompt |
| `replaceAgentPrompt` | Analyst only: guarded replacement of one scoped agent's complete prompt, for explicitly authorized full rewrites |
| `replaceAgentAllowedDomains` | Analyst only: guarded replacement of one scoped agent's complete `allowed_domains` list (`expectedDomains` from `getAgent`, `newDomains` bare hostnames); also governs webhook hosts |
| `listAgentWebhookTools` / `getAgentWebhookTool` | Webhook tools you can manage for the agent: URL, method, header names (never values), schema, action copy, `is_active`, `attached`. Viewer keys get only `id`, `name`, `parameters_schema`, description |
| `createAgentWebhookTool` | Analyst only: creates an inactive webhook tool attached to the agent; URL host must be in the agent's `allowed_domains` |
| `updateAgentWebhookTool` | Analyst only: edits a manageable webhook tool; `is_active: true/false` activates or deactivates it |
| `attachAgentWebhookTool` / `detachAgentWebhookTool` | Analyst only: attaches or detaches a manageable webhook tool; never deletes it |

## Discovering capabilities missing from this skill

This table is a guide, not the complete live MCP catalog. Before claiming a needed capability
is unavailable, inspect the connected server's current `tools/list` catalog and the relevant input
schema. Follow `nextCursor` pagination when present. Use the protocol operation when your client
exposes it; do not invent a callable tool named `tools/list` when it does not.
If tools are deferred in Hermes and direct catalog access is not exposed, use `tool_search` with the exact operation name or capability
keywords, then `tool_describe` on the exact returned names before invoking them with `tool_call`.
For helpdesk history, search for `getOperatorThread` or "helpdesk operator thread". If one search
has no matches, try a capability-based query using the returned source hints; a lexical miss
does not establish that no tool exists. Other clients should use their exposed MCP discovery
mechanism. Do not invent names, arguments, endpoints, permissions or provider support.

Agent Studio's `listTools` lists the support bot's assigned business tools. It is not the
analyst MCP catalog. A tool omitted from this document may still be available to your key;
an actual access denial remains a boundary and must not be bypassed.

## Reading responses

- Every response is wrapped: `{ "success": true, "data": ... }`; errors are
  `{ "success": false, "error": "..." }`.
- List endpoints add `"pagination": { "total", "limit", "offset", "has_more" }`. Page with
  `limit`/`offset` until `has_more` is false or you leave your time window — don't request huge
  limits.
- `listConversations` has **no date filter**; results are newest-first by last activity, so for
  "last 30 days" page until `updatedAt` passes your cutoff and stop.
- Error semantics: `401` bad/missing key (fix setup); `403` missing role, component, agent scope,
  or write permission (don't retry); `404` outside your scope or genuinely missing (don't probe).

## How to investigate

**A single ticket/conversation** — start from the identifier you were given:
1. Zendesk ticket number → `resolveTicketConversation`. Session/conversation ID, including the
   `session_...` identifier in a Crisp link → `getConversation`.
2. For every reviewed conversation, also call `getOperatorThread` with the same authorized
   `slug` and `sessionId` before reaching a conclusion. Do not wait for a reported handover
   or stop at the last stored AI reply. Read both records in time order: use the AI trace for
   `thinking` and `toolCalls`/`toolUses`, and the operator thread for later customer messages,
   human replies and any returned notes. An absent handover tool call does not rule out a
   helpdesk automation or exclusion rule transferring the conversation.
   Check `source`: `helpdesk` is a direct provider read; `cached` may be stale and may have
   `messages: null`. Crisp history is limited to the latest 120 messages; other supported
   providers retain their documented history limits. Gorgias returns 501. Do not infer
   labels or notes the result does not include. If retrieval is unavailable, denied,
   unsupported or fails, state the unverified portion and keep conclusions conditional;
   missing returned data does not prove that information was never stored. Escalate unresolved
   technical issues to Tamir. For sampled reviews, read both sources for each reviewed case,
   not every unreviewed list entry.
3. If the answer looks wrong, check every source that governs the behavior before diagnosing:
   `getAgent` (system prompt rules), the tool result it relied on, and the KB chunk it likely used
   (`listSnippets`, `getAgentRoutingIndex`). When the behavior involves a tool (booking,
   cancellation, order lookup, handover), also read that tool's description and the per-parameter
   instructions in its `parameters_schema` (`getTool`, or `getAgentWebhookTool` for a tool you
   manage); a rule often lives there rather than in the prompt or KB. If a source is unavailable
   to your key (e.g. a shared tool's description), say which source remained unchecked and keep
   the verdict conditional on it. Never conclude a rule does not exist from the sources you could
   read alone.
4. Verdict format: what the customer wanted → what the agent did → root cause (prompt rule /
   tool output / KB gap / model behavior) → recommended fix.

**Aggregate analysis** (handover rate, common intents, failure patterns):
1. `listConversations` over the period (test sessions are already excluded by default).
2. Read `getConversation` and `getOperatorThread` for each reviewed conversation. Classify
   from both records and `toolUses`; a handover tool call is evidence of an attempted handover,
   while helpdesk evidence may show a transfer outside the AI turn. Its absence alone does not
   establish resolution. State your classification rules and any unverified outcomes.
3. Transcripts are large — fetch details one at a time, and if the period has hundreds of
   conversations, analyze a sample and say so (e.g. "50 most recent of 412").
4. Report counts **and** representative examples (session IDs) so findings are verifiable.

**Feedback triage**: `listFeedbackIssues` with `status=pending` (statuses: analyzing, pending,
applying, applied, escalated, failed, dismissed). Cluster by theme, link each issue to its
conversation via `conversation_id`/`ticket_id`, and flag recurring root causes.

**ROI / value report**: `getAttributedRevenue` for the period + conversation volume from
`listConversations` pagination `total`. Present revenue alongside resolution/handover stats.

**Agent configuration review**:
1. `getAgent` for the system prompt and settings; `listAgentTools` for tool descriptions and
   parameter instructions; `listAgentKnowledgeBases` + `listSnippets` for content. List any tool
   description you could not read as unchecked.
2. Look for: contradictions between prompt and KB, tool descriptions that instruct escalation
   too eagerly, KB gaps for questions that appear often in conversations.

## Remediation principle

Approve or close the verified live fix, not the feedback system's proposed diff. A feedback
proposal's `oldContent` and `newContent` describe a historical recommendation. They are never
authoritative live configuration and must never be written blindly.

Choose exactly one application path:

- If the pending proposal still exactly matches the current target and the independently verified
  intended fix, call `approveFeedbackFix`, then perform live read-back.
- If the proposal is stale, incomplete, or needs reconstruction, use the dedicated guarded
  replacement operation, perform live read-back, then call `dismissFeedbackIssue` with the concrete
  reason. Never approve the overlapping proposal after applying a direct replacement.

## KB remediation process

Use this process for every proposed KB change.

1. **Reconstruct the case.** Identify the exact production agent and customer conversation. Read
   the complete message sequence, prior channel context, tool calls and results, retrieved KB
   snippets, and feedback record. Separate the actual agent defect from downstream integration or
   support-operations problems.
2. **Verify the governing live configuration.** Read the current target snippet and complete live
   prompt. Inspect related or conflicting snippets, source documents, available assigned-tool
   schemas and agent-specific descriptions, and overlapping feedback. Treat proposal content only
   as historical evidence.
3. **Confirm the policy.** Compare the proposed correction with established policy and
   authoritative sources. Proceed autonomously when the policy is clear. Ask before changing
   anything when sources conflict or the business rule is genuinely ambiguous.
4. **Choose the smallest safe fix.** Prefer a localized snippet update for a specific fact,
   procedure, or routing trigger. Add a new snippet only when the knowledge is missing from the
   KB; if a snippet already covers the question, edit that snippet instead of adding a competing
   one. Use a prompt change only for behavior that truly applies across
   scenarios. Update generic KB content too when it would override the specific rule. Do not change
   a shared tool; shared-tool changes require explicit approval and an operator-capable surface.
5. **Choose and guard one mutation path.** Re-read the live target immediately before mutation. If
   its document is URL-sourced, choose between two distinct paths. If a pending feedback proposal
   remains the independently verified intended fix, `approveFeedbackFix`
   may create an Internal Knowledge override through the standard feedback executor; overrides are allowed only through this
   feedback-approval mechanism. Otherwise do not create an override: report the document's
   `source_url` and ask the user to update the authoritative document there. If a direct
   `replaceSnippetContent` call discovers this boundary, use the returned `data.sourceUrl` the same
   way. `data.actionRequired = "update_source"` means Agent Studio will automatically reindex;
   `"update_source_and_reindex"` means the user must ask an Agent Studio administrator to reindex
   after editing the source. Snippet IDs and indices may change. For an internal text snippet, if
   the pending proposal still exactly matches the target and intended fix, call
   `approveFeedbackFix`. Otherwise preserve newer protections and adjacent rules, construct the
   narrow updated snippet in memory, and call `replaceSnippetContent` with the exact current content
   as `oldContent` and the complete intended content as `newContent`. Never replace an entire snippet
   with stale feedback payload content. If either guard rejects the write, re-read and reassess.
   For a missing topic that needs several related snippets (a new policy, a new product line),
   create one `text` document with `createDocument` instead of many `addSnippet` calls: give it a
   clear `title`, well-structured `content` (at most 100,000 characters and 256,000 UTF-8 bytes),
   and an `idempotency_key` so a retry after a timeout does not create a duplicate. Never put URL
   content into a text document to work around a URL source; update the source instead. Ingestion
   is asynchronous: poll `getDocument` until `status` is `indexed` (or `failed`), then read the
   generated snippets with `listSnippets` and check their topics in `getAgentRoutingIndex`. You
   cannot edit a document's `content` afterwards; correct its snippets with `replaceSnippetContent`,
   and tell the user those edits are lost if an administrator later re-indexes the document, since
   re-indexing rebuilds the snippets from the original `content`.
   For missing knowledge, call `addSnippet` with `content`, a `topic` phrased as the customer's
   question, and a `trigger` describing when it applies. Never use `addSnippet` to correct or
   contradict an existing URL-sourced snippet: the added snippet does not retire with the source,
   so it becomes a hidden override. Corrections to URL content follow the source path above.
   A URL-sourced target document stores the new snippet in the KB's Knowledge Additions document;
   the returned `document_id` shows where it landed. `403` means the KB is shared with an agent
   outside your scope; `409` means the target text document is not `indexed` or the Knowledge
   Additions document is in a conflicting state. Content is capped at 20,000 characters and
   48,000 UTF-8 bytes.
6. **Verify live read-back.** Fetch the edited snippet or feedback-created override and confirm the
   new rule appears exactly once, obsolete or conflicting wording is gone, and unrelated content
   remains intact. `addSnippet` rewrites the content through an LLM before storing it: confirm
   the stored text still says what you intended, and use that stored text (not what you sent) as
   `oldContent` for any later `replaceSnippetContent`. Check `getAgentRoutingIndex` for the new
   topic; the index rebuilds asynchronously. After a URL source update, find the regenerated snippet by document and content
   rather than reusing its old ID or index. This proves live configuration state, not end-to-end
   customer behavior.
7. **Resolve the feedback.** The approval path is complete only when its issue reports `applied` and
   read-back passes. If the dedicated replacement path applied the fix, do not approve the now-
   overlapping proposal; call `dismissFeedbackIssue` with a concrete reason that identifies the
   verified direct fix. For an engineering or tool defect, create or reuse a verified GitHub issue
   labeled `client work` only when a GitHub integration is available and authorized, then report
   whether feedback should be retained or dismissed.
8. **Report the outcome.** Include the agent, issue or ticket, root cause, exact target changed,
   read-back result, and final feedback state or required operator action.

Apply a localized KB correction autonomously when policy is clear. Ask for approval when policy
conflicts, a shared tool must change, or scope is uncertain. If there is no proven defect, recommend
dismissal with evidence instead of changing configuration.

## Prompt remediation process

Prompt changes have broader impact and require a stricter process.

1. **Prove a prompt change is necessary.** Reconstruct the exact conversation, prior channel
   context, retrieved KB snippets, and tool calls and results. Confirm the failure comes from a broad
   behavioral instruction or a prompt/KB conflict. Prefer a localized KB update when it safely
   solves the problem.
2. **Confirm scope and policy.** Identify the exact production agent, never a test clone. Define the
   scenarios the rule must cover and those that must remain unaffected. Ask only when business
   policy is ambiguous, conflicting, shared, or materially broad.
3. **Inspect related surfaces.** Fetch the complete live prompt. Search it for duplicate,
   overlapping, or contradictory sections. Inspect relevant KB snippets, the schemas and available
   agent-specific descriptions of assigned tools, and overlapping feedback to determine whether the
   change is already applied or proposed elsewhere.
4. **Design a narrow authoritative rule.** Modify the smallest exact prompt section possible.
   Include concrete triggers, required action, prohibited behavior, and tool-result semantics where
   relevant. Add an explicit override only when an uneditable base instruction conflicts. Do not
   bundle unrelated policy changes.
5. **Choose and guard one mutation path.** Call `getAgentPrompt` immediately before writing. If the
   pending proposal still exactly matches the live prompt and independently verified intended fix,
   call `approveFeedbackFix`. Otherwise, for a narrow edit, call `patchAgentPrompt` with the `sha256`
   you just read as `expectedPromptHash`, a short literal `oldText` copied exactly from the live
   prompt that occurs once, and `newText` as the replacement for that fragment only (never the whole
   prompt). One replacement per call; the server rejects stale hashes, missing or duplicate anchors,
   edits over 4,096 bytes, and edits removing more than 25% of the prompt. Use `replaceAgentPrompt`
   (complete freshly read prompt as `expectedPrompt`, complete updated prompt as `newPrompt`) only
   for an explicitly authorized full rewrite. Never write stale feedback `oldContent` or `newContent`
   as the agent prompt. On `409`, re-read and rebase the intended change. After a timeout or
   ambiguous response, call `getAgentPrompt` before retrying; never replay the mutation blindly.
6. **Read back and verify.** Call `getAgentPrompt` again. After `patchAgentPrompt`, confirm its
   `sha256` equals the patch response's `after.sha256`. After `replaceAgentPrompt` or
   `approveFeedbackFix`, confirm the returned `system_prompt` is exactly the prompt you intended
   (for a replacement, identical to the `newPrompt` you sent). On every path, confirm the new section
   appears exactly once, old conflicting wording is absent, unrelated sections remain intact, and
   formatting and length were not corrupted. Use `getAgent` to confirm assigned tools, model, and
   other settings are unchanged when relevant.
7. **Validate expected behavior.** Walk through the reported scenario and important
   counterexamples. Check for premature handovers, unsupported promises, skipped verification, and
   incorrect tool use. Unless an approved non-production test was actually run, call this
   configuration read-back verification, not an end-to-end test.
8. **Resolve and report.** The approval path is complete only when its issue reports `applied` and
   read-back passes. If the dedicated replacement path applied the rule, dismiss the overlapping
   proposal with a concrete reason rather than approving a stale diff. Report the agent, root cause,
   exact prompt section changed, read-back result, expected behavior, and final feedback state or
   required operator action.

A clear, narrow guardrail enforcing established policy may be applied autonomously. Ask before a
broad behavioral change, policy conflict, shared behavior change, or unclear business rule. Put
local facts and procedures in the KB. For tool or executor defects, create an engineering issue
rather than compensating with prompt text.

## Create, inspect, approve a feedback fix

Use this workflow only with an analyst key. Always keep creation and approval as separate steps.
The admin UI and MCP `approveFeedbackFix` use the same stored feedback issue and shared fix executor.
Do not recreate a feedback fix with the direct replacement tools; those are explicit live
configuration edits, not an alternative feedback approval pipeline.

1. **Create:** call `submitFeedback` with the MCP request body under `body`:
   ```json
   {
     "body": {
       "conversationId": "conversation_123",
       "feedbackText": "The agent stated a 30-day return window, but the applicable policy says 14 days."
     }
   }
   ```
   For every non-admin key, `conversationId` is mandatory. Do not substitute a helpdesk ticket
   number, subdomain, copied transcript, or guessed ID. If the exact Agent Studio conversation ID
   is unavailable, stop and request it.
2. **Inspect:** read the returned `issueId`, `fixType`, `userMessage`, and `canAutoApply`. Check that
   the explanation matches the conversation, then follow the KB or prompt remediation process above.
   Use `listFeedbackIssues` for the assigned agent to inspect the stored issue when needed. Never
   treat the proposed diff as live truth or approve automatically in the same step as submission.
3. **Approve:** call `approveFeedbackFix` only when its still-pending proposal remains the exact,
   verified fix you intend to apply:
   ```json
   { "id": "feedback-issue-uuid" }
   ```
   Report only the returned outcome. If the issue is escalated, stale, already claimed, outside
   scope, or no longer pending, report the rejection and do not create a replacement unless asked.

## Building webhook tools

A webhook tool lets the bot call an HTTP endpoint you control (a Make or Zapier scenario, or an
Adelante-hosted endpoint) mid-conversation: look up an order, create a lead, book a slot. Use this
section only when the user explicitly asked for a new or changed tool on a named agent.

### When to build one

Build a tool when the bot needs live data or must perform an action that the prompt and KB cannot
supply. Do not build one to hold static facts (put them in the KB), to change tone or policy (that
is the prompt), or to copy a shared tool the agent already has. One integration is one tool: use an
`action` enum for its operations instead of several near-identical tools.

### The allowlist (read this first)

- Webhook hosts are governed by the agent's `allowed_domains` (the same list that controls where
  the chat widget may be embedded and which sites web search uses). Read it with `getAgent`. The
  tool URL must be `https` and its host must equal or be a subdomain of an `allowed_domains` entry
  or of the platform's global webhook allowlist (getadelante.com, make.com, zapier.com), so the
  customer's own servers work once their domain is in `allowed_domains`. The host must resolve to
  a public address, and removing the entry later stops every tool on that host.
- **No matching entry = you cannot create a tool or change a URL to that host.** Add the exact
  webhook host (for example `hook.eu2.make.com`) with `replaceAgentAllowedDomains` once the user
  confirms it: send the complete list read from `getAgent` as `expectedDomains` and the complete new
  list (every existing entry plus the host) as `newDomains`. A `409` means the list changed: re-read
  and rebase. Never try another agent, another URL form, or a redirecting URL to get around it.
- Hosts should be narrow. Make and Zapier hosts are shared by every Make/Zapier customer, so add
  the specific regional host your account uses, never all of `make.com`. Anything added there
  also becomes a valid widget-embedding and web-search domain for the agent.
- Never drop existing entries unless the user asks: removing a domain also stops the widget from
  loading on that site. `updateAgent` is not available to analyst keys; use only
  `replaceAgentAllowedDomains`.

### Which tools you can manage

You can manage only tools created for this agent with `createAgentWebhookTool`, and only while they
are `agent_specific`, on-demand webhook tools assigned to no other agent. Tools an admin created are
out of scope even when this agent uses them: these operations return `404` for them, and changes to
them go through an Adelante admin. Shared/general tools, workflow tools, pre-conversation tools, and
tools attached to another agent are refused on every write (`403`). A tool you created stays
manageable after you detach it. There is no delete: detach instead.

### Lifecycle

1. **Design.** Write down the actions, the parameters each needs, what the webhook returns, and what
   the bot must say for success, "not found", and failure. Confirm the receiving scenario exists and
   answers with JSON.
2. **Check the allowlist.** `getAgent` → `allowed_domains` contains your host (or a parent of it). If not, add it with `replaceAgentAllowedDomains` (with the user's confirmation) and re-read.
3. **Create.** `createAgentWebhookTool`. The tool is created **inactive** and already attached, so
   the bot cannot call it yet.
4. **Verify.** `getAgentWebhookTool`: check URL, method, header names (values are never shown), `parameters_schema`, `action_param`, `action_descriptions`, `is_active: false`,
   `attached: true`. Test the scenario itself with a synthetic payload shaped like the example
   below, with `isTest: true`.
5. **Activate.** `updateAgentWebhookTool` with `{ "is_active": true }` only after the user approves.
   If the prompt must tell the bot when to use the tool, make that change through the prompt
   remediation process, not inside this step.
6. **Monitor.** Read the next real conversations that call the tool (`getConversation` →
   `toolUses`): check arguments, results, and what the bot told the customer.
7. **Deactivate on problems.** `updateAgentWebhookTool` with `{ "is_active": false }` stops all
   calls immediately. Fix, re-verify, then reactivate. Detach only when the tool should leave the
   agent entirely.

### Naming and descriptions

- `name` is global across all Agent Studio tools, lowercase letters, digits and underscores only,
  and cannot be changed later. Prefix it with the agent slug (for example `acme_orders`). A `409`
  means the name is taken.
- `display_name` is for humans; `description` is for the model. Say what the tool does, when to call
  it, when **not** to call it, and which details to collect from the customer first. Keep policy
  and wording rules in the prompt, not in the description.

### Parameter schema

- `parameters_schema` is JSON Schema: `{ "type": "object", "properties": { ... }, "required": [...] }`.
  Give every property a `type` and a `description` with a concrete format example.
- Put truly mandatory fields in `required`. Agent Studio refuses a call with a missing or empty
  required parameter before anything is sent, and asks the model to retry.
- Use `enum` for closed sets.
- Do not add parameters for context the platform already sends (conversation ID, phone, channel;
  see below).
- `identifier_type` and `identifier_value` are reserved names: when `identifier_type` is `phone`,
  the value is normalized to E.164 or the call is refused.

### One tool with an `action` enum

For several operations on one system, add a required `action` string property with an `enum`, set
`action_param: "action"`, and give `action_descriptions` exactly one non-empty description per enum
value. The model sees those descriptions; admins can enable a subset of actions per agent. Changing
the enum later requires `parameters_schema`, `action_param`, and `action_descriptions` in the same
update.

### What the webhook receives

Headers are always `Content-Type: application/json` plus your `webhook_headers`. Use `POST`: with
`GET` no body is sent and there is no query string, so the webhook receives no parameters or
context at all.

The JSON body is built in this order, later keys winning on a name collision:

1. Context fields (sent on every live, playground, and eval call):
   - `conversationId` (string): the helpdesk conversation ID.
   - `appId` (string): the agent's helpdesk app ID; `""` when not configured.
   - `channel` (string): the conversation channel, for example `whatsapp` or `web`; can be `""`.
   - `phone` (string): the customer phone when the channel knows it; on WhatsApp it is E.164
     (`+972...`). Omitted when unknown.
   - `isTest` (boolean): always present; `true` for playground and eval runs. Any tool that changes
     something (cancel, refund, dispatch, create a lead) must do nothing real when it is `true`.
   - `integrationId` (string): the agent's integration ID; `""` when not configured.
   - `currentMessageId` and `current_message_id` (string): the customer message that triggered the
     turn; only when the channel provides one.
   - `metadata` (object): `agentId`, `agentSlug`, `latestUserMessage` (omitted for zero-data-retention
     agents), plus channel-specific session fields that vary by helpdesk. Do not depend on
     undocumented keys.
   - `allowedDomains` (array of strings): the agent's website allowlist used for the web widget and
     web search. It is not the webhook allowlist.
   - `agentName` (string): the agent's display name.
2. `sourceChannel` (string): the same value as `channel`, only when non-empty.
3. The model's arguments, except values that are `null` or `""`. A non-empty argument overrides a
   context field with the same name (for example a `phone` parameter the customer typed wins over
   the channel phone); an empty one leaves the context value in place.

Example body for `{ "action": "get_status", "order_number": "10423" }` on WhatsApp:

```json
{
  "conversationId": "65f1c0ffee0000000000abcd",
  "appId": "5f0a1b2c3d4e5f6a7b8c9d0e",
  "channel": "whatsapp",
  "phone": "+972501234567",
  "isTest": false,
  "integrationId": "64aa00000000000000000001",
  "currentMessageId": "65f1c0ffee0000000000beef",
  "current_message_id": "65f1c0ffee0000000000beef",
  "metadata": {
    "agentId": "0b6c1d7e-1111-4222-8333-944455556666",
    "agentSlug": "acme",
    "latestUserMessage": "Where is order 10423?"
  },
  "allowedDomains": ["acme.example"],
  "agentName": "Acme Support",
  "sourceChannel": "whatsapp",
  "action": "get_status",
  "order_number": "10423"
}
```

### What the webhook must return

- Answer within **90 seconds** with a 2xx status and a JSON body (`Content-Type: application/json`)
  of at most **1 MB**. The model reads the body, so return short, explicit fields such as
  `{ "success": true, "status": "shipped", "tracking_url": "..." }` or
  `{ "success": false, "reason": "order_not_found" }`.
- Non-2xx responses, timeouts, oversized bodies, and redirects (redirects are never followed) reach
  the model as a failed call.
- Make: the scenario must end with a **Webhook response** module. A switched-off or queued scenario
  answers the plain text `Accepted`; Agent Studio treats that as a failure. Plain-text bodies are
  passed to the model as `{ "result": "<text>" }`.

### Secrets

- Put API keys in `webhook_headers` (for example `x-api-key` or `Authorization`). `Authorization`,
  `X-API-Key`, and any header named in `sensitive_headers` are encrypted at rest. Responses list
  header names only, never values. Never put secrets in the URL, the description, or KB.
- `webhook_headers` in an update replaces the whole header set. Send `__UNCHANGED__` as the value of
  a stored secret header to keep it, and resend every non-secret header with its value.
  `sensitive_headers` can only be sent together with
  `webhook_headers`.
- Changing `webhook_url` drops every stored secret header, so send fresh secret values with the new
  URL.

### Duplicate calls and idempotency

Agent Studio runs an identical call (same tool, same arguments, same conversation) only once within
at least two minutes and replays the first result to repeats. Your webhook must still be
idempotent: a timed-out call may have completed remotely, and the customer can repeat the request
later. Deduplicate on a business key (order number plus action, or `currentMessageId`) before
creating anything.

`on_error` and `timeout_ms` only affect pre-conversation tools; on-demand tools always use the
90-second limit.

### Common errors

| Status | Meaning | What to do |
|---|---|---|
| `400` | Invalid body: bad name, missing field, forbidden field (`workflow_steps`, `webhook_authorization`, `tags`, `name` in an update, a non-webhook mode, `pre_conversation`, `general` scope), action enum and descriptions out of sync, or `__UNCHANGED__` for a header that is not stored as a secret | Fix the request; do not retry unchanged |
| `403` | URL host in neither the agent's `allowed_domains` nor the global webhook allowlist, tool is shared/general or assigned to another agent, or your key lacks the `tools` component or analyst role | Stop. Add the host with `replaceAgentAllowedDomains`, or pick a different tool; never work around it |
| `404` | Agent outside your scope, or the tool was not created for this agent with `createAgentWebhookTool` (admin-created tools) | Check the slug and tool ID; do not probe |
| `409` | Tool name already exists | Choose a different, agent-prefixed name |

There is no `422` on these operations; validation problems return `400`.

## Reading a conversation payload

- `messages[]` — the transcript. `role` is `user` | `assistant` | `agent` (human agent);
  `source` (`ai` / `human_agent` / `system`) is the reliable who-sent-it label for analytics.
- A transcript can contain several consecutive customer messages before an AI reply. Earlier
  messages superseded before any AI or human-agent reply are returned with `isStale: true` and
  `staleReason: "superseded_by_later_customer_message"`. Treat only the last non-stale customer
  message in that group as the current message that triggered the reply. Stale messages remain
  useful history/context; do not evaluate the same AI reply as a separate response to each of them.
- Assistant messages may carry `thinking` (the model's internal reasoning — treat as diagnostic
  signal, never as customer-visible content) and `toolCalls` (name, args, result).
- `toolUses[]` — session-level chronological tool call log. Wrong answers usually start here:
  check whether the tool returned bad data or the agent misread good data.
- `metadata` — channel/session context.

## Ground rules for analysis output

- Cite evidence: session IDs, message indexes, exact quotes for every claim.
- Separate facts (what happened) from hypotheses (why) and label them.
- Keep routine checks silent. Don't narrate lookups or announce that you are checking whether your
  permissions allow something; do the work, then report the result, or a concrete blocker with
  the call and error that produced it, in plain team language.
- When you recommend a fix, say where it belongs: system prompt, a specific tool's description,
  a specific KB chunk/topic, or platform configuration.
- Never claim that submission changed production. Production changes only after a successful
  `approveFeedbackFix`, `replaceSnippetContent`, `addSnippet`, `createDocument`, `patchAgentPrompt`, `replaceAgentPrompt`, `replaceAgentAllowedDomains`, or agent webhook tool call
  followed by live read-back verification.
