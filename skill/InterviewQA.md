## Go语言：

1. 基础，数组与切片的区别

~~~go
s3 := []int{1, 2, 3, 4, 5, 6, 7, 8}
s4 := s3[3:6]
fmt.Printf("The length of s4: %d\n", len(s4))
fmt.Printf("The capacity of s4: %d\n", cap(s4))
fmt.Printf("The value of s4: %d\n", s4)
/*
The length of s4: 3
The capacity of s4: 5
The value of s4: [4 5 6]
*/
~~~

2. 知识前导：为什么字典的键类型会受到约束？Go 语言字典的键类型不可以是函数类型、字典类型和切片类型？在值为nil的字典上执行读操作会成功吗？除了添加键 - 元素对，我们在一个值为nil的字典上做任何操作都不会引起错误。当我们试图在一个值为nil的字典中添加键 - 元素对的时候，Go 语言的运行时系统就会立即抛出一个 panic。

结构体不可以、切片可以、函数可以作为key

不能读，会引发panic。不能添加。