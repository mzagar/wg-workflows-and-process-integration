# Daily activity report — Use Case

## Business problem

Each working day I have to report to my managers what I did yesterday and what I plan to do
today. Compiling this by hand means manually re-reading my own activity across several
systems, which is slow and easy to get wrong or forget. I want a workflow to collect my
activity from those systems and have a model draft the report, then deliver it to me for a
final human check. I am **not** trying to send the report to my managers automatically — the
workflow's job ends when the draft reaches me; I copy, edit, and forward it myself.

## Requirements

- Trigger on a schedule each working-day morning (no manual kickoff in the normal path).
- Collect my recent activity from several source systems (issue tracker, chat, meetings,
  mail/calendar) over a defined time window.
- Draft a short "Yesterday / Today" report from that collected activity.
- Deliver the draft to a private channel that only I can see, so I can review and edit it.
- Keep a record of what sources were read and what draft was produced.
- The delivery of the draft (posting it to my private channel) is the only externally
  visible action; no human approval gate is required before it, because the channel is
  private to me and I review the draft there before doing anything with it.

## Constraints and context

- The collected activity is **untrusted content** — issue text, chat messages and mail
  bodies are free-form and may contain anything; it flows into a model prompt.
- The source systems remain the **authoritative record** of my activity. The report is a
  derived artifact; the workflow must not become a shadow system of record for my work.
- A weak or wrong draft is cheap to fix — I edit it in the channel before forwarding — so
  the impact of a bad result is minor. The real failure to avoid is a run that **silently
  drops** (no report appears and I don't notice) or **double-posts**.
- Runs must be effectively instant; there is no long human wait inside the run, and the run
  does not need to survive a restart mid-way — if it fails it simply re-runs next schedule.
- Vendor-neutral: no dependency on a specific issue tracker, chat product, mail provider,
  model provider, or workflow engine.

## Out of scope

- Sending the report to my managers, or to anyone but me. The workflow stops at my private
  channel.
- Taking any action in the source systems (closing tickets, replying to messages, changing
  calendar) — the harvest is strictly read-only.
- Deciding *for* me what to prioritise today in a way that acts on it; the "Today" section
  is a suggestion I edit, never an instruction the workflow executes.
- Autonomous scheduling of meetings or any customer-visible action.

## Success criteria

Given a normal working day, a good run reads my activity across the configured sources over
the time window, produces a concise "Yesterday / Today" draft, and posts exactly one copy to
my private channel with a record of which sources were read. The closed set of ways a run can
end: `COMPLETED` (draft posted); `COMPLETED_WITH_LIMITATION` (draft posted, but one or more
sources were unavailable and this is stated in the draft); `ABSTAINED` (no material activity
found — a short "nothing material to report" note rather than a padded draft); or
`FAILED_TECHNICAL` (could not post; surfaced for the next run, never a silent drop).