# Egret-lang Language Reference

Load this file when writing or reviewing `.eg` source syntax, types, control
flow, classes, generics, Norms, async functions, native/unsafe code, or
conditional compilation.

## Related Style Rules

For formatting, naming, module boundaries, comments, and repository-level code
style, load `references/coding-style.md`. This file focuses on language syntax
and semantic patterns.

## Complete Stable Syntax Checklist

This checklist is the minimum grammar surface an AI agent must remember before
claiming it knows Egret-lang syntax.

Lexical and literals:

- UTF-8 source files.
- Line comments `// ...`.
- Block comments `/* ... */`.
- AI block comments `@AI ... @END`.
- Identifiers: ASCII letter or `_`, followed by ASCII letters, digits, or `_`.
- Integer literals: decimal, binary `0b...`, octal `0o...`, hexadecimal
  `0x...`.
- Floating literals with a decimal point.
- String literals `"..."`.
- C string literals `c"..."`.
- Escapes: `\\`, `\"`, `\0`, `\n`, `\r`, `\t`, `\xHH`.
- Bool and nil literals: `true`, `false`, `nil`.

Program structure:

- Optional `module pkg.sub;`.
- `use module.path;`.
- `use module.path as alias;`.
- Qualified names through `.` in module paths and through `::` in source
  qualified identifiers.

Declarations:

- `func name(params) -> Type { ... }`.
- `async func name(params) -> Type { ... }`.
- `extern func name(params) -> Type;`.
- `extern async func name(params) -> Type;`.
- `builtin func name(params) -> Type;` for runtime/compiler builtins.
- `const NAME: Type = expr;`.
- `type Name = Type;`.
- `enum Name { A, B = 2, internal C = -1 }`.
- `class Name { ... }`.
- `class Child : Base { ... }`.
- `norm Name<T> { ... }`.
- `norm impl Name<Type> { ... }`.
- `internal` may prefix supported top-level declarations, fields, methods, norm
  signatures, and enum variants.

Types:

- Builtins: `Int`, `UInt`, `Int64`, `Int32`, `Int16`, `Int8`, `UInt64`,
  `UInt32`, `UInt16`, `UInt8`, `Bool`, `Float`, `Float32`, `Float64`,
  `String`, `CString`, `Void`, `Ptr`, `Any`, `ErrCode`.
- Runtime-backed: class types, `Future(T)`, `Future(T, ErrCode)`, `Lazy(T)`.
- Optional types: `T?`.
- Immutable view types: `immut T`.
- Function types: `(A, B) -> R`.
- Generic class types: `Name<T, 42, "key", true>`.

Statements:

- Block `{ ... }`.
- `let` and `var` local declarations, with or without explicit type.
- Pair local declaration: `let value: T, err: ErrCode = expr;`.
- Assignment `lhs = expr;`.
- Pair assignment `a, b = expr;`.
- Compound assignment `lhs += expr;`, `lhs -= expr;`.
- `if` / `else if` / `else`.
- `loop condition { ... }`.
- `loop (init; cond; step) { ... }`.
- `break;`.
- `skip;`.
- `return;`, `return expr;`, `return value, err;`.
- `defer expr;`.
- `dispose expr;`.
- Match statement.
- Expression statement.

Expressions:

- Literals and identifiers.
- Unary `-`, `!`, `~`, `&`.
- Binary arithmetic, shift, comparison, equality, bitwise, logical operators.
- Ternary `cond ? then_expr : else_expr`.
- Function/method calls.
- Generic call type arguments: `fn<T>(arg)`.
- Argument spreading: `fn(xs...)`.
- Field/member access: `obj.field`.
- Norm member access form: `Norm<T>.method` through parser-supported generic
  member syntax.
- Casts: `expr => Type`, `expr =>? Type`.
- Type test: `expr is Type`.
- Error propagation: `expr?`.
- `new Type(args...)`.
- Lambda `|params| -> Type { ... }`.
- `match expr { pattern => expr, ... }`.
- Field initializer expression: `Type { field: value, ... }`.
- `super`, `super.method(...)`, `super.init(...)`, `super<Base>.method(...)`.
- Async expressions: `await expr`, `spawn expr`, `join expr`.
- Lazy expressions: `lazy expr`, `force(expr)`.
- Infinite/range stream sugar: `[start, step, ...]` and `[start ...]`.

## File Shape

```egret
module demo.app;

use std;
use strconv;

const LIMIT: Int = 10;

type UserId = Int;

func main(argc: Int, argv: Int) -> Int {
    print("hello " + strconv.itoa(argc));
    return 0;
}
```

`module` is optional for simple single-file builds. `use` declarations must come
before ordinary declarations. Use dot-separated module paths such as
`net.tcp`.

`::` is accepted for source qualified identifiers in expression/type contexts,
while module paths use dots:

```egret
use collections;

let v: collections::Vector<Int> = new collections.Vector<Int>();
```

## Lexical Grammar

Source files are UTF-8 text. The portable syntax surface uses ASCII
punctuation, keywords, and identifiers. Non-ASCII data is safe inside string
literals as UTF-8 bytes.

Comments:

```egret
// line comment

/*
block comment
*/

@AI
agent metadata block
@END
```

Identifiers:

```text
[A-Za-z_][A-Za-z0-9_]*
```

Reserved words cannot be used as ordinary identifiers unless the grammar has a
special escape for a specific context. Some keyword-looking names such as
`match`, `dispose`, `spawn`, and `join` are accepted in selected identifier
positions by the parser; avoid relying on that for public APIs.

Literal forms:

```egret
let dec: Int = 123;
let bin: Int = 0b1010;
let oct: Int = 0o755;
let hex: Int = 0x2a;
let f: Float = 3.14;
let s: String = "line\n";
let c: CString = c"native\0";
let ok: Bool = true;
let no: Bool = false;
let maybe: String? = nil;
```

Stable string escapes are `\\`, `\"`, `\0`, `\n`, `\r`, `\t`, and `\xHH`.
C string literals must not contain embedded NUL after escape processing except
where the C string rule explicitly permits the terminator.

Reserved keyword groups:

```text
declarations/modules: func const internal extern class type enum norm impl module use as
values/control/objects: var let if else loop break skip return dispose new super immut match is defer true false nil
async: async await spawn join
lazy: lazy force
```

## Main Functions

```egret
func main() -> Int {
    return 0;
}

func main(argc: Int, argv: Int) -> Int {
    if argc > 1 {
        print(std.argv_get(argc, argv, 1));
    }
    return 0;
}

async func main(argc: Int, argv: Int) -> Int {
    return await run(argc, argv);
}
```

The parser accepts functions and methods without an explicit `-> Type`, but new
code should prefer explicit return types unless matching nearby code:

```egret
func log_ok() {
    print("ok");
}
```

## Types

Common types:

```text
Int UInt Int8 Int16 Int32 Int64
UInt8 UInt16 UInt32 UInt64
Float Float32 Float64
Bool String CString Void Ptr Any ErrCode
T?
immut T
Lazy(T)
Future(T)
Future(T, ErrCode)
(Int, String) -> Bool
```

Examples:

```egret
let n: Int = 42;
let ok: Bool = true;
let text: String = "hello";
let maybe: String? = nil;
let f: (Int, Int) -> Int = |a: Int, b: Int| -> Int {
    return a + b;
};
```

Cast and type test:

```egret
let any_value: Any = 42;
let n: Int = any_value => Int;
let maybe_text: String? = any_value =>? String;

if any_value is Int {
    print("int");
}
```

Optional values:

```egret
class Node {
    var next: Node?;
}

let maybe_node: Node? = nil;
```

Lazy values:

```egret
func expensive() -> Int {
    return 42;
}

func demo() -> Int {
    let value: Lazy(Int) = lazy expensive();
    return force(value);
}
```

## Declarations

```egret
const SIZE: Int = 4096;
internal const SECRET_LIMIT: Int = 64;

type UserId = Int;

enum Color {
    Red,
    Green = 2,
    internal Blue,
}

func add(a: Int, b: Int = 1) -> Int {
    return a + b;
}

async func load() -> String, ErrCode {
    return "ok", std.OK;
}

extern func c_strlen(s: CString) -> Int;
extern async func c_async_load() -> Int, ErrCode;
```

Default parameters must be trailing. Variadic parameters are stable only in the
trailing position.

`internal` can prefix top-level declarations, constants, classes, type aliases,
enums, enum variants, fields, methods, Norms, Norm signatures, and Norm impl
methods where the grammar allows it.

Variadic and spread examples:

```egret
func join_all(prefix: String, parts: ...String) -> String {
    return prefix;
}

func call_join(xs: collections.Vector<String>) -> String {
    return join_all("p:", xs...);
}
```

`builtin func` exists for runtime/compiler builtins. Production code should not
invent builtins; use it only when extending the compiler/runtime surface.

```egret
builtin func dispose_safepoint() -> Void;
```

Declaration syntax forms:

```text
internal func name(params) -> Type { ... }
async func name(params) -> Type { ... }
internal async func name(params) -> Type { ... }
extern func name(params) -> Type;
extern async func name(params) -> Type;
internal builtin func name(params) -> Type;
```

## Control Flow

```egret
let x: Int = 1;
x = x + 1;
x += 2;

if x > 0 {
    print("positive");
} else if x == 0 {
    print("zero");
} else {
    print("negative");
}

let i: Int = 0;
loop (; i < 5; ) {
    print(i);
    i = i + 1;
}

loop (let j: Int = 0; j < 4; j = j + 1) {
    if j == 2 {
        skip;
    }
    print(j);
}

loop true {
    break;
}
```

Ternary:

```egret
let label: String = ok ? "yes" : "no";
```

Match:

```egret
match code {
    0 => {
        print("zero");
    }
    1 => {
        print("one");
    }
    _ => {
        print("other");
    }
}
```

Match expression:

```egret
let name: String = match code {
    0 => "zero",
    1 => "one",
    _ => "other",
};
```

Stable match patterns are integer literals, string literals, `true`, `false`,
and `_` default.

Defer and dispose:

```egret
defer cleanup();
dispose obj;
```

`defer expr;` runs cleanup when leaving the current function/scope path.
`dispose expr;` lowers to `expr.dispose()` plus a dispose safepoint. Resource
`dispose` methods should be idempotent.

Local declarations and assignment forms:

```egret
let a: Int = 1;
let b = 2;
let c: Int;
var d: String = "mutable";
let value: String, err: ErrCode = load();
value, err = reload();
target.field = 10;
target.field += 1;
target.field -= 1;
```

## Operators

Stable precedence from high to low:

1. Postfix calls, field access, indexing.
2. Unary `!`, unary `-`, unary `~`, address-of `&`, `lazy`, `await`, `spawn`,
   `join`.
3. `*`, `/`, `%`.
4. `+`, `-`.
5. `<<`, `>>`.
6. `<`, `>`, `<=`, `>=`.
7. `==`, `!=`.
8. Bitwise `&`, `^`, `|`.
9. Logical `&&`, `||`.
10. Ternary `?:`.

Postfix operators:

```egret
let a: Int = obj.field;
let b: Int = fn<Int>(10);
let c: Rect = shape => Rect;
let maybe: Rect? = shape =>? Rect;
let ok: Bool = shape is Rect;
let value: String = read_text(path)?;
```

`expr?` propagates the `ErrCode` from a `T, ErrCode` expression. Use `_ = expr?;`
when intentionally discarding the successful value.

Object construction and field initialization:

```egret
let p: Point = new Point(1, 2);
let q: Point = Point { x: 1, y: 2 };
```

The grammar supports range/stream sugar in expression position:

```egret
let stream1 = [1, 2, ...];
let stream2 = [10 ...];
```

## Lambdas

```egret
let add: (Int, Int) -> Int = |a: Int, b: Int| -> Int {
    return a + b;
};
```

Prefer no-capture lambdas unless existing code proves capture support for the
target compiler version.

## Classes

```egret
class Counter {
    internal var value: Int = 0;

    func init(self: Counter, value: Int = 0) -> Void {
        self.value = value;
    }

    func inc(self: Counter) -> Int {
        self.value = self.value + 1;
        return self.value;
    }
}

func main() -> Int {
    let c: Counter = new Counter(41);
    print(c.inc());
    return 0;
}
```

Inheritance:

```egret
class Base {
    var x: Int;

    func init(self: Base, x: Int) -> Void {
        self.x = x;
    }

    func get(self: Base) -> Int {
        return self.x;
    }
}

class A<T> : Base {
    var y: T;

    func init(self: A<T>, x: Int, y: T) -> Void {
        super.init(x);
        self.y = y;
    }

    func get(self: A<T>) -> Int {
        return self.x + 1;
    }
}
```

`super`:

```egret
class Animal {
    var name: String;

    func init(self: Animal, name: String) -> Void {
        self.name = name;
    }

    func speak(self: Animal) -> String {
        return "unknown";
    }
}

class Dog : Animal {
    func init(self: Dog, name: String) -> Void {
        super.init(name);
    }

    func speak(self: Dog) -> String {
        return super<Animal>.speak() + ": woof";
    }
}
```

Rules:

- Egret uses single inheritance.
- `super.init(...)` is for derived-class construction and must be placed where
  constructor rules allow it; current diagnostics require it as the first
  statement for derived class initialization when needed.
- `super.method(...)` calls the immediate base implementation.
- `super<Base>.method(...)` selects an explicit base owner and must name a valid
  base class.

Field initializer expression:

```egret
class Point {
    var x: Int;
    var y: Int;
}

func origin() -> Point {
    return Point { x: 0, y: 0 };
}
```

Lifecycle:

```egret
class Guard {
    var id: Int;

    func init(self: Guard, id: Int) -> Void {
        self.id = id;
    }

    func deinit(self: Guard) -> Void {
        print("deinit");
    }
}
```

## Generics

```egret
func id<T>(x: T) -> T {
    return x;
}

class Box<T=Int> {
    var v: T;

    func init(self: Box<T>, v: T) -> Void {
        self.v = v;
    }

    func get(self: Box<T>) -> T {
        return self.v;
    }
}

func addN<N>(x: Int) -> Int {
    return x + N;
}
```

Norm-constrained type parameters:

```egret
func keep_hashable<T: collections.Hashable>(value: T) -> T {
    return value;
}
```

Generic method:

```egret
class Converter {
    func id<T>(self: Converter, value: T) -> T {
        return value;
    }
}
```

Class specialization:

```egret
class Cache<T> {
    var value: T;
}

class Cache<String> {
    var value: String;

    func len(self: Cache<String>) -> Int {
        return std.len(self.value);
    }
}
```

Rules:

- Type parameters and value parameters share `<...>`.
- Defaults must be trailing.
- Value generic arguments must be compile-time constants.
- Value generic arguments can be integers, floats, strings, booleans, or named
  constants accepted by the compiler.
- Type parameters can have Norm constraints: `<T: NormName>`.
- Provide explicit type arguments for ambiguous calls.

Generic argument grammar accepts both type arguments and value arguments:

```egret
class Field<Name, Required> {
    var label: String;
    var required: Bool;
}

let f: Field<"email", true> = new Field<"email", true>();
```

Generic call syntax uses parser-disambiguated type arguments before the call:

```egret
let x: Int = id<Int>(42);
```

## Norms

```egret
norm Printable<T> {
    func format(x: T) -> String;
}

class User {
    var name: String;
}

norm impl Printable<User> {
    func format(x: User) -> String {
        return x.name;
    }
}
```

Async Norm signatures and implementations must match async-ness:

```egret
norm AsyncRead<T> {
    async func read(path_text: String) -> T, ErrCode;
}

norm impl AsyncRead<String> {
    async func read(path_text: String) -> String, ErrCode {
        return "", std.OK;
    }
}
```

Custom `collections.Map` key:

```egret
use collections;

class MyKey {
    var a: Int;
    var b: Int;
}

norm impl collections.Hashable<MyKey> {
    func hash(x: MyKey) -> Int {
        return x.a * 31 + x.b;
    }

    func eq(a: MyKey, b: MyKey) -> Bool {
        return a.a == b.a && a.b == b.b;
    }
}
```

Norm impl syntax supports multiple target types for Norms that require them:

```egret
norm PairNorm<A, B> {
    func combine(a: A, b: B) -> String;
}

norm impl PairNorm<Int, String> {
    func combine(a: Int, b: String) -> String {
        return b;
    }
}
```

## Error Handling

Egret uses explicit `ErrCode`, not exceptions.

```egret
use std;

func pair(code: Int) -> Int, ErrCode {
    if code == 0 {
        return 0, std.ERR_INVALID;
    }
    return code + 10, std.OK;
}

func main() -> Int {
    let value: Int, err: ErrCode = pair(7);
    if err != std.OK {
        return 1;
    }
    print(value);
    return 0;
}
```

Async multi-return:

```egret
async func async_pair(code: Int) -> String, ErrCode {
    if code == 0 {
        return "", std.ERR_INVALID;
    }
    return "value", std.OK;
}

async func main() -> Int {
    let text: String, err: ErrCode = await async_pair(1);
    if err != std.OK {
        return 100;
    }
    print(text);
    return 0;
}
```

Error propagation with `?`:

```egret
use fs;
use std;

func load_config(path_text: String) -> String, ErrCode {
    let text: String = fs.file_read_text_result(path_text)?;
    return text, std.OK;
}

func ensure(path_text: String) -> ErrCode {
    _ = fs.ensure_dir(path_text)?;
    return std.OK;
}
```

Rules:

- `expr?` expects an expression returning `T, ErrCode` or compatible error
  shape.
- If the error is not `std.OK`, the current function returns the error.
- If success value is intentionally discarded, use `_ = expr?;`.
- Bare `expr?;` is rejected for value-returning expressions; make discarding
  explicit.

## Async, Threads, Native, Unsafe

Basic async:

```egret
async func nop() -> Void {
    return;
}

async func main() -> Int {
    await nop();
    return 0;
}
```

Async expressions:

```egret
async func work(id: Int) -> Int {
    return id * 2;
}

async func demo() -> Int {
    let f1 = spawn work(1);
    let f2 = spawn | | -> Int {
        return 2;
    }();

    let a: Int = join f1;
    let b: Int = await f2;
    return a + b;
}
```

Rules:

- `async func` marks a function that may suspend.
- `await expr` waits for a Future-style value.
- `spawn expr` starts an async task/coroutine expression.
- `join expr` joins a spawned task.
- `await`, `spawn`, and `join` are async-context operations; use them inside
  async functions unless an existing runtime API documents otherwise.

Thread spawn:

```egret
use thread;
use std;

func worker(arg: Any) -> Int {
    let n: Int = arg => Int;
    return n + 1;
}

func main() -> Int {
    let th, err = thread.spawn(worker, 41);
    if err != std.OK {
        return 1;
    }
    let rc, join_err = th.join();
    if join_err != std.OK {
        return 2;
    }
    print(rc);
    return 0;
}
```

Unsafe requires `--unsafe`:

```egret
use unsafe;

func demo(ptr: Ptr) -> Int {
    return unsafe.load64(ptr);
}
```

Native and unsafe syntax surface:

```egret
extern func native_len(s: CString) -> Int;

func ptr_demo(p: Ptr) -> Int {
    let addr: Ptr = &p;
    _ = addr;
    return 0;
}
```

Use native and pointer syntax only behind small APIs. Builds using `unsafe`
module APIs require `--unsafe`.

Conditional compilation:

```egret
#if EG_OS_LINUX
    print("linux");
#else
    print("not-linux");
#endif

#if EG_CPU_X64 && EG_ARCH_AMD64
    print("x64");
#endif
```
