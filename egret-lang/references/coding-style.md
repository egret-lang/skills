# Egret-lang Coding Style Reference

Load this file before creating or modifying Egret-lang source when style,
readability, module boundaries, API shape, or repository consistency matters.

## Formatting

- Use 4 spaces for indentation.
- Do not use tabs.
- Remove trailing whitespace.
- Write one simple statement per line.
- End simple statements with `;`.
- Do not compress multiple statements into one line.
- Block statements do not need a semicolon after the closing brace.
- Prefer readable vertical layout over dense code.

Recommended:

```egret
module app;

use std;
use strconv;

func add(a: Int, b: Int) -> Int {
    return a + b;
}

func main() -> Int {
    let value: Int = add(20, 22);
    print(strconv.itoa(value));
    return 0;
}
```

Avoid:

```egret
func main() -> Int { let x = 1; print("x"); return x; }
```

## Top-level Spacing

- `module` comes first.
- `use` comes after `module`.
- Keep one blank line between `module` and `use`.
- Keep one blank line between `module`/`use` and later declarations.
- Do not put blank lines between adjacent `use` declarations.
- Keep one blank line between `func` declarations.
- Keep one blank line between `class` and `class`, and between `class` and
  `func`.
- Keep one blank line around `norm` declarations.
- Do not insert blank lines between adjacent `extern` declarations.
- Keep one blank line between `extern` blocks and non-extern declarations.

## Language Keyword Choices

- Use `func`, not legacy `fn`.
- Use `loop`, not `for`.
- Use `skip`, not `continue`.
- Use `use`, not `import`.
- Use `ErrCode` returns, not exception-style control flow.
- Prefer dot module paths such as `net.tcp.connect`.
- Use `as` aliases only when they improve readability or resolve name clashes.

## Naming

Current repository examples use pragmatic names rather than a heavily enforced
style guide. Follow nearby code first. When creating new code:

- Module names: lowercase dot paths, e.g. `net.smtp`, `image.detail`.
- Package names: lowercase snake case in `egret.toml`, e.g.
  `hello_egret_package`.
- Functions and local variables: lowercase snake case, e.g. `read_config`,
  `path_text`.
- Constants: uppercase snake case for public constants when appropriate, e.g.
  `LIMIT`, `SIZE`.
- Classes and Norms: PascalCase, e.g. `Counter`, `StringBuilder`,
  `Hashable`.
- Generic type parameters: short PascalCase names such as `T`, `K`, `V`.
- Avoid vague names like `data`, `tmp`, `obj` unless the scope is tiny.

## Module and API Boundaries

- Put public API in short, stable modules.
- Put implementation details under `.detail`, `.internal`, or deeper modules.
- Use `internal` for symbols users should not depend on.
- Library files should usually declare `module`; tiny single-file CLIs may omit
  it.
- Use `use` declarations to show dependencies clearly at the top of the file.

Recommended public/internal split:

```egret
module image;

use image.detail;

func load(path: String) -> Image, ErrCode {
    return image.detail.load_impl(path);
}
```

```egret
module image.detail;

use std;

internal func load_impl(path: String) -> Image, ErrCode {
    return new Image(), std.OK;
}
```

## Error Handling Style

- Treat `ErrCode` as ordinary control flow.
- Handle every `ErrCode` in production-style code.
- Return `std.OK` on success.
- Prefer `T, ErrCode` for APIs that may fail and return a value.
- Prefer `ErrCode` for APIs that may fail without returning a value.
- Do not encode errors as empty strings or `-1` unless the API explicitly
  documents that convention.
- In tests, returning a nonzero `Int` from `main` is acceptable.

Recommended:

```egret
func read_config(path_text: String) -> String, ErrCode {
    let text: String, err: ErrCode = fs.read_text(path_text);
    if err != std.OK {
        return "", err;
    }
    return text, std.OK;
}
```

## Object-oriented Style

- Use `class` for state plus behavior.
- Use plain functions for pure computation or stateless helpers.
- `init` should leave the object usable.
- Avoid half-initialized objects that require a second setup call.
- Keep inheritance shallow.
- Prefer composition over inheritance for code reuse.
- Use `internal` for fields that represent resource handles, caches, internal
  indexes, or invariants.
- Resource classes should provide idempotent cleanup where practical.

## Resource and Native Style

- Wrap raw handles or `Ptr` values in a small typed API.
- Mark native handle fields `internal`.
- Keep unsafe code in narrow modules.
- Do not spread `unsafe` calls across business logic.
- If code uses `use unsafe`, make sure the build command includes `--unsafe`.

## Standard Library Style

- Prefer standard modules over custom helpers for common tasks.
- Use `std` for common primitives and error codes.
- Use `strings` for string utilities beyond basic `std` operations.
- Use `strconv` for number/string conversion.
- Use `fs` and `path` for filesystem work.
- Use `collections.Vector` for ordered append/read.
- Use `collections.Map` for keyed lookup.
- Use `collections.Set` for uniqueness.
- Use `collections.Heap` for priority behavior.
- Use `net.foo` for synchronous networking.
- Use `aio.foo` for async networking.
- Collections should use Norm constraints rather than ad hoc comparator
  function pointers.

## Async and Service Style

- Use `async func` and `await` consistently within async flows.
- Close sockets/files on every error path.
- Prefer multi-process workers plus async IO for high-performance services when
  practical.
- Use threads for explicit CPU or blocking work after measuring runtime costs.
- Keep cross-process/shared mutable state minimal.

## Test Style

- Preserve `// EXPECT_EXIT`, `// EXPECT_STDOUT`, `// EXPECT_COMPILE_FAIL`, and
  `// EXPECT_BUILD_ARGS`.
- Keep tests small and single-purpose.
- Make failure return codes distinct when possible.
- For compile-fail tests, keep the failure focused so diagnostics remain clear.

Example:

```egret
// EXPECT_EXIT: 0
// EXPECT_STDOUT: ok

func main() -> Int {
    print("ok");
    return 0;
}
```

## Comments

- Use comments to explain non-obvious invariants, ownership, platform behavior,
  or unsafe/native assumptions.
- Do not comment obvious assignments.
- Preserve special generated or test comments such as `@AI %{{-- ... --}}%` and
  `// EXPECT_*`.

## Generated Code Checklist

Before finalizing generated Egret code:

- Does it use 4 spaces and no tabs?
- Are `module` and `use` ordered correctly?
- Are declarations separated by readable blank lines?
- Are all `ErrCode` values handled?
- Are public APIs short and internal implementation details hidden?
- Does unsafe/native code stay behind a narrow API?
- Is the smallest useful `egret build` or test command clear?
