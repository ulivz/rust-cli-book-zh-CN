# 解析命令行参数

一个典型的调用我们的 CLI 工具的方式如下：

```console
$ grrs foobar test.txt
```

我们期望我们的程序去查找 `test.txt`
并打印那些包含 `foobar` 的行，
那么我们如何拿到这两个值呢？

在 CLI Program 名称之后的文本通常被称为 `command-line arguments`
或者 `command-line flags` (他们通常长这样: `--this`).
操作系统内部通常将它们认为是一些字符串的清单 —— 粗略地说，这些字符串被空格分开。

这里有好几种方式去思考 arguments，
以及如何将它们解析成某种利用使用的形式。
你也需要去告诉你的 CLI Program 的用户有哪些可以传递的 arguments
以及这些 arguments 所期望的格式。

## Getting the arguments

标准库中包含了一个函数 [`std::env::args()`], 
为你提供了一个用于获取指定的 arguments 的 [iterator]
第一个入口 (索引为 `0`) 将会是你的 CLI Program 的名称 (如 `grrs`),
接下来的部分则是用户写的。

[`std::env::args()`]: https://doc.rust-lang.org/1.39.0/std/env/fn.args.html
[iterator]: https://doc.rust-lang.org/1.39.0/std/iter/index.html

以这种方式获取原始的 arguments 非常容易 (写在文件 `src/main.rs` 中 `fn main() {` 的之后。)

```rust,ignore
{{#include cli-args-struct.rs:10:13}}
```

使用 `cargo run -- pattern path` 测试输出:

```console
$ cargo run -- pattern path
    Finished dev [unoptimized + debuginfo] target(s) in 0.03s
     Running `target/debug/grrs pattern path`
pattern: pattern
path: path
```

## CLI arguments 作为类型

与其把他们想象成一堆文本，
将 CLI arguments 视为一个表示你的程序的输入的自定义的数据类型通常是值得的。

让我们看 `grrs foobar test.txt`:
这里有两个 arguments:
第一个是 `pattern` (要查找的字符串),
接着第二个是 `path` (要搜索的文件).

关于他们我们还需要说哪些东西吗？
好吧，首先，两者都是必须的。
我们还没有谈及任何默认值，
所以我们希望我们的用户总是提供两个值，
因此，我们可以说一下这些参数的类型：
`pattern` 应该是一个字符串，
而第二个参数 `path` 应该是一个文件的路径。

在 Rust 中，围绕所处理的数据来组织程序是非常常见的，
所以这种查看 CLI arguments 的方式非常适合。
让我们从这里开始（在文件 `src/main.rs` 中 `fn main() {` 之前）:

```rust,ignore
{{#include cli-args-struct.rs:3:7}}
```

这里定义了一个新的结构（一个 [`struct`]），
他包含了两个字段 `pattern`、`path` 来存储数据

[`struct`]: https://doc.rust-lang.org/1.39.0/book/ch05-00-structs.html

<aside>

**Aside:**
[`PathBuf`] 与 [`String`] 类似，但是（设计上）是为了跨平台工作的路径。

[`PathBuf`]: https://doc.rust-lang.org/1.39.0/std/path/struct.PathBuf.html
[`String`]: https://doc.rust-lang.org/1.39.0/std/string/struct.String.html

</aside>

现在，我们仍然需要将 Program 的实际参数转换为这种形式。
一种方式是手动解析我们从操作系统拿到的字符串列表，然后自己构建这个结构，可能是这样：

```rust,ignore
{{#include cli-args-struct.rs:10:17}}
```

这可以工作，但是不太方便，
你将如何去处理 `--pattern="foo"` 或  `--pattern "foo"` 的需求呢？
你会如何去实现 `--help`?

## 使用 Clap 解析 CLI arguments

一个更棒的方式是去使用众多可用的库的一个，
在 Rust 生态中，解析命令行参数最流行的库叫作 [`clap`]，
它拥有所有你期望的能力，
包括对 [Sub Command]、[Shell completion] 还有好的 Help 信息的支持。

[`clap`]: https://docs.rs/clap/
[Sub Command]: https://docs.rs/clap/latest/clap/trait.Subcommand.html
[Shell completion]: https://docs.rs/clap_complete/

首先，让我们通过在 `Cargo.toml` 的 `[dependencies]` 中
增加 `clap = { version = "4.0", features = ["derive"] }` 
来引入 `clap`

现在，我们可以在代码中写 `use clap::Parser;` 了，
并在我们的 `struct Cli` 上方增加 `#[derive(Parser)]`
让我们在这个过程中也写一些文档注释。

代码最终会像这样（在文件 `src/main.rs` 中 `fn main() {` 之前)：

```rust,ignore
{{#include cli-args-clap.rs:3:12}}
```

<aside class="node">

**注意:**
你可以将许多自定义的属性添加到字段中，
举例来说,
如果您想将此字段用于 `-o` 或 `--output` 之后的参数，
你可以增加 `#[arg(short = 'o', long = "output")]`，
想要了解更多信息，
请移步 [clap 的文档][`clap`].

</aside>

在 `Cli` struct 的正下方，我们的模板包含它的 `main` 函数。
当程序启动时，它会调用这个函数，
其第一行是：

```rust,ignore
{{#include cli-args-clap.rs:14:16}}
```

这将尝试将命令行参数解析到我们的 `Cli` struct 中。

但如果失败了怎么办？
这就是这种方法的美妙之处：
Clap 知道可以哪些字段期望出现，
以及它们的预期格式是什么。
它可以自动生成一个不错的 `--help` 消息，
以及给出一些不错的错误信息
并在你输入 `--putput` 时建议你传入 `--output`。

<aside class="note">

**注意:**
`parse` 方法是在你的 `main` 函数中使用的。
失败时，
它会打印出错误或帮助信息，
并立即退出程序。
请不要在其他地方使用！

</aside>

## Wrapping up

你的代码现在应该长这样：

```rust,ignore
{{#include cli-args-clap.rs}}
```

如果不带任何参数运行它：

```console
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 10.16s
     Running `target/debug/grrs`
error: The following required arguments were not provided:
    <pattern>
    <path>

USAGE:
    grrs <pattern> <path>

For more information try --help
```

我们可以在使用 `cargo run` 时直接通过在 `--` 后面写参数来传递参数：

```console
$ cargo run -- some-pattern some-file
    Finished dev [unoptimized + debuginfo] target(s) in 0.11s
     Running `target/debug/grrs some-pattern some-file`
```

如你看到的，
没有输出。
这是对的：
这只是意味着没有错误，我们的程序结束了。

<aside class="exercise">

**读者练习:**
让这个程序输出它的参数！

</aside>
