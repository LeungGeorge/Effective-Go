
### 多个返回值

Go的一个不同寻常的特性就是，函数和方法可以返回多个值。这种形式可以用来改进C程序中一些笨拙的习语：带内错误返回，例如-1表示EOF并修改地址传递的参数。

> One of Go's unusual features is that functions and methods can return multiple values. This form can be used to improve on a couple of clumsy idioms in C programs: in-band error returns such as -1 for EOF and modifying an argument passed by address.
> 
> In C, a write error is signaled by a negative count with the error code secreted away in a volatile location. In Go, Write can return a count and an error: “Yes, you wrote some bytes but not all of them because you filled the device”. The signature of the Write method on files from package os is:


```go
func (file *File) Write(b []byte) (n int, err error)
```