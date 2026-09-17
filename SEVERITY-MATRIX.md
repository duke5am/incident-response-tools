# Severity Matrix

**Purpose:** agree on how bad something is, in under a minute, while people are stressed.
**Owner:** [TEAM OR ORG NAME] · **Last reviewed:** [YYYY-MM-DD] · **Next review:** [YYYY-MM-DD]

---

## The one rule that matters

**Severity describes customer impact. It does not describe how interesting, novel, or
technically alarming the bug is.**

These two things feel identical during an incident and are almost unrelated.

- A one-line config typo that locks every customer out of login is **SEV1**.
- A fascinating distributed-systems race condition that corrupts an internal metrics counter,
  reproduced once in staging, is **not an incident at all**.
- A memory leak that will definitely take down the service in six hours is **SEV2 or SEV1 now**,
  even though nothing is broken yet — because expected impact is impact.

Ask the impact question, not the engineering question:

> **How many customers cannot do what they came to do, and how badly?**

If you cannot answer that within two minutes, you do not yet know the severity. Either get the
answer or **declare high and downgrade later** (see *Upgrade and downgrade*, below).

### The costly failure is declaring too low

Under-declaring happens because declaring high feels embarrassing, wakes people up, and commits
you to communication you have not thought through. So teams drift downward: "it's probably just
[cache/DNS/one region], let's look for twenty minutes first."

That instinct costs you the two things you cannot buy back: **time** and **the right people in
the room**. A SEV1 declared late is a SEV1 that ran unsupervised for an hour while one person
googled. Over-declaring is cheap and reversible: you can downgrade in ten minutes with a single
message. Under-declaring is expensive and irreversible.

**When in doubt, declare one level higher than feels comfortable, then downgrade with evidence.**
Nobody should ever be criticised for declaring high and being wrong. People should be coached
when they declared low and it turned out to matter — as a process problem, not a personal one.

---

## The scale

Fill in the response-time columns with what your team can actually honour. If you cannot honour
a number, change the number rather than writing an aspiration.

### SEV1 — Critical

**Definition:** A large-scale outage, or an event that destroys or risks destroying customer data,
or a security breach with confirmed customer exposure. Customers cannot use the product at all,
or a core function is unusable for effectively all customers.

**Typical triggers (examples, not an exhaustive list):**
- Primary datastore down, unreachable, or failing writes for most requests
- Authentication or login broken for everyone — nobody can get in
- Complete loss of a critical dependency with no fallback (payment provider, primary cloud region,
  DNS, certificate expiry taking the site offline)
- Data loss, data corruption, or confirmed unauthorised access to customer data
- A deploy or migration that is actively corrupting data as it runs

**User-visible impact:**
> "I cannot use [PRODUCT] at all." Support inbox filling within minutes. Social/media mentions.
> Revenue-affecting for transactional products. Customers asking for a status update.

**Who must be paged (immediately, not by email):**
- Primary on-call for the affected service — **page**
- Incident Commander — **page** (see `ROLES.md`)
- Engineering leadership / director on call — **page**
- Secondary on-call as Operations Lead — **page**
- Support / customer-success lead — **page**
- Communications Lead — **page** if status page and customer email are required
- Executive stakeholder — **notify** (direct message or call), not paged

**Response-time expectation:** acknowledge within **[5] minutes**; first internal declaration
message within **[10] minutes**; first status-page update within **[30] minutes**.

**Status page required:** **Yes — immediately.** Open the incident on the status page before you
know the cause. "We are investigating reports of [SYMPTOM]" is a complete, honest update.

**Customer comms required:** **Yes.** Status-page updates on a fixed cadence plus a direct email
to affected customers. Executive summary to leadership within **[24] hours**.

**Who can declare:** **Anyone** — any engineer, support agent, or on-call responder who believes
customer impact meets this bar. No approval needed. Declaring is always allowed; withdrawing is
what needs sign-off.

**First-move action:** declare, open the incident channel, page the IC. Do not start debugging
alone first.

---

### SEV2 — Major

**Definition:** Significant customer-visible degradation, or a core function broken for a
substantial subset of customers, or a defect that is certain to escalate into SEV1 if left alone.
Workaround may exist but is painful or requires support involvement.

**Typical triggers:**
- A core feature entirely broken for one customer tier, one large customer, or one region
- Error rate or latency elevated well beyond normal, affecting a meaningful share of requests
- Checkout, signup, or a key integration failing for many but not all users
- A dependency degraded and failing over: service is up, but slow or partially wrong
- Background processing badly backed up: queues growing with no sign of draining
- A security issue with elevated risk but no confirmed customer exposure

**User-visible impact:**
> "It mostly works, but [KEY ACTION] fails or takes far too long." Support tickets rising. Some
> customers blocked, others inconvenienced.

**Who must be paged:**
- Primary on-call for the affected service — **page**
- Secondary on-call — **notify**, escalate to page if not improving within **[15] minutes**
- Incident Commander — **notify**, page if not stabilised within **[30] minutes** or if user
  impact is growing
- Support lead — **notify** (they will field the tickets)

**Response-time expectation:** acknowledge within **[15] minutes**; first internal declaration
within **[20] minutes**; first status-page update within **[60] minutes** if customers can see it.

**Status page required:** **Yes**, if customers can observe the problem. If the impact is
genuinely invisible to customers (internal-only degradation), status page is optional and the
decision is recorded in the incident channel.

**Customer comms required:** **Yes** — status page plus a proactive email if the impact is
sustained beyond **[2] hours** or affects a named account. Support gets a holding reply template
immediately.

**Who can declare:** **Anyone.** Same rule as SEV1.

**First-move action:** declare, find the Operations Lead, start the mitigation-first sequence in
`RUNBOOK.md`.

---

### SEV3 — Minor

**Definition:** Degraded or partially broken functionality with a reasonable workaround, or impact
limited to a small number of customers, or a problem that is real but not urgent. The product is
fundamentally doing its job.

**Typical triggers:**
- A non-core feature broken (reporting export, an admin screen, a rarely used integration)
- Elevated errors for a small percentage of requests with an automatic retry succeeding
- One customer affected by a data or configuration problem
- A noisy but non-critical alert indicating a resource trending toward a limit
- Cosmetic or correctness bug in a secondary surface

**User-visible impact:**
> "There is a bump in the road, but I can get my work done." Occasional support ticket. Users may
> not notice at all.

**Who must be paged:** **Nobody is woken up.** Notify the on-call channel during working hours.
Assign an owner the next business day if unresolved.

**Response-time expectation:** triage within **[1 business day]**; acknowledged in the on-call
channel within **[4 working hours]**.

**Status page required:** **No.** Update only if customers are actively reporting the issue in
volume, which itself is a signal to reconsider the severity.

**Customer comms required:** **No** proactively. Support answers if asked, using the actual
technical status.

**Who can declare:** On-call engineer, or any engineer raising it as a ticket. No incident process
required — this is intended to stay lightweight. If two people are working on it simultaneously
and it is growing, that is a signal to upgrade to SEV2 and run the process properly.

**First-move action:** create a ticket, link the alert or report, notify the channel. Move on.

---

### SEV4 — Low / No customer impact

**Definition:** No customer-visible impact. Internal-only problems, cosmetic defects, single-user
issues, or something noticed in passing. Technical debt made visible.

**Typical triggers:**
- Typo, layout defect, or copy error with negligible user impact
- An internal tool, dashboard, or non-production environment misbehaving
- Alert firing correctly but for a condition with no user consequence
- A latent bug found by reading code, not by observing symptoms

**Who must be paged:** nobody, ever.

**Response-time expectation:** none. Goes to the normal backlog and gets prioritised against
everything else.

**Status page required:** No. **Customer comms required:** No.

**Who can declare:** anyone, or nobody — a ticket is enough. There is no incident.

**First-move action:** write it down before you lose it. Then go back to what you were doing.

**Important:** if a SEV4 is being worked on by more than one person for more than an hour, ask
whether it is actually a SEV3 or SEV2 that was misfiled because it arrived through the ticket
queue instead of the pager queue. Severity is about impact, not about the route it came in by.

---

## Quick reference

| | SEV1 | SEV2 | SEV3 | SEV4 |
|---|---|---|---|---|
| **Customer impact** | All / critical function down, or data at risk | Large subset affected, core function broken | Small subset, workaround exists | None visible |
| **Page on-call** | Yes, primary + secondary | Yes, primary | No (notify in hours) | No |
| **Page Incident Commander** | Yes, immediately | Notify, escalate in [15] min | No | No |
| **Acknowledge within** | [5] min | [15] min | [4] working hours | n/a |
| **Status page** | Yes, first update ≤ [30] min | Yes if customer-visible | No | No |
| **Customer email** | Yes + exec summary | If > [2] h or named account | No | No |
| **Cadence of updates** | Every [30] min | Every [60] min | Ticket updates | n/a |
| **Postmortem required** | Yes, always | Yes, always | Optional, on request | No |
| **Who can declare** | Anyone | Anyone | On-call / ticket | Anyone |
| **Downgrade requires** | IC or on-call lead + evidence | Same | Anyone | n/a |

**The SEV2/SEV3 boundary is the one your team will argue about.** The test: *does the customer have
a workaround they can find on their own, without contacting support?* If yes, it is probably SEV3.
If they must contact you, or cannot work at all, it is SEV2 or higher. Write your own version of
that sentence once you have argued it out, so the next argument is shorter.

Replace every `[NUMBER]` with a number your team will actually meet. A missed response time is a
process problem to fix in the next review — it is not evidence that the number should be ignored.

---

## Special cases

### Suspected data loss or corruption
Treat as **SEV1** regardless of apparent blast radius until you have evidence it is smaller.
Stop the thing that is writing bad data before you investigate the cause. Preserve evidence
(see `RUNBOOK.md` → *Stabilise*).

### Security incidents
Suspected breach, credential exposure, or unauthorised access follows the SEV1 path *plus*
whatever your legal, privacy, and regulatory obligations require. Those obligations — notification
deadlines, who must be told, who must approve external wording — must be documented **before** the
incident, not decided during it. Fill in here:

- Security escalation contact: [NAME / CHANNEL / PHONE]
- Legal / privacy contact: [NAME / CHANNEL]
- Notification obligations and deadlines that apply to us: [SUMMARY OR LINK]
- Who may approve external wording about a security incident: [ROLE]

### Data-integrity and "correct but wrong" incidents
A service can be fast, available, and producing wrong answers — for example reporting the wrong
balance, sending the wrong notification, or writing to the wrong tenant. This is often **SEV1**
even though no dashboard is red, because customers are acting on incorrect information and may be
making decisions or payments based on it. **Correctness failures should be taken at least as
seriously as availability failures.** If your monitoring cannot see this class of problem, that
is an action item, not a reason to declare lower.

### Anticipated but not yet realised impact
Expected impact counts. If you can demonstrate that a resource will be exhausted at a predictable
time, that is a live incident at the severity of the impact you expect, not a curiosity to watch.
Responders should act *before* customers are affected — this is the cheapest incident you will
ever run.

---

## Upgrade and downgrade

**Upgrade:** anyone may upgrade the severity at any time, by saying so and stating the impact
evidence. No approval is required. Announce it in the incident channel using the declaration
template in `COMMS-TEMPLATES.md`.

**Downgrade:** requires the Incident Commander (or, for SEV2 → SEV3, the on-call lead) to state
the evidence in the incident channel. Downgrades are decisions with reasons, not vibes. Record
the reason in the timeline — postmortems are much easier when the severity decisions are dated.

**Downgrade rule of thumb:** downgrade only when the fix is verified by customer-visible signal
(real traffic succeeding), not when the dashboard looks better.

---

## Reviewing this matrix

Look at this document again after every SEV1 and SEV2, and at least once a quarter. Three
questions:

1. **Did we argue about severity during the incident?** If yes, the definitions are ambiguous —
   tighten the wording of the level that caused the argument.
2. **Did we consistently declare too low?** Look at incidents that were upgraded. If upgrades are
   common, the *initial* definitions are wrong, not the people declaring.
3. **Are the response-time targets real?** If [5] minutes is never met, either staff to meet it or
   change the commitment. An unmet commitment trains people to ignore the document.

Change it, date it, and tell the team. A severity matrix nobody has read since it was written is
decoration.
