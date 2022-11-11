# 更好的错误处理

我们都只能接受会发生错误的现实，
与很多其他语言相比，
在使用 Rust 时，很难不去注意和处理这个现实。
因为没有例外，
所有可能的错误状态通常都编码在函数的返回类型中。

## Results

像 [`read_to_string`] 这样的函数不会返回字符串，
相反，它返回一个 [`Result`]，
其中包含一个 `String`，
或者某种类型的错误，
（在这个 case 中是  [`std::io::Error`]）。

[`read_to_string`]: https://doc.rust-lang.org/1.39.0/std/fs/fn.read_to_string.html
[`Result`]: https://doc.rust-lang.org/1.39.0/std/result/index.html
[`std::io::Error`]: https://doc.rust-lang.org/1.39.0/std/io/type.Result.html

你该如何知道这些错误是哪个呢？
由于 `Result` 是一个 `enum` (枚举)，
你可以使用 `match` 来检查它是哪个变体（`variant`）：

```rust,no_run
let result = std::fs::read_to_string("test.txt");
match result {
    Ok(content) => { println!("File content: {}", content); }
    Err(error) => { println!("Oh noes: {}", error); }
}
```

<aside>

**注意:**
如果你不确定在 Rust 中的 `enums` 是什么，以及是如何工作的？
[请移步 Rust Handbook 的这一章节](https://doc.rust-lang.org/1.39.0/book/ch06-00-enums.html)
赶上进度。

</aside>

## Unwrapping

现在，我们能够访问文件的内容了，
但是在 `match` block 之后我们真的不能用它做任何事，
为此，我们需要以某种方式来处理错误情，
这里的挑战在于，`match` block 的所有分支都必须返回类型相同的东西，
但是这里有一个巧妙的技巧可以解决这一问题：

```rust,no_run
let result = std::fs::read_to_string("test.txt");
let content = match result {
    Ok(content) => { content },
    Err(error) => { panic!("Can't deal with {}, just exit here", error); }
};
println!("file content: {}", content);
```

我们可以使用在 match block 之后的 `content` 中的 String，
如果 `result` 是一个错误，这个 String 将不会存在，
但是由于程序会在运行到我们使用 `content` 的位置之前就会退出，
所以没关系。

<aside>
</aside>

这可能看起来有点粗暴，
但是很方便。
如果你的程序需要去访问去访问一个文件，同时在这个文件不存在时不做任何事，
退出程序是有个有效的策略，
在 `Results` 中甚至有一个快捷方法 `unwrap` 能够直接完成这件事：

```rust,no_run
let content = std::fs::read_to_string("test.txt").unwrap();
```

## 无需 panic

当然，中断程序不是处理错误的唯一方式，
除了 `panic!`，我们也可以简单地书写 `return`:

```rust,no_run
# fn main() -> Result<(), Box<dyn std::error::Error>> {
let result = std::fs::read_to_string("test.txt");
let content = match result {
    Ok(content) => { content },
    Err(error) => { return Err(error.into()); }
};
println!("file content: {}", content);
Ok(())
# }
```

然而，这改变我们的函数所需要的返回值类型，
事实上，我们的示例中一种隐藏着一些东西：
即函数签名在代码中的位置，
在上一个有 `return` 的示例中，
它变得很重要，
这里是 _完整_ 的例子：

```rust,no_run
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let result = std::fs::read_to_string("test.txt");
    let content = match result {
        Ok(content) => { content },
        Err(error) => { return Err(error.into()); }
    };
    println!("file content: {}", content);
    Ok(())
}
```

我们返回值是一个 `Result`!
这也是我们为什么可以在 `match` block 的第二个分支写 `return Err(error);`。
有看到最底部有一个 `Ok(())` 吗？
这是这个函数的默认的函数返回值，
同时意味着 "Result 是 OK 的，以及没有内容"。

<aside>

**注意:**
为什么这里没有写成 `return Ok(());` 这样?
这很容易 —— 也是完全有效的。
在 Rust 中最后一个任意的表达式都是函数的返回值，
并且在习惯上省略掉不必要的 `return`。

</aside>

## Question Mark

和调用 `.unwrap()` 是一个 shortcut 方法一样，
对于错误分支中包含有 `panic!` 的 `match`，
对于错误分支的 `return` 的 `match`，
我们有另一个 shortcut: `?`

没错，一个问号，
你可以将这个操作符插在一个类型为 `Result` 的值之后，
接着 Rust 将在内部展开成某种和我们写的
`match` 很相似的东西。

让我们尝试一下：

```rust,no_run
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let content = std::fs::read_to_string("test.txt")?;
    println!("file content: {}", content);
    Ok(())
}
```

非常简洁！

<aside>

**注意:**
这里还有一些事情发生，
不需要理解就可以使用它，
举例来说，
在 `main` 函数中的错误类型是 `Box<dyn std::error::Error>`，
但是我们看到上述的 `read_to_string` 返回了一个 [`std::io::Error`]，
这里能够工作，是因为 `?` 展开的代码中 _转换_ 了错误类型。

`Box<dyn std::error::Error>` 同时也是一个非常有趣的类型，
这是一个 Box，可以包含 _任意_
的实现了标准的 [`Error`](`std::error::Error`) trait 的类型
这意味着基本上所有的错误都可以被放置在这个 box 中，
所以我们可以使用 `?` 在所有返回 `Result` 的常用函数中使用 `?`。

[`std::error::Error`]: https://doc.rust-lang.org/1.39.0/std/error/trait.Error.html

</aside>

## Providing Context

当在你的 `main` 函数中使用 `?` 出现错误是正常，
但这并不太好，
举例来说，
当你运行 `std::fs::read_to_string("test.txt")?` 时，
但是文件 `test.txt` 又不存在，
你将会获得这样的输出：

```text
Error: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

在这种情况下，你的代码没有字面上包含文件名，
因此想要告诉用户哪个文件 `NotFound` 会很困难，
这里有好几种方式去解决这个问题。

举例来说，我们可以创建我们自己的错误类型，
接着使用它来构建一个自定义的错误信息：

```rust,ignore
{{#include errors-custom.rs}}
```

现在，
运行代码我们会得到我们的自定义的错误信息：

```text
Error: CustomError("Error reading `test.txt`: No such file or directory (os error 2)")
```

看起来不是很漂亮，
但是我们稍后可以非常轻松地为我们的输出调整调试的输出。

这种模式其实非常常见，
但是它存在一个问题：
我们没有存储原始的错误，
只有它的字符串标识。
一个经常被使用的库 [`anyhow`] 对这个问题有一个巧妙的解决方案：
和我们使用 `CustomError` 类似，
其 [`Context`] trait 可以在使用时增加一个 description，
此外，它也保留了原始错误，
所以我们得到了一个指出根本原因的错误信息的 "chain"。

[`anyhow`]: https://docs.rs/anyhow
[`Context`]: https://docs.rs/anyhow/1.0/anyhow/trait.Context.html

首先，让我们通过在 `Cargo.toml` 的 `[dependencies]` 中
增加 `anyhow = "1.0"` 来引入 `anyhow`。

完整的 example 看起来是这样：

```rust,ignore
{{#include errors-exit.rs}}
```

将会打印出一个错误：

```text
Error: could not read file `test.txt`

Caused by:
    No such file or directory (os error 2)
```
