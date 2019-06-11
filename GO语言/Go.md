### 数组

数组的初始化：

var array [5]int 

go 语言初始化时，会使用对应的零值来给元素初始化。

快速创建：

array := [5]int{1,2,3,4,5}或者 array := [...]int{1,2,3}

声明并给指定元素赋值：

array := [5]int{1: 10, 2: 20}

指针对象如果没有初始化直接赋值，会报错，如下所示：

ptr1 := [3]*int{0:new(int),1:new(int)}

*ptr1[2] = 10

数组变量的类型包括数组长度和每个元素的类型。只有这两部分都相同的数组，才是类型相同的数组，才能互相赋值。

编译器会阻止类型不同的数组互相赋值。



append的用法有两种：

slice = append(slice, elem1, elem2)

slice = append(slice, anotherSlice...)

第一种用法中，第一个参数为slice,后面可以添加多个参数。

如果是将两个slice拼接在一起，则需要使用第二种用法，在第二个slice的名称后面加三个点，而且这时候append只支持两个参数，不支持任意个数的参数。

### 切片：

s := arr[startIndex:endIndex] ，从startIndex~endIndex-1，如果缺省endIndex则表示到最后一个元素

**对应TOML配置文件时，对应的Struct必须大写，小写时没法映射**

\#string到int

int,err:=strconv.Atoi(string)

\#string到int64

int64, err := strconv.ParseInt(string, 10, 64)

\#int到string

string:=strconv.Itoa(int)

\#int64到string

string:=strconv.FormatInt(int64,10)

使用下划线_可以导入未使用的包

不想跳出case循环，即执行完一个分支再进入下一个分支时使用fallthrough关键字

var identifier []type声明切片，切片在未初始化之前默认为nil，长度为0

var slice []type=array[start:end]，表示的是从数组array start到end-1索引之间的元素构成的子集。

切片、接口、map、通道默认都是引用传递

变参函数可以接受slice作参数

**new 和 make 区别**

new 和 make 均是用于分配内存：new 用于值类型和用户定义的类型，如自定义结构，make 用于内置引用类型（切片、map 和管道）。它们的用法就像是函数，但是将类型作为参数：new(type)、make(type)。new(T) 分配类型 T 的零值并返回其地址，也就是指向类型 T 的指针（详见第 10.1 节）。它也可以被用于基本类型：v := new(int)。make(T) 返回类型 T 的初始化之后的值，因此它比 new 进行更多的工作（详见第 7.2.3/4 节、第 8.1.1 节和第 14.2.1 节）new() 是一个函数，不要忘记它的括号



接口类型检测：

switch t :=  xxx.(type)

case condition:

XXXXXXXX



if   v,ok:=xxx.(type);ok{

......}



每个 interface {} 变量在内存中占据两个字长：一个用来存储它包含的类型，另一个用来存储它包含的数据或者指向数据的指针。

如果将某个类型的切片赋值给一个空接口切片，需要显示赋值，原因是在内存中结构不一样。

通过反射修改值

v :=  reflect.valueof(x)

v  =  v.Elem()

**There's no reason to use a map when an array or slice will do.**

**并发控制方式：**

Wait   WaitGroup 

Cancel Context模式



在main.main函数执行之前所有代码都运行在同一个goroutine，也就是程序的主系统线程中。因此，如果某个init函数内部用go关键字启动了新的goroutine的话，新的goroutine只有在进入main.main函数之后才可能被执行到。

### GO的数组与切片

数组创建时需要制定长度

var a=[3]{}  这种或者var a=[...]{}这种  ，如果不指定即为切片~

对于类型，和数组的最大不同是，切片的类型和长度信息无关，只要是相同类型元素构成的切片均对应相同的切片类型。

a := [10]{1,2,3,4,5,6}

b :=  a[0:2:cap(a)]      初始化切片时右边的是开区间，左边是闭区间，第三个参数是容量

切片高效操作的要点是要降低内存分配的次数，尽量保证append操作不会超出cap的容量，降低触发内存分配的次数和每次分配内存大小。

并发模型：

Actor模型和CSP模型

<<1 

命令行翻墙：

启用：source socks-cli/activate

禁用：source socks-cli/deactivate

返回值是值类型还是引用类型：

- 当返回类型不涉及状态变更并且是较简单的数据结构，一律返回值类型
- 当返回类型可能遇到状态变更或者你关心它的生命周期则使用指针类型
- 当返回的结构比较大的时候使用指针类型