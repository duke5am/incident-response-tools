# Blameless Postmortem Template

**Purpose:** turn an incident into changes that make the next one smaller, and do it without
punishing the people who were on the keyboard.

Copy everything below the line into `[POSTMORTEM LOCATION — e.g. a doc, a wiki page, a repo
directory]`, fill it in, and link it from the incident channel before you archive it.

- **Postmortem owner:** [NAME] — the owner writes it, gathers the input, and drives the actions
- **Review meeting:** [DATE, TIME, DURATION — recommend [45–60] minutes]
- **Attendees:** responders, the IC, the Comms Lead, an engineer who was *not* involved, and
  [A MANAGER IF AND ONLY IF THEY CONTRIBUTE — never as an observer evaluating people]
- **Publish to:** [LOCATION VISIBLE TO THE WHOLE ENGINEERING TEAM]
- **Deadline for first draft:** within [48] hours of resolution, while memory is intact

> **Write the postmortem within [48 hours] and review it within [5 business days].** A postmortem
> written three weeks later is a work of historical fiction: details get smoothed over, and the
> people who were there have reconstructed a tidier story. Facts decay fast. Write the timeline
> down while the logs still exist and the responders still remember what they were thinking.

---

## Why blameless (read this before the review meeting)

**Blameless does not mean accountability-free.** It means the subject of the review is the system,
not the person. Action items are still owned, dated, and tracked — see `ACTION-ITEMS.md`.

The reason is practical, not sentimental. When a postmortem can end in blame, three things happen
and all of them make your systems less safe:

1. **People stop reporting.** The next near-miss that would have been caught early goes
   unreported, because reporting it invites scrutiny.
2. **Information gets withheld during incidents.** Responders hedge, delay escalating, and avoid
   saying "I changed X" while the clock runs.
3. **You fix the wrong thing.** "Be more careful" is not a control. The mistake will be made again
   by someone else, in a different service, because nothing structural changed.

The productive question is never *who*, it is:

> **What did the person see, what did they reasonably expect to happen, and what in the system
> made that action look like the right one at that moment?**

If the answer is "they should have known better", ask what the system did to make the wrong action
easy and the right action hard: unclear UI, a default that should not be the default, a missing
guardrail, no test, no staging parity, a deploy pipeline with no confirmation, a runbook that was
wrong, a dashboard that lied, a rule that everyone breaks.

**Practical blameless language:**

| Instead of | Write |
|---|---|
| "Engineer X ran the wrong command" | "The runbook's command block is ambiguous about [PARAMETER] and returns success when it fails" |
| "Someone forgot to update the config" | "The config requires manual synchronisation between [A] and [B], and nothing detects drift" |
| "The team was careless with the migration" | "The migration tooling does not validate against production-scale data; [TABLE] is [N]× larger than staging" |
| "Human error caused the outage" | "The deploy pipeline allowed an untested path to reach production because [SPECIFIC GAP]" |
| "Should have tested it" | "There is no automated check for [BEHAVIOUR]; the manual check is not on the release checklist" |
| "Poor communication" | "No one owned outbound updates; the status page had a single holder of access who was asleep" |
| "Monitoring failed" | "Alerting covers [SYMPTOM CLASS A] but not [CLASS B]; the gap was known and had no owner" |

Use "the responder", "the operator", "we". Use names only to record who did what (useful in the
timeline), never to attach fault.

### "Human error" is never a root cause

"Human error" is where analysis stops, not where it ends. **Saying a human erred tells you nothing
you can act on.** The follow-up question always has a real answer, and that answer is the finding:

- *Why did the action seem correct?* → the interface was misleading, the doc was stale, the
  confirmation dialog was ignored because it fires 50 times a day.
- *Why was the error not caught?* → no test, no canary, no verification step, alerts that cannot
  see this failure mode.
- *Why was the impact so large?* → no blast-radius limit, no flag, no isolated environment, no
  rollback path.
- *Why was it hard to recover?* → runbook missing, rollback untested, access held by one person.

Every one of those answers is a system property, and every one of them can be changed. A postmortem
whose final cause is "a person made a mistake" has produced zero action items of value. Close it
and start again with the questions above.

Equally: **do not write "root cause" as a single thing.** See the contributing-factors section.

---

# POSTMORTEM — [INCIDENT TITLE]

## Metadata

| Field | Value |
|---|---|
| Incident ID | [ID] |
| Title | [SHORT, CUSTOMER-DESCRIPTIVE TITLE — e.g. "Login failures for all customers" not "Auth NPE in prod"] |
| Severity at declaration | SEV[1-4] |
| Highest severity reached | SEV[1-4] |
| Date | [YYYY-MM-DD] |
| Customer impact window | [START HH:MM UTC] → [END HH:MM UTC] ([TOTAL DURATION]) |
| Time to acknowledge | [N] minutes |
| Time to mitigate | [N] minutes (service restored) |
| Time to resolve | [N] minutes (fully resolved) |
| Postmortem owner | [NAME] |
| Review date | [YYYY-MM-DD] |
| Status | [DRAFT / IN REVIEW / FINAL / ACTIONS OPEN / ACTIONS COMPLETE] |
| Related incidents | [LINKS — especially previous incidents with the same contributing factor] |
| Services affected | [SERVICES] |

---

## 1. Summary

*Two to four sentences. Someone who was not involved should understand what happened, who it hurt,
and how it ended. Customer language, not system language.*

[SUMMARY]

---

## 2. Impact

**Quantify in user-visible terms.** Not "increased 5xx rate on the API" — how many people, for how
long, unable to do what? Fill in what you can measure; write "unknown" for what you cannot, and
make measuring it an action item. Do not invent numbers.

### Customers affected

| Measure | Value | How we know |
|---|---|---|
| Users affected | [NUMBER OR "unknown — see action item"] | [SOURCE: analytics / logs / support volume] |
| Requests failed | [NUMBER OR %] | [SOURCE] |
| Requests degraded (slow but succeeded) | [NUMBER OR %] | [SOURCE] |
| Accounts / tenants affected | [NUMBER OR "ALL"] | [SOURCE] |
| Regions or cohorts affected | [LIST] | [SOURCE] |
| Duration of customer impact | [DURATION] | [FROM TIMELINE] |

### What customers could not do

*Write it as a customer would say it. This is the most important paragraph in the document.*

- [CUSTOMER-VISIBLE CAPABILITY LOST — e.g. "could not log in"]
- [CUSTOMER-VISIBLE CAPABILITY DEGRADED — e.g. "could not complete checkout; saw a generic error"]
- [CUSTOMER-VISIBLE CAPABILITY LOST — e.g. "scheduled reports did not run"]

### Downstream and secondary effects

- Support tickets / contacts: [NUMBER OR "unknown"]
- Named accounts who contacted us: [LIST OR "unknown"]
- Revenue or billing impact: [AMOUNT, RANGE, OR "not quantified"] — *state the method or say you
  have none; do not estimate loosely in a written document*
- Data written incorrectly, lost, or not processed: [DETAIL — and whether it was recovered]
- Backlog accumulated and time to drain: [DETAIL]
- Customer-visible workarounds offered: [DETAIL]
- Trust/compliance consequences (SLA credits, notification duties): [DETAIL OR "none"]

### Impact we did not see

*Any impact that should have been visible but was not, and any impact discovered after resolution
— e.g. a customer who told us a month later. Missing observability is a finding.*

[DETAIL OR "none identified"]

---

## 3. Timeline

**All times in [UTC] — state the timezone once and never mix.** Timestamps matter more than prose:
a good timeline lets a reader reconstruct the incident without asking questions.

Mark entries: **[DETECT]** detection, **[DECIDE]** a decision, **[ACT]** a change made,
**[COMMS]** a message sent, **[OBS]** an observation, **[RECOVER]** user-visible recovery.

Record the *reasoning* at decision points, not just the action. "Decided not to roll back at 14:20
because [REASON]" is far more valuable in review than the action itself.

| Time (UTC) | Type | Event | Source |
|---|---|---|---|
| [HH:MM] | | [Last known good state / last deploy / change made] | |
| [HH:MM] | DETECT | [First symptom appeared — the actual start, if knowable] | [Alert / log / report] |
| [HH:MM] | DETECT | [Alert fired / customer reported / someone noticed] | [Link] |
| [HH:MM] | | [First acknowledgement] | |
| [HH:MM] | DECIDE | [Severity declared SEV[N]. Reason: [REASON]. IC: [NAME]] | |
| [HH:MM] | COMMS | [Status page opened / internal declaration sent] | [Link] |
| [HH:MM] | ACT | [First mitigation attempt: [ACTION]. Owner: [NAME]] | |
| [HH:MM] | OBS | [Result of that attempt] | |
| [HH:MM] | ACT | [Next action] | |
| [HH:MM] | DECIDE | [Severity upgraded/downgraded because [EVIDENCE]] | |
| [HH:MM] | ACT | [Action that worked] | |
| [HH:MM] | RECOVER | [Customer-visible signal recovered] | [Graph link] |
| [HH:MM] | COMMS | [Monitoring update sent] | |
| [HH:MM] | | [Incident declared resolved] | |
| [HH:MM] | COMMS | [Resolved update sent; customer email sent] | |
| [HH:MM] | | [Postmortem scheduled] | |

**Time-to-detect:** [TIME FROM ACTUAL START TO DETECTION — the most under-examined number in most
postmortems. If detection was a human noticing, that is a finding.]

**Time spent on each phase:** detection [N] min · acknowledgement [N] min · mitigation [N] min ·
investigation-after-stabilisation [N] min.

*Note where time was lost and why. Was it ambiguity about severity, missing access, an untested
rollback, waiting on a vendor, an approval, or investigating instead of mitigating?*

---

## 4. What went well

*Not decoration, and not false praise. These are the things to protect and repeat — and people need
to know which of their instincts were right.*

- [e.g. "The rollback path worked first time and took 3 minutes; it was last tested on [DATE]."]
- [e.g. "Detection was automatic and paged within 60 seconds of the first failure."]
- [e.g. "Comms cadence held for the full 90 minutes; no update was missed."]
- [e.g. "The IC handed off at the 4-hour mark and the second IC caught an unchecked assumption."]
- [e.g. "The feature flag allowed us to disable the affected surface without a deploy."]
- [e.g. "Support had a holding reply within 10 minutes of declaration, which kept the ticket queue
  manageable."]

---

## 5. What went wrong

*Systems and process, not people. For each item, say what the consequence was.*

- [e.g. "Time-to-detect was 22 minutes because the alert threshold is 5% error rate; the customer
  impact was severe before it fired."]
- [e.g. "Two people made changes simultaneously between [TIME] and [TIME]; neither change can be
  evaluated independently."]
- [e.g. "The rollback was attempted before evidence was captured, so the exception trace is lost."]
- [e.g. "The status page was opened 40 minutes after declaration; the first customer update went
  out at [TIME]."]
- [e.g. "The runbook for this service is outdated; the documented command no longer exists."]
- [e.g. "No one could confidently state the customer impact for the first 15 minutes."]

---

## 6. Where we got lucky

**The most underrated section in any postmortem.** Luck is a real component of outcome and it is
invisible in a document that only records causes and fixes. Two incidents with identical causes can
differ enormously in impact purely by timing and coincidence. If you do not write down the luck,
you will mistake a narrow escape for competence and you will not fix the thing that would have made
it much worse.

Ask: *what would have made this 10× worse, and how close were we to it?*

- [e.g. "This happened at [LOW-TRAFFIC TIME]; at peak, roughly [N]× more requests would have
  failed and the backlog would have taken hours to clear."]
- [e.g. "The affected region was not the one handling [CRITICAL CUSTOMER]."]
- [e.g. "The bad deploy went to 1 of 6 instances; the canary was orthogonal luck, not a control —
  there is no canary by design."]
- [e.g. "The write that corrupted data failed on a NOT NULL constraint; nothing was persisted."]
- [e.g. "The on-call engineer happened to be the service owner and recognised the symptom
  immediately. The next responder would not have."]
- [e.g. "If the incident had begun 20 minutes later, the person who fixed it would have been
  asleep."]

**Rule: every item in this section should produce an action item.** Luck is not a mitigation. If
the outcome depended on chance, the underlying vulnerability is still there — convert the luck into
a control.

---

## 7. Contributing factors

**There is almost never a single root cause.** Complex systems fail through combinations: a trigger,
pre-existing conditions, gaps that let it spread, and gaps that made it hard to see or stop. Writing
one "root cause" hides the factors that are cheapest to fix and makes recurrence likely.

Use these four categories. Include everything that genuinely contributed — a factor is not a
criticism, and more factors mean more options.

### 7.1 Trigger — what started it

*The proximate event. Necessary but rarely sufficient on its own.*

- [e.g. "Deploy [SHA] at [TIME] introduced [CHANGE]."]
- [e.g. "A dependency's latency increased."]
- [e.g. "A certificate expired."]

### 7.2 Pre-existing conditions — why the trigger had that effect

*What was already true before the trigger. Usually the richest source of action items.*

- [e.g. "The service had no blast-radius control: a single bad request pattern could saturate the
  shared connection pool."]
- [e.g. "Staging runs [N]× less data than production, so the query plan differs."]
- [e.g. "Retry logic is unbounded, so an upstream slowdown amplifies into self-inflicted load."]
- [e.g. "Access to [SYSTEM] is held by one person."]

### 7.3 Amplifiers — why it spread or lasted

- [e.g. "The queue consumer kept retrying poison messages, keeping the backlog from draining."]
- [e.g. "The autoscaler scaled on CPU while the bottleneck was database connections."]
- [e.g. "The status page had one holder of edit access, who was asleep."]

### 7.4 Detection and response gaps — why it was hard to see or stop

- [e.g. "No alert exists for [FAILURE MODE]; it was found by a customer."]
- [e.g. "Dashboards show per-service error rates but nothing that answers 'can a customer complete
  [KEY JOURNEY]?'."]
- [e.g. "The rollback procedure was documented but had not been executed in [N] months."]
- [e.g. "Severity definitions were ambiguous for degraded-but-working, so declaration was delayed."]

### 7.5 The "five whys" — use it, with this caution

Ask "why?" repeatedly to get below the surface, and **stop when the answer is a system property you
can change** — not when you reach a person.

Example chain (illustrative, not a real incident):

- *Why did customers see errors?* The API returned 500s because the connection pool was exhausted.
- *Why was the pool exhausted?* A new query held connections for the duration of an upstream call.
- *Why did that query reach production?* The change passed review; the review checklist does not
  cover connection-holding behaviour and there is no lint rule for it.
- *Why is there no check?* The guideline exists in `[DOC]` but is not enforced by tooling, and the
  author had not read it. → **System property, and actionable: add a lint rule / test.**
- Stop here. Do **not** continue to "why did the author not read the doc?" — that direction produces
  blame and no control.

**Every contributing factor should map to at least one action item, or be explicitly accepted as a
known risk.** Accepted risks are written down with a reason and a review date. "We decided not to
fix this" is legitimate; "we forgot" is not.

---

## 8. Action items

**Cap this list. A postmortem with 25 action items produces zero completed action items.** Three to
five is a good target, and the cap should be enforced by the postmortem owner in the review meeting.
Classification, tracking, and review are covered in `ACTION-ITEMS.md` — read it before filling in
this table.

**Priority order:** prevent > detect > mitigate > respond. A fix that stops the incident happening
beats a fix that makes it visible sooner, which beats one that makes it hurt less. But do not
neglect detection: an incident you cannot see is an incident that lasts longer, and detection work
is often cheap.

| # | Class | Action | Owner | Due | Tracked in | Status |
|---|---|---|---|---|---|---|
| 1 | Prevent | [SPECIFIC, VERIFIABLE ACTION] | [NAME] | [DATE] | [EXISTING TICKET/PROJECT — prefer this over a new tracker] | Open |
| 2 | Detect | | | | | Open |
| 3 | Mitigate | | | | | Open |
| 4 | Respond | | | | | Open |
| 5 | [Process] | | | | | Open |

Rules for writing these (details in `ACTION-ITEMS.md`):

- **Specific and verifiable.** "Add a connection-pool lint rule to CI" is an action. "Improve
  testing" is a wish. If you cannot tell whether it is done, it is not an action item.
- **One owner per item.** A team is not an owner. Two owners means no owner.
- **A real date.** Not "next quarter". If the date is wrong later, move it deliberately and
  visibly — do not quietly let it pass.
- **Prefer the existing backlog.** An action item filed in a dedicated postmortem tracker is an
  action item nobody will ever work on.
- **Say what it prevents.** Every action should name the incident or failure mode it addresses.
- **Do not create actions for things already planned.** Link the existing ticket instead.

### Accepted risks (no action, by decision)

| Risk | Why we are accepting it | Decided by | Review date |
|---|---|---|---|
| [RISK] | [REASON — cost, rarity, tolerance] | [NAME] | [DATE] |

### Actions from previous postmortems that are still open

*If this list is long, the problem is your action-item process, not this incident.*

| From incident | Action | Owner | Original due | Status / why late |
|---|---|---|---|---|
| | | | | |

---

## 9. Lessons and guidance

*Optional but valuable: what should the next responder know that this incident taught us? Often this
becomes a runbook edit rather than prose.*

- [e.g. "When [SYMPTOM] appears, check [THING] first — it was invisible in every dashboard."]
- [e.g. "Rolling back [CHANGE] requires [STEP]; without it, [CONSEQUENCE]."]
- [e.g. "The vendor's status page lagged reality by [N] minutes; use their API endpoint instead."]

---

## 10. Supporting material

- Incident channel: [LINK]
- Dashboards / graphs: [LINK]
- Relevant logs, traces, error samples: [LINK — save them; retention will delete them]
- Deploy / diff / PR: [LINK]
- Status page history: [LINK]
- Customer communications sent: [LINKS]
- Evidence preserved during response: [LOCATION]
- External references (vendor postmortem, dependency incident): [LINKS]

---

## Review meeting agenda ([45–60] minutes)

1. **[5 min] Read the summary and impact aloud.** Correct any factual errors immediately.
2. **[10 min] Walk the timeline.** Ask "what were you thinking at this point?" — not "why did you
   do that?" The goal is to understand the information available at the time, not to audit
   decisions with hindsight.
3. **[10 min] Contributing factors.** Add anything missing. Challenge single-cause explanations.
4. **[10 min] Where we got lucky.** Make sure every item generates an action or an accepted risk.
5. **[10 min] Action items.** Enforce the cap. Assign owners and dates in the room, and confirm
   each owner accepts it. Nothing is agreed until an owner says yes.
6. **[5 min] Process check.** Did the severity matrix, roles, comms cadence, and runbook work? What
   should change in those documents? Edit them in the meeting if it is quick.
7. **[5 min] Close.** Thank the responders specifically. Confirm who publishes this, where, and when.

**Meeting rules:**
- The **postmortem owner facilitates**, not the IC. The IC was in the incident and has a
  perspective to contribute.
- Ask **"what did you know at the time?"** rather than "why did you…" — the second form is
  interrogative even when it is not meant to be.
- **Include someone who was not involved.** They catch unstated assumptions and jargon.
- **Do not let the meeting become the fix.** Investigation needed to complete the document gets
  its own ticket and owner.

---

## Sign-off

| Role | Name | Date | Notes |
|---|---|---|---|
| Postmortem owner | [NAME] | [DATE] | |
| Incident Commander | [NAME] | [DATE] | |
| Engineering lead / manager | [NAME] | [DATE] | Confirms actions are scheduled into real work |
| Published to [LOCATION] | | [DATE] | |

**Actions reviewed on:** [DATES — see `ACTION-ITEMS.md`. A postmortem is not finished when it is
published; it is finished when the actions have landed or been deliberately dropped.]**
