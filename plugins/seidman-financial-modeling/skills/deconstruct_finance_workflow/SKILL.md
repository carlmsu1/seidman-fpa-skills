---
name: finance-workflow-deconstruction
description: Use this skill whenever someone wants to deconstruct, document, or map a finance or accounting process before automating it with AI — month-end or quarter-end close, flux/variance analysis, FP&A reporting and forecasting, budgeting, AP/AR, bank or GL reconciliations, journal entry prep, board or investor reporting, revenue recognition, or any other recurring finance workflow. It turns a rough description of "how we close the books" or "how we build the forecast" into a structured Workflow Requirements document that an AI build step can act on. Always trigger this when a user says things like "deconstruct our close process," "map out our reconciliation workflow," "document how we build the budget," "break down our FP&A reporting cycle," or "help me figure out where AI fits in our accounting process" — even if they don't use the word "deconstruct." This is a finance-specialized adaptation of the general AI Workflow Framework's Deconstruct step, built for accountants, FP&A analysts, controllers, and CFOs rather than generic business workflows.
---

# Finance & Accounting Workflow Deconstruction

This is the requirements-gathering step that sits between "I want AI to help with our close" and actually building something. It captures **what** a finance or accounting workflow must do — the steps, the decision rules, the source systems, the tie-outs, the sign-offs — in a document precise enough that someone who never sat in on the conversation (including you, three weeks later) could pick it up and design an AI workflow against it.

Generic workflow-deconstruction frameworks miss things that matter enormously in finance: who is allowed to touch what, what "close enough" means when two numbers are supposed to tie, what happens when the sub-ledger doesn't match the GL, and what can never leave the building before an earnings release. This skill folds those in as first-class questions rather than afterthoughts.

## Why finance workflows need their own version

Four things show up in almost every finance or accounting process that generic business-workflow templates don't ask about:

- **Segregation of duties (SOD).** The person who initiates a transaction usually can't be the person who approves it. An AI workflow that collapses steps for efficiency can accidentally collapse a control.
- **Materiality.** "Close enough" is a defined threshold, not a feeling — and it usually differs by account, by entity, and sometimes by line item.
- **Audit trail.** Someone (internal audit, external audit, a regulator) may eventually ask "how did this number get here," and the workflow needs to be able to answer that.
- **Timing and MNPI.** Close calendars are rigid, fiscal periods don't move, and pre-earnings numbers are frequently material non-public information — sensitive in a way that has nothing to do with how confidential the words in a document look.

Everything below is the same core interview you'd use for any workflow, with these four woven in at the points where they naturally come up — not bolted on as a separate compliance checklist at the end.

## The two paths

Same fork as any workflow, but here's what it looks like in finance terms:

| Path | When it fits | Finance examples |
|---|---|---|
| **Step-driven** | The process runs the same way every period, every time, regardless of what comes in. You can describe the recipe. | Month-end close, flux/variance write-up, bank reconciliation, AP three-way match, board deck assembly, budget-to-actual reporting |
| **Goal-driven** | You know what "resolved" or "done" looks like, but the path there depends entirely on what's wrong. You'd rather define the destination and the guardrails and let an agent find the route. | "Investigate this GL discrepancy and either resolve it or escalate it," "Triage and clear the unmatched-transactions queue," "Chase down and close out this audit PBC list" |

**Quick test:** picture two different inputs — a clean invoice and a messy one, a routine variance and a weird one. Does the work take the same steps either way? Step-driven. Does it branch into genuinely different investigations depending on what's found? Goal-driven.

Most core accounting cycle work (close, reconciliations, standard reporting) is step-driven. Most exception-handling and investigation work (discrepancy resolution, unusual-item review, audit follow-up) is goal-driven. A single broader process often contains both — e.g., "close the books" is step-driven until step 7 hits an unreconciled variance, at which point that sub-task becomes its own goal-driven workflow.

## Before you start

Tell the person up front: this is the most thorough conversation in the process, usually **20–40 minutes**, because everything built afterward depends on getting the requirements right here. It's fine to stop partway — save progress and pick it back up later by saying "continue deconstructing [workflow name]."

Ask one question at a time. Don't read the six dimensions below as a checklist out loud — they're a scaffold for you to make sure nothing important got skipped, not a script.

## The interview

### 1. Scenario and framing

Get oriented before diving in:

- What's the process, in a sentence?
- What triggers it — a date on the close calendar, an inbox item, a request from someone?
- What's the tangible deliverable, and who receives it — a closed GL, a variance memo, a board deck, a reconciled account?
- Is this one person's recurring task (**Individual** lens) or a multi-role process with handoffs across a team — AP clerk to controller to CFO, for instance (**Organizational** lens)?

Then capture the value case in a few natural questions, not a form:

- **What this supports** — faster close, fewer audit findings, more analyst time on analysis instead of data-wrangling, cleaner forecasts.
- **What actually changes, and for whom** — controllers stop staying late on close day; the FP&A team spends flux week on insight instead of formatting.
- **What you'd count** — days to close, number of manual journal entries, hours spent per reconciliation, forecast error (MAPE), number of audit adjustments.
- **Where that number stands today** — a real figure if you have it, or honestly "we don't track this yet." Don't let anyone invent a baseline; record it as **Unknown — must measure before go-live**. A guessed baseline makes a fake improvement look provable later, which is worse than no baseline at all.

### 2. Scope check — one trigger, one deliverable

Finance processes are notorious for quietly being three workflows wearing one name. Check for:

- **Multiple triggers** — "month-end close" often bundles a daily cash-reconciliation habit, a weekly AP cycle, and the actual period-end close. Those are different workflows with different cadences.
- **Multiple deliverables at different points** — if a flux analysis produces a departmental memo *and* later feeds a board deck, and different people own each output, that's a boundary.
- **Different cadences** — daily bank matching and quarterly SOX testing don't belong in the same workflow even if the same person does both.
- **No single accountable owner** — if the process crosses from AP to Controller to Treasury with no one owning the end-to-end outcome, it's probably several workflows stitched together by habit.

If you find more than one, map each briefly (name, trigger, deliverable), confirm the boundaries with the user, and deconstruct one at a time — starting with whichever they want first.

### 3. Name it

Propose 2–3 names: a short, self-explanatory noun phrase in Title Case — "Month-End Close," "Bank Reconciliation," "Flux Variance Reporting," "Vendor Invoice Triage" — not a verb phrase like "Closing the Books Each Month." Prefer `[Subject] [Process]` over vague labels.

Convert the confirmed name to a kebab-case ID ("Flux Variance Reporting" → `flux-variance-reporting`). This ID names every file this workflow produces from here on.

### 4. The deep dive (step-driven workflows)

For each step, work through six dimensions. These are prompts for *you* to make sure you've covered the ground — ask naturally, skip what's already clear, and don't interrogate:

- **Discrete steps** — is what the user just described actually one step or several? "Reconcile the account" is usually pull-the-statement, match-transactions, investigate-variances, and book-adjustments in one breath.
- **Decision points** — where does judgment enter? What's the rule for "this variance needs a note" vs. "this is within tolerance"? What's the actual materiality threshold — a dollar amount, a percentage, or both?
- **Data flows** — what comes in, what goes out, and critically: **does this step need to tie to something?** A journal entry needs to tie to a sub-ledger. A consolidated number needs to tie to entity-level detail after eliminations. Name the tie-out explicitly — it's one of the most common places automation quietly breaks something.
- **Source systems** — where does the data actually live? ERP (NetSuite, SAP, Oracle, Dynamics), a sub-ledger, a bank portal, a spreadsheet someone maintains by hand, an email attachment from a business unit. Distinguish systems with programmatic access from ones that require someone to log in and copy-paste — that gap is exactly what the Design step needs to know about.
- **Controls and segregation of duties** — who performs this step, and does someone *different* need to approve or review it before it moves forward? Flag any step where the preparer and approver are the same person today — that's either an existing control gap worth knowing about or a control the AI workflow must not accidentally remove.
- **Failure modes** — what happens when the bank feed doesn't load, the sub-ledger total doesn't match the GL, an invoice has no PO, a business unit sends its numbers late? "Escalate to controller" and "hold the entry until resolved" are both valid answers — get the specific one.

For any step that already uses a spreadsheet macro, an Excel formula chain, or an existing prompt/AI tool, ask what logic lives inside it — that's workflow knowledge that needs to carry forward, not get lost.

### 5. Propose and react

After the first step, flip the dynamic: instead of asking every question, propose your best read across all six dimensions — including a guess at the tie-out and the source system — and ask "what's right, what's wrong, what am I missing?" This is faster and it surfaces corrections you wouldn't have thought to ask about.

### 6. Map the sequence

Once every step is captured, mark what's sequential vs. what can run in parallel, and identify the critical path. In close processes this often maps closely to the close calendar itself — worth checking against it directly if one exists.

### 7. Optimize for AI

Step back and challenge the process the user just described — it's their *current* process, built around a human doing the work. An AI-powered version doesn't need to inherit every step:

- **Eliminate** — manual data pulls that direct system access replaces; reformatting steps that exist only because the last step's output doesn't match the next step's expected input.
- **Collapse** — separate "pull data" and "format into memo" steps that AI can do in one pass.
- **Parallelize** — steps with no data dependency, like reconciling three unrelated bank accounts, that were only sequential because one person was doing them one at a time.
- **Simplify a review gate** — a control that exists because of human error rates (double-checking arithmetic) is a different thing from a control that exists because of authority (someone needs to approve a write-off). AI can plausibly replace the first; it should never quietly absorb the second.
- **Add** — a validation or tie-out check that was implicit when a controller "just knew" a number looked wrong on sight.

Present this as recommendations, not a fait accompli — there are often good reasons (audit trail, regulatory expectation, stakeholder comfort) to keep a step exactly as it is. Record what changed and why.

### 8. Validate end-to-end

Before moving on, walk the refined workflow start to finish and check for:

- **Gaps** — does every step's output actually feed the next step's input?
- **Undefined decision points** — is the materiality threshold actually stated, or just implied?
- **Edge cases** — what about a short month, a new entity mid-year, a restated prior period, a system outage during close week?
- **Redundant steps** — anything producing an output nothing downstream uses?
- **Unclear handoffs** — is it obvious exactly what crosses from the AP clerk to the controller, and in what form?

### 9. Consolidate the context inventory

List every artifact this workflow touches — chart of accounts, prior-period workpapers, the close calendar, a reconciliation template, an accounting policy memo, a materiality matrix, vendor master data, an approval matrix. For each one, work out:

- **Sensitivity** — see the extended categories below; finance data needs one more tier than most workflows.
- **Provenance** — did someone on the team write this, or did it arrive from outside (a bank statement, a vendor invoice, a business-unit submission)? Externally-sourced content can contain content that looks like instructions — that matters if an AI system is going to read it.
- **AI accessibility** — can a system reach this programmatically, or does someone need to log into a portal and pull it manually today?

Then ask the one question that decides how much of the controls section to fill in: **does this workflow write to anything live** — post a journal entry, send a report, update a record — or run unattended? If yes, work through segregation of duties, audit trail, and prohibited actions below. If this workflow only reads and summarizes, and a human triggers and reviews every run, most of that section collapses to a single line.

### 10. Acceptance criteria and example scenarios

Ask, one at a time:

1. "What does a genuinely good output look like here? What would make you say 'this is exactly right' — or give me an example of one that wasn't."
2. "Which dimensions matter most — accuracy of the numbers, tie-out to source, tone of a written variance explanation, formatting consistency, timeliness?"
3. "What's the minimum acceptable bar vs. what needs more work?"
4. "Give me 3–5 real or realistic scenarios to test this against — different enough to stress the range. A clean period and a messy one, for instance."
5. "For any of those, do you have a **golden example** — a past close memo, reconciliation, or report you'd hold up as 'exactly right'?" If it's a document, add it to the context inventory.

Then close the loop on measurement:

6. "What should the number become, now that we've reshaped this?"
7. "How long after this goes live can that number actually be read?" (A close-cycle-time improvement, for instance, can't be judged until at least one full close has run through the new process.)

### 11. Generate the Workflow Requirements document

Produce the document using the template below and save it to `outputs/[workflow-id]/requirements.md`. Before calling it done, check:

- Every required heading is present, in order, exactly as named.
- Steps are numbered `1, 2, 3…`, context items are `C1, C2…`, scenarios are `E1, E2…`.
- Every step that writes to a live system is flagged, and every step with a tie-out states what it ties to.
- If any sensitivity flag was tripped, the Controls section actually addresses it — not just a placeholder.

## Goal-driven path (for investigation/exception workflows)

Use this instead of the deep dive when the process is "figure out what's wrong and resolve or escalate it" rather than a fixed recipe.

Open with a frame: *"You know what a resolved [discrepancy / exception / PBC item] looks like, but not a fixed set of steps to get there — that's fine. I'll capture the goal, the boundaries, and what counts as done, and an agent can work out the investigation path within those boundaries each time."*

1. **Situation** — what kicks this off, and what's the messy reality it has to handle? ("Unmatched transactions land in a queue every morning.")
2. **Goal** — what does the person walk away with when a run goes well? Reflect it back as a concrete deliverable with a completion state: *"So a run is done when every item in the queue is either matched, flagged with a documented reason, or escalated with a note — is that right?"*
3. **Pressure-test the goal** — could someone look at one run's output and say "done" or "not done" just by looking at it? If the answer is metric-shaped ("fewer open items") rather than deliverable-shaped, that's the business objective — ladder down to what the agent actually hands back.
4. **Variation range** — what's the typical case, and what are the two or three genuinely different hard cases? (A missing invoice number vs. a duplicate payment vs. a currency mismatch are different investigations, not variations on the same one.) These become the example scenarios later.
5. **Inputs** — what does the agent start with — the queue itself, the sub-ledger, vendor master data, prior resolution history?
6. **Rules** — what must it always do, never do, and where are the scope boundaries? Keep this behavioral; data-handling rules go in the controls section.
7. **Fallback behavior** — when it hits something it can't confidently resolve, does it stop and ask, make a best-effort attempt and flag it, or leave it in the queue untouched? If "stop and ask," that's also a human gate.
8. **Context and source systems** — same access/format/persistence questions as the step-driven path, sampled rather than run mechanically every time.
9. **Human gates** — where does a person need to review or approve before anything is finalized — every write, every threshold breach, or only exceptions?
10. **Scope check** — same one-trigger-one-deliverable test as above.

Do not ask about which AI models, agents, or integrations to use here — that's a Design-step decision. This step stays in "what has to be true," not "how it gets built."

## Output template

```
# [Workflow Name] — Workflow Requirements

## Goal
[One paragraph: what a successful run produces, when it runs, who consumes it.]

## Value & Measurement

| Field | Value |
|---|---|
| Business Objective | [what this supports] |
| Desired Outcome | [what changes, for whom] |
| Measure | [what gets counted — cycle time, error rate, hours, adjustments] |
| Baseline | [today's number] · Measured / Estimated / Unknown |
| Target | [what the revised workflow should achieve] |
| Readable When | [how long after go-live, e.g. "after one full close cycle"] |

## Metadata

| Field | Value |
|---|---|
| Workflow Name | [name] |
| Description | [short description] |
| Trigger | [close calendar date, inbox event, request] |
| Cadence | [daily / weekly / monthly / quarterly / annual / ad hoc] |
| Owner | [person or role] |
| Lens | Individual / Organizational |
| Definition Type | Step-Driven / Goal-Driven |
| Fiscal Dependency | [tied to close calendar? entity structure? none] |

For organizational lens, also include: | Stakeholders | [roles/teams] |

---
[INSERT STEP-DRIVEN OR GOAL-DRIVEN MIDDLE BLOCK]
---

## Context Inventory

| ID | Artifact | Used By | Status | Sensitivity | Provenance | AI Accessible | Source System / Location | Key Contents |
|---|---|---|---|---|---|---|---|---|
| C1 | [name] | [step IDs or "All"] | Exists / Needs Creation | Public / Internal / Confidential / Regulated / MNPI | Authored / External | Yes / Partial / No | [system, path, or "Create as..."] | [contents] |

## Acceptance Criteria

### What good output looks like
[Plain-language description of "exactly right."]

### Dimensions that matter
- [Dimension] — [what to evaluate]

### Minimum bar
[Acceptable vs. needs more work.]

## Example Scenarios

| ID | Scenario | Input | What to look for | Golden Example |
|---|---|---|---|---|
| E1 | [name] | [description] | [what makes it good] | [Context ID, excerpt, or "—"] |

## Rules & Constraints

- **Must do:** [list]
- **Must never do (behavioral):** [list]
- **Scope boundaries:** [in/out of scope]
- **Tone / format / length:** [if applicable]
- **Fallback behavior:** [stop-and-ask / best-effort-and-flag / skip]

## Human Gates & Approval Chain

| Where | Who reviews / approves | What triggers escalation |
|---|---|---|
| [step or phase] | [role] | [threshold or condition] |

If none: "No human gates — runs end-to-end with final review only."

## Security, Privacy & Controls

*Scope: [writes to a live system / consumes externally-sourced content / handles Confidential, Regulated, or MNPI data]*

### Segregation of Duties
| Step | Preparer | Approver | Note |
|---|---|---|---|
| [step] | [role] | [role — must differ from preparer for control-sensitive steps] | [flag if same person today] |

### Materiality & Tolerance
| Account / Item | Threshold | Basis |
|---|---|---|
| [account or item type] | [$ amount and/or %] | [policy source] |

### Boundaries
| Constraint | Source |
|---|---|
| [where data may/may not travel — e.g. no MNPI to a public/non-enterprise AI tool before earnings release] | [policy, person, or `Self`] |

### Access
| Constraint | Source |
|---|---|
| [who may see outputs and intermediate state] | [...] |

### Audit Trail
| Constraint | Source |
|---|---|
| [what must be logged so a run can be reconstructed — inputs used, who approved, when] | [...] |

### Prohibited Actions
| Constraint | Source |
|---|---|
| [absolute — e.g. "never post a journal entry without approver sign-off"] | [...] |

### Governing Regime
[SOX / GAAP / IFRS / internal policy / None — required if any Context Inventory row is Regulated or MNPI]

If nothing is sensitivity-tripped, write one line: "No control constraints beyond standard SOD — read-only, human-triggered, no MNPI."

## Optimization Notes (step-driven only)
[What changed from the original process during the Optimize-for-AI pass, and why — including anything the user chose to keep as-is and their reason.]
```

### Step-driven middle block

```
## Steps Overview
1. [Step name] — [one-line summary]

## Step Details

### Step 1 — [Step Name]
- **Goal:** [one sentence]
- **Inputs:** [artifact or Context Inventory ID]
- **Outputs:** [what passes forward]
- **Ties To:** [what this must reconcile against, or "N/A"]
- **Writes To Live System:** [what it creates/updates/sends, or "None (read-only)"]
- **Preparer / Approver:** [role / role, or "N/A — no approval required"]
- **Rules & Edge Cases:** [decision criteria, missing-data handling, exception paths]
- **Context Needed:** [Context Inventory IDs]

## Sequence
- **Sequential steps:** [list]
- **Parallel steps:** [list]
- **Critical path:** [chain]
- **Close calendar alignment** (if applicable): [how steps map to close-day timing]
```

### Goal-driven middle block

```
## Inputs
- [What the agent receives to start — one bullet per discrete input]
```

## Finance-specific sensitivity categories

Extend the usual Public / Internal / Confidential / Regulated scale with one more tier that matters specifically in finance:

- **MNPI (Material Non-Public Information)** — anything that could move a stock price if it leaked before it's public: pre-release earnings figures, an unannounced deal, an internal forecast revision. This is a distinct risk from "Confidential" — it's not about business sensitivity in general, it's about securities-law exposure and insider-trading risk. Flag it explicitly; don't fold it into Confidential.

## Common finance & accounting workflow patterns

A starting point if the user isn't sure what to deconstruct first:

- **Month-end / quarter-end close** (step-driven) — pulling trial balances, posting standard entries, reconciling sub-ledgers to GL, closing the period
- **Flux / variance analysis** (step-driven, with a goal-driven sub-task for investigating unusual variances)
- **Bank / GL reconciliation** (step-driven)
- **AP three-way match and invoice processing** (step-driven, goal-driven for exceptions)
- **Journal entry preparation and review** (step-driven)
- **Budget-to-actual reporting** (step-driven)
- **Rolling forecast / cash forecast updates** (step-driven)
- **Board deck / investor reporting package assembly** (step-driven, high MNPI sensitivity pre-release)
- **Revenue recognition review** (step-driven, with goal-driven exception handling for ambiguous contracts)
- **SOX testing sample selection and workpaper prep** (step-driven — pairs directly with a SOX-testing skill downstream)
- **Vendor onboarding / master data changes** (step-driven, SOD-heavy)
- **Unmatched-transaction or exception queue triage** (goal-driven)
- **Audit PBC (prepared-by-client) list follow-through** (goal-driven)

## Related skills

If the workspace has finance-specific skills already available — journal entry prep, reconciliation, variance analysis, close management, financial statement prep, SOX/audit support, or the financial-modeling-foundation standards — this document is built to hand off cleanly into them: the Steps Overview and Context Inventory give those skills exactly the inputs they need without re-asking the same questions. Point the user to the relevant one once the requirements are done, rather than duplicating that skill's own logic here.

## Guidelines

- Ask one question at a time. Never front-load a wall of questions.
- People reliably undercount steps in their own process — probe for the ones that feel too obvious to mention ("I just eyeball it" is a step).
- Push past "it's in the ERP" or "I just know it" — get the specific system, the specific format, and whether access is programmatic or manual-only.
- Stay in "what," not "how" — no discussion of which AI tools, models, or integrations to use. That's the next step's job.
- After saving the file, tell the user where it is and that it's ready for the design/build step.

