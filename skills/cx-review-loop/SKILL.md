---
name: cx-review-loop
description: Run the recurring improvement loop for an assigned Agent Studio support agent in the team's chat - a weekly review of the bot's handovers and a daily follow-up on open feedback. The weekly report shows only recommendations supported by at least 10% of the agent's handover conversations for the week. Propose each change, apply it only after someone in the team chat approves, then confirm it landed. Use for a scheduled weekly review, a scheduled daily feedback follow-up, when the team asks for a summary of what the bot hands over, or when the team replies to one of those messages.
---

# CX review loop

Two routines for the support agent you are assigned to, both ending in the same
propose, approve, apply steps:

- **Weekly handover review**: measure last week's automation against the weeks
  before, read every handover and what the human agents did next, and recommend
  the changes that would have let the bot handle them. Only recommendations
  supported by at least 10% of the week's handover conversations reach the team.
- **Daily feedback follow-up**: ask the team what to do with feedback items that
  are still open.

You propose. The team decides in the chat. You apply only what they approved.
Reviewing or recommending a change does not authorize applying or dismissing it.

## Before you start

- Use the installed `adelante-agent-studio-analyst` skill for the Agent Studio MCP
  tools and for how to read conversations, prompts and knowledge. Discover the
  live tool list first. This skill uses `getConversationAnalytics`,
  `listHandovers`, `getConversation`, `getOperatorThread`, `getAgentPrompt`,
  `listSnippets`, `getTool`, `listFeedbackIssues`, `submitFeedback`,
  `approveFeedbackFix` and `dismissFeedbackIssue`. If one is missing or returns
  403, say which one and run the rest read-only. Use only the scoped connection
  you were given.
- Write to the team in their language, in the chat where this routine runs.
- Read the review log (see "Review log") before anything else, so a reply days
  later still maps to the right item and decisions already made are not asked again.
- When you review more than one agent, run every step below separately for each
  agent. Handover sets, thresholds and recommendations are never pooled across agents.

## Weekly handover review

### 1. Measure

1. Call `getConversationAnalytics` for the agent over the last 7 complete days,
   and once over the 28 days before them. Report the week's conversations, bot
   handovers and automation rate next to the 4-week rate. When the agent runs on
   more than one channel, give the rate per channel from `byChannel`; channels
   behave very differently. If the analytics tool is unavailable, report counts
   without a rate and say why.
2. Note anything that changed in the period: changes applied from the review log,
   and model or prompt changes you can see. A rate moves for many reasons; name the
   ones you know and do not credit or blame a change from one week of numbers.

The analytics count conversations by their start date and count every
conversation where a handover tool ran. `listHandovers` matches the handover
call's time. Expect small differences between the two. The threshold below always
uses the `listHandovers` set, never the analytics count.

### 2. Collect the full handover set

1. Call `listHandovers` for the agent and the same 7 days. Page with
   `limit`/`offset` until you have every row that `pagination.total` reports.
   Do not stop at a cap and do not sample.
2. The handover set is the distinct `sessionId` values returned. Remove
   duplicates: a conversation returned twice, or handed over more than once,
   counts once.
3. The set is **complete** only when every page was fetched without an error and
   the number of distinct IDs matches what the listing reports. If a page failed,
   the listing was cut short, or the totals disagree, the set is **incomplete**:
   say so, and do not claim that any recommendation met a verified threshold.
4. Every distinct ID returned stays in the set, including conversations that were
   attempted but not handed over, contacts that are not customers, and
   conversations whose transcript you could not open or could not classify. The
   size of this set is the denominator for the week.

### 3. Read every handover

1. Open the transcript (`getConversation`) when `handoverCompleted` is false or
   `handoverCount` is above 1. A conversation with no completed handover call was
   not handed over: count it apart as "attempted, not handed over", and check
   whether the customer was left waiting (that is a bot error).
2. Count apart, and leave out of the findings, contacts that are not customers:
   vendor pitches, newsletters, phishing, automated notices, internal staff
   threads. If the bot replied to one wrongly, list it as a bot error.
3. For every remaining handover read both sides: `getConversation` for what the
   customer asked, what the bot looked up and what it said before transferring,
   and `getOperatorThread` for what the human did next. When the human side is
   unavailable, count it as "no human reply available" and keep it out of the
   conclusions.
4. Keep a list of the conversations you could not read or classify. They stay in
   the denominator, and the report says how many there were.

### 4. Classify each handover on two axes

**Why the bot handed over.** Decide from the transcript. `handoverReasonType` is the
bot's own label and only a hint: "other" and missing labels are common.

- **Rule**: the prompt or knowledge tells the bot to transfer this case. Quote the rule.
- **Customer asked**: the customer asked for a person. If the request followed a
  weak or wrong answer, classify by that cause instead.
- **Missing information**: the bot lacked a fact the customer needed. Name the fact.
- **Missing tool**: the request needs an action or data the bot has no tool for.
- **Tool failure**: a tool errored, or returned wrong or contradictory data.
- **Bot-caused**: the bot offered a transfer the customer did not ask for, hedged,
  or said it could not confirm something, while its knowledge, a product page or a
  tool result had the answer. Cite where the answer was.

**What the human did**, from the operator thread:

- **Reply only**: answered with information the bot had or could have had.
- **One action**: one back-office step, then a reply.
- **Judgment**: several steps, an exception, or back-and-forth.

### 5. Group into findings and decide what removes each

A finding is one underlying issue and one proposed change. Put conversations in
the same finding only when they show the same issue and the same change would
have fixed each of them. Never merge unrelated issues into a broad category
("knowledge gaps", "shipping questions") to reach the threshold; two different
missing facts are two findings. A conversation may support several genuinely
distinct findings, and counts once within each.

For each finding record, privately: the type, every supporting conversation ID,
the cause, what the human typically answered (without customer personal data),
and the exact proposed edit if there is one. Every supporting ID must come from
this agent's handover set for this week.

| Cause | Human did | Finding type and what to propose |
|---|---|---|
| Missing information | Reply only | Fix: a knowledge addition holding the answer the team gave. First search the knowledge for it (`listSnippets`): if the fact is already there, the problem is retrieval or a contradiction, not a missing fact, and the fix corrects that entry. |
| Bot-caused | any | Fix: the smallest wording change to the prompt or knowledge entry involved, written as what the bot should do ("answer the fit question from both product pages"). |
| Two sources disagree | any | Fix: one wording that removes the conflict. Prompt, knowledge entries and tool descriptions (`getTool`) can each disagree with the others, and two knowledge entries can disagree with each other. If the conversations do not show which is correct, it is a policy question instead. |
| Rule | Reply only | Policy question: keep the rule, or let the bot answer with the text the team used? Give your recommendation. |
| Rule | Action or judgment | Nothing to change. Count it. |
| Customer asked, at the start | any | Nothing to change. Count it. |
| Missing tool | any | Technical gap: a tool for Adelante to build, with the decisions it depends on. |
| Tool failure | any | Technical gap: the failing tool and what it returned, for Adelante. |
| Team answers differ from policy | any | Policy question to the team. |
| An open feedback item or earlier recommendation no longer matches what the conversations show | any | Closure: close or dismiss it, with the reason. |

**Verified and unconfirmed causes.** A cause is verified when you read the
instruction, knowledge entry or tool result that produced the behaviour, and the
transcripts show it. Anything else is an unconfirmed explanation: say so in the
private record and in the "Today" line ("the conversations suggest...", "not
confirmed in the instructions"). Do not state an unconfirmed cause as fact.

Before recommending, check `listFeedbackIssues` for an open item on the same
finding, and the review log for a decision already taken. Ask a decided question
again only when its weekly support has clearly grown.

### 6. Apply the 10% threshold

For each agent separately:

```
minimum_support = ceil(total_unique_handover_conversations × 0.10)
```

- `total_unique_handover_conversations` is the size of the complete handover set
  from step 2. Never lower it for missing transcripts, unclassified
  conversations, non-customer contacts or attempted handovers.
- A finding's support is the number of distinct IDs in its
  `supporting_conversation_ids`. Count conversations, not messages, tool calls or
  repeated handover events.
- A finding qualifies when its support is at least `minimum_support`.
- The threshold applies to every recommendation type: fixes, policy questions,
  technical gaps and closures. There is no severity exception in the weekly
  report. An urgent incident (a customer harmed, data exposed, the bot down) goes
  through the separate incident-handling process the moment you see it, not into
  this report.
- A finding below the threshold stays in the private analysis with its evidence.
  It does not appear in the team message, is not filed with `submitFeedback`, and
  gets no approval number. Note it in the review log so a recurring issue can be
  seen across weeks.

When a hosted runtime validates the report, it verifies the handover set,
checks every supporting ID, calculates the counts and enforces the cutoff. Give
it accurate evidence and every finding, qualifying or not. Do not pre-filter,
split, merge or reword findings to get one past its validation. If it rejects or
drops a finding, that finding stays private. Show the counts and percentages it
returns, not your own.

### 7. Check last week's applied changes

For each change applied since the previous review, compare the finding's count in
the 7 days before the change with the days since. Read one or two conversations
of that finding since the change and check the bot handled them and its answer was
right. Report one line per change: held, partly, not seen yet, or worse.

### 8. File and report

1. Build the private structured report (see "Private structured report") with
   every finding.
2. For each qualifying recommendation, assign the next approval number from the
   review log and file it (see "Propose, approve, apply").
3. Send one team message (see "Weekly message").

A handover in `listHandovers` is a conversation where the bot called its handover
tool. Human takeovers and system-triggered handovers are not in it, so the count
can be lower than the helpdesk's. Say "bot handovers", and give every count with
its week.

## Daily feedback follow-up

1. Call `listFeedbackIssues` for the agent with status pending, escalated and
   failed. Start with the ones created yesterday.
2. If nothing is open, send nothing.
3. Otherwise send one message with up to six items, newest first, and say how
   many more are waiting. For each: what the feedback was about, the exact change
   proposed, and "approve, dismiss, or change?". For an escalated item say plainly
   what is blocking it.

The weekly threshold and weekly message format do not apply here.

## Propose, approve, apply

**Propose**

- A qualifying weekly fix is filed with `submitFeedback` on one representative
  conversation from its supporting IDs (`source: Handover Analysis`). Put the
  approval number, the finding, its support for the week and the supporting
  conversation IDs in the feedback text. Then read the created item back: the fix
  it holds is what would be applied, and it must match the exact edit in the
  private report.
- The exact text that will change (current wording and new wording, or the new
  knowledge entry) is kept in the private report under the approval number. The
  weekly message describes it in plain words; if someone in the chat asks to see
  the exact wording, reply with it. Daily follow-ups show the exact change, as
  above.
- Keep each change small and write it as what the bot should do. Prefer adding a
  fact to the knowledge over adding a rule to the prompt. One change per finding.
- Approval numbers are permanent: continue from the highest number in the review
  log, never reuse one, and use the same number in the message, the feedback text,
  the private report and the log.
- Technical gaps and policy questions are questions, not changes. With each give
  your recommendation and the reason for it, so the team can answer with a yes.
- A technical gap is built or fixed by Adelante. Record the request and its
  supporting IDs in the review log, and tell the team Adelante will confirm when
  it is ready.

**Approve**

- Anyone in the team chat can approve, dismiss or change a proposal. Act on a
  reply that clearly refers to a numbered item ("yes to 2", "dismiss 4", or a
  reply to that message).
- If a reply is unclear or two people disagree, ask one short question before
  doing anything.
- An approval covers the exact edit stored under that number. If the live prompt
  or knowledge changed since, or the fix in the feedback item differs from the
  stored edit, describe the current change again and ask again.

**Apply**

1. Yes: call `approveFeedbackFix` for that item. If it reports the text no longer
   matches, re-run the proposal and ask again.
2. Read the live prompt or snippet back and check the new text is there.
3. Reply in the chat with what changed, and note who approved and when in the
   review log.
4. No: call `dismissFeedbackIssue` with the reason they gave.
5. "Change it to...": submit a new proposal with their wording, show its exact
   text, and ask again.
6. An item you cannot apply with your tools stays open. Say that it needs
   Adelante and what exactly is needed.

A closure recommendation is applied the same way: dismiss or close only after an
approval of that number.

## Private structured report

One record per weekly run, kept with the review log and passed to the runtime
when it validates reports. Use the runtime's schema when it provides one; it
must carry at least:

```
period: start and end dates
agents:
  - agent: slug
    handover_listing_complete: true | false
    handover_conversation_ids: [distinct sessionIds from listHandovers]
    unread_conversation_ids: [in the set, transcript or human side unavailable, or unclassified]
    findings:
      - approval_number: number, or none when below the threshold
        type: fix | policy_question | technical_gap | closure
        title: short name
        today: current procedure or limitation, marked verified or unconfirmed
        recommendation: proposed behaviour change and expected result
        decision: what the team must approve or answer
        evidence: instruction or knowledge quoted, what the transcripts show
        proposed_edit: exact current and new text, when the type is fix
        supporting_conversation_ids: [distinct ids from this agent's handover set]
```

Keep customer personal data out of it; use session IDs. It is never sent to the
team chat.

## Review log

Keep one file, `cx-review-log.md`, in your persistent workspace. One line per
item: approval number, date, feedback issue ID, short title, finding, state
(proposed, approved by whom and when, applied on which date, dismissed with
reason, waiting on Adelante, waiting on team). Record team decisions on rules and
policy questions too ("keep the transfer rule for X, decided by whom, when"). Add
one line per week with the conversations, bot handovers, distinct handover
conversations, minimum support, whether the handover listing was complete, and
the automation rate, plus one line per below-threshold finding with its support.
Read it at the start of every run and before acting on a reply. It is the source
for "what is still waiting", for the applied-change check and for not asking a
decided question again. Keep customer personal data out of it; use session IDs.

## Weekly message

- One message per agent per week. Short lines, numbered items, no tables.
- First line: "Last week: N bot handovers in M conversations, X% automated
  (4-week average Y%)."
- Second line: the threshold, e.g. "Recommendations below are each supported by
  at least 7 of the 64 conversations handed over last week (10%)."
- The applied-change check, one line per change.
- Then every qualifying recommendation, numbered with its approval number. No
  item cap and no "ask for more". Each one:
  1. Support, as the runtime calculated it, in the team's language, e.g.
     "7 מתוך 64 פניות שהועברו לצוות — 10.9%" or "7 of 64 handed-over
     conversations — 10.9%".
  2. **Today:** the current procedure or limitation that explains what the bot
     did. Say when it is not confirmed.
  3. **Recommendation:** the behaviour change and the expected result.
  4. **Decision:** the approval or the specific policy answer needed.
- Plain, concise language for the team. No source snippets, no prompt or trigger
  directives, no old/new text blocks, no session IDs, feedback IDs or other
  internal IDs.
- No recommendation qualified: a short message with the rate line and "No
  recurring issue reached 10% of last week's handed-over conversations." Do not
  list smaller findings instead.
- Say when evidence is incomplete: how many conversations in the set could not be
  read or classified, human replies unavailable, analytics unavailable, how many
  contacts were not customers.
- Handover listing incomplete: say the full handover set could not be collected,
  give the rows you have, and do not present any recommendation as meeting a
  verified 10% threshold.

## Checks before sending the weekly message

- Every page of `listHandovers` was fetched, or the message says the listing is
  incomplete.
- The denominator is the number of distinct handover conversation IDs, with no
  conversation removed.
- `minimum_support` was calculated per agent with a ceiling, not rounding.
- Each supporting ID list has no duplicates and every ID is in that agent's
  handover set for the week.
- Each shown recommendation has support at least `minimum_support`. Nothing below
  it is shown, filed or numbered.
- Each finding is one issue and one change, not a merged category.
- The message contains no internal IDs, snippets, directives or old/new text.
- Every finding, qualifying or not, is in the private structured report.

## Examples

- **50 handover conversations**: minimum support 5. A finding with 5
  conversations (10.0%) is shown; one with 4 (8.0%) stays private.
- **64 handover conversations**: minimum support 7 (6.4 rounds up). 7 conversations
  (10.9%) is shown; 6 (9.4%) is not.
- **51 handover conversations**: minimum support 6 (5.1 rounds up). 5 conversations
  (9.8%) is not shown.
- **8 handover conversations**: minimum support 1. Every finding with a
  supporting conversation qualifies; say the week was small.
- **Duplicate IDs**: `listHandovers` returns 66 rows with 2 repeated session IDs.
  The set has 64 conversations and minimum support is 7. A finding whose evidence
  lists one conversation three times counts it once.
- **Unread transcripts**: 64 conversations, 5 transcripts unavailable. The
  denominator stays 64 and minimum support stays 7; the message says 5
  conversations could not be read.
- **Multiple agents**: agent A has 64 conversations (minimum 7), agent B has 12
  (minimum 2). A finding with 4 conversations from A and 2 from B is two
  findings: A's 4 stays private, B's 2 is shown in B's message. The two sets are
  not combined into 76.
- **Incomplete listing**: page 3 of `listHandovers` fails after retries. The set
  is incomplete, so no recommendation is presented as passing a verified
  threshold; the message says the handover list could not be fully collected.
- **No qualifying recommendation**: 40 conversations (minimum 4), largest finding
  has 3. Send only the rate line, the applied-change check and "No recurring
  issue reached 10% of last week's handed-over conversations."
- **One conversation, two findings**: a conversation shows both a missing
  delivery-time fact and a tool error. It counts toward both findings, once each.
- **Broad grouping refused**: 3 conversations missing a return-window fact and 4
  missing a sizing fact (64 total, minimum 7) are two findings of 3 and 4, both
  private, not one "missing knowledge" finding of 7.

## Scheduling in a hosted runtime

When the administrator asks for it and the runtime has a scheduler:

- Daily feedback follow-up: weekday mornings in the team's timezone.
- Weekly handover review: one fixed weekday morning.
- Each job loads `adelante-agent-studio-analyst` and `cx-review-loop`, delivers to
  the team chat it was created in, and keeps continuity with the chat session.
- Check for an existing job before creating one. A scheduled run only proposes
  and reports; applying happens when someone replies in the chat.
