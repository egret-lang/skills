# Egret-lang Build and Project Reference

Load this file when creating `egret.toml`, compiling `.eg` files, selecting
targets, linking native code, installing/building the toolchain, or working with
package layouts.

## Repository Paths

- `egret`: toolchain entrypoint.
- `egretc`: compiler entrypoint.
- `system/`: standard library modules written in Egret-lang.
- `runtime/`: C runtime for GC, IO, threads, networking, AIO, TLS, and support.
- `handbook/`: user-facing language handbook.
- `docs/spec/`: stable specification fragments.
- `tests/*.eg`: compiler/runtime conformance tests.
- `examples/*`: real package examples.
- `build/`: generated binaries, test outputs, and intermediate files.

## Build the Toolchain

```bash
./build-mac.sh
./build-linux.sh
build-windows.bat
```

Manual:

```bash
make -j4
make test
```

If LLVM is not discoverable:

```bash
export LLVM_CONFIG=/opt/homebrew/opt/llvm/bin/llvm-config
make -j4
```

## Single-file Builds

```bash
./egret build main.eg -o app
./app
```

Optimization and debug info:

```bash
./egret build main.eg -O2 -g -o app
```

Emit object or assembly:

```bash
./egret build main.eg -c -o app.o
./egret build main.eg -S -o app.s
```

Print linker command:

```bash
./egret build main.eg -o app --print linker-command
```

Freestanding examples:

```bash
./egret build kernel.eg -S -o kernel --freestanding --target riscv64-unknown-elf
./egret build kernel.eg -c -o kernel.o --freestanding --target x86_64-unknown-elf
```

## Package Builds

```bash
./egret build --manifest-path egret.toml
./egret build --manifest-path egret.toml --bin server
./egret build --manifest-path egret.toml --bins
```

Package builds default to `build/<bin-name>`. Windows targets append `.exe`.

## Manifest Forms

Single executable:

```toml
[package]
name = "hello_egret_package"
version = "0.1.0"

[bin]
main = "main.eg"

[lib]
modules = ["hello_egret_package"]
```

Multiple executable targets:

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

Rules:

- Do not mix `[bin]` and `[[bin]]` in one manifest.
- `[lib].modules` lists library module roots that belong to the package.
- Keep package names and module roots consistent.

Recommended layout:

```text
egret.toml
main.eg
src/<package_name>/<package_name>.eg
src/<package_name>/detail/detail.eg
```

Entry:

```egret
use hello_egret_package;

func main() -> Int {
    _ = hello_egret_package.print_hello_world();
    return 0;
}
```

Library:

```egret
module hello_egret_package;

func print_hello_world() -> Int {
    print("hello world");
    return 0;
}
```

## Build Flags

```text
--json-error
--sysroot <path>
--target <triple>
--manifest-path <egret.toml>
--bin <name>
--bins
-P, --package-path <path>
--locked
--offline
--linker <path>
--link-arg <arg>
--print linker-command
--freestanding
--entry <sym>
--ldscript <path>
--no-std
--no-gc
--no-class
--no-exception
--no-reflect
--unsafe
--log-perf
--gc=<on|off>
-O0|-O1|-O2|-O3
-g
-c
-S
```

Target triples:

```text
aarch64-apple-darwin
x86_64-apple-darwin
aarch64-unknown-linux-gnu
x86_64-unknown-linux-gnu
loongarch64-unknown-linux-gnu
mips64el-unknown-linux-gnuabi64
riscv64-unknown-linux-gnu
x86_64-pc-windows-msvc
aarch64-pc-windows-msvc
riscv64-unknown-elf
x86_64-unknown-elf
aarch64-unknown-elf
```

Environment:

```bash
EGRET_SYSROOT=/path/to/sysroot
EGRET_PACKAGE_PATH=/path/to/packages
EGRET_TARGET=x86_64-unknown-linux-gnu
EGRET_LINKER=cc
EGRET_LINK_ARGS=-no-pie
EGRET_FREESTANDING_LINKER=ld.lld
LLVM_CONFIG=/opt/homebrew/opt/llvm/bin/llvm-config
```

## Install Script Context

The repository may contain `install.sh` for release packages. A pipe installer
must read interactive input from `/dev/tty`, not ordinary stdin:

```bash
curl -fsSL https://egret-lang.org/install.sh | sh
curl -fsSL https://egret-lang.org/install.sh | sh -s -- --yes
curl -fsSL https://egret-lang.org/install.sh | sh -s -- --dir ~/.local/bin --yes
```

## Make Targets

Common repository targets:

```bash
make -j4
make test
make run-perf
make runtime-stress
make gc-stress
make fuzz-smoke
make clean
```

Stress script:

```bash
sh tests/egretperf_mem_stress_complex.sh --threads 10 --iters 100000000
```
