---
name: cx-review-loop
description: Run the recurring improvement loop for an assigned Agent Studio support agent in the team's chat - a weekly review of the bot's handovers and a daily follow-up on open feedback. Propose each change with its exact text, apply it only after someone in the team chat approves, then confirm it landed. Use for a scheduled weekly review, a scheduled daily feedback follow-up, when the team asks for a summary of what the bot hands over, or when the team replies to one of those messages.
---

# CX review loop

Two routines for the support agent you are assigned to, both ending in the same
propose, approve, apply steps:

- **Weekly handover review**: measure last week's automation against the weeks
  before, read every handover and what the human agents did next, and propose the
  changes that would have let the bot handle them.
- **Daily feedback follow-up**: ask the team what to do with feedback items that
  are still open.

You propose. The team decides in the chat. You apply only what they approved.

## Before you start

- Use the installed `adelante-agent-studio-analyst` skill for the Agent Studio MCP
  tools and for how to read conversations, prompts and knowledge. Discover the
  live tool list first. This skill uses `getConversationAnalytics`, `getAgentStats`,
  `listHandovers`, `getConversation`, `getOperatorThread`, `getAgentPrompt`,
  `listSnippets`, `getTool`, `listFeedbackIssues`, `submitFeedback`,
  `approveFeedbackFix` and `dismissFeedbackIssue`. If one is missing or returns
  403, say which one and run the rest read-only. Use only the scoped connection
  you were given.
- Write to the team in their language, in the chat where this routine runs.
- Read the review log (see "Review log") before anything else, so a reply days
  later still maps to the right item and decisions already made are not asked again.

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
call's time. Expect small differences between the two and say which number
you quote.

### 2. Read every handover

1. Call `listHandovers` for the same 7 days. Page until you have every row. If
   there are more than 150, take the newest 150 and say so.
2. Open the transcript (`getConversation`) when `handoverCompleted` is false or
   `handoverCount` is above 1. A conversation with no completed handover call was
   not handed over: count it apart as "attempted, not handed over", and check
   whether the customer was left waiting (that is a bot error).
3. Count apart, and leave out of the themes, contacts that are not customers:
   vendor pitches, newsletters, phishing, automated notices, internal staff
   threads. If the bot replied to one wrongly, list it as a bot error.
4. For every remaining handover read both sides: `getConversation` for what the
   customer asked, what the bot looked up and what it said before transferring,
   and `getOperatorThread` for what the human did next. When the human side is
   unavailable, count it as "no human reply available" and keep it out of the
   conclusions.

### 3. Classify each handover on two axes

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

### 4. Group into themes and decide what removes each

Group by cause and topic. For each theme record the week's count, two or three
session IDs, and what the human typically answered, without customer personal data.

| Cause | Human did | What to propose |
|---|---|---|
| Missing information | Reply only | A knowledge addition holding the answer the team gave. First search the knowledge for it (`listSnippets`): if the fact is already there, the problem is retrieval or a contradiction, not a missing fact, and the proposal fixes that entry. |
| Bot-caused | any | The smallest wording change to the prompt or knowledge entry involved, written as what the bot should do ("answer the fit question from both product pages"). |
| Two sources disagree | any | One wording that removes the conflict, quoting both. Prompt, knowledge entries and tool descriptions (`getTool`) can each disagree with the others, and two knowledge entries can disagree with each other. Ask which is correct if the conversations do not show it. |
| Rule | Reply only | A question: keep the rule, or let the bot answer with the text the team used? Give your recommendation. |
| Rule | Action or judgment | Nothing to change. Count it. |
| Customer asked, at the start | any | Nothing to change. Count it. |
| Missing tool | any | A tool-gap question for Adelante, with the decisions it depends on. |
| Tool failure | any | List the session IDs for Adelante. The team does not need to decide anything. |
| Team answers differ from policy | any | A question to the team. |

Before proposing, check `listFeedbackIssues` for an open item on the same theme,
and the review log for a decision already taken. Ask a decided question again only
when its weekly count has clearly grown.

### 5. Check last week's applied changes

For each change applied since the previous review, compare the theme's count in
the 7 days before the change with the days since. Read one or two conversations
of that theme since the change and check the bot handled them and its answer was
right. Report one line per change: held, partly, not seen yet, or worse.

### 6. Send one message

See "Message format": the rate line, the health lines, the top themes with counts, the
applied-change check, then the numbered proposals and questions.

The health lines, each against the previous 4 weeks:

- Satisfaction (CSAT where collected) and time from handover to the first human reply,
  from the operator threads you read.
- Any contact reason or handover theme that rose by more than half, with its count.
- Incidents: tool failures (`getAgentStats`), outages, carrier or website problems seen
  in conversations.
- One thing to watch before next week.

If nothing moved, say so in one line. Record the health lines in the review log; the
monthly director review reads them.

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

## Propose, approve, apply

**Propose**

- A proposal from the weekly review is filed with `submitFeedback` on one
  representative conversation (`source: Handover Analysis`). Put the theme, the
  weekly count and the example session IDs in the feedback text. Then read the
  created item back: the fix it holds is what would be applied.
- Always show the exact text that will change: the current wording and the new
  wording, or the new knowledge entry. Never ask for approval of a summary.
- Keep each change small and write it as what the bot should do. Prefer adding a
  fact to the knowledge over adding a rule to the prompt. One change per theme.
- Number every proposal and question in the message and write the same numbers
  in the review log.
- Tool gaps and team decisions are questions, not changes. Ask them in the same
  message and record the answer. With each question give your recommendation and
  the reason for it, so the team can answer with a yes.
- A tool gap is built by Adelante. Ask for it in the chat with the decisions it
  depends on, record the request in the review log, and tell the team Adelante
  will confirm when it is ready.

**Approve**

- Anyone in the team chat can approve, dismiss or change a proposal. Act on a
  reply that clearly refers to a numbered item ("yes to 2", "dismiss 4", or a
  reply to that message).
- If a reply is unclear or two people disagree, ask one short question before
  doing anything.
- An approval covers the exact text you showed. If the live prompt or knowledge
  changed since, or the fix in the item differs from what you showed, show the
  current exact change again and ask again.

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

## Review log

Keep one file, `cx-review-log.md`, in your persistent workspace. One line per
item: number, date, feedback issue ID, short title, theme, state (proposed,
approved by whom and when, applied on which date, dismissed with reason, waiting on
Adelante, waiting on team). Record team decisions on rules and policy questions
too ("keep the transfer rule for X, decided by whom, when"). Add one line per week
with the conversations, bot handovers and automation rate. Read it at the start of
every run and before acting on a reply. It is the source for "what is still
waiting", for the applied-change check and for not asking a decided question
again. Keep customer personal data out of it; use session IDs.

## Message format

- One message per run. Short lines, numbered items, no tables.
- First line: "Last week: N bot handovers in M conversations, X% automated
  (4-week average Y%)."
- Then the health lines, then the one or two things that need an answer today.
- Every count carries its period ("12 bot handovers last week").
- Weekly: at most six proposals or questions. Daily: at most six items.
- Say when a number is partial (capped at 150, human replies unavailable,
  analytics unavailable) and how many contacts were left out as not customers.

## Scheduling in a hosted runtime

When the administrator asks for it and the runtime has a scheduler:

- Daily feedback follow-up: weekday mornings in the team's timezone.
- Weekly handover review: one fixed weekday morning.
- Each job loads `adelante-agent-studio-analyst` and `cx-review-loop`, delivers to
  the team chat it was created in, and keeps continuity with the chat session.
- Check for an existing job before creating one. A scheduled run only proposes
  and reports; applying happens when someone replies in the chat.
