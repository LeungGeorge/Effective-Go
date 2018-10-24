和其他语言一样，名字在Go中同样重要。它们甚至有着一样的语义：一个包内的名字是否对外可见，取决于其首字母是否大写。因此，花一点时间讨论一波Go程序的命名约定是很有必要的。

> Names are as important in Go as in any other language. They even have semantic effect: the visibility of a name outside a package is determined by whether its first character is upper case. It's therefore worth spending a little time talking about naming conventions in Go programs.

### 包名

当一个导入一个包时，包名就是包内容的访问器。在如下标识之后：

> When a package is imported, the package name becomes an accessor for the contents. After

```
import "bytes"
```

以导入包`bytes.Buffer`为例讨论包导入。如果可以使用相同的名字来访问一个包都内容将会非常有用，这也意味着包名是好的：简短，简洁，有吸引力。按惯例，包名通常时一个小写的单词；不需要使用下划线或者混合大小写。要力求简洁，因为每个用这个包都人都会敲这个名字。另外，不必担心与先前的名字有冲突。包名仅仅是导入后的默认名字，在所有代码中不必唯一，在少有的冲突情况下，可以选一个不同的名称在本地使用。不管怎样，冲突很少见，因为导入的文件名确定了要使用的包。

> the importing package can talk about`bytes.Buffer`. It's helpful if everyone using the package can use the same name to refer to its contents, which implies that the package name should be good: short, concise, evocative. By convention, packages are given lower case, single-word names; there should be no need for underscores or mixedCaps. Err on the side of brevity, since everyone using your package will be typing that name. And don't worry about collisionsa priori. The package name is only the default name for imports; it need not be unique across all source code, and in the rare case of a collision the importing package can choose a different name to use locally. In any case, confusion is rare because the file name in the import determines just which package is being used.

另外一个约定是，包名是源目录的基础名。程序包`src/encoding/base64`以`encoding/base64`导入，但是包名是`base64`，不是`encoding_base64`也不是`encodingBase64`。

> Another convention is that the package name is the base name of its source directory; the package in`src/encoding/base64`is imported as`"encoding/base64"`but has name`base64`, not`encoding_base64`and not`encodingBase64`.

程序包引入者将使用这个名字引用包内容。因此，包中的导出名字可以利用这个事实来避免残迹。（不要使用 `import .`标记，这将会简化那些必须运行在包之外而不可避免的测试）例如，在包`bufio`中缓冲阅读类型叫做`Reader`，而不是`BufReader`，因为用户会把它看作是`bufio.Reader`这样是清晰的、简洁的名字。此外，还因为导入实体通常使用他们他们的包名称寻址的，`bufio.Reader`与`io.Reader`并不冲突。同样，创建一个`ring.Ring`的新实例——Go中构造的定义——通常会叫做`NewRing`，但是由于包名是`ring`，而`Ring`仅仅是包的导出类型，它只叫做`New`。这样，客户端看到的将是`ring.New`。使用包结构可以帮助你选择好的名字。

> The importer of a package will use the name to refer to its contents, so exported names in the package can use that fact to avoid stutter. \(Don't use the`import .`notation, which can simplify tests that must run outside the package they are testing, but should otherwise be avoided.\) For instance, the buffered reader type in the`bufio`package is called`Reader`, not`BufReader`, because users see it as`bufio.Reader`, which is a clear, concise name. Moreover, because imported entities are always addressed with their package name,`bufio.Reader`does not conflict with`io.Reader`. Similarly, the function to make new instances of`ring.Ring` —which is the definition of a _constructor_ in Go—would normally be called`NewRing`, but since `Ring` is the only type exported by the package, and since the package is called `ring` , it's called just `New`, which clients of the package see as `ring.New`. Use the package structure to help you choose good names.

另一个简短的例子是`once.Do`；`once.Do(setup)`的可读性很好，写成`once.DoOrWaitUntilDone(setup)`并不会有所改善。更长的名字并不会自动变得更易读。通常，一条有用的文档注释比额外的长名字更有价值。

> Another short example is`once.Do`;`once.Do(setup)`reads well and would not be improved by writing`once.DoOrWaitUntilDone(setup)`. Long names don't automatically make things more readable. A helpful doc comment can often be more valuable than an extra long name.

### Get方法

Go 并未提供自动的`get/se`t方法。不过，你自行提供`get/set`方法也无可厚非，通常这么做也是合适的。但是，在 `Get` 方法前面加`get`，既不是习惯也没有必要。如果你有一个字段`owner`（小写的，非导出型），它的`get`方法应该叫作`Owner`（大写的，导出型），而不是`GetOwner`

。大写名字导出的用法，提供了区分字段和方法的钩子。一个`set`方法，如果有必要的话，可能叫作`SetOwner`。这两个名字在实际应用中可读性都很好。

> Go doesn't provide automatic support for getters and setters. There's nothing wrong with providing getters and setters yourself, and it's often appropriate to do so, but it's neither idiomatic nor necessary to put`Get`into the getter's name. If you have a field called`owner`\(lower case, unexported\), the getter method should be called`Owner`\(upper case, exported\), not`GetOwner`. The use of upper-case names for export provides the hook to discriminate the field from the method. A setter function, if needed, will likely be called`SetOwner`. Both names read well in practice:

```
owner := obj.Owner()
if owner != user {
    obj.SetOwner(user)
}
```

### 接口名

依管理，一个方法接口由方法名加上`er`后缀或者类似的修改来命名，以构造一个替代性的名词：Reader，Writer，Formatter，CloseNotifier 等等。

> By convention, one-method interfaces are named by the method name plus an -er suffix or similar modification to construct an agent noun:`Reader`,`Writer`,`Formatter`,`CloseNotifier`etc.

由很多类似的名字，尊重它们和它们说体现的函数名，是最富有成效的了。Read，Write，Close，Flush，String 等等有着最具规范的签名和含义。以免混乱，不要给你的方法起这些名字，除非它有相同的签名和含义。相反的，如果你的类型实现了一个有相同含义的方法，跟常用类型的方法一样，可以给他一个相同的名字和函数签名：可以把你的字符串转换方法叫作`String`而不是`ToString`。

> There are a number of such names and it's productive to honor them and the function names they capture.`Read`,`Write`,`Close`,`Flush`,`String`and so on have canonical signatures and meanings. To avoid confusion, don't give your method one of those names unless it has the same signature and meaning. Conversely, if your type implements a method with the same meaning as a method on a well-known type, give it the same name and signature; call your string-converter method`String`not`ToString`.

### 混合大小写

最后，在Go中有这样一个约定，使用混合大小写的形式（类似驼峰，但是注意首字母），而不是下划线，来写多单词的名字。

> Finally, the convention in Go is to use`MixedCaps`or`mixedCaps`rather than underscores to write multiword names.



