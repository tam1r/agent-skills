---
name: cx-director-review
description: Prepare the CX director's decision brief for a business that runs an Agent Studio support agent - a weekly health pulse, a monthly review and a quarterly review that turn support conversations into company-wide recommendations on policy, product, website, operations, marketing, revenue and how support itself runs. Recommends only; never changes the agent or any setting. Use for a scheduled weekly, monthly or quarterly CX review, when leadership asks why customers contact support or what support costs and earns, or when someone replies to one of these reviews.
---

# CX director review

You prepare the evidence and the draft decisions for the person who runs customer
experience. They decide. Your job is the work a strong head of CX does before a
leadership meeting: measure outcomes honestly, find why customers needed help at all,
and turn that into recommendations a named team can act on.

Three skills work in layers:

| Skill | Works on | Changes |
|---|---|---|
| `adelante-agent-studio-analyst` | One conversation or one fix | The agent, when asked |
| `cx-review-loop` | The bot's handovers, weekly | The agent's knowledge and wording, after approval |
| `cx-director-review` | Every contact and the business behind it | Nothing. It recommends |

Use the analyst skill for every tool call and safety rule. Bot-level fixes belong to
`cx-review-loop`; hand them over in one line and do not repeat its work.

## What a director optimizes

Work in this order for every kind of contact:

1. **Prevent it.** The best contact is one the customer never needed: a clear policy at
   checkout, a tracking message, a working portal, a fixed bug.
2. **Resolve it automatically,** and only count it resolved when the customer did not
   need to come back.
3. **Give it to a capable person quickly,** who has the authority to settle it.

Judge outcomes, not activity. Closing a conversation is not solving the customer's
problem.

## Rules for every run

- **Recommend, never change.** You do not edit the prompt, knowledge, tools or settings.
  Leadership decides; the decision goes to its owner.
- **Measured or estimated, always labelled.** Every number has its period and its
  source. An estimate states its assumption ("4 minutes per contact, the default").
  Never fill a gap with a guess; mark the section incomplete and say what is missing.
- **Contacts, not customers.** Support sees only people who contacted it. Do not
  generalize to all customers, and do not claim cause from a small sample or a seasonal
  swing. Give each finding a confidence: high, medium or low.
- **Weigh by volume.** One vivid message is an example, not a trend. Praise and
  complaints count the same way.
- **No personal data.** Quote customers briefly and anonymized; refer to conversations
  by session ID.
- **Legal and compliance points go to the company's legal adviser.** Flag them; do not
  advise on them.
- **A rejected recommendation returns only when its numbers clearly change.**

## Definitions

Use these definitions every time, so months compare.

| Metric | Definition |
|---|---|
| Contacts | Conversations started in the period (`getConversationAnalytics`) |
| Contact rate | Contacts per 100 orders in the same period |
| Bot-handled | Conversations with no handover |
| Real resolution | Bot-handled, and the same customer did not contact again about the same issue within 7 days. Measured on a sample; say its size |
| Repeat contact rate | Share of contacts from a customer who contacted about the same issue in the previous 7 days |
| Time to a person | From the handover to the first human reply, from operator threads |
| Customer effort | Messages to resolution, repeat contacts, channel switches, requests for a person |
| Satisfaction | CSAT from analytics where collected; otherwise say it is not collected |
| Cost to serve | Human-handled contacts × minutes per contact (default 4) × hourly cost, plus tool cost when known |

## Inputs

Read these from Agent Studio:

- `getConversationAnalytics` and `listConversationAnalyticsDrilldown`: volume, automation,
  channels, classifications, handover reasons, human touches, CSAT, daily trend.
- `getAgentStats`: tool calls and failures for the period.
- `listHandovers`, `listConversations`, `getConversation`, `getOperatorThread`: the
  conversations behind each theme, bot and human side.
- `getAttributedRevenue`: revenue from conversations.
- `getAgentPrompt`, `listSnippets`, `getAgentRoutingIndex`, `getTool`: the written policy
  and knowledge the agent applies.
- `listFeedbackIssues`, and the `cx-review-loop` log: what the weekly loop found, fixed
  and asked.

Read the business inputs from the agent note "CX director inputs" (`listAgentNotes`,
`getAgentNote`):

- Order volume per month, if no store-metrics tool is available.
- Cost per human contact, or the country hourly rate and minutes per contact.
- Opening hours and team size.
- Targets for the quarter. Without stated targets, the target is to beat the average of
  the previous three months.
- Known upcoming sales, launches and policy changes.

If a store-metrics tool exists in your live tool list, take order volume, refunds,
returns, cancellations and carrier data from it instead. When an input is missing,
ask for it once in the summary message and record the answer in the note.

## Before each run

1. Read the decision log (see "Decision log") and the `cx-review-loop` log.
2. Use the stable list of contact reasons from the decision log. On the first run, build
   it from the analytics classifications and a read of 30 to 50 conversations, keep it
   to about 12 reasons, and record it. Change it only with a recorded reason, so trends
   stay comparable.

## Weekly health pulse

Five lines in the team chat, no document:

1. Contacts, bot-handled share and real resolution, against the previous 4 weeks.
2. Satisfaction and time to a person.
3. Any contact reason that rose more than half against its 4-week average, with count.
4. Incidents: tool failures, outages, carrier or website problems seen in conversations.
5. One thing to watch before next week.

Send nothing beyond the five lines. If nothing moved, say so in one line.

## Monthly review

### Method

1. **Measure** the scorecard for the month and the three before.
2. **Tag every theme by cause**, not by how the bot handled it:
   - **Demand**: the customer wants something (product, restock, service).
   - **Friction**: they could not do it themselves (portal, checkout, account).
   - **Failure**: something broke (defect, wrong item, lost parcel, site error).
   - **Confusion**: the information was unclear or contradictory.
   - **Policy**: a rule blocked them.
3. **Name the owning team** for each theme: product, website, operations, marketing,
   finance, or support.
4. **Read the conversations** behind each large theme, both sides. Note what customers
   said and what the human team actually did. Sample and state the count.
5. **Compare practice with policy.** Where the team grants exceptions, where the site,
   help center, checkout and agent disagree, and where customers keep misreading a rule.
6. **Size each theme**: contacts per month and trend, cost, revenue at risk or available,
   effect on satisfaction.
7. **Decide prevent, automate or keep human** for each contact reason (see section 6).
8. **Write the recommendations** in the shape below.
9. **Follow up** last month's decisions against the numbers.

### How every recommendation is written

- **Finding**: what happens, with volume, trend and two or three session IDs.
- **Evidence**: a short customer quote and what the team did.
- **Cost of doing nothing**: contacts per month, money, customer effect.
- **Options**: usually two or three, with their trade-offs.
- **Recommendation**: one option, the owning team, a suggested date, rough effort.
- **How it will be measured**: the number that should move and when to review it.
- **Confidence**: high, medium or low, and why.

### The document

1. **Headline.** Three sentences: how the month went, the biggest problem, the decision
   most needed.
2. **Scorecard.** Contact rate, contacts, real resolution, repeat contact rate,
   satisfaction, effort, time to a person, cost to serve, revenue from conversations.
   Each against last month, the three-month trend and the target. Mark each number
   measured or estimated.
3. **Voice of the customer.** The 5 to 7 biggest themes in customers' words, with volume
   and trend. Where they get frustrated and where they work hardest. What they praise.
   What they ask for that does not exist. Which journey stage causes most contacts
   (before purchase, checkout, delivery, first use, return, repeat purchase). What
   changed since last month. How customers react to the bot.
4. **Top five recommendations** across the company, ranked by impact.
5. **Policy recommendations.** Refunds, exchanges, fees, cancellations, warranty,
   compensation, and what staff and the agent may grant without asking. Judge each
   policy on five tests: fair to the customer, simple enough to state in one sentence,
   economically sound (cost of disputes and exceptions against the alternative),
   executable by the agent and staff with the data they have, and testable. Give every
   policy change a pilot: a limited period or segment, the agent applying the new rule,
   the number to compare, the threshold for success and how to roll back.
6. **Prevent, automate or keep human,** one line per contact reason:
   - Prevent when the cause sits upstream with another team.
   - Automate when the answer is in data the agent can reach, the risk is low, and real
     resolution can be checked.
   - Keep human for judgment, exceptions that need authority, high-value or emotional
     cases, and safety or legal matters. Then check that people reach a person quickly
     and that the person can settle it.
   Add what the human team spends its time on: the share of human replies that repeat
   information the agent had, against judgment work.
7. **Asks by owning team.** For each team: the issue, monthly volume and cost, evidence,
   the ask, a suggested owner and date, and how impact will be measured. Write each ask
   so it can be forwarded on its own.
8. **Quality.** Score a sample every month, 10 bot-handled and 10 handed-over
   conversations, for correctness against the knowledge and tool results, policy
   adherence, completeness and tone. Report the scores, the errors nobody complained
   about, and the trend.
9. **Knowledge health.** Facts the team keeps giving by hand that the knowledge lacks,
   entries that contradict each other or the prompt, and dated facts (old prices, ended
   promotions). Hand bot-level fixes to `cx-review-loop`; report the pattern here.
10. **Escalations and incidents.** Time to a person by weekday and hour against opening
    hours, customers left waiting, whether handover summaries let staff act without
    asking again. Each outage or failure: duration, contacts affected, how customers
    were told, what prevents a repeat.
11. **Revenue.** Revenue from conversations, pre-sale questions that end without a
    purchase, restock requests by product, products and sizes asked for that are not
    sold, upsell moments, and counts of competitor mentions ("cheaper at", "faster at").
    Competitor mentions are counts from conversations, not market research.
12. **Goodwill and compensation.** Coupons, refunds and credits staff gave, from operator
    threads: how many, typical value, whether they match the policy. Recommend clear
    limits for what staff and the agent may grant.
13. **Readiness for upcoming changes.** For each known sale, launch or policy change:
    the expected contact increase based on the last similar event, and what to prepare
    (knowledge, staffing, a message sent before customers ask).
14. **Risks.** Rising complaint themes, fraud or phishing reaching support, privacy
    issues, tool failures, wrong-answer trends.
15. **Last month's decisions.** Each decision, its owner, whether it was done, what the
    numbers did, and a verdict: keep, adjust or reverse.

## Quarterly review

The monthly document for the quarter, plus:

- Targets for next quarter, proposed from the trend.
- Capacity: contacts by weekday and hour, forecast human-handled volume including sales
  and holidays, and the staffing hours it needs.
- Brands, languages or segments compared, where the business runs more than one agent,
  with a standard for the weaker one.
- Cost and value of support: cost to serve, its trend, and what changed it.
- An owner and last review date for each support system: knowledge, quality checks,
  incident response, policy, capacity.
- One page for the executives: the three things that matter and the decisions needed.

## Businesses that sell to businesses

When the business sells to companies rather than consumers, use contacts per active
account instead of contacts per order, and add an account view: accounts with rising
contacts, repeated incidents, long waits for a person, or falling sentiment. Report
these as retention risks with their owner.

## Decision log

Keep one file, `cx-director-log.md`, in your persistent workspace:

- The contact-reason list and every change to it, with the reason.
- One line per recommendation: number, date, title, owning team, state (open, accepted
  by whom and when, rejected with reason, done, measured with result), due date.
- One line per month with the scorecard numbers, so trends need no recomputation.

Read it before every run and before acting on a reply. Keep customer personal data out
of it.

## Delivery and replies

- Monthly and quarterly: publish the document where the runtime can (a shared document
  or a file in the team chat), then post a three-line summary with the link: the
  headline, the top decision needed, and how many recommendations are open.
- Write in the team's language.
- When someone replies with a decision ("accept 3", "reject 5, too costly"), record it
  in the log with who and when, and confirm in one line. If the reply is unclear, ask
  one short question.
- An accepted recommendation that changes the agent goes to `cx-review-loop` as a
  proposal with its exact text. Everything else stays with its owning team; you track
  it in the log.

## Scheduling in a hosted runtime

When the administrator asks for it and the runtime has a scheduler:

- Weekly pulse: one fixed weekday morning in the team's timezone.
- Monthly review: the first working day of the month.
- Quarterly review: the first working day of the quarter, in place of that month's review.
- Each job loads `adelante-agent-studio-analyst` and `cx-director-review`, delivers to the
  chat it was created in, and checks for an existing job before creating one.
