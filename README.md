本书使用`gitbook`编写，采用中英文对照的形式编写。欢迎参与进来，只为写更好的 Go 代码。

gitbook：[点击阅读](https://leunggeorge.gitbooks.io/effective-go/content/)

github：[https://github.com/LeungGeorge/Effective-Go](https://github.com/LeungGeorge/Effective-Go)

---

Go 是一种新的编程语言。虽然借鉴了现有编程语言的思想，但它有一些不同寻常的特性，使得高效的 Go 程序，在性质上不同于它的亲属语言编写的程序。直截了当的把 C++ 程序或者 Java 程序翻译成 Go 程序，不大可能打到令人满意度结果（毕竟 Java 程序不是用 Go 写的）。另一方面，从 Go 的角度看来这个问题，可能创造一个成功的但完全不同的程序。换句话说，要想把 Go 写好，理解它的特性和习惯用法是很重要的。了解 Go 编程中既有的约定也很重要，比如命名，格式化，程序结构等等，这会让你使编写的 Go 程序更易于理解。

> Go is a new language. Although it borrows ideas from existing languages, it has unusual properties that make effective Go programs different in character from programs written in its relatives. A straightforward translation of a C++ or Java program into Go is unlikely to produce a satisfactory result—Java programs are written in Java, not Go. On the other hand, thinking about the problem from a Go perspective could produce a successful but quite different program. In other words, to write Go well, it's important to understand its properties and idioms. It's also important to know the established conventions for programming in Go, such as naming, formatting, program construction, and so on, so that programs you write will be easy for other Go programmers to understand.

本文提供了编写清晰、惯用的 Go 代码的技巧。建议先行阅读 [language specification](https://golang.org/ref/spec)，[the Tour of Go](https://tour.golang.org/) 和 [How to Write Go Code](https://golang.org/doc/code.html)。

> This document gives tips for writing clear, idiomatic Go code. It augments the [language specification](https://golang.org/ref/spec), [the Tour of Go](https://tour.golang.org/), and [How to Write Go Code](https://golang.org/doc/code.html), all of which you should read first.

### 示例

[Go package sources](https://golang.org/src/) 旨在不仅作为核心库，也充当着如何使用本语言的示例。此外，一些包包含可运行的、自包含的可执行示例，你可以从网站 [golang.org](https://golang.org/) 直接运行。比如 [这个](https://golang.org/pkg/strings/#example_Map)（如果需要，点击“Example”展开）。如果你有如何处理问题，或者如何实现问题点的问题，基础库中断文档、代码和示例可以为你提供答案、思路、背景。

> The [Go package sources](https://golang.org/src/) are intended to serve not only as the core library but also as examples of how to use the language. Moreover, many of the packages contain working, self-contained executable examples you can run directly from the[golang.org](https://golang.org/)web site, such as [this one](https://golang.org/pkg/strings/#example_Map) \(if necessary, click on the word "Example" to open it up\). If you have a question about how to approach a problem or how something might be implemented, the documentation, code and examples in the library can provide answers, ideas and background.



