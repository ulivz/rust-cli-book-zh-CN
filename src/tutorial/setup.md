# 项目设置

如果你还没有准备好，
首先在你的电脑上[安装 Rust]
(他可能会耗费几分钟)，
接着，打开 Terminal，进入你想要放置你的应用代码的项目目录。

[安装 Rust]: https://www.rust-lang.org/tools/install

通过在目录中运行 `cargo new grrs` 来初始化一个项目
如果你查看这个被创建的 `grrs` 目录
你将会发现对于一个 Rust 项目需要的典型的设置：

- `Cargo.toml` 文件包含了当前项目的元信息
  包含了我们用到的 `dependencies`、`external` 的库的青岛。
- `src/main.rs` 文件，是当前项目的 entry 的入口文件。

如果你在 `grrs` 目录中运行 `cargo run`，
你将会得到一个 "Hello World"，至此，你的项目准备好了。

## 可能的运行过程

```console
$ cargo new grrs
     Created binary (application) `grrs` package
$ cd grrs/
$ cargo run
   Compiling grrs v0.1.0 (/Users/pascal/code/grrs)
    Finished dev [unoptimized + debuginfo] target(s) in 0.70s
     Running `target/debug/grrs`
Hello, world!
```
