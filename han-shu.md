
### 多个返回值

`Go`的一个不同寻常的特性就是，函数和方法可以返回多个值。这种形式可以用来改进`C`程序中一些笨拙的习语：一些标志性的错误，例如用`-1`表示`EOF`，并且修改传递的地址参数。

在`C`中，一个写错误由一个带有错误码的负计数标识的，它存储在一个可变的地方。在`Go`中，`Write`可以返回一个计数和一个错误：“是的，你写了一些字节，但没有全部写完，由于你填满了设备”。在包`os`中，文件`Write`方法的签名是：

> One of Go's unusual features is that functions and methods can return multiple values. This form can be used to improve on a couple of clumsy idioms in C programs: in-band error returns such as -1 for EOF and modifying an argument passed by address.
> 
> In C, a write error is signaled by a negative count with the error code secreted away in a volatile location. In Go, Write can return a count and an error: “Yes, you wrote some bytes but not all of them because you filled the device”. The signature of the Write method on files from package os is:


```go
func (file *File) Write(b []byte) (n int, err error)
```

如文档所说，它返回写入的字节数，当`n != len(b)`时会返回一个非`nil`的错误。这是一种常见的风格；有关更多示例，请参考错误处理部分。


> and as the documentation says, it returns the number of bytes written and a non-nil error when n != len(b). This is a common style; see the section on error handling for more examples.


类似的方法排除了向返回值传递指针以模拟引用参数的必要。这里有一个简单的函数，从直接切片中获取一个数字，并返回这个数字和下一个位置。
 

> A similar approach obviates the need to pass a pointer to a return value to simulate a reference parameter. Here's a simple-minded function to grab a number from a position in a byte slice, returning the number and the next position.

```go
func nextInt(b []byte, i int) (int, int) {
    for ; i < len(b) && !isDigit(b[i]); i++ {
    }
    x := 0
    for ; i < len(b) && isDigit(b[i]); i++ {
        x = x*10 + int(b[i]) - '0'
    }
    return x, i
}

```

你可以使用它（上面的函数），去扫描切片`b`中的数字。


> You could use it to scan the numbers in an input slice b like this:

```go
    for i := 0; i < len(b); {
        x, i = nextInt(b, i)
        fmt.Println(x)
    }
```

### 命名返回参数

Go中函数的返回值是可以指定名字的，并可以像常规变量一样使用，就像是入参一样。当返回参数被命名时，在函数开始时它们会被初始化为对应类型的零值；如果函数执行了没有参数的return语句，则结果参数的当前值将被用作返回值。

> The return or result "parameters" of a Go function can be given names and used as regular variables, just like the incoming parameters. When named, they are initialized to the zero values for their types when the function begins; if the function executes a return statement with no arguments, the current values of the result parameters are used as the returned values.

返回值的名字并非强制性的，但是它们可以让代码看起来更简短、更清晰：它们就是文档本身。如果我们给nextInt的返回值指定名字话，每个int所代表的含义就很明显了。


> The names are not mandatory but they can make code shorter and clearer: they're documentation. If we name the results of nextInt it becomes obvious which returned int is which.

```go
func nextInt(b []byte, pos int) (value, nextPos int) {
```

### 延迟执行

Go的defer语句会安排一个函数调用（defer函数），使得在return之前立即执行defer函数。这是一个不同寻常但高效的方式，去处理类似的场景：不论从哪个函数分支返回，资源必须得到释放。典型的例子是解锁互斥锁，或者关闭文件。

> Go's defer statement schedules a function call (the deferred function) to be run immediately before the function executing the defer returns. It's an unusual but effective way to deal with situations such as resources that must be released regardless of which path a function takes to return. The canonical examples are unlocking a mutex or closing a file.


```go
// Contents returns the file's contents as a string.
func Contents(filename string) (string, error) {
    f, err := os.Open(filename)
    if err != nil {
        return "", err
    }
    defer f.Close()  // f.Close will run when we're finished.

    var result []byte
    buf := make([]byte, 100)
    for {
        n, err := f.Read(buf[0:])
        result = append(result, buf[0:n]...) // append is discussed later.
        if err != nil {
            if err == io.EOF {
                break
            }
            return "", err  // f will be closed if we return here.
        }
    }
    return string(result), nil // f will be closed if we return here.
}
```

延迟类似`Close`这样的函数执行，有两个好处。第一，它可以保证你永远不会忘记关闭文件，特别是在以后改写这个函数并添加新的返回分支的情况下，极易犯的一个错误。第二，这意味着`close`紧挨着`open`，这远比放在函数末尾要清晰得多。

> Deferring a call to a function such as Close has two advantages. First, it guarantees that you will never forget to close the file, a mistake that's easy to make if you later edit the function to add a new return path. Second, it means that the close sits near the open, which is much clearer than placing it at the end of the function.

`defer`函数的参数（如果函数是方法，则包括接收方）是在延迟执行时计算的，而非调用执行时计算。此外，不必担心变量的值在函数执行时变化，也就是说，一个延迟执行也会延迟其他延迟函数的执行。举一个简单的例子。

> The arguments to the deferred function (which include the receiver if the function is a method) are evaluated when the defer executes, not when the call executes. Besides avoiding worries about variables changing values as the function executes, this means that a single deferred call site can defer multiple function executions. Here's a silly example.


```go
for i := 0; i < 5; i++ {
    defer fmt.Printf("%d ", i)
}
```

`defer`函数的执行遵循`LIFO`的顺序，因此，这段代码在返回时会打印出 `4 3 2 1`。另一个更加可信的例子是，这是一种简单的方式，它可以跟踪程序的执行过程。我们可以写出很多跟踪执行路径的例子，比如：


> Deferred functions are executed in LIFO order, so this code will cause 4 3 2 1 0 to be printed when the function returns. A more plausible example is a simple way to trace function execution through the program. We could write a couple of simple tracing routines like this:

```go 
func trace(s string)   { fmt.Println("entering:", s) }
func untrace(s string) { fmt.Println("leaving:", s) }

// Use them like this:
func a() {
    trace("a")
    defer untrace("a")
    // do something....
}
```

我们可以更好地利用延迟函数的参数在延迟执行时计算这一事实。
`traceing`为`untracing`设置参数。例如：


> We can do better by exploiting the fact that arguments to deferred functions are evaluated when the defer executes. The tracing routine can set up the argument to the untracing routine. This example:


```go
func trace(s string) string {
    fmt.Println("entering:", s)
    return s
}

func un(s string) {
    fmt.Println("leaving:", s)
}

func a() {
    defer un(trace("a"))
    fmt.Println("in a")
}

func b() {
    defer un(trace("b"))
    fmt.Println("in b")
    a()
}

func main() {
    b()
}
```

输出

> prints

```go
entering: b
in b
entering: a
in a
leaving: a
leaving: b
```

来自于其他语言已经习惯于块资源管理的程序员来说，`defer`可能看起来有点奇怪，但是它是最有趣的，强大的应用恰恰来自于这样一个事实：它不是基于块而是基于函数的。在异常和恢复一节中，我们会看到另外一个例子，和它的可能性。

> For programmers accustomed to block-level resource management from other languages, defer may seem peculiar, but its most interesting and powerful applications come precisely from the fact that it's not block-based but function-based. In the section on panic and recover we'll see another example of its possibilities.

