# 输出

## 打印 "Hello World"

```rust
println!("Hello World");
```

好吧，这很容易，
Great，让我们进入下一个主题。

## 使用 println

你可以使用 `println!` macro 来打印
几乎全部的东西。
这个 macro 有一写非常赞的能力，
但也需要写一些特别的语法。
它期望你能够写一个字符串文本作为第一个参数，
其中包含了一些占位符，
这些占位符将会被接下来的 arguments 参数的值所填充，

举个例子：

```rust
let x = 42;
println!("My lucky number is {}.", x);
```

将会打印出：

```console
My lucky number is 42.
```

上面的字符串中的大括号（`{}`）便是其中的占位符之一，
这是默认的占位符类型，
它尝试以人类可读的方式来打印给定的值。
但并非所有的类型都可以做到这一点，
这也是为什么还有一个 "调试标识"，
你可以通过填充占位符的大括号来实现，就像这样： `{:?}`.。

举例来说：

```rust
let xs = vec![1, 2, 3];
println!("The list is: {:?}", xs);
```

会打印出：

```console
The list is: [1, 2, 3]
```

如果你期望你的数据类型可以被打印用于调试或者日出输出，
在大多数情况下，你可以在其定义上方增加 `#[derive(Debug)]`。

<aside>

**注意:**
“用户友好的” 打印是使用 [`Display`] trait,
调试输出使用 [`Debug`] trait（人类可读，但是是面向开发者的），
关于更多你可以在 `println!` 中使用的语法的信息，
你可以在 [`std::fmt` 的文档] 找到。

[`Display`]: https://doc.rust-lang.org/1.39.0/std/fmt/trait.Display.html
[`Debug`]: https://doc.rust-lang.org/1.39.0/std/fmt/trait.Debug.html
[std::fmt]: https://doc.rust-lang.org/1.39.0/std/fmt/index.html

</aside>

## 打印错误

打印错误应当通过 `stderr` 来完成，
能够更加容易地将它们通过管道传输到文件或者更多工具中。

<aside>

**Tip：**
在大多数操作系统中，
一个程序可以写到两个输出流中，
On most operating systems， `stdout` 和 `stderr`。
`stdout` 是程序的实际输出，
而 `stdout` 允许错误，还有其他信息能够与 `stdout` 分开，
这样的话，
输出可以被存储到一个文件或者通过管道传输到另一个程序中，
而错误可以被展示给用户。
</aside>

在 Rust 中，这是通过
`println!` 和 `eprintln!` 实现的，
前者打印到 `stdout`，
而后者输出到 `stderr`：

```rust
println!("This is information");
eprintln!("This is an error! :(");
```

<aside>

**请留意**: 打印 [escape codes] 可能会非常危险，
它会使用户的 Terminal 进入一个奇怪的状态，
手动打印时请务必非常小心！

[escape codes]: https://en.wikipedia.org/wiki/ANSI_escape_code

理想情况下，处理原始的 escape codes 时，
你应该使用一个类似 `ansi_term` 的 crate，
来让你（还有你的用户）的生活更轻松。

</aside>

## 关于 printing 的性能说明

打印信息到 Terminal 是出奇地慢！
如果你在一个循环中调用类似 `println!` 的方法，
它很容易成为一个原本很快的程序的瓶颈。
为了提升速度，
你有两件事可以做。

首先，
你可能期望减少
实际 “flush” 到终端的写入次数
_每次_, `println` 告诉系统刷新到终端，
因为打印到每一个新行是非常常见的， 
如果你不需要这样做，
你可以将你的 `stdout` 句柄包装在一个 [`BufWriter`] 里，
默认情况下，最多可以缓冲 8kb，
(当你想要立即打印的时候，
你仍然可以在这个 `BufWriter` 上调用 `.flush()`。)

```rust
use std::io::{self, Write};

let stdout = io::stdout(); // get the global stdout entity
let mut handle = io::BufWriter::new(stdout); // optional: wrap that handle in a buffer
writeln!(handle, "foo: {}", 42); // add `?` if you care about errors here
```

第二，
这有助于获得对 `stdout` (或 `stderr`) 的锁定，
并使用 `writeln!` 直接打印到他们。
这可以避免系统一遍又一遍地锁定和解锁 `stdout`。

```rust
use std::io::{self, Write};

let stdout = io::stdout(); // get the global stdout entity
let mut handle = stdout.lock(); // acquire a lock on it
writeln!(handle, "foo: {}", 42); // add `?` if you care about errors here
```

你也可以结合这两种方法。

[`BufWriter`]: https://doc.rust-lang.org/1.39.0/std/io/struct.BufWriter.html

## 展示一个 progress bar

一些 CLI 应用运行小于一秒，
其他的一些可能花费几分钟或者几小时。
如果你正在编写后一种类型的程序，
你可能想要向用户展示正在发生的事情，
为了达到这一目的，你应该打印出一些有用的状态更新，
理想的情况下，是一个易于消费的结构。

使用 [indicatif] crate，
你可以为你的程序添加 progress bar，
以及小的 spinners，
这里有一个简单的示例：

```rust,ignore
{{#include output-progressbar.rs:1:9}}
```

想了解更多信息，
请查看 `indicatif` 的 [文档][indicatif docs] 和 [示例][indicatif examples]。

[indicatif]: https://crates.io/crates/indicatif
[indicatif docs]: https://docs.rs/indicatif
[indicatif examples]: https://github.com/mitsuhiko/indicatif/tree/master/examples

## Logging

为了更容易理解我们的程序中发生的事情，
我们可能想添加一些日志（Logging）语句。
这在编写应用程序时通常很容易。
但是半年后再次运行这个程序时，它会变得非常有用。

在某些方面，
日志记录与使用 `println` 相同，
除了你可以指定消息的重要性。
通常可以使用的级别是 _error_、_warn_、_info_、_debug_ 和 _trace_
（_error_ 具有最高优先级，_trace_ 最低）。

要将简单的日志记录添加到你的应用程序，
你需要两件事：
[log] crate（包含以日志级别命名的宏）
和一个 _adapter_，它实际上会将日志输出写入有用的地方。
能够使用日志适配器（将会让程序）非常灵活：
例如，您可以使用它们不仅将日志写入终端
也可以到 [syslog] 或者一个中心的日志服务器。

[syslog]: https://en.wikipedia.org/wiki/Syslog

由于我们现在只关心编写 CLI 应用程序，
因此（我们会建议你使用）[env_logger] ，这是一个易于使用的适配器。
它被称为 “env” 记录器，
因为你可以通过环境变量来指定应用程序打印哪些部分
（以及你要在哪个级别记录它们）。
它将在您的日志消息前面加上时间戳，
以及日志消息来自的模块。
由于库也可以使用 `log`，
你也可以轻松配置他们的日志输出。

[log]: https://crates.io/crates/log
[env_logger]: https://crates.io/crates/env_logger

这里有一个简单的示例：

```rust,ignore
{{#include output-log.rs}}
```

假设你在 Linux 和 macOS 上有一个名为 `src/bin/output-log.rs` 的文件，
你可以像这样运行它：

```console
$ env RUST_LOG=info cargo run --bin output-log
    Finished dev [unoptimized + debuginfo] target(s) in 0.17s
     Running `target/debug/output-log`
[2018-11-30T20:25:52Z INFO  output_log] starting up
[2018-11-30T20:25:52Z WARN  output_log] oops, nothing implemented!
```

在 Windows 的 PowerShell 里, 你可以这样运行：

```console
$ $env:RUST_LOG="info"
$ cargo run --bin output-log
    Finished dev [unoptimized + debuginfo] target(s) in 0.17s
     Running `target/debug/output-log.exe`
[2018-11-30T20:25:52Z INFO  output_log] starting up
[2018-11-30T20:25:52Z WARN  output_log] oops, nothing implemented!
```

在 Windows 的 CMD 里, 你可以这样运行：

```console
$ set RUST_LOG=info
$ cargo run --bin output-log
    Finished dev [unoptimized + debuginfo] target(s) in 0.17s
     Running `target/debug/output-log.exe`
[2018-11-30T20:25:52Z INFO  output_log] starting up
[2018-11-30T20:25:52Z WARN  output_log] oops, nothing implemented!
```

`RUST_LOG` is the name of the environment variable
you can use to set your log settings.
`env_logger` also contains a builder
so you can programmatically adjust these settings,
and, for example, also show _info_ level messages by default.

There are a lot of alternative logging adapters out there,
and also alternatives or extensions to `log`.
If you know your application will have a lot to log,
make sure to review them,
and make your users' life easier.

`RUST_LOG` 是环境变量的名称，
你可以使用它来设置您的日志设置。
`env_logger` 还包含一个构建器，
因此你可以通过编程的方式来调整这些设置，
并且，例如，默认情况下还显示 _info_ 级别的消息。

这里有很多可供选择的日志适配器，
以及 `log` 的替代或者扩展。
如果你知道您的应用程序需要记录很多日志，
请确保对它们进行 Review，
并使你的用户的生活更轻松。

<aside>

**Tip:**
经验表明，即使是不太有用的 CLI 程序最终也可能会被使用多年
（特别是如果它们是作为临时解决方案），
如果你的应用程序不起作用，
并且有人（例如，将来的你）想要弄清楚为什么，
是都支持传递 `--verbose` 以获得额外的日志输出，
可以会产生几分钟或者几小时的调试时间的差异。
在 [clap-verbosity-flag] crate 中包含了一个快速的方法，
它使用 `clap` 将 `--verbose` 添加到项目中。

[clap-verbosity-flag]: https://crates.io/crates/clap-verbosity-flag

</aside>
