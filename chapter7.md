### 使用`new`分配

`Go`有两个分配原语。内置函数`new`和`make`。它们会干不同的事情，分配不同的类型，虽然有点迷惑人，但区分规则很简单。我们先来说`new`。它是一个内置的分配内存的函数，但它在其他语言中的同名兄弟，它不会初始化内存，仅仅是初始为零值。也就是说，`new(T)`分配了一个类型为T的条目，并且被初始化为零值的存储空间，并返回它的地址，类型为`*T`。在Go的术语中，它返回了一个指针，指向类型为`T`的新分配的零值。

> Go has two allocation primitives, the built-in functions new and make. They do different things and apply to different types, which can be confusing, but the rules are simple. Let's talk about new first. It's a built-in function that allocates memory, but unlike its namesakes in some other languages it does not initialize the memory, it only zeros it. That is, new\(T\) allocates zeroed storage for a new item of type T and returns its address, a value of type \*T. In Go terminology, it returns a pointer to a newly allocated zero value of type T.

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

### 构造函数和复合文字

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

The fields of a composite literal are laid out in order and must all be present. However, by labeling the elements explicitly as field`:`value pairs, the initializers can appear in any order, with the missing ones left as their respective zero values. Thus we could say

```go
 return &File{fd: fd, name: name}
```

As a limiting case, if a composite literal contains no fields at all, it creates a zero value for the type. The expressions`new(File)`and`&File{}`are equivalent.

Composite literals can also be created for arrays, slices, and maps, with the field labels being indices or map keys as appropriate. In these examples, the initializations work regardless of the values of`Enone`,`Eio`, and`Einval`, as long as they are distinct.

```go
a := [...]string   {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
s := []string      {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
m := map[int]string{Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
```

### 用make分配

Back to allocation. The built-in function`make(T,`args`)`serves a purpose different from`new(T)`. It creates slices, maps, and channels only, and it returns an_initialized_\(not_zeroed_\) value of type`T`\(not`*T`\). The reason for the distinction is that these three types represent, under the covers, references to data structures that must be initialized before use. A slice, for example, is a three-item descriptor containing a pointer to the data \(inside an array\), the length, and the capacity, and until those items are initialized, the slice is`nil`. For slices, maps, and channels,`make`initializes the internal data structure and prepares the value for use. For instance,

```go
make([]int, 10, 100)
```

allocates an array of 100 ints and then creates a slice structure with length 10 and a capacity of 100 pointing at the first 10 elements of the array. \(When making a slice, the capacity can be omitted; see the section on slices for more information.\) In contrast,`new([]int)`returns a pointer to a newly allocated, zeroed slice structure, that is, a pointer to a`nil`slice value.

These examples illustrate the difference between`new`and`make`.

```go
var p *[]int = new([]int)       // allocates slice structure; *p == nil; rarely useful
var v  []int = make([]int, 100) // the slice v now refers to a new array of 100 ints

// Unnecessarily complex:
var p *[]int = new([]int)
*p = make([]int, 100, 100)

// Idiomatic:
v := make([]int, 100)
```

Remember that`make`applies only to maps, slices and channels and does not return a pointer. To obtain an explicit pointer allocate with`new`or take the address of a variable explicitly.

### Arrays {#arrays}

Arrays are useful when planning the detailed layout of memory and sometimes can help avoid allocation, but primarily they are a building block for slices, the subject of the next section. To lay the foundation for that topic, here are a few words about arrays.

There are major differences between the ways arrays work in Go and C. In Go,

* Arrays are values. Assigning one array to another copies all the elements.
* In particular, if you pass an array to a function, it will receive a copy of the array, not a pointer to it.
* The size of an array is part of its type. The types`[10]int`and`[20]int`are distinct.

The value property can be useful but also expensive; if you want C-like behavior and efficiency, you can pass a pointer to the array.

```go
func Sum(a *[3]float64) (sum float64) {
    for _, v := range *a {
        sum += v
    }
    return
}

array := [...]float64{7.0, 8.5, 9.1}
x := Sum(
&
array)  // Note the explicit address-of operator
```

But even this style isn't idiomatic Go. Use slices instead.

