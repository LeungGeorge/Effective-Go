# effective go（中英对照）

格式问题是最具争议但影响最小的问题。人们可以适应不同的格式风格，但如果他们不必这样做就更好了（鬼知道有没有强迫症），这样就不必专门花时间去讨论遵循相同风格的问题了。问题是如何在没有冗长风格指南的情况下实现这个乌托邦。

> Formatting issues are the most contentious but the least consequential. People can adapt to different formatting styles but it's better if they don't have to, and less time is devoted to the topic if everyone adheres to the same style. The problem is how to approach this Utopia without a long prescriptive style guide.

在 Go 中，我们使用一种不同寻常的方法，让机器础处理大多数格式化问题。程序 `gofmt`（或者 `go fmt`，在包级别操作而不是源文件级别）读取 Go 程序，以标准风格缩进和垂直对齐，保留并在必要时重新格式化注释。如果你想知道如何处理一些新的布局情况，运行 `gofmt`，如果结果看起来不对，那就重新组织你的程序（或者报个 `gofmt` 的 bug ），不要绕过这个问题。

> With Go we take an unusual approach and let the machine take care of most formatting issues. The`gofmt`program \(also available as`go fmt`, which operates at the package level rather than source file level\) reads a Go program and emits the source in a standard style of indentation and vertical alignment, retaining and if necessary reformatting comments. If you want to know how to handle some new layout situation, run`gofmt`; if the answer doesn't seem right, rearrange your program \(or file a bug about`gofmt`\), don't work around it.

例如，不必花时间去对注释进行排版，`gofmt` 对帮助你做到。给定一个声明：

> As an example, there's no need to spend time lining up the comments on the fields of a structure.`Gofmt`will do that for you. Given the declaration

```go
type T struct {
    name string // name of the object
    value int // its value
}
```

`gofmt` 将会重新按列进行排列：

> `gofmt`will line up the columns:

```go
type T struct {
    name    string // name of the object
    value   int    // its value
}
```

标准包中断所有`Go`代码都已经使用 `gofmt` 格式化过。

保留了一些格式细节，非常简洁：

**缩进**

* `gofmt` 默认使用 `tabs` 缩进，只有在必要时才会使用空格。

**每行长度**

* Go 中没有对行长度作限制。不必担心溢出穿孔卡。如果觉得一行太长，可以改为多行并以tab缩进。

**括号**

* Go 比 C 和 Java 需要的括号更少：控制结构（`if`，`for`，`switch`）无需括号。此外，运算符优先层次也更短、更清晰。因此，`x<<8 + y<<16` 的含义也就由空格表明了，这是与其他语言不同的。

> All Go code in the standard packages has been formatted with`gofmt`.
>
> Some formatting details remain. Very briefly:
>
> **Indentation**
>
> * We use tabs for indentation and`gofmt`emits them by default. Use spaces only if you must.
>
> **Line length**
>
> * Go has no line length limit. Don't worry about overflowing a punched card. If a line feels too long, wrap it and indent with an extra tab.
>
> **Parentheses**
>
> * Go needs fewer parentheses than C and Java: control structures \(`if`,`for`,`switch`\) do not have parentheses in their syntax. Also, the operator precedence hierarchy is shorter and clearer, so  `x<<8 + y<<16` means what the spacing implies, unlike in the other languages.



