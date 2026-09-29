# Test Cases

High-level UAT, E2E and SIT cases that your human partner runs after the
build. No unit tests, no test code: builders choose their own unit tests
to make these cases pass.

## Sort behaviors into levels

| Level | Question it answers | Actor | Example |
|---|---|---|---|
| **UAT** | Does it meet what the business accepts? | A person — a user or an operator — in their own terms | Owner sets an announcement → visitors see it |
| **E2E** | Does a whole journey work across the deployed stack? | A person | Owner sets an announcement, the watcher restarts → visitors still see it |
| **SIT** | Do two systems work correctly at their boundary? | A system | Service answers 500 → watcher resends on the next poll |

**Each behavior appears once, at the first level in the order UAT → E2E →
SIT that can observe it.**

- UAT takes every behavior a person can check against the requirements.
- E2E takes only journeys across several steps or systems that no single
  UAT case already covers.
- SIT takes only behavior at the boundary between two separately
  deployed systems — this service and the one it calls, a worker
  process and the API it claims from — that no person can observe:
  contracts, retries, failures at that boundary. The actor is the
  system on one side, and the other side is real, not faked. The
  service's own database, files and queues are part of the service,
  not a second system.

**What is not a test case at all.** Behavior inside one process —
replay logic, a parser, an ordering rule, what a module does when a
write fails — is a builder's unit or in-process test, not SIT, even
though no person can observe it. The test: *could a builder run this in
the test project with nothing else deployed?* If yes, drop the case;
the builder writes that test. A SIT case that names no second system is
one of these.

The dropped case's requirement still needs a case, where a person or
another system sees the outcome, through something the design defines:
a log line, a status field, a response. When the design defines nothing
to see, that is a gap, and the choice is your human partner's. Ask
before presenting the test cases, as a choice (Showing choices in
SKILL.md):

- **Add something to see** — name the log line or field; it goes into
  the section it belongs to (Telemetry, usually), and the case cites it.
- **Make it a builder's concern** — the requirement loses its number,
  the other numbers staying as they are, and the statement stays,
  unnumbered, in the section that describes it (Flows, Error handling),
  so the builders still get it.

Either answer changes sections already approved. Show each with the
test cases, one line per section — "This also changes section N: …" —
so approving the test cases approves those changes too.

A level with no cases is left out.

## One line per case

```
TC-NN — As a <role>, when <action>, then <observable result>. (R-NN[, R-NN])
```

- **One behavior per line.** A case that needs "and then" in its result
  is two cases — unless it is an E2E journey.
- **Then is observable.** Something a tester can see pass or fail: what
  appears, what is stored, what is sent. Never "works correctly".
- **Every case cites the requirements it checks.** A case with no
  requirement behind it is invented scope — remove it, or add the
  requirement to the design first.
- **Numbering runs across levels** in order: TC-01, TC-02, … from UAT
  through SIT.

## Presenting them

Present the cases as three lists under the headings UAT, E2E and SIT —
in chat, or, when the list is long, in a temporary file in your
scratchpad or temp directory (not the repo). For a file, the chat message
gives its path and the number of cases at each level. Within a level,
past seven cases, group them — by actor or flow at UAT and E2E, by the
boundary at SIT (Long lists in SKILL.md) — numbering still running in
order. Then ask "Are the test cases approved?" They close like a design
section (see Closing a section in SKILL.md). Revise in the same place,
and say what changed.

The approved cases go into the design doc; the temporary file is not
part of it.

**Redundant cases — before and after:**

```
Before: one behavior at three levels
UAT  TC-01 — As a project owner, when I set an announcement, then visitors see it on the project page. (R-01)
E2E  TC-02 — As a project owner, when I set an announcement, then the watcher picks it up and visitors see it. (R-01)
SIT  TC-03 — As the watcher, when the owner's announcement changes, then I PUT it to the service and visitors see it. (R-01)

After: kept once, at UAT; SIT keeps only the boundary behavior no person can see
UAT  TC-01 — As a project owner, when I set an announcement, then visitors see it on the project page. (R-01)
SIT  TC-02 — As the watcher, when the service answers 500, then I resend the same state on the next poll. (R-04)
```

**Unit test as SIT — before and after:**

```
Before: internal behavior dressed as SIT
SIT  TC-17 — As the service, when a segment's last line is cut mid-way, then replay stops there and truncates the file. (R-15)

After: not a test case; the builder's unit test covers it. R-15 still
needs a case where a person sees the outcome, using the warning the
Telemetry section defines:
UAT  TC-17 — As the operator, when I start the service with a journal whose last line is cut, then the startup log warns with the truncated segment's name. (R-15)
```

## Check before presenting

1. **Coverage:** every requirement has at least one case — a
   requirement covered only by a builder's unit test still needs one,
   at the level where a person or another system can see the outcome,
   and what they see is defined in the design.
2. **Level:** no case runs inside one process; every SIT case has a
   real second system on the other side.
3. **Traceability:** every case cites at least one requirement.
4. **Redundancy:** no behavior appears twice, within a level or across
   levels.
5. **Shape:** one line each, in the format above, levels in UAT → E2E →
   SIT order.
