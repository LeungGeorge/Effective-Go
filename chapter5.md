Go的控制结构与C相关，但是在重要的方面存在差异。Go中没有`do`、`while`循环，只有一个略微广义的`for`；`switch`则更加灵活，`if`和`switch`接受类似于for的可选初始化语句；`break`和`continue`语句接受一个可选的标号来标识中断或继续什么；也有一些新的控制结构，包括类型`switch`和多路通信多路复用器（a multiway communications multiplexer），select。语法上，也略微不同：没有括号，控制结构体必须总是由大括号包起来。

> The control structures of Go are related to those of C but differ in important ways. There is no`do`or`while`loop, only a slightly generalized`for`;`switch`is more flexible;`if`and`switch`accept an optional initialization statement like that of`for`;`break`and`continue`statements take an optional label to identify what to break or continue; and there are new control structures including a type switch and a multiway communications multiplexer,`select`. The syntax is also slightly different: there are no parentheses and the bodies must always be brace-delimited.

### If

在Go中，一个`if`的例子大概长这样：

> In Go a simple`if`looks like this:

```go
if x > 0 {
    return y
}
```

强制性的大括号g7里在多行中编写简单语句。不管怎样，这样做是一个好的风格，尤其是当控制体包含一条控制一句时，比如`return`或者`break`。

由于if和switch可以接受初始化语句，那么常见的情况是用来创建一个局部变量。

> Mandatory braces encourage writing simple`if`statements on multiple lines. It's good style to do so anyway, especially when the body contains a control statement such as a`return`or`break`.
>
> Since`if`and`switch`accept an initialization statement, it's common to see one used to set up a local variable.

```go
if err := file.Chmod(0664); err != nil {
    log.Print(err)
    return err
}
```

在Go的基础库中，if语句不会不会流向下一条语句时——也就是说，控制体以`break`，`continue`，`goto`或者`return`结尾——这时，没有不要的`else`会被省略。

> In the Go libraries, you'll find that when an`if`statement doesn't flow into the next statement—that is, the body ends in`break`,`continue`,`goto`, or`return`—the unnecessary`else`is omitted.

```go
f, err := os.Open(name)
if err != nil {
    return err
}
codeUsing(f)
```

这是一个常见的示例，代码需要防范这这一系列的错误。如果代码走下去的都是成功流，可读性会很好。当出现错误时，立即消除它们。由于错误情况会以`return`语句结尾，因此代码不需要`else`语句。

> This is an example of a common situation where code must guard against a sequence of error conditions. The code reads well if the successful flow of control runs down the page, eliminating error cases as they arise. Since error cases tend to end in`return`statements, the resulting code needs no`else`statements.

```go
f, err := os.Open(name)
if err != nil {
    return err
}
d, err := f.Stat()
if err != nil {
    f.Close()
    return err
}
codeUsing(f, d)
```

### 重新声明和重新赋值

题外话， 上一节点最后一个例子演示了`:=`短声明的细节。该声明调用`os.Open`进行读取，

> An aside: The last example in the previous section demonstrates a detail of how the`:=`short declaration form works. The declaration that calls`os.Open`reads,

```go
f, err := os.Open(name)
```

这个语句声明了两个变量，`f`和`err`。几行之后，调用`f.Stat`进行读取，

> This statement declares two variables,`f`and`err`. A few lines later, the call to`f.Stat`reads,

```go
d, err := f.Stat()
```

这看起来像是声明了`d`和`err`。但是注意，这连个语句中都出现了`err`。这种重复是合法的：因为`err`是被第一个语句声明，在第二个语句中仅仅是重新赋值。这意味着`f.Stat`使用了前面已经声明的变量`err`，仅仅是给他赋予了新值而已。

> which looks as if it declares`d`and`err`. Notice, though, that`err`appears in both statements. This duplication is legal:`err`is declared by the first statement, but only\_re-assigned\_in the second. This means that the call to`f.Stat`uses the existing`err`variable declared above, and just gives it a new value.

在`:=`声明中，变量`v`即使声明过，也是可以再次出现的，前提是：

* 这个声明和已有声明`v`在相同的作用域（如果`v`已经在外面的作用域声明了，这里的声明将会创建一个新的变量`§`）
* 初始化中相应的值可以赋值给`v`，或者
* 在声明中至少有一个新的变量。

> In a`:=`declaration a variable`v`may appear even if it has already been declared, provided:
>
> * this declaration is in the same scope as the existing declaration of`v`\(if`v`is already declared in an outer scope, the declaration will create a new variable §\),
> * the corresponding value in the initialization is assignable to`v`, and
> * there is at least one other variable in the declaration that is being declared anew.

这种不同寻常的特性是纯粹的实用主义，例如，你会经常看到，在一个长的`if-else`链中，更易于使用单个`err`值。

§ 值得一提的是，在Go中，函数参数和返回值的作用域跟函数体是一致的。尽管它们在词法上在包裹函数体的大括号之外。

> This unusual property is pure pragmatism, making it easy to use a single`err`value, for example, in a long`if-else`chain. You'll see it used often.
>
> § It's worth noting here that in Go the scope of function parameters and return values is the same as the function body, even though they appear lexically outside the braces that enclose the body.

### For

在Go中，for循环跟C的相似——但又不完全相同。它统一了for和while，在Go中没有do-while语句。一共有三种形式，只有只有带有分号。

> The Go`for`loop is similar to—but not the same as—C's. It unifies`for`and`while`and there is no`do-while`. There are three forms, only one of which has semicolons.

```go
// Like a C for
for init; condition; post { }

// Like a C while
for condition { }

// Like a C for(;;)
for { }
```

短声明使得在循环中很容易声明索引变量。

> Short declarations make it easy to declare the index variable right in the loop.

```go
sum := 0
for i := 0; i < 10; i++ {
    sum += i
}
```

如果你正在循环遍历一个`array`，`slice`，`string`，或者`map`，亦或是从`channel`中读数据，那么可以使用`range`子句管理循环。

> If you're looping over an array, slice, string, or map, or reading from a channel, a`range`clause can manage the loop.

```go
for key, value := range oldMap {
    newMap[key] = value
}
```

如你仅仅需要`range`的第一项（健或者索引），就丢掉第二个：

> If you only need the first item in the range (the key or index), drop the second:

```go
for key := range m {
    if key.expired() {
        delete(m, key)
    }
}
```

如果你仅需要`range`的第二项（值），可以使用空白标识`_`下划线，以丢弃第一个：

> If you only need the second item in the range (the value), use the blank identifier, an underscore, to discard the first:

```go
sum := 0
for _, value := range array {
    sum += value
}
```

空白标识符有很多用处，接下来的一节我们会对此详述。

对于字符串，`range`为你可以做更过的事情，通过解析`UTF-8`编码来拆分单个`Unicode`编码点。错误的编码会小号一个字节，并产生一个替换性的符文`U+FFFD`。
（`rune`（相关的内置类型）这个词是`Go`中的术语，用以标记单个`Unicode`编码点。到[语言规范](https://golang.org/ref/spec#Rune_literals)查看更多详情）。循环

> The blank identifier has many uses, as described in a later section.
> 
> For strings, the range does more work for you, breaking out individual Unicode code points by parsing the UTF-8. Erroneous encodings consume one byte and produce the replacement rune U+FFFD. (The name (with associated builtin type) rune is Go terminology for a single Unicode code point. See [the language specification](https://golang.org/ref/spec#Rune_literals) for details.) The loop

```go
for pos, char := range "日本\x80語" { // \x80 is an illegal UTF-8 encoding
    fmt.Printf("character %#U starts at byte position %d\n", char, pos)
}
```

打印如下：

> prints

```go
character U+65E5 '日' starts at byte position 0
character U+672C '本' starts at byte position 3
character U+FFFD '�' starts at byte position 6
character U+8A9E '語' starts at byte position 7
```

最后，`Go`没有都好运算符，`++`和`--`谁语句而不是表达式。因此，如果你想在for中运行多个变量，你应该是用并行赋值（虽然这样会妨碍使用`++`和`--`）


> Finally, Go has no comma operator and ++ and -- are statements not expressions. Thus if you want to run multiple variables in a for you should use parallel assignment (although that precludes ++ and --).


```go
// Reverse a
for i, j := 0, len(a)-1; i < j; i, j = i+1, j-1 {
    a[i], a[j] = a[j], a[i]
}
```

### Switch

Go的`switch`比`C`的更加通用。表达式不必是常量，甚至是整数，`case`从上到下的顺序进行求值，直到匹配。如果`switch`没有表达式，则它回匹配`true`。`switch`就像是一个`if-else-if-else`的链。


> Go's switch is more general than C's. The expressions need not be constants or even integers, the cases are evaluated top to bottom until a match is found, and if the switch has no expression it switches on true. It's therefore possible—and idiomatic—to write an if-else-if-else chain as a switch.

```go
func unhex(c byte) byte {
    switch {
    case '0' <= c && c <= '9':
        return c - '0'
    case 'a' <= c && c <= 'f':
        return c - 'a' + 10
    case 'A' <= c && c <= 'F':
        return c - 'A' + 10
    }
    return 0
}
```

Go中，switch不会自动通过case，但是case条件可以以逗号表达式列表的形式展示。

> There is no automatic fall through, but cases can be presented in comma-separated lists.

```go
func shouldEscape(c byte) bool {
    switch c {
    case ' ', '?', '&', '=', '#', '+', '%':
        return true
    }
    return false
}
```

break语句可以用来结束一个switch，尽管在Go中并不像在其他语言中那么常见。但是，有时候需要需要跳出一个循环，而不是switch，在Go中可以通过设置一个标号来中断循环。下面的例子会演示这两种用法：


> Although they are not nearly as common in Go as some other C-like languages, break statements can be used to terminate a switch early. Sometimes, though, it's necessary to break out of a surrounding loop, not the switch, and in Go that can be accomplished by putting a label on the loop and "breaking" to that label. This example shows both uses.


```go
Loop:
	for n := 0; n < len(src); n += size {
		switch {
		case src[n] < sizeOne:
			if validateOnly {
				break
			}
			size = 1
			update(src[n])

		case src[n] < sizeTwo:
			if n+1 >= len(src) {
				err = errShortInput
				break Loop
			}
			if validateOnly {
				break
			}
			size = 2
			update(src[n] + src[n+1]<<shift)
		}
	}
```

当然，continue语句可以接受一个可选的标号，但是它只能应用于循环。

为结束这一节，这里有一个使用switch语句对字节切片进行比较的例子。

> Of course, the continue statement also accepts an optional label but it applies only to loops.
> 
> To close this section, here's a comparison routine for byte slices that uses two switch statements。

```go
// Compare returns an integer comparing the two byte slices,
// lexicographically.
// The result will be 0 if a == b, -1 if a < b, and +1 if a > b
func Compare(a, b []byte) int {
    for i := 0; i < len(a) && i < len(b); i++ {
        switch {
        case a[i] > b[i]:
            return 1
        case a[i] < b[i]:
            return -1
        }
    }
    switch {
    case len(a) > len(b):
        return 1
    case len(a) < len(b):
        return -1
    }
    return 0
}
```

### Type switch

`switch`也可以用来发现接口变量的动态类型。这种`type switch`使用类型断言，括号中使用关键字`type`。如果在switch在表达式中声明了一个变量，则变量在每个子句中就会有响应的类型。在这种情况下复用名字也是习惯用法，声明一个变量，在每个子句中类型不同，但有着相同的名字。

> A switch can also be used to discover the dynamic type of an interface variable. Such a type switch uses the syntax of a type assertion with the keyword type inside the parentheses. If the switch declares a variable in the expression, the variable will have the corresponding type in each clause. It's also idiomatic to reuse the name in such cases, in effect declaring a new variable with the same name but a different type in each case.