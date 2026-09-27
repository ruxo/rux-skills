---
name: writing-builder-plans
description: Use when an approved design document is ready to be built by several builders working in parallel, before any implementation starts
---

# Writing Builder Plans

## Overview

Turn an approved design document into plans for a fixed number of
builders who work in parallel, each alone in their own session, until
done. They never talk to each other, so the plans make that possible:
every shared contract is fixed first in plan 0, every project and file
has one owner, and each builder's plan can be finished and tested with
only plan 0 and its own files.

Plans say what to build; builders write and verify most of the code,
test-first. Plan 0's contract is given in full. Elsewhere, a plan shows
code only where it says something more clearly than words — an exact
query, format or pattern — never a task's whole implementation.

**Announce at start:** "I'm using the writing-builder-plans skill to split the design into builder plans."

## 1. Inputs

- **The design doc** — a file, or a GitHub issue. If your human partner
  didn't give its path or issue URL, ask. Read it in full.
- **The builder count.** Look in your harness's persistent memory, then
  the project instruction file (`CLAUDE.md`, `AGENTS.md`, `GEMINI.md`).
  If it is not there, ask (suggest 3), then record the answer: in your
  harness's per-project memory if it has one; otherwise add a line to
  the project instruction file.
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
  sees — names, signatures, internal types. Plan 0 defines them.
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

## 3. Plan 0 — the contract

Plan 0 holds everything more than one builder depends on, written out in
full so every builder codes against the same thing:

- **API contracts** — the OpenAPI document (created or updated) for HTTP
  APIs between services; the schema additions for GraphQL
- **Shared types and interfaces** — exact names, fields, signatures
- **Storage schema** — collections or tables, fields, indexes
- **Messages, events and config keys**
- **Every hotspot edit, done once** — for example the registration line
  for a service a builder will own. Plan 0 creates that service as a
  compiling stub; the builder who owns it fills it in.
- **Test projects** — the new test projects and the moved tests of
  section 4's "One test project per code project".

Plan 0 is contract, not feature: no behavior, no business logic. It
compiles and leaves existing tests passing. Its contract files are
given in full; moved tests are listed as `from → to`.

One builder runs plan 0 alone, commits it and stops — builders 1 to N
start from that commit, each in its own working copy (a branch or
worktree) your human partner sets up.

That is the one commit a builder makes. Otherwise builders leave git
alone: no staging, no commits, unless the work cannot go on without
one. Staging and committing are your human partner's job.

## 4. Split the work

Split the rest into one plan per builder:

- **Disjoint files.** After plan 0, every file belongs to exactly one
  plan. A plan never edits a file another plan owns, nor a plan 0 file
  other than a stub it owns.
- **Independent.** Each plan can be finished, with its tests passing,
  using only plan 0 and its own files. Where it calls another builder's
  part, it calls the plan 0 interface and its tests use a fake.
- **One builder per project.** A project is source code that builds
  into one library or executable, or, for an interpreted language,
  deploys as one service — a .NET project, a Cargo crate, an npm
  package, a Go module, a Python service. Give each project to one
  builder; one builder may own several. Two builders in one project
  share its project file, lockfile and build, and one builder's
  half-finished work breaks the other's compile.
- **One test project per code project.** Builders test first, so each
  needs test projects of their own: every code project gets its own
  test project, owned by the builder who owns the code project. Where
  tests live inside the code project (Go test files, Rust test modules),
  there is nothing to create. When the solution has one test project
  shared by several code projects, never give it to one builder or
  split it between builders. Plan 0 creates a test project for each
  code project the build touches, empty and building, and adds it to
  the solution; builders' new tests go there. Plan 0 also moves,
  unchanged, the shared project's tests that the assigned work needs —
  tests of behavior a builder changes, or that a builder must update —
  into the new test project for their code project; they belong there
  anyway. A helper that
  tests left behind still use stays put, and the new test project
  references it. Every other test stays where it is: this is not a
  test migration.
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

Save plans 0 to N beside the design doc as Markdown (`.md`), whatever
the design doc's format. When the design doc is a GitHub issue, save
them where your human partner said (section 1). Each file name starts
with today's date, `YYYY-MM-DD-`, then a name you choose, and ends with
`- Plan N` — for example `2026-09-27-customer-tags - Plan 0.md` …
`2026-09-27-customer-tags - Plan 3.md`.

Plans have no diagrams. Where a task follows a flow, state or model the
design doc draws, name that design doc section instead of redrawing it.

Each builder plan contains, in order:

1. **Header** — title, the design doc's path or issue URL, the goal in
   one sentence, and "Start from the commit that contains plan 0."
2. **How to work** — this text, word for word:
   > Work alone until this plan is done. Build every task test-first — REQUIRED SUB-SKILL: Use rux-skills:test-driven-development. Before calling a task done, use rux-skills:verification-before-completion. Leave git alone: don't stage or commit unless the work cannot go on without it; your human partner commits your work. Change only the files under Files you own; if a task seems to need another file, the plan is wrong — stop and tell your human partner.
3. **Before you start** — only when section 5 gives this plan steps.
4. **Files you own** — every file this plan creates or changes.
5. **Tasks**, in build order. Each task has:
   - **Files** — the files it creates or changes
   - **Uses** — the plan 0 contracts it relies on, by name
   - **Tests pin** — each behavior as one line, situation → expected
     result; unit tests, or integration tests inside this plan's own
     part
   - **Enables** — the design doc's test-case IDs this task helps pass
6. **Done when** — the commands that must pass, from the project
   instruction file.

Plan 0 has the same header without the start line, then How to work
with two changes: drop the test-first sentence (plan 0 adds no
behavior), and replace the "Leave git alone" sentence with "When plan 0
is done, commit it: builders 1 to N start from that commit." Then
**Files you own** — every file it creates, changes or moves — then the
**contract** in full, then **Done when**.

## 7. Self-review

1. **Coverage:** every requirement and design section has a task; every
   test-case ID appears under Enables in some plan.
2. **Ownership:** across plans 1 to N, no file appears twice, and no
   project appears in two plans unless the hand-off says why.
3. **Independence:** no task needs anything from another plan except
   plan 0 contracts.
4. **Contract:** every name a task uses from another builder's part is
   defined in plan 0.
5. **Your human partner:** their steps appear only in Before you start.
6. **No design decisions:** every behavior traces to the design doc.
7. **No placeholders:** no "TBD", "handle errors appropriately",
   "similar to task N".
8. **Git:** only plan 0 tells its builder to commit; no task in plans 1
   to N stages or commits.

Fix issues inline.

## 8. Hand off

> "Plans written: `<plan 0 path>` … `<plan N path>`. Run plan 0 with one builder; it commits plan 0. Then start builders 1 to N from that commit, each in its own branch or worktree. Plans 1 to N merge in any order. Please review them."

Add the reasons section 4 asks for: why there are fewer plans than
builders, or why two builders share a project.

If they request changes, make them and run the self-review again. Then
stop. Building starts when your human partner starts the builders.

## Common Mistakes

| Mistake | Fix |
|---|---|
| A plan tells the builder to commit after each task | Builders leave staging and commits to your human partner; only plan 0 ends with a commit |
| Two plans each add a line to the same registration file | That edit is a hotspot: it goes in plan 0 |
| A task waits for a service another builder writes | Put its interface in plan 0; test against a fake |
| A final "integration" or "end-to-end" task in a builder plan | Cross-builder checks are the design doc's test cases, run by your human partner |
| A plan carries a task's whole implementation | Builders write and verify most of the code; show code only where it is clearer than words |
| Writing plans in AsciiDoc to match the design doc, or copying its diagrams | Plans are Markdown with no diagrams; point to the design doc section |
| Three plans for work that splits two ways | Write two and say why |
| Two builders in one project because the files are disjoint | Disjoint files still share one build; give the project to one builder |
| Every builder adds tests to the solution's one shared test project | Plan 0 creates one test project per code project; each goes to that project's builder |
| Moving all of a project's tests out of the shared test project | Plan 0 moves only the tests the assigned work needs |
| A plan resolves something the design left open | Ask your human partner, write the answer into the design doc, then plan |
| Stopping for a choice only one builder's code can see | Not a gap; the builder decides |
| Asking what to name an interface between two builders | Not a gap; plan 0 defines it |
| Writing plans with "Assumption:" notes because you were told not to ask | Ask anyway; assumptions become N builders' wrong code |
