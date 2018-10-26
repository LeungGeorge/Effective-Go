
### 使用`new`分配

`Go`有两个分配原语。内置函数`new`和`make`。它们会干不同的事情，分配不同的类型，虽然有点迷惑人，但区分规则很简单。我们先来说`new`。它是一个内置的分配内存的函数，但它在其他语言中的同名兄弟，它不会初始化内存，仅仅是初始为零值。也就是说，`new(T)`分配了一个类型为T的条目，并且被初始化为零值的存储空间，并返回它的地址，类型为`*T`。在Go的术语中，它返回了一个指针，指向类型为`T`的新分配的零值。

> Go has two allocation primitives, the built-in functions new and make. They do different things and apply to different types, which can be confusing, but the rules are simple. Let's talk about new first. It's a built-in function that allocates memory, but unlike its namesakes in some other languages it does not initialize the memory, it only zeros it. That is, new(T) allocates zeroed storage for a new item of type T and returns its address, a value of type *T. In Go terminology, it returns a pointer to a newly allocated zero value of type T.

由于`new`返回的内存是零值，这会有助于设计你的数据结构，每个类型的零值都可以直接使用，无需额外初始化操作。这意味着，结构体的使用者可以直接使用`new`创建，并可以直接使用。例如，`bytes.Bufer`的文档中说`Buffer`的零值是一个空的待用的缓存。类似的，`sync.Mutex`没有显式构造或者初始化方法。相反的，`sync.Mutex`的零值被定义为一个未加锁的互斥量。


> Since the memory returned by new is zeroed, it's helpful to arrange when designing your data structures that the zero value of each type can be used without further initialization. This means a user of the data structure can create one with new and get right to work. For example, the documentation for `bytes.Buffer` states that "the zero value for Buffer is an empty buffer ready to use." Similarly, `sync.Mutex` does not have an explicit constructor or Init method. Instead, the zero value for a `sync.Mutex` is defined to be an unlocked mutex.

零值的属性是可以传递的，请看如下类型声明：

> The zero-value-is-useful property works transitively. Consider this type declaration.

```go
type SyncedBuffer struct {
    lock    sync.Mutex
    buffer  bytes.Buffer
}
```

`SyncedBuffer`类型的值也可以在分配或者声明时立即使用。在下一个片段，`p`和`v`无需进一步的处理即可正常使用。

> Values of type `SyncedBuffer` are also ready to use immediately upon allocation or just declaration. In the next snippet, both p and v will work correctly without further arrangement.

```go
p := new(SyncedBuffer)  // type *SyncedBuffer
var v SyncedBuffer      // type  SyncedBuffer
```

### 构造函数和符合文字

有时候，零值并不是很好，这时一个初始化构造函数就很有必要了，就如同这个从`os`包中提取的例子。

> Sometimes the zero value isn't good enough and an initializing constructor is necessary, as in this example derived from package os.


```go
func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    f := new(File)
    f.fd = fd
    f.name = name
    f.dirinfo = nil
    f.nepipe = 0
    return f
}
```

注意，与`C`不同，在`Go`中返回局部变量的地址是完全没问题的，与变量相关的存储在函数返回后不会释放。事实上，


> Note that, unlike in C, it's perfectly OK to return the address of a local variable; the storage associated with the variable survives after the function returns. In fact, taking the address of a composite literal allocates a fresh instance each time it is evaluated, so we can combine these last two lines.

```go
    return &File{fd, name, nil, 0}
```
