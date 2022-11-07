# _grrs_ 的第一个实现

在关于命令行参数的一章后，
我们已经有了输入数据，
我们可以开始编写实际的工具了，
现在，我们的 `main` 函数只包含这一行：

```rust,ignore
{{#include impl-draft.rs:15:15}}
```

让我们从读取我们得到的文件开始：

```rust,ignore
{{#include impl-draft.rs:16:16}}
```

<aside>

**注意:**
看到这里的 [`.expect`] 方法了吗？
这是一个当这个值（这个 case 中是这个输入文件）无法被读取时能够使程序立即退出的快捷功能，
看起来不是很漂亮，
在下一章[更好的错误报告]中，
我们将研究如何改进这一点。

[`.expect`]: https://doc.rust-lang.org/1.39.0/std/result/enum.Result.html#method.expect
[更好的错误报告]:./errors.html

</aside>

现在，让我们遍历这些行，
并打印初满足我们（指定的）pattern 的每一行：

```rust,ignore
{{#include impl-draft.rs:18:22}}
```

## Wrapping up

你的代码现在应该如下所示：

```rust,ignore
{{#include impl-draft.rs}}
```

试试运行 `cargo run -- main src/main.rs`，它现在应该可以工作！

<aside class="exercise">

**读者练习:**
这不是最好的实现：
无论文件有多大，
它都将整个文件读入内存,
让我们想想办法优化
（一个可能的思路是使用 [`BufReader`]，而不是 `read_to_string()`）。

[`BufReader`]: https://doc.rust-lang.org/1.39.0/std/io/struct.BufReader.html

</aside>
