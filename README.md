# Egret-lang Skill

This directory contains the Egret-lang skill for AI Coding Agents. Its purpose
is to let an agent progressively load focused reference documents and then write
Egret-lang code that matches the project syntax, coding style, build workflow,
and verification expectations.

## Purpose

This skill solves three practical problems:

1. It helps an agent quickly decide when Egret-lang-specific rules apply.
2. It splits syntax, standard library usage, build commands, templates, and
   debugging workflows into reference files that can be loaded on demand.
3. It avoids loading the full language handbook for every task, reducing context
   noise while improving generated-code consistency.

## Directory Structure

```text
skills/egret-lang/
├── README.md
├── SKILL.md
└── references/
    ├── build-and-projects.md
    ├── coding-style.md
    ├── language.md
    ├── stdlib.md
    ├── templates.md
    └── testing-and-debugging.md
```

## File Responsibilities

- `SKILL.md`: the skill entry point. It should only contain trigger conditions,
  mandatory rules, reference routing, and minimal verification commands.
- `references/coding-style.md`: coding style, including indentation, spacing,
  naming, module boundaries, error handling, OOP, unsafe/native usage, test
  comments, and the generated-code checklist.
- `references/language.md`: language syntax and semantic patterns, including
  types, declarations, control flow, classes, generics, Norms, `ErrCode`, async,
  unsafe, and conditional compilation.
- `references/build-and-projects.md`: build commands, `egret.toml`, package
  layout, targets, link flags, install scripts, and Makefile targets.
- `references/stdlib.md`: standard library modules and common APIs, including
  `std`, `strings`, `strconv`, `collections`, `fs`, `path`, `net`, and `aio`.
- `references/templates.md`: copyable templates, including single-file CLIs,
  packages, multi-bin projects, file tools, async main functions, thread workers,
  and test files.
- `references/testing-and-debugging.md`: verification, diagnostics, test
  metadata, common failures, stress testing, and debugging workflows.

## Loading Strategy

An agent should read `SKILL.md` first, then load the smallest necessary
reference set for the task:

- Writing or modifying `.eg` source: `coding-style.md` + `language.md`
- Creating a project or package: `build-and-projects.md` + `templates.md`
- Using the standard library: `stdlib.md`
- Writing tests or handling compile failures: `testing-and-debugging.md`
- Writing network services: `coding-style.md` + `language.md` + `stdlib.md` +
  `templates.md`
- Debugging runtime, GC, performance, or stress tests:
  `testing-and-debugging.md`, plus build docs when needed

## Maintenance Rules

- Keep `SKILL.md` short. Do not move large syntax sections, API lists, or
  templates back into the entry file.
- Add new language syntax to `references/language.md`.
- Add new coding conventions to `references/coding-style.md`.
- Add new build flags, package layouts, and installation workflows to
  `references/build-and-projects.md`.
- Add new standard library APIs to `references/stdlib.md`.
- Add new copyable examples to `references/templates.md`.
- Add new testing, diagnostics, stress, and debugging workflows to
  `references/testing-and-debugging.md`.
- Prefer repository facts as source material: `handbook/`, `docs/spec/`,
  `system/`, `examples/`, `tests/`, and `./egret help build`.

## Verification

After modifying this skill, at least check:

```bash
find tools/skills/egret-lang -maxdepth 3 -type f -print | sort
wc -l tools/skills/egret-lang/SKILL.md tools/skills/egret-lang/references/*.md
```

If example code changes, verify it with real `./egret build ...` commands when
practical so the examples continue to match current language behavior.
