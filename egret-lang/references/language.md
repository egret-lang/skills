# Egret-lang Language Reference

Load this file when writing or reviewing `.eg` source syntax, types, control
flow, classes, generics, Norms, async functions, native/unsafe code, or
conditional compilation.

## Related Style Rules

For formatting, naming, module boundaries, comments, and repository-level code
style, load `references/coding-style.md`. This file focuses on language syntax
and semantic patterns.

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

## Types

Common types:

```text
Int Int8 Int16 Int32 Int64
UInt8 UInt16 UInt32 UInt64
Float Float64
Bool String CString Void Ptr Any
T?
immut T
Lazy(T)
Future(T)
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

## Declarations

```egret
const SIZE: Int = 4096;
internal const SECRET_LIMIT: Int = 64;

type UserId = Int;

enum Color {
    Red,
    Green,
    Blue,
}

func add(a: Int, b: Int = 1) -> Int {
    return a + b;
}

async func load() -> String, ErrCode {
    return "ok", std.OK;
}

extern func c_strlen(s: CString) -> Int;
```

Default parameters must be trailing. Variadic parameters are stable only in the
trailing position.

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

Defer and dispose:

```egret
defer cleanup();
dispose obj;
```

## Operators

Stable precedence from high to low:

1. Postfix calls, field access, indexing.
2. Unary `!`, unary `-`.
3. `*`, `/`, `%`.
4. `+`, `-`.
5. `<<`, `>>`.
6. `<`, `>`, `<=`, `>=`.
7. `==`, `!=`.
8. Bitwise `&`, `^`, `|`.
9. Logical `&&`, `||`.
10. Ternary `?:`.

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
    var value: Int;

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

Rules:

- Type parameters and value parameters share `<...>`.
- Defaults must be trailing.
- Value generic arguments must be compile-time constants.
- Provide explicit type arguments for ambiguous calls.

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
