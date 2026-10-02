---
name: writing-builder-plans
description: Use when an approved design document is ready to be built by several builders working in parallel, before any implementation starts
---

# Writing Builder Plans

## Overview

Turn an approved design document into plans for builders who work in
parallel, each alone in their own session and their own worktree, until
done, without talking to each other. The plans make that possible with
one rule: **a plan depends only on plans it starts after, and plans
that can run at the same time touch no file in common**. Two plans can
run at the same time when neither reaches the other through Starts
after. Contracts that plans share go in contract plans that the plans
using them start after.

**Quality comes before parallelism.** No plan may leave anything broken:
when a plan is done, the code builds and every test passes. If the work
cannot be split without breaking that, write fewer plans — down to one.

Plans say what to build; builders write and verify most of the code,
test-first. Contracts are given in full. Elsewhere, a plan shows code
only where it says something more clearly than words — an exact query,
format or pattern. The rest — the detailed design and its
implementation — is left to the builders.

**Announce at start:** "I'm using the writing-builder-plans skill to split the design into builder plans."

## 1. Inputs

- **The design doc** — a file, or a GitHub issue. If your human partner
  didn't give its path or issue URL, ask. Read it in full.
- **The builder count** — how many plans to aim to have running at the
  same time. Look in your harness's persistent memory, then the project
  instruction file (`CLAUDE.md`, `AGENTS.md`, `GEMINI.md`). If it is
  not there, ask, then add a line to the project instruction file.
- **Where the plans go.** Beside the design doc. Plans are temporary
  files: they serve the build and are deleted once it is merged; they
  never become GitHub issues or part of the design doc. When the design
  doc is a GitHub issue, there is nothing to sit beside: ask your human
  partner where the plans go, in the same message as any gaps.
- **The code the design touches.** Read every file it will change, and
  the patterns builders must follow.

**Gaps.** Plans never settle design questions. A gap is anything the
design leaves open that someone outside the build can observe —
behavior, stored data, a public API, an event another system reads — or
a step your human partner must take after the build, such as a deploy
or a migration run. These are not gaps:

- Interfaces between plans that nothing outside the build sees — names,
  signatures, internal types. The contract plans define them.
- Choices inside one plan — internal structure, helper names, library
  calls. Its builder makes them.
- Setup before building — secrets, accounts, access. They go in Before
  you start (section 5).

Ask before writing any plan: all gaps in one message, each with your
recommended answer. Ask even when told to just write the plans —
several builders building on a guess cost more than the wait. A gap you
find later, while writing, stops you the same way: ask, then continue.

When your human partner answers ("you decide" means your
recommendation), write the answers into the design doc — the next
unused requirement number for each new rule, never a removed one, and a
test case where one is needed, written by the rules of produce-design's
test-cases.md — tell them what you changed, and carry on. For a GitHub
issue, save its body to a file with
`gh issue view <n> --json body -q .body`, edit the file, and put it
back with `gh issue edit <n> --body-file <file>`.

## 2. Map the files

List every file to be created or changed, and every generated artifact.
Mark the **hotspots** — files that more than one part of the work would
touch: service or DI registration, route tables, the exported or
generated API schema, package manifests and lockfiles, module index
files (`lib.rs`, `mod.rs`, `index.ts`), solution or workspace files
that list the projects, shared config, shared test fixtures. Note which
project each file belongs to.

## 3. Contract plans

The contract plans hold everything more than one plan depends on,
written out in full so every builder codes against the same thing:

- **API contracts** — the OpenAPI document (created or updated) for HTTP
  APIs between services; the schema additions for GraphQL
- **Shared types and interfaces** — exact names, fields, signatures
- **Storage schema** — collections or tables, fields, indexes
- **Messages, events and config keys**
- **Every hotspot edit, done once** — for example the registration line
  for a service a build plan will fill in, with that service as a stub.
- **Test projects** — the new test projects and the moved tests of
  section 4's "One test project per code project".

What else a contract plan does — behavior it changes, code it moves or
removes — follows the design. A contract plan also updates whatever its
change breaks, so it too leaves everything building and passing. Moved
tests are listed as `from → to`.

Split the contract the way the plans use it: contracts that different
plans depend on go in different contract plans, so no plan waits for a
contract it does not use; contracts always used together go in one.
Contract plans follow the same rules as every other plan.

**Order follows the dependencies, not phases.** A plan builds on
another when it uses that plan's contract or code, or touches a file
that plan touches. Each plan lists under **Starts after** the plans it
builds on, unless it already reaches them through one it lists; a plan
that builds on none starts at once. Every plan listed under Starts
after has a lower number (`0a` < `0b` < … < `1` < `2`), so when two
plans touch the same file, the lower number goes first, and a contract
plan never waits for a build plan.

Where two build plans would share something, first try moving the
shared part into a contract plan. A build plan starts after another
build plan only when it needs that plan's finished work.

**Git is your human partner's.** Staging, committing, merging and
worktrees are all theirs; builders never touch git. A plan starts from
a commit that contains every plan it starts after, and runs in its own
worktree, so no builder sees another's unfinished work: two builders
never share a working folder, even when their plans touch different
files. Your human partner sets all of that up. How to work tells the
builder to change nothing in git, and that is all a plan says about
git.

## 4. Split the work

**Split where work can run side by side, or must wait for other
plans.** Work that runs as a pipeline, each step needing the one
before, is one plan, however long: two plans in a straight line are one
plan. Splitting is not free: every builder first has to read the
design, the contracts and the code. Split only when the time saved is
worth that; you judge. There is no size rule.

Every split keeps these:

- **Parallel plans share no file.** Two plans may touch the same file
  only when one starts after the other, directly or through others —
  a build plan filling a stub its contract plan created, for example.
  A project's manifest, lockfile and module index are files too: two
  plans that both add a dependency or a module to one project share
  them.
- **Split along projects where you can.** A project is source code
  that builds into one library or executable, or, for an interpreted
  language, deploys as one service — a .NET project, a Cargo crate, an
  npm package, a Go module, a Python service. Plans in different
  projects are the cleanest split. Plans inside one project may still
  run at the same time when they share no file: each builder works in
  its own worktree, so neither sees the other's half-finished code. One
  plan may touch several projects.
- **Independent.** Each plan can be finished, with the build and every
  test passing, using only the plans it starts after and its own files.
  Code from a plan it starts after, directly or through others, it uses
  as it is. Anything else it reaches only through a contract's
  interface, with a fake in its tests.
- **One test project per code project.** Builders test first, so every
  code project gets its own test project, touched only by the plans
  that change that code project. Where tests live inside the code
  project (Go test files, Rust test modules), there is nothing to
  create. When the solution has one test project shared by several code
  projects, the contract plans create a test project for each code
  project the build touches, empty and building, and add it to the
  solution; new tests go there. They also move, unchanged, the shared
  project's tests that the work needs — tests of behavior a plan
  changes, or that a plan must update — into the new test project for
  their code project; they belong there anyway. A helper that tests left behind still use stays put, and the
  new test project references it. Every other test stays where it is:
  this is not a test migration.
- **Cohesive.** Split along the contract — API, dashboard, background
  job — so each plan covers a whole part.

Aim for as many plans running at the same time as the builder count.
Fewer is fine when the work does not divide further: say why in the
hand-off. Never invent work to fill a plan. When more plans are ready
than there are builders, your human partner starts them as builders
come free.

## 5. Your human partner's steps

Anything a builder needs from your human partner — secrets, accounts,
access, environment setup — goes in that plan's **Before you start**
list, so the builder then works to the end without stopping. Nothing
that needs your human partner comes later in a plan.

Checks after development — UAT, E2E, SIT — are not in plans. They are
the design doc's test cases, and your human partner runs them. If a
check is needed after the build and no test case covers it, that is a
gap: ask with the others (section 1) so the design doc gets the test
case.

## 6. Write the plans

Save the plans beside the design doc as Markdown (`.md`), whatever the
design doc's format. When the design doc is a GitHub issue, save them
where your human partner said (section 1). Each file name starts with
today's date, `YYYY-MM-DD-`, then a name you choose, and ends with the
plan's number: `- Plan 0` for a single contract plan, `- Plan 0a`,
`- Plan 0b` … for several, and `- Plan 1`, `- Plan 2` … for the build
plans — for example `2026-09-27-customer-tags - Plan 0.md` …
`2026-09-27-customer-tags - Plan 3.md`.

Plans have no diagrams. Where a task follows a flow, state or model the
design doc draws, name that design doc section instead of redrawing it.

Each build plan contains, in order:

1. **Header** — title, the design doc's path or issue URL, the goal in
   one sentence, and **Starts after** — the plans it starts after, by
   number, or "none".
2. **How to work** — this text, word for word:
   > Work alone until this plan is done. Git is your human partner's: read it if you need to, change nothing in it. Work in the folder you were started in. Build every task test-first: turn each of its Tests pin lines into a test, run it and watch it fail, then write the code that makes it pass — so the tests prove the plan's goal, not just that the code runs. Think in the functional paradigm: data as values, logic as functions whose result follows from their inputs. Side effects are allowed but used sparingly. A mutable variable is acceptable, and the right choice when it would ease memory allocation pressure. Accessing global variables is a side effect and is fine, but keep global variables to a minimum. A transactional variable, one whose value keeps changing at run time, is preferably passed along as state rather than kept global; it stays global only where the design doc puts it there. Before calling a task done, use rux-skills:verification-before-completion. Give every test run a time limit — 120 seconds, unless the project says otherwise or the suite is known to take longer — and treat a run that hits it as a failure to fix, not a reason to wait longer. Leave nothing broken: when this plan is done, the build and every test pass. Change only the files under Files you own; if a task seems to need another file, the plan is wrong — stop and tell your human partner. Track your token spend and your time as you go, and report both to your human partner when you are done, with the time split into coding, testing and other. Measure them the simple way, such as noting the clock when you switch from one to another; rough numbers are fine, and say which ones are estimates.
3. **Before you start** — only when section 5 gives this plan steps.
4. **Files you own** — every file this plan creates or changes, grouped
   by project.
5. **Tasks**, in build order. Each task has:
   - **Files** — the files it creates or changes
   - **Uses** — the contracts it relies on, by name
   - **Tests pin** — each behavior as one line, situation → expected
     result; unit tests, or in-process tests of this plan's own part
   - **Enables** — the design doc's test-case IDs this task helps pass
6. **Done when** — the commands that must pass, from the project
   instruction file.

A contract plan has the same parts, and the **contract** comes in full
before the tasks. Its **Files you own** lists every file it creates,
changes or moves.

## 7. Self-review

1. **Coverage:** every requirement and design section has a task; every
   test-case ID appears under Enables in some plan.
2. **Parallel plans share no file:** take every pair of plans where
   neither reaches the other through Starts after — contract or build;
   their Files you own lists have no file in common.
3. **Independence:** no task needs anything from a plan its plan does
   not start after, directly or through others.
4. **Contract:** every name a task uses from a plan its plan does not
   start after is defined in a contract plan it starts after.
5. **Order:** each plan reaches through Starts after every plan it
   builds on, and lists no plan it does not build on; every plan under
   Starts after has a lower number.
6. **No straight line:** no plan starts after exactly one plan that
   nothing else starts after; those two are one plan.
7. **Nothing broken:** after each plan, on its own and merged with the
   others, the build and every test pass.
8. **Your human partner:** their steps appear only in Before you start.
9. **No design decisions:** every behavior traces to the design doc.
10. **No placeholders:** no "TBD", "handle errors appropriately",
    "similar to task N".
11. **Git:** outside the How to work text, no plan, contract or build,
    mentions git.

Fix issues inline.

## 8. Hand off

First show the **execution diagram**: one box per plan, labelled with
its number and one-line goal, and an arrow into each plan from every
plan it lists under Starts after. A plan can start once every plan with
an arrow into it is done. Render the diagram with the `show_widget`
tool when your harness has one (read its `read_me` first); otherwise
put a PlantUML block in chat. Then say:

> "Plans written: `<every plan's path>`. The diagram shows the start order: a plan can start once every plan with an arrow into it is done. Start each plan, one builder each, in its own worktree, from a commit that contains the plans it starts after. Merge a plan after the plans it starts after; plans that ran at the same time merge in any order. The plans are temporary: delete them once the build is merged. Please review them."

When fewer plans can run at the same time than the builder count, add
the reason.

If they request changes, make them and run the self-review again. Then
stop. Building starts when your human partner starts the builders.

## Common Mistakes

| Mistake | Fix |
|---|---|
| A plan that leaves the build or a test broken for a later plan to fix | Every plan leaves everything building and passing; if the split can't, use fewer plans |
| Splitting pipeline work into Plan 1 then Plan 2 to keep plans short | Two plans in a straight line are one plan, however long |
| Three small side-by-side tasks become three plans | Each builder pays a start-up cost; split only when the time saved is worth it |
| Splitting more ways to keep every builder busy | Quality before parallelism: fewer plans beat a broken one |
| Every build plan waits for all contract plans | A plan starts after only the plans it builds on |
| Two contract plans both edit the solution file and run side by side | The higher-numbered one starts after the other, or they become one plan |
| A plan tells the builder to commit, stage, or make a worktree or branch | Git is your human partner's; a plan says only "change nothing in it", in How to work |
| Two parallel plans each add a line to the same registration file | That edit is a hotspot: it goes in one contract plan both start after |
| A task waits for a service a parallel plan writes | Put its interface in a contract plan; test against a fake |
| Faking code from a plan this plan starts after | That code exists: use it as it is |
| A final "integration" or "end-to-end" task in a builder plan | Cross-builder checks are the design doc's test cases, run by your human partner |
| A plan carries a task's whole implementation | Builders write and verify most of the code; show code only where it is clearer than words |
| Writing plans in AsciiDoc to match the design doc, or copying its diagrams | Plans are Markdown with no diagrams; point to the design doc section |
| Refusing to split work inside one project | Plans in one project can run side by side when they share no file; its manifest, lockfile and module index count as files |
| Every plan adds tests to the solution's one shared test project | The contract plans create one test project per code project |
| Moving all of a project's tests out of the shared test project | Move only the tests the work needs |
| A plan resolves something the design left open | Ask your human partner, write the answer into the design doc, then plan |
| Stopping for a choice only one plan's code can see | Not a gap; the builder decides |
| Asking what to name an interface between two plans | Not a gap; the contract plans define it |
| Writing plans with "Assumption:" notes because you were told not to ask | Ask anyway; assumptions become several builders' wrong code |
