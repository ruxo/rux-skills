# Test Cases

High-level UAT, E2E and SIT cases that your human partner runs after the
build. No unit tests, no test code: builders choose their own unit tests
to make these cases pass.

## Sort behaviors into levels

| Level | Question it answers | Actor | Example |
|---|---|---|---|
| **UAT** | Does it meet what the business accepts? | A person, in business terms | Owner sets an announcement → visitors see it |
| **E2E** | Does a whole journey work across the deployed stack? | A person | Owner sets an announcement, the watcher restarts → visitors still see it |
| **SIT** | Do two systems work correctly at their boundary? | A system | Service answers 500 → watcher resends on the next poll |

**Each behavior appears once, at the first level in the order UAT → E2E →
SIT that can observe it.**

- UAT takes every behavior a person can check against the requirements.
- E2E takes only journeys across several steps or systems that no single
  UAT case already covers.
- SIT takes only behavior no person can observe: contracts, retries,
  failures at a boundary.

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
gives its path and the number of cases at each level. Then ask "Are the
test cases approved?" They close like a design section (see Closing a
section in SKILL.md). Revise in the same place, and say what changed.

The approved cases go into the design doc; the temporary file is not
part of it.

**Redundant cases — before and after:**

```
Before: one behavior at three levels
UAT  TC-01 — As a project owner, when I set an announcement, then visitors see it on the project page. (R-01)
E2E  TC-02 — As a project owner, when I set an announcement, then the watcher picks it up and visitors see it. (R-01)
SIT  TC-03 — As the watcher, when the owner's announcement changes, then I PUT it to the service and visitors see it. (R-01)

After: kept once, at UAT; SIT keeps only what no person can see
UAT  TC-01 — As a project owner, when I set an announcement, then visitors see it on the project page. (R-01)
SIT  TC-02 — As the watcher, when the service answers 500, then I resend the same state on the next poll. (R-04)
```

## Check before presenting

1. **Coverage:** every requirement has at least one case.
2. **Traceability:** every case cites at least one requirement.
3. **Redundancy:** no behavior appears twice, within a level or across
   levels.
4. **Shape:** one line each, in the format above, levels in UAT → E2E →
   SIT order.
