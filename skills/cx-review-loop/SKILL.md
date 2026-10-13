---
name: cx-review-loop
description: Run the recurring improvement loop for an assigned Agent Studio support agent in the team's chat - a weekly review of the bot's handovers and a daily follow-up on open feedback. Propose each change with its exact text, apply it only after someone in the team chat approves, then confirm it landed. Use for a scheduled weekly review, a scheduled daily feedback follow-up, or when the team replies to one of those messages.
---

# CX review loop

Two routines for the support agent you are assigned to, both ending in the same
propose, approve, apply steps:

- **Weekly handover review**: read last week's handovers and what the human agents
  did next, and propose the changes that would have let the bot handle them.
- **Daily feedback follow-up**: ask the team what to do with feedback items that
  are still open.

You propose. The team decides in the chat. You apply only what they approved.

## Before you start

- Use the installed `adelante-agent-studio-analyst` skill for the Agent Studio MCP
  tools and for how to read conversations, prompts and knowledge. Discover the
  live tool list first; if `listHandovers`, `getOperatorThread`,
  `listFeedbackIssues`, `submitFeedback`, `approveFeedbackFix` or
  `dismissFeedbackIssue` is missing or returns 403, say which one is missing and
  run the rest read-only. Use only the scoped connection you were given.
- Write to the team in their language, in the chat where this routine runs.
- Keep a review log (see "Review log") so a reply days later still maps to the
  right item.

## Weekly handover review

1. Call `listHandovers` for the last 7 complete days. Page until you have every
   row. If there are more than 150, take the newest 150 and say so in the report.
2. For each handover read `getOperatorThread` to see what the human did after the
   bot handed over. When the human side is unavailable for a conversation, count
   it as "no human reply available" and leave it out of the conclusions.
3. Sort every handover into one of three kinds, from the human's reply:
   - **Reply only**: the human answered with information the bot had or could have had.
   - **One action**: the human did one back-office step, then replied.
   - **Judgment**: several steps, an exception, or back-and-forth.
4. Group them into themes. For each theme record the count for the week, two or
   three session IDs, and what the human typically answered.
5. For each theme decide what would remove it:
   - **Wording**: the team already answers it the same way, or the prompt and
     knowledge disagree with each other. This is the only kind you propose as a
     change to apply.
   - **Tool gap**: the bot lacks an action or a piece of data.
   - **Team decision**: the bot would be doing something it does not do today,
     or the team's replies differ from the written policy.
6. Check last week's applied changes: for each one, did the same theme come back
   this week? Report it in one line per change.
7. Send one message (see "Message format"): the week's handover count and rate,
   the top themes with counts, then the numbered proposals and questions.

A handover in `listHandovers` is a conversation where the bot called its handover
tool. Human takeovers and system-triggered handovers are not in it, so the count
can be lower than the helpdesk's. Say "bot handovers", and give counts with their
week.

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

- A wording proposal from the weekly review is filed with `submitFeedback` on one
  representative conversation (`source: Handover Analysis`). Put the theme, the
  weekly count and the example session IDs in the feedback text. Then read the
  created item back: the fix it holds is what would be applied.
- Always show the exact text that will change: the current wording and the new
  wording, or the new knowledge entry. Never ask for approval of a summary.
- Number every proposal and question in the message and write the same numbers
  in the review log.
- Tool gaps and team decisions are questions, not changes. Ask them in the same
  message and record the answer. Tell the team when something needs Adelante to
  build or apply it.

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
item: number, date, feedback issue ID, short title, state (proposed, approved by
whom and when, applied, dismissed with reason, waiting on Adelante, waiting on
team). Read it at the start of every run and before acting on a reply. It is the
source for "what is still waiting" and for next week's check on applied changes.
Keep customer personal data out of it; use session IDs.

## Message format

- One message per run. Short lines, numbered items, no tables.
- Lead with the one or two things that need an answer today.
- Every count carries its period ("12 bot handovers last week").
- Weekly: at most six proposals or questions. Daily: at most six items.
- Say when a number is partial (capped at 150, human replies unavailable).

## Scheduling in a hosted runtime

When the administrator asks for it and the runtime has a scheduler:

- Daily feedback follow-up: weekday mornings in the team's timezone.
- Weekly handover review: one fixed weekday morning.
- Each job loads `adelante-agent-studio-analyst` and `cx-review-loop`, delivers to
  the team chat it was created in, and keeps continuity with the chat session.
- Check for an existing job before creating one. A scheduled run only proposes
  and reports; applying happens when someone replies in the chat.
