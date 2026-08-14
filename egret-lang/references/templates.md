# Egret-lang Code Templates

Load this file when the user asks to create new Egret-lang code, packages,
tests, services, examples, or boilerplate.

## Single-file CLI

```egret
use std;
use strconv;

func main(argc: Int, argv: Int) -> Int {
    if argc < 2 {
        print("usage: app <name>");
        return 2;
    }

    let name: String = std.argv_get(argc, argv, 1);
    print("hello " + name);
    print("argc=" + strconv.itoa(argc));
    return 0;
}
```

Build:

```bash
./egret build main.eg -o app
./app world
```

## Package

`egret.toml`:

```toml
[package]
name = "demo"
version = "0.1.0"

[bin]
main = "main.eg"

[lib]
modules = ["demo"]
```

`src/demo/demo.eg`:

```egret
module demo;

use strconv;

func describe(x: Int) -> String {
    return "value=" + strconv.itoa(x);
}
```

`main.eg`:

```egret
use demo;

func main() -> Int {
    print(demo.describe(42));
    return 0;
}
```

Build:

```bash
./egret build --manifest-path egret.toml
./build/demo
```

## Multiple Binaries

```toml
[package]
name = "toolbox"
version = "0.1.0"

[[bin]]
name = "server"
main = "cmd/server.eg"

[[bin]]
name = "client"
main = "cmd/client.eg"

[lib]
modules = ["toolbox"]
```

Build:

```bash
./egret build --manifest-path egret.toml --bin server
./egret build --manifest-path egret.toml --bins
```

## Error-returning File Tool

```egret
use fs;
use std;

func read_config(path_text: String) -> String, ErrCode {
    let text: String, err: ErrCode = fs.read_text(path_text);
    if err != std.OK {
        return "", err;
    }
    return text, std.OK;
}

func main(argc: Int, argv: Int) -> Int {
    if argc < 2 {
        return 2;
    }

    let text: String, err: ErrCode = read_config(std.argv_get(argc, argv, 1));
    if err != std.OK {
        return 1;
    }

    print(text);
    return 0;
}
```

## Async Main

```egret
use std;
use aio.tcp;

async func run() -> Int, ErrCode {
    let fd: Int, err: ErrCode = await aio.tcp.connect("127.0.0.1", 6379, 1000);
    if err != std.OK {
        return 1, err;
    }
    aio.tcp.close(fd);
    return 0, std.OK;
}

async func main() -> Int {
    let rc: Int, err: ErrCode = await run();
    if err != std.OK {
        return 100;
    }
    return rc;
}
```

## Thread Worker

```egret
use std;
use thread;

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

## Generic Collection With Custom Key

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

func main() -> Int {
    let m: collections.Map<MyKey, Int> = new collections.Map<MyKey, Int>();
    let k: MyKey = new MyKey();
    k.a = 1;
    k.b = 2;
    _ = m.set(k, 9);
    print(m.get_or(k, 0));
    return 0;
}
```

## Test File

```egret
// EXPECT_EXIT: 0
// EXPECT_STDOUT: ok

func main() -> Int {
    print("ok");
    return 0;
}
```

Compile-fail test:

```egret
// EXPECT_COMPILE_FAIL

func main() -> Int {
    let x: Int = "not an int";
    return x;
}
```

Unsafe test:

```egret
// EXPECT_EXIT: 0
// EXPECT_BUILD_ARGS: --unsafe

use unsafe;

func main() -> Int {
    let p: Ptr = nil;
    _ = p;
    return 0;
}
```
