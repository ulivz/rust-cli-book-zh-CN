# 通过 15 分钟写一个命令行应用来学习 Rust

本教程将会引导你使用 [Rust] 写一个 CLI (command line interface) 应用，
大约需要十五分钟达到让你有一个正在运行的程序的地步，
在那之后，我们将继续调整我们的程序，
直到我们可以发布我们的小工具。

[Rust]: https://rust-lang.org/

你将学习有关如何开始的所有要领，
以及在哪里可以找到更多信息。
你可以随意跳过现在不需要知道的部分，
或随时返回阅读。

<aside>

**先决条件:**
本教程不会取代对编程的一般介绍，
同时希望您熟悉一些常见的概念。
你应该对使用命令行/终端感到满意。
如果您已经了解其他几种语言，
这可能是第一次接触 Rust 的好机会。

**寻求帮助:**
如果您在任何时候对所使用的功能感到不知所措或感到困惑，
请查看 Rust 随附的大量官方文档，
首先是《The Rust Programming Language》一书。 它带有大多数 Rust 安装（rustup doc）的知识，可在 [doc.rust-lang.org] 上在线获取。

[doc.rust-lang.org]: https://doc.rust-lang.org

也非常欢迎你提问——
Rust 社区以友好和乐于助人而著称。
看看 [community page]
查看人们讨论 Rust 的地方列表。

[community page]: https://www.rust-lang.org/community

</aside>

你想写什么样的项目？ 
我们从简单的事情开始怎么样：
让我们写一个小 `grep` 克隆。 
这是一个我们可以提供字符串和路径的工具，
它只会打印包含给定字符串的行。
我们称它为 `grrs`（发音为 “grass”）。

最后，我们希望能够像这样运行我们的工具：

```console
$ cat test.txt
foo: 10
bar: 20
baz: 30
$ grrs foo test.txt
foo: 10
$ grrs --help
[some help text explaining the available options]
```

<aside class="note">

**Note:**
这本书是为 [Rust 2018] 编写的。
代码示例也可用于 Rust 2015，
但你可能需要稍微调整一下；
例如，添加 `extern crate foo;` 调用。

确保你运行 Rust 1.31.0（或更高版本）
并且您在 `Cargo.toml` 文件的 `[package]` 中设置了 `edition = "2018"`

[Rust 2018]: https://doc.rust-lang.org/edition-guide/index.html

</aside>
