跟C一样，Go 的正式语法使用分号结束语句，但与C不同的是，这些分号在源码中不可见。取而代之的是，词法分析器会使用一个简单的规则，在扫描时自动插入分号，所以输入文本中几乎没有它。

具体规则是这样的。在新的一行之前，最后如果是一个这样的标识符（包括`int`、`float64`），一个基本的文字，如数字、字符串常量、或者是下面符号中的一个

> Like C, Go's formal grammar uses semicolons to terminate statements, but unlike in C, those semicolons do not appear in the source. Instead the lexer uses a simple rule to insert semicolons automatically as it scans, so the input text is mostly free of them.
>
> The rule is this. If the last token before a newline is an identifier \(which includes words like`int`and`float64`\), a basic literal such as a number or string constant, or one of the tokens

```go
break continue fallthrough return ++ -- ) }
```

词法分析器总会在标记后面插入一个分号。这可以总结为：“如果新的一行紧跟一个语句的结束符，就会插入一个分号”。

也可以在右大括号前省略分号，如下语句

> the lexer always inserts a semicolon after the token. This could be summarized as, “if the newline comes after a token that could end a statement, insert a semicolon”.
>
> A semicolon can also be omitted immediately before a closing brace, so a statement such as

```go
  go func() { for { dst <- <-src } }()
```

无需分号。习惯用法是，Go程序只在像`for`循环这样的语句中使用分号，以分割初始化器，条件，延续元素。分号还用于分割一行的多条语句。

插入分号的一个结果是，你不能把控制语句的左大括号放在下一行。如果你确实这样做了，括号前就会插入一个分号，这将导致不可预知的结果。要这样写：

> needs no semicolons. Idiomatic Go programs have semicolons only in places such as`for`loop clauses, to separate the initializer, condition, and continuation elements. They are also necessary to separate multiple statements on a line, should you write code that way.
>
> One consequence of the semicolon insertion rules is that you cannot put the opening brace of a control structure \(`if`,`for`,`switch`, or`select`\) on the next line. If you do, a semicolon will be inserted before the brace, which could cause unwanted effects. Write them like this

```go
if i < f() {
    g()
}
```

而不是这样：

> not like this

```go
if i < f()  // wrong!
{           // wrong!
    g()
}
```



