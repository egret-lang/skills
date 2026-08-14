# Egret-lang Standard Library Reference

Load this file when selecting imports, using standard library APIs, writing IO,
networking, collections, strings, async IO, crypto, encoding, threading, or OS
code.

Import style:

```egret
use std;
use strconv;
use strings;
use collections;
use fs;
use path;
use net.tcp;
use aio.tcp;
```

If repository files are available, verify exact APIs in `system/**/*.eg`.

## Core `std`

Common APIs:

```text
std.len(s)
std.byte_at(s, i)
std.streq(a, b)
std.strcmp(a, b)
std.concat(a, b)
std.substr(s, start, len)
std.argv_get(argc, argv, idx)
std.stdin_read_line()
std.cpu_count()
std.StringView
std.StringBuilder
std.OK
std.ERR_INVALID
std.ERR_TIMEOUT
std.ERR_NOT_FOUND
```

Example:

```egret
use std;

func demo() -> Void {
    print("hello");
    if std.streq("a", "a") {
        print("same");
    }
}
```

## `strconv`

```text
strconv.atoi(s)
strconv.itoa(x)
strconv.parse_uint64(s)
strconv.format_uint64(x)
strconv.parse_float(s)
strconv.format_float(x)
strconv.parse_float64(s)
strconv.format_float64(x)
strconv.parse_bool(s)
strconv.format_bool(b)
```

```egret
use strconv;

func demo() -> Void {
    let text: String = strconv.itoa(42);
    let value: Int = strconv.atoi("42");
    print(text);
    print(strconv.itoa(value));
}
```

## `strings`

```text
strings.contains(s, sub)
strings.index(s, sub)
strings.index_from(s, sub, start)
strings.last_index(s, sub)
strings.count(s, sub)
strings.trim_space(s)
strings.trim_left(s)
strings.trim_right(s)
strings.trim(s)
strings.trim_prefix(s, prefix)
strings.trim_suffix(s, suffix)
strings.strip_prefix(s, prefix)
strings.strip_suffix(s, suffix)
strings.to_lower(s)
strings.to_upper(s)
strings.to_lower_ascii(s)
strings.to_upper_ascii(s)
strings.starts_with(s, prefix)
strings.ends_with(s, suffix)
strings.repeat(s, n)
strings.replace_all(s, pat, repl)
strings.replace_first(s, pat, repl)
strings.split(s, sep)
strings.fields(s)
strings.join_parts(parts, sep)
```

```egret
use strings;

func demo() -> Void {
    if strings.starts_with("egret-lang", "egret") {
        print(strings.to_upper_ascii("ok"));
    }
}
```

## Collections

```egret
use collections;

func demo() -> Int {
    let xs = new collections.Vector<Int>();
    xs.push(10);
    xs.push(20);
    let first: Int = xs.get(0);
    xs.set(0, 30);

    let m = new collections.Map<String, Int>();
    m.set("a", 1);
    if m.has("a") {
        return m.get_or("a", 0);
    }
    return first;
}
```

Common collection classes:

```text
collections.Vector<T>
collections.Map<K, V>
collections.Set<T>
collections.ZSet<T>
collections.List<T>
collections.Heap<T>
collections.Ring<T>
```

Custom keys usually need `collections.Hashable<T>`:

```egret
norm impl collections.Hashable<MyKey> {
    func hash(x: MyKey) -> Int {
        return x.a * 31 + x.b;
    }

    func eq(a: MyKey, b: MyKey) -> Bool {
        return a.a == b.a && a.b == b.b;
    }
}
```

## Filesystem and Paths

Filesystem:

```text
fs.file_exists(path) -> Bool
fs.file_write_text(path, data) -> Bool
fs.file_read_text(path) -> String
fs.open(path, flags, mode) -> Int, ErrCode
fs.open_file(path, flags, mode) -> fs.File, ErrCode
fs.create(path) -> fs.File, ErrCode
fs.read_text(path) -> String, ErrCode
fs.write_text(path, data) -> Int, ErrCode
fs.remove(path) -> Int, ErrCode
fs.exists(path) -> Bool, ErrCode
fs.mkdir(path) -> Int, ErrCode
fs.mkdir_all(path) -> Int, ErrCode
fs.close(handle)
```

Paths:

```text
path.join_path(a, b)
path.base(path)
path.dir(path)
path.ext(path)
path.clean(path)
```

Example:

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
```

## Networking

Synchronous TCP:

```text
net.tcp.connect(host, port, timeout_ms) -> Int, ErrCode
net.tcp.listen(host, port, backlog) -> Int, ErrCode
net.tcp.accept(fd, timeout_ms) -> Int, ErrCode
net.tcp.sock_port(fd) -> Int, ErrCode
net.tcp.send(fd, data, timeout_ms) -> Int, ErrCode
net.tcp.recv(fd, max_bytes, timeout_ms) -> String, ErrCode
net.tcp.close(fd)
```

AIO TCP:

```text
aio.tcp.connect(host, port, timeout_ms) -> Future(Int, ErrCode)
aio.tcp.listen(host, port, backlog) -> Int, ErrCode
aio.tcp.accept(fd, timeout_ms) -> Future(Int, ErrCode)
aio.tcp.send(fd, data, timeout_ms) -> Future(Int, ErrCode)
aio.tcp.recv(fd, max_bytes, timeout_ms) -> Future(String, ErrCode)
aio.tcp.close(fd)
```

AIO example:

```egret
use aio.tcp;
use std;

async func fetch(host: String, port: Int) -> Int, ErrCode {
    let fd: Int, err: ErrCode = await aio.tcp.connect(host, port, 5000);
    if err != std.OK {
        return 0, err;
    }

    let sent: Int, send_err: ErrCode = await aio.tcp.send(fd, "ping", 5000);
    if send_err != std.OK {
        aio.tcp.close(fd);
        return 0, send_err;
    }

    let data: String, recv_err: ErrCode = await aio.tcp.recv(fd, 4096, 5000);
    aio.tcp.close(fd);
    if recv_err != std.OK {
        return sent, recv_err;
    }
    return std.len(data), std.OK;
}
```

## Module Families

```text
aio.redis
aio.mysql
aio.smtp
aio.http
aio.https
aio.memcache
aio.websocket
bytes
compress.zlib
concurrent.thread_pool
coroutine
crypto.md5
crypto.sha1
crypto.sha256
crypto.aes
crypto.rsa
crypto.rand
encoding.base64
encoding.json
encoding.xml
fmt
io
math
net.http
net.https
net.memcache
net.mysql
net.redis
net.smtp
net.tls
net.udp
net.websocket
os
os.env
os.process
os.signal
protocol.redis
protocol.memcache
protocol.mysql
protocol.websocket
regexp
runtime
sort
sync
testing
thread
time
unsafe
```

## Service Runtime Guidance

For high-performance services in the current runtime:

- Prefer multi-process workers plus async IO when possible.
- Keep per-process worker state isolated.
- Use threads for explicit CPU or blocking work after measuring.
- Track GC/STW cost under load before choosing a thread-heavy design.
