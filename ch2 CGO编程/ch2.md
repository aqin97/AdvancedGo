# 第二章 CGO编程

## 2.1 入门

### 2.1.1 最简单的CGO程序

```go
package main

import "C"

func main() {
 println("hello world")
}

```

通过`import "C"`语句启用了CGO特性，虽然没有调用任何CGO相关函数，但是go build命令在编译和连接阶段启动gcc编译器，这就是一个完整的CGO程序了

### 2.1.2 基于C标准库输出字符串

```go
package main

//#include <stdio.h>
import "C"

func main() {
 C.puts(C.CString("hello world\n"))
}

```

### 2.1.3 使用自己编写的c函数

```go
package main

/*
static void SayHello(const char *s) {
 puts(s);
}
*/
import "C"

func main() {
 C.SayHello(C.CString("hello, world\n"))
}

```

当然也可以将SayHello()函数放到当前目录下的一个c文件当中,为了导出到外部，需要将修饰符`static`去掉，还需要在CGO文件中声明需要使用的函数。

### 2.1.4 C代码的模块化

抽象和模块化是将复杂问题简单化的通用手段,SayHello()函数的声明放到hello.h，实现放到hello.c，在hello.c中`#include "hello.h"`,在CGO部分就不需要重新对需要使用的函数进行声明，而是直接通过包含头文件的方式来调用。

### 2.1.5 GO重新实现C函数

CGO不管可以让GO语言调用C语言函数，还可以导出GO实现的函数给C用。

通过CGO函数之前的`//export SayHello`来导出这个函数，这样就可以将他作为一个已经实现的C函数来使用了。

### 2.1.6 面向C接口的GO编程

从2.1.1到2.1.5，最开始全部的CGO代码都在一个GO文件当中，之后慢慢通过面向C接口编程技术将SayHello()拆分到不同的C文件当中去，而main仍然是GO语言编写，再之后又用GO重新实现了C函数。但是对目前的例子来说，只有一个函数，拆到三个不同的文件实在是繁琐。重新尝试合并：

```go
// +build go1.10
package main

//void SayHello(_GoString_ s);
import "C"

import "fmt"

func main() {
 C.SayHello("hello world\n")
}

//export SayHello
func SayHello(s string) {
 fmt.Print(s)
}

```
