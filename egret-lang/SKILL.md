---
name: "egret-lang"
description: "Writes, builds, tests, and reviews Egret-lang or egret-lang code. Invoke when creating/modifying .eg files, egret.toml, stdlib, or Egret build scripts."
---

# Egret-lang Coding Skill

Use this skill when a task involves Egret-lang source code, packages, examples,
tests, standard library usage, build commands, release/install scripts, or
runtime/performance workflows.

Keep this file small. Load reference files only when the task needs them.

## Mandatory First Steps

1. Identify the task type: language syntax, package/build, standard library,
   template generation, testing/debugging, or runtime/performance.
2. Load the matching reference file(s) from `references/` before writing code.
3. Inspect nearby project files when available; repository-local style wins over
   generic examples.
4. After editing `.eg`, `egret.toml`, build scripts, or tests, run the smallest
   useful verification command.

## Reference Routing

- `references/coding-style.md`: load before creating or modifying `.eg` code
  when formatting, naming, module boundaries, API shape, comments, or repository
  consistency matters.
- `references/language.md`: load for `.eg` syntax, types, `func`, `module`,
  `use`, control flow, `loop`, classes, `init`/`dispose`/`deinit`, inheritance,
  generics, Norms, `ErrCode`, async, unsafe, extern, and conditional
  compilation.
- `references/build-and-projects.md`: load for `egret build`, `egret.toml`,
  package layout, targets, link flags, install scripts, Makefile targets, and
  toolchain build commands.
- `references/stdlib.md`: load for standard library modules, imports, strings,
  strconv, collections, fs/path, net/aio, threads, protocols, crypto, encoding,
  and service-runtime guidance.
- `references/templates.md`: load when creating new files/packages/tests or when
  the user asks for example code/boilerplate.
- `references/testing-and-debugging.md`: load for validation, diagnostics,
  compile failures, runtime stress, perf checks, GC/performance work, or shell
  installer checks.

When multiple areas apply, load the smallest set that covers the task. Example:
new async network service = `coding-style.md` + `language.md` + `stdlib.md` +
`templates.md` + `build-and-projects.md`.

## Non-negotiable Egret Rules

- Use 4 spaces, no tabs.
- Use `func`, not legacy `fn`.
- Use `loop`, not `for`.
- Use `skip`, not `continue`.
- Put `module` first, then `use`, then declarations.
- Preserve `// EXPECT_*` test metadata.
- Treat `ErrCode` as explicit control flow; do not ignore errors in production
  style code.
- Compile changed `.eg` code with `./egret build ...` or the relevant package
  command.

## Source-of-truth Paths

If the repository is available, prefer these sources before guessing:

- `handbook/*.md` and `docs/spec/syntax.md` for language behavior.
- `docs/spec/abi.md` for entry points and ABI behavior.
- `system/**/*.eg` for standard library APIs.
- `examples/*` for real package layout and application patterns.
- `tests/*.eg` for compiler/runtime edge cases and expected behavior.
- `./egret help build` for current build flags.

## Minimal Verification Commands

```bash
./egret build main.eg -o app
./egret build --manifest-path egret.toml
make test
/bin/sh -n install.sh
```

Choose the command that matches the changed surface. Do not run broad perf or
stress tests unless the task touches runtime, GC, allocator, networking,
collections, or performance-sensitive behavior.
