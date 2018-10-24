Go 提供了 `C-style` 的块注释`/* */`和`C++ style`的行注释。通常情况下用行注释；块注释大多数作为包注释，在表达式中或者禁用大量代码时也是非常有用得。

> Go provides C-style`/* */`block comments and C++-style `//` line comments. Line comments are the norm; block comments appear mostly as package comments, but are useful within an expression or to disable large swaths of code.

程序——同时又是网络服务器——`godoc`，处理 Go 源文件并导出包内容生成文档。在顶层声明出现前，没有出现换行符的注释，回和声明一起被导出作为该项的解释性文本。注释的性质和风格决定了`godoc`生成文档的质量。

> The program—and web server—`godoc`processes Go source files to extract documentation about the contents of the package. Comments that appear before top-level declarations, with no intervening newlines, are extracted along with the declaration to serve as explanatory text for the item. The nature and style of these comments determines the quality of the documentation`godoc`produces.

每个包都应该有一个包子句前面的块注释。对于多文件包，包注释只需要出现在一个文件中，任何一个都可以。包注释应该介绍包，并提供与整个包相关的信息。它（注释）会出现在`godoc`页面，并应该建立后续的详细文档。

> Every package should have apackage comment, a block comment preceding the package clause. For multi-file packages, the package comment only needs to be present in one file, and any one will do. The package comment should introduce the package and provide information relevant to the package as a whole. It will appear first on the`godoc`page and should set up the detailed documentation that follows.

```
/*
Package regexp implements a simple library for regular expressions.

The syntax of the regular expressions accepted is:

    regexp:
        concatenation { '|' concatenation }
    concatenation:
        { closure }
    closure:
        term [ '*' | '+' | '?' ]
    term:
        '^'
        '$'
        '.'
        character
        '[' [ '^' ] character-ranges ']'
        '(' regexp ')'
*/
package regexp
```

如果包比较简单的话，包注释也可以简洁一些。

> If the package is simple, the package comment can be brief.

```
// Package path implements utility routines for
// manipulating slash-separated filename paths.
```

注释并不需要额外格式，比如星号横幅。生成的输出甚至可能不以固定宽度字体展示，所以，不要依赖空格去对齐——`godoc`，比如`gofmt`，记得这一点。注释是不被解释的纯文本，因此`HTML`，还有诸如 `_this_`的这类注释将会被逐字复制，最好不要使用（效果非预期）。godoc使用等宽文本，适合于程序片段。包 `fmt package` 使用了这种方式，获得了很好的效果。

> Comments do not need extra formatting such as banners of stars. The generated output may not even be presented in a fixed-width font, so don't depend on spacing for alignment—`godoc`, like`gofmt`, takes care of that. The comments are uninterpreted plain text, so HTML and other annotations such as`_this_`will reproduce verbatim and should not be used. One adjustment`godoc`does do is to display indented text in a fixed-width font, suitable for program snippets. The package comment for the[`fmt`package](https://golang.org/pkg/fmt/) uses this to good effect.

根据上下文的不同，`godoc`甚至可能不会重新格式化注释，因此，确保这些注释看起来非常直观：拼写正确，标点和句子结构，折叠过长的行，等等。

> Depending on the context,`godoc`might not even reformat comments, so make sure they look good straight up: use correct spelling, punctuation, and sentence structure, fold long lines, and so on.

在一个包里，在顶级声明之前的任何注释都将作为文档注释。程序的每个导出的（大写的）名称都应该有一个文档注释。

> Inside a package, any comment immediately preceding a top-level declaration serves as a doc commentfor that declaration. Every exported \(capitalized\) name in a program should have a doc comment.

文档注释最好是完整的句子，它允许各种各样的自动化展示。第一句应该是一句总结性的话，并以被声明的名称开头。

> Doc comments work best as complete sentences, which allow a wide variety of automated presentations. The first sentence should be a one-sentence summary that starts with the name being declared.

```
// Compile parses a regular expression and returns, if successful,
// a Regexp that can be used to match against text.
func Compile(str string) (*Regexp, error) {
```

如果每个文档注释都以所描述的条目的名称开头，那么，就可以很方便的通过grep来处理`godoc`的输出。假设你正在找正则表达式的解析函数，却未能记住“Compile”这个名字，这时就可以运行命令：

> If every doc comment begins with the name of the item it describes, the output of`godoc`can usefully be run through`grep`. Imagine you couldn't remember the name "Compile" but were looking for the parsing function for regular expressions, so you ran the command,

```
$ godoc regexp | grep -i parse
```

如果包中注释都以“This function”开头，grep将不会有任何帮助。但由于包文档注释都以名称开头，你将会看到类似的场景，来帮助你回忆起你所要找寻东西。

> If all the doc comments in the package began, "This function...",`grep`wouldn't help you remember the name. But because the package starts each doc comment with the name, you'd see something like this, which recalls the word you're looking for.

```
$ godoc regexp | grep parse
    Compile parses a regular expression and returns, if successful, a Regexp
    parsed. It simplifies safe initialization of global variables holding
    cannot be parsed. It simplifies safe initialization of global variables
$
```

Go的声明语法允许对声明进行分组。单个文档注释可以引入一组相关的常量或变量。因为展示的是整个声明，这种注释看起来就略显敷衍了。

> Go's declaration syntax allows grouping of declarations. A single doc comment can introduce a group of related constants or variables. Since the whole declaration is presented, such a comment can often be perfunctory.

```
// Error codes returned by failures to parse an expression.
var (
    ErrInternal      = errors.New("regexp: internal error")
    ErrUnmatchedLpar = errors.New("regexp: unmatched '('")
    ErrUnmatchedRpar = errors.New("regexp: unmatched ')'")
    ...
)
```

分组还可以表明不同条目间的关系，比如一组变量受到互斥对象的保护。

> Grouping can also indicate relationships between items, such as the fact that a set of variables is protected by a mutex.

```
var (
    countLock   sync.Mutex
    inputCount  uint32
    outputCount uint32
    errorCount  uint32
)
```



