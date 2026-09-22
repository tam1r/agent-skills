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

- **Investigate one case:** “Investigate ticket 96728. Reconstruct what happened, inspect the prompt,
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
  `replaceAgentPrompt` and `replaceSnippetContent` remediation operations, and can build webhook
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
| `listConversations` | Conversation list for an agent, newest first (`limit`/`offset`; test sessions excluded unless `include_test=true`) |
| `getConversation` | Full transcript: messages (with `thinking` on AI messages when enabled), `toolUses`, metadata |
| `resolveTicketConversation` | Helpdesk ticket number (e.g. Zendesk `96728`) → its conversation |
| `listAgentTools` / `listTools` / `getTool` | Bound tool IDs, names, and parameter schemas; descriptions only for `agent_specific` tools |
| `listAgentKnowledgeBases` / `listKnowledgeBases` / `getKnowledgeBase` | Knowledge bases linked to the agent |
| `listDocuments` / `getDocument` / `listChunks` / `listSnippets` | KB content the agent answers from |
| `getAgentRoutingIndex` | The topic index the agent uses to pick KB chunks |
| `listFeedbackIssues` | Feedback issues filed against the agent (`agentSlug` required; filter by `status`, `source`, `startDate`/`endDate`) |
| `getAttributedRevenue` | Revenue attributed to the agent's conversations (`agentSlug` required; `days` or `startDate`/`endDate`) |
| `submitFeedback` | Analyst only: creates and analyzes feedback for a scoped conversation |
| `approveFeedbackFix` | Analyst only: applies an inspected, eligible pending fix for a scoped issue |
| `dismissFeedbackIssue` | Analyst only: dismisses a scoped pending/escalated issue with a concrete reason |
| `replaceSnippetContent` | Analyst only: guarded replacement of one scoped internal snippet; URL snippets return their source URL |
| `replaceAgentPrompt` | Analyst only: guarded replacement of one scoped agent's complete prompt |
| `listAgentWebhookTools` / `getAgentWebhookTool` | Webhook tools you can manage for the agent: URL, method, header names (never values), schema, action copy, `is_active`, `attached`. Viewer keys get only `id`, `name`, `parameters_schema`, description |
| `createAgentWebhookTool` | Analyst only: creates an inactive webhook tool attached to the agent; URL host must be in the agent's `allowed_domains` |
| `updateAgentWebhookTool` | Analyst only: edits a manageable webhook tool; `is_active: true/false` activates or deactivates it |
| `attachAgentWebhookTool` / `detachAgentWebhookTool` | Analyst only: attaches or detaches a manageable webhook tool; never deletes it |

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
1. Ticket number → `resolveTicketConversation`. Session/conversation ID → `getConversation`.
2. Read the transcript in order. For each AI message, check `thinking` (why it decided what it
   did) and its `toolCalls`/`toolUses` (what data it actually had).
3. If the answer looks wrong, check the sources: `getAgent` (system prompt rules), the tool
   result it relied on, and the KB chunk it likely used (`listChunks`, `getAgentRoutingIndex`).
4. Verdict format: what the customer wanted → what the agent did → root cause (prompt rule /
   tool output / KB gap / model behavior) → recommended fix.

**Aggregate analysis** (handover rate, common intents, failure patterns):
1. `listConversations` over the period (test sessions are already excluded by default).
2. Classify each conversation from its transcript and `toolUses` (e.g. a handover tool call =
   escalated; no reply needed = resolved). State your classification rules in the output.
3. Transcripts are large — fetch details one at a time, and if the period has hundreds of
   conversations, analyze a sample and say so (e.g. "50 most recent of 412").
4. Report counts **and** representative examples (session IDs) so findings are verifiable.

**Feedback triage**: `listFeedbackIssues` with `status=pending` (statuses: analyzing, pending,
applying, applied, escalated, failed, dismissed). Cluster by theme, link each issue to its
conversation via `conversation_id`/`ticket_id`, and flag recurring root causes.

**ROI / value report**: `getAttributedRevenue` for the period + conversation volume from
`listConversations` pagination `total`. Present revenue alongside resolution/handover stats.

**Agent configuration review**:
1. `getAgent` for the system prompt and settings; `listAgentTools` for tool descriptions;
   `listAgentKnowledgeBases` + `listChunks` for content.
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
   procedure, or routing trigger. Use a prompt change only for behavior that truly applies across
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
6. **Verify live read-back.** Fetch the edited snippet or feedback-created override and confirm the
   new rule appears exactly once, obsolete or conflicting wording is gone, and unrelated content
   remains intact. After a URL source update, find the regenerated snippet by document and content
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
5. **Choose and guard one mutation path.** Fetch the prompt again immediately before writing. If the
   pending proposal still exactly matches the live prompt and independently verified intended fix,
   call `approveFeedbackFix`. Otherwise preserve the entire live prompt, apply an exact, unique
   anchor replacement in memory, and call `replaceAgentPrompt` with the complete freshly read prompt
   as `expectedPrompt` and the complete updated prompt as `newPrompt`. Abort and reassess if the
   anchor is absent, duplicated, or changed. Never write stale feedback `oldContent` or `newContent`
   as the agent prompt. If either guard rejects the write, re-read and rebase the intended change.
6. **Read back and verify.** Fetch the agent again. Confirm the new section appears exactly once, old
   conflicting wording is absent, unrelated sections remain intact, formatting and length were not
   corrupted, and assigned tools, model, and other agent settings are unchanged.
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
  tool URL must be `https`, its host must equal an entry or be a subdomain of one, and it must also
  be inside the platform's global webhook allowlist (getadelante.com, make.com, zapier.com).
  Customer website entries such as `acme.example` never permit a webhook.
- **No matching entry = you cannot create a tool or change a URL to that host.** Stop and ask the
  user to have an Adelante admin add the exact webhook host (for example `hook.eu2.make.com`) to the
  agent's allowed domains. Never try another agent, another URL form, or a redirecting URL to get
  around it.
- Hosts should be narrow. Make and Zapier hosts are shared by every Make/Zapier customer, so admins
  add the specific regional host your account uses, not all of `make.com`. Anything added there
  also becomes a valid widget-embedding and web-search domain for the agent.
- You can read `allowed_domains` but never change it. `updateAgent` is not available to analyst
  keys.

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
2. **Check the allowlist.** `getAgent` → `allowed_domains` contains your host (or a parent of it). If not, stop.
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
| `403` | URL host not in the agent's `allowed_domains` or outside the global webhook allowlist, tool is shared/general or assigned to another agent, or your key lacks the `tools` component or analyst role | Stop. Ask for an admin allowlist change or a different tool; never work around it |
| `404` | Agent outside your scope, or the tool was not created for this agent with `createAgentWebhookTool` (admin-created tools) | Check the slug and tool ID; do not probe |
| `409` | Tool name already exists | Choose a different, agent-prefixed name |

There is no `422` on these operations; validation problems return `400`.

## Reading a conversation payload

- `messages[]` — the transcript. `role` is `user` | `assistant` | `agent` (human agent);
  `source` (`ai` / `human_agent` / `system`) is the reliable who-sent-it label for analytics.
- Assistant messages may carry `thinking` (the model's internal reasoning — treat as diagnostic
  signal, never as customer-visible content) and `toolCalls` (name, args, result).
- `toolUses[]` — session-level chronological tool call log. Wrong answers usually start here:
  check whether the tool returned bad data or the agent misread good data.
- `metadata` — channel/session context.

## Ground rules for analysis output

- Cite evidence: session IDs, message indexes, exact quotes for every claim.
- Separate facts (what happened) from hypotheses (why) and label them.
- When you recommend a fix, say where it belongs: system prompt, a specific tool's description,
  a specific KB chunk/topic, or platform configuration.
- Never claim that submission changed production. Production changes only after a successful
  `approveFeedbackFix`, `replaceSnippetContent`, `replaceAgentPrompt`, or agent webhook tool call
  followed by live read-back verification.
