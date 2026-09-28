---
name: writing-builder-plans
description: Use when an approved design document is ready to be built by several builders working in parallel, before any implementation starts
---

# Writing Builder Plans

## Overview

Turn an approved design document into plans for a fixed number of
builders who work in parallel, each alone in their own session, until
done, without talking to each other. The plans make that possible with
one rule: **a plan depends only on plans it starts after, and plans
that can run at the same time touch nothing in common** — no file, no
project. Contracts builders share go in contract plans that the plans
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
- **The builder count.** Look in your harness's persistent memory, then
  the project instruction file (`CLAUDE.md`, `AGENTS.md`, `GEMINI.md`).
  If it is not there, ask, then add a line to the project instruction file.
- **Where the plans go.** Beside the design doc. When the design doc is
  a GitHub issue, there is nothing to sit beside: ask your human partner
  where the plans go, in the same message as any gaps.
- **The code the design touches.** Read every file it will change, and
  the patterns builders must follow.

**Gaps.** Plans never settle design questions. A gap is anything the
design leaves open that someone outside the build can observe —
behavior, stored data, a public API, an event another system reads — or
a step your human partner must take after the build, such as a deploy
or a migration run. These are not gaps:

- Interfaces between builders' parts that nothing outside the build
  sees — names, signatures, internal types. The contract plans define
  them.
- Choices inside one builder's part — internal structure, helper names,
  library calls. That builder makes them.
- Setup before building — secrets, accounts, access. They go in Before
  you start (section 5).

Ask before writing any plan: all gaps in one message, each with your
recommended answer. Ask even when told to just write the plans — N
builders building on a guess cost more than the wait. A gap you find
later, while writing, stops you the same way: ask, then continue.

When your human partner answers ("you decide" means your
recommendation), write the answers into the design doc — a new
requirement number for each new rule, and a test case where one is
needed — tell them what you changed, and carry on. For a GitHub issue,
save its body to a file with `gh issue view <n> --json body -q .body`,
edit the file, and put it back with
`gh issue edit <n> --body-file <file>`.

## 2. Map the files

List every file to be created or changed, and every generated artifact.
Mark the **hotspots** — files that more than one part of the work would
touch: service or DI registration, route tables, the exported or
generated API schema, package manifests and lockfiles, solution or
workspace files that list the projects, shared config, shared test
fixtures. Note which project each file belongs to.

## 3. Contract plans

The contract plans hold everything more than one builder depends on,
written out in full so every builder codes against the same thing:

- **API contracts** — the OpenAPI document (created or updated) for HTTP
  APIs between services; the schema additions for GraphQL
- **Shared types and interfaces** — exact names, fields, signatures
- **Storage schema** — collections or tables, fields, indexes
- **Messages, events and config keys**
- **Every hotspot edit, done once** — for example the registration line
  for a service a builder will own, with that service as a stub the
  owning builder fills in.
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

**Order follows the dependencies, not phases.** Each plan lists under
**Starts after** the contract plans whose contracts it uses; a plan
that uses none starts at once. A contract plan can start after another
contract plan. A build plan never starts after a build plan — where it
would, the shared part belongs in a contract plan.

Each contract plan's builder commits it at the end. A plan starts from
a commit that contains every plan it starts after — your human partner
merges them when there are several — in its own working copy (a branch
or worktree) your human partner sets up.

Those are the only commits builders make. Otherwise builders leave git
alone: no staging, no commits, unless the work cannot go on without
one. Staging and committing are your human partner's job.

## 4. Split the work

Split the rest into one build plan per builder:

- **Parallel plans share nothing.** Two plans may touch the same file
  only when one starts after the other, directly or through others —
  a build plan filling a stub its contract plan created, for example.
  Plans that can run at the same time never touch the same file.
- **Independent.** Each plan can be finished, with the build and every
  test passing, using only the contract plans it starts after and its
  own files. Where it calls another builder's part, it calls the
  contract's interface and its tests use a fake.
- **Parallel plans share no project either.** A project is source code
  that builds into one library or executable, or, for an interpreted
  language, deploys as one service — a .NET project, a Cargo crate, an
  npm package, a Go module, a Python service. Plans that can run at the
  same time never touch the same project, even in different files: they
  share its project file, lockfile and build, and one builder's
  half-finished work breaks the other's compile. One builder may own
  several projects.
- **One test project per code project.** Builders test first, so each
  needs test projects of their own: every code project gets its own
  test project, owned by the builder who owns the code project. Where
  tests live inside the code project (Go test files, Rust test modules),
  there is nothing to create. When the solution has one test project
  shared by several code projects, never give it to one builder or
  split it between builders. The contract plans create a test project
  for each code project the build touches, empty and building, and add
  it to the solution; builders' new tests go there. They also move,
  unchanged, the shared project's tests that the assigned work needs —
  tests of behavior a builder changes, or that a builder must update —
  into the new test project for their code project; they belong there
  anyway. A helper that tests left behind still use stays put, and the
  new test project references it. Every other test stays where it is:
  this is not a test migration.
- **Cohesive.** Within that, split along the contract — API, dashboard,
  background job — so each builder owns a whole part.
- **Balanced by effort**, not by task count.

If the work cannot be split N ways under these rules, write fewer plans
and say why. Never invent work to fill a plan. Put two builders in one
project only when it holds so much of the work that the others would
finish far earlier; split it, and its test project, by disjoint files
and say why in the hand-off.

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
`- Plan 0b` … for several, and `- Plan 1` … `- Plan N` for the build
plans — for example `2026-09-27-customer-tags - Plan 0.md` …
`2026-09-27-customer-tags - Plan 3.md`.

Plans have no diagrams. Where a task follows a flow, state or model the
design doc draws, name that design doc section instead of redrawing it.

Each build plan contains, in order:

1. **Header** — title, the design doc's path or issue URL, the goal in
   one sentence, and **Starts after** — the contract plans it starts
   after, by number, or "none".
2. **How to work** — this text, word for word:
   > Work alone until this plan is done. Build every task test-first — REQUIRED SUB-SKILL: Use rux-skills:test-driven-development. Before calling a task done, use rux-skills:verification-before-completion. Leave nothing broken: when this plan is done, the build and every test pass. Leave git alone: don't stage or commit unless the work cannot go on without it; your human partner commits your work. Change only the files under Files you own; if a task seems to need another file, the plan is wrong — stop and tell your human partner.
3. **Before you start** — only when section 5 gives this plan steps.
4. **Files you own** — every file this plan creates or changes.
5. **Tasks**, in build order. Each task has:
   - **Files** — the files it creates or changes
   - **Uses** — the contracts it relies on, by name
   - **Tests pin** — each behavior as one line, situation → expected
     result; unit tests, or integration tests inside this plan's own
     part
   - **Enables** — the design doc's test-case IDs this task helps pass
6. **Done when** — the commands that must pass, from the project
   instruction file.

A contract plan has the same parts, with two changes: How to work
replaces the "Leave git alone" sentence with "When this plan is done,
commit it: the plans that start after it start from that commit."; and
the **contract** comes in full before the tasks. Its **Files you own**
lists every file it creates, changes or moves.

## 7. Self-review

1. **Coverage:** every requirement and design section has a task; every
   test-case ID appears under Enables in some plan.
2. **Parallel plans share nothing:** no two plans that can run at the
   same time touch the same file or project — a project only when the
   hand-off says why.
3. **Independence:** no task needs anything from another build plan;
   what it needs from other builders' parts comes from a contract plan
   it starts after.
4. **Contract:** every name a task uses from another builder's part is
   defined in a contract plan.
5. **Order:** each plan's Starts after lists every contract plan whose
   contract it uses and no other; no build plan starts after a build
   plan; no plan starts after itself, directly or through others.
6. **Nothing broken:** after each plan, on its own and merged with the
   others, the build and every test pass.
7. **Your human partner:** their steps appear only in Before you start.
8. **No design decisions:** every behavior traces to the design doc.
9. **No placeholders:** no "TBD", "handle errors appropriately",
   "similar to task N".
10. **Git:** only contract plans tell their builder to commit; no task
    in a build plan stages or commits.

Fix issues inline.

## 8. Hand off

First show the **execution diagram**: one box per plan, labelled with
its number and one-line goal, and an arrow from each plan to every plan
that starts after it, so plans on the same level can run at the same
time. Render it with the `show_widget` tool when your harness has one
(read its `read_me` first); otherwise put a PlantUML block in chat.
Then say:

> "Plans written: `<contract plan paths>`, `<plan 1 path>` … `<plan N path>`. The diagram shows the start order: start each plan, one builder each, in its own branch or worktree, from a commit that contains the plans it starts after; contract plans commit themselves when done. Plans 1 to N merge in any order. Please review them."

Add the reasons section 4 asks for: why there are fewer plans than
builders, or why two builders share a project.

If they request changes, make them and run the self-review again. Then
stop. Building starts when your human partner starts the builders.

## Common Mistakes

| Mistake | Fix |
|---|---|
| A plan that leaves the build or a test broken for a later plan to fix | Every plan leaves everything building and passing; if the split can't, use fewer plans |
| Every build plan waits for all contract plans | A plan starts after only the contract plans it uses |
| Splitting more ways to keep every builder busy | Quality before parallelism: fewer plans beat a broken one |
| A plan tells the builder to commit after each task | Builders leave staging and commits to your human partner; only contract plans end with a commit |
| Two parallel plans each add a line to the same registration file | That edit is a hotspot: it goes in one contract plan both start after |
| A task waits for a service another builder writes | Put its interface in a contract plan; test against a fake |
| A final "integration" or "end-to-end" task in a builder plan | Cross-builder checks are the design doc's test cases, run by your human partner |
| A plan carries a task's whole implementation | Builders write and verify most of the code; show code only where it is clearer than words |
| Writing plans in AsciiDoc to match the design doc, or copying its diagrams | Plans are Markdown with no diagrams; point to the design doc section |
| Three plans for work that splits two ways | Write two and say why |
| Two parallel plans in one project because their files differ | Different files still share one build; parallel plans share no project |
| Every builder adds tests to the solution's one shared test project | The contract plans create one test project per code project; each goes to that project's builder |
| Moving all of a project's tests out of the shared test project | Move only the tests the assigned work needs |
| A plan resolves something the design left open | Ask your human partner, write the answer into the design doc, then plan |
| Stopping for a choice only one builder's code can see | Not a gap; the builder decides |
| Asking what to name an interface between two builders | Not a gap; the contract plans define it |
| Writing plans with "Assumption:" notes because you were told not to ask | Ask anyway; assumptions become N builders' wrong code |
