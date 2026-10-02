# rux-skills

A Claude Code plugin of independent development-workflow skills — design, high-level test cases, debugging, verification, and code-review discipline. Each skill stands alone; compose them as needed.

Derived from [Superpowers](https://github.com/obra/superpowers) by Jesse Vincent (MIT).

## Skills

| Skill | Use when |
|---|---|
| `produce-design` | Before any creative work — explores intent, requirements, and design before implementation; architectural designs get a design doc with diagrams and UAT, E2E, and SIT test cases |
| `writing-builder-plans` | An approved design doc is ready to be built by several builders in parallel — writes contract plans and build plans, split where work can run side by side and ordered by dependency, so parallel plans share no file and none leaves anything broken |
| `systematic-debugging` | Any bug, test failure, or unexpected behavior, before proposing fixes |
| `verification-before-completion` | About to claim work is complete, fixed, or passing |
| `receiving-code-review` | Receiving code review feedback, before implementing suggestions |
| `writing-skills` | Creating, editing, or verifying skills |

## Install

In Claude Code:

```
/plugin marketplace add D:/tools/rux-skills
/plugin install rux-skills@rux-skills
```

Skills are invoked as `rux-skills:<skill-name>`.
