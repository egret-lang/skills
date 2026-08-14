# Egret-lang Testing and Debugging Reference

Load this file when validating code, writing tests, handling diagnostics,
debugging build failures, performance regressions, runtime stress, or GC issues.

## Basic Verification

Single file:

```bash
./egret build file.eg -o /tmp/file
/tmp/file
```

Package:

```bash
./egret build --manifest-path egret.toml
```

Full repository:

```bash
make test
```

Performance/runtime:

```bash
make run-perf
make runtime-stress
make gc-stress
sh tests/egretperf_mem_stress_complex.sh --threads 10 --iters 100000000
```

## Diagnostics

Emit JSON diagnostics:

```bash
./egret build bad.eg -o /tmp/bad --json-error
```

Print linker command:

```bash
./egret build main.eg -o app --print linker-command
```

Build for a specific target:

```bash
./egret build main.eg -o app --target aarch64-apple-darwin
```

## Test Metadata

Preserve these comments in `tests/*.eg`:

```egret
// EXPECT_EXIT: 0
// EXPECT_STDOUT: ok
// EXPECT_COMPILE_FAIL
// EXPECT_BUILD_ARGS: --unsafe
```

They are consumed by test tooling and are not ordinary comments.

## Common Failure Patterns

- `egret.toml not found`: use explicit input build
  `./egret build main.eg -o app`, run inside a package, or pass
  `--manifest-path`.
- Unsafe module unavailable: add `--unsafe` or avoid `use unsafe`.
- Linker errors: run with `--print linker-command`, inspect `EGRET_LINKER` and
  `EGRET_LINK_ARGS`.
- Wrong platform: pass explicit `--target`.
- Package resolution errors: check `[lib].modules`, `module` declarations,
  `EGRET_PACKAGE_PATH`, `-P`, `--locked`, and `--offline`.
- Type/norm errors: add explicit type arguments and verify required
  `norm impl` exists.

## AI Coding Verification Workflow

Before editing:

1. Identify whether the task touches syntax, package config, stdlib, runtime, or
   tests.
2. Load only the relevant reference files.
3. Inspect nearby existing `.eg` files for local style.

After editing:

1. Compile the direct target.
2. Run the generated binary if behavior matters.
3. Run `make test` for language, stdlib, compiler, runtime, or shared behavior.
4. Run performance/stress commands only when the task touches runtime,
   allocator, GC, collections, networking, or perf scripts.

## Runtime Performance Notes

Known performance-sensitive areas:

- GC allocation and safepoints.
- STW pause time.
- Conservative stack scanning.
- Allocator/free/trim paths.
- String concat/substr/streq hot paths.
- Vector/Map typed hot paths.
- AIO/TCP/SMTP/Redis/Memcache services.

For high-performance network services, prefer multi-process workers plus async IO
unless the runtime’s multithreaded GC/allocator behavior has been measured and
shown acceptable for the workload.

## Shell Script Checks

For shell scripts such as `install.sh`:

```bash
/bin/sh -n install.sh
```

For pipe installers:

```bash
curl -fsSL https://egret-lang.org/install.sh | sh
curl -fsSL https://egret-lang.org/install.sh | sh -s -- --yes
```

Interactive prompts in a pipe installer must read from `/dev/tty`, not standard
input.
