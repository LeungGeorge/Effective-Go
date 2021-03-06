# 简介（Introduction）

Go 是一门新的编程语言。尽管它借鉴了现有语言的思想，但其与众不同的特性使得在编写高效的 Go 程序时，与其他语言有着本质的区别。简单的把 C++ 程序或者 Java 程序直接翻译成 Go 程序，不大可能达到令人满意度结果（毕竟 Java 程序是用 Java 写的，不是用 Go）。另一方面，从 Go 的角度去思考问题，可以编写出完全不同但同样可行程序。换句话说，要想把 Go 写好，理解它的特性和风格是很重要的。了解 Go 语言的既定规则也很重要，如：命名，格式化，程序结构等等，这会让你使编写的 Go 程序便于其他程序员理解。

Go is a new language. Although it borrows ideas from existing languages, it has unusual properties that make effective Go programs different in character from programs written in its relatives. A straightforward translation of a C++ or Java program into Go is unlikely to produce a satisfactory result—Java programs are written in Java, not Go. On the other hand, thinking about the problem from a Go perspective could produce a successful but quite different program. In other words, to write Go well, it's important to understand its properties and idioms. It's also important to know the established conventions for programming in Go, such as naming, formatting, program construction, and so on, so that programs you write will be easy for other Go programmers to understand.


本文提供了编写简洁、地道的 Go 代码的技巧。建议先行阅读 [language specification](https://golang.org/ref/spec)，[the Tour of Go](https://tour.golang.org/) 和 [How to Write Go Code](https://golang.org/doc/code.html)。

This document gives tips for writing clear, idiomatic Go code. It augments the [language specification](https://golang.org/ref/spec), [the Tour of Go](https://tour.golang.org/), and [How to Write Go Code](https://golang.org/doc/code.html), all of which you should read first.

## 示例\(Examples\)
[Go package sources](https://golang.org/src/) 旨在不仅作为核心库，也充当着如何使用本语言的示例。此外，一些包包含可运行的、自包含的可执行示例，你可以从网站 [golang.org](https://golang.org/) 直接运行。比如 [这个](https://golang.org/pkg/strings/#example_Map)（如果需要，点击“Example”展开查看）。如果你有任何问题，诸如：如何解决问题，某些东西是如何实现的疑问，文档、代码以及基础库中的示例可以提供答案、思路、背景知识。

The [Go package sources](https://golang.org/src/) are intended to serve not only as the core library but also as examples of how to use the language. Moreover, many of the packages contain working, self-contained executable examples you can run directly from the [golang.org](https://golang.org/) web site, such as [this one](https://golang.org/pkg/strings/#example_Map) \(if necessary, click on the word "Example" to open it up\). If you have a question about how to approach a problem or how something might be implemented, the documentation, code and examples in the library can provide answers, ideas and background.

> 本书使用`gitbook`编写，采用中英文对照的形式编写。欢迎参与进来，只为写更好的 Go 代码。  
> [gitbook/effective-go](https://leunggeorge.gitbooks.io/effective-go/content/)  
> [github/effective-go](https://github.com/LeungGeorge/effective-go)  

## 目录

* [简介](README.md)
* [格式](/docs/chapter1.md)
* [注释](/docs/chapter2.md)
* [名字](/docs/chapter3.md)
* [分号](/docs/chapter4.md)
* [控制结构](/docs/chapter5.md)
* [函数](/docs/chapter6.md)
* [数据](/docs/chapter7.md)
* [初始化](/docs/chapter8.md)
* [方法](/docs/chapter9.md)
* [接口和其他类型](/docs/chapter10.md)
* [空白标识符](/docs/chapter11.md)
* [内嵌](/docs/chapter12.md)
* [并发](/docs/chapter13.md)
* [错误](/docs/chapter14.md)
* [web服务器](/docs/chapter15.md)



