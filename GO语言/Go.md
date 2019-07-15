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

**iota**

每次const出现时，都会让iota的值初始化成0。一行中有两个及以上的iota时，在下一行增长，而不是立即取得它的引用，比如 const(A，B = iota+1，iota+2)两个iota的值是一样的

#### CSP（Communicating Sequential Processes）模型与Actor模型

二者的格言都是：

Don't communicate by sharing memory，share memory by communicating.



#### 获取命令行的参数

- 使用os库，如：

  ```go
  func main() {
      args := os.Args
      if args == nil  { // 校验参数并输出提示信息
          return
      }
      fmt.Printf("%T\n", args) 
      fmt.Printf("%v\n", args)
  }
  ```

go build main.go 

./main -name DomXiao

输出：[]string [./main -name DomXiao]

- 使用flag库，步骤：

  1）定义各个参数的类型、名字、默认值与提示信息

  2）解析

  3）获取参数值

```go
func main() {
    name := flag.String("name", "", "Your name")
    var age int
    flag.IntVar(&age, "age", -1, "Your age")

    flag.Parse()

    println("name", *name)
    println("age", age)
}
```



#### defer问题

官方对defer的执行时机做的阐述，分别是：

- 包裹defer的函数返回时
- 包裹defer的函数执行到末尾时
- 所在的goroutine发生panic时

当有多个defer时，执行顺序是LIFO。

```go
func unameReturnValues() int {
    var result int
    defer func() {
        result++
        fmt.Println("I'm unamed defer~")
    }()
    return result
}

func namedReturnValues() (result int) {
    defer func() {
        result++
        fmt.Println("I'm named defer~")
    }()
    return result
}
```

上面的方法会输出0，下面的方法输出1。上面的方法使用了匿名返回值，下面的使用了命名返回值，除此之外其他的逻辑均相同，为什么输出的结果会有区别呢？

Defer过程如下:

- 将result赋值给返回值（可以理解成Go自动创建了一个返回值retValue，相当于执行retValue = result）
- 然后检查是否有defer，如果有则执行
- 返回刚才创建的返回值（retValue）

在这种情况下，defer中的修改是对result执行的，而不是retValue，所以defer返回的依然是retValue。在命名返回值方法中，由于返回值在方法定义时已经被定义，所以没有创建retValue的过程，result就是retValue，defer对于result的修改也会被直接返回。

**当调用os.Exit()方法退出程序时，defer并不会被执行。**

**Go 中如何判断一个变量的类型？？？**

通过类型断言，如：

~~~go
s1 := map[int]string{0:"zero",1:"one",2:"two"}
value,ok := interface{}(s1).(map[int]string)
fmt.Println(value,ok)
//输出：map[0:zero 1:one 2:two] true
~~~

#### 类型转换时值得注意的点

- 对于整数类型值、整数常量之间的类型转换，原则上只要源值在目标类型的可表示范围之内就是合法的。但如果源整数类型的范围比较大，目标类型的范围比较小的时候，会出现截掉高位的二进制数据。比如：

```go
var srcInt = int16(-255)
dsInt := int8(srcInt)
fmt.PrintLn(dsInt)//1
```

首先你要知道，整数在 Go 语言以及计算机中都是以补码的形式存储的。这主要是为了简化计算机对整数的运算过程。补码其实就是原码各位求反再加 1。比如，int16 类型的值<code>-255</code>的补码是<code>1111111100000001</code>。如果我们把该值转换为<code>int8</code>类型的值，那么 Go 语言会把在较高位置（或者说最左边位置）上的 8 位二进制数直接截掉，从而得到<code>00000001</code>。又由于其最左边一位是0，表示正整数，以及**正整数的补码就等于源码（负数的补码等于源码+1）**，所以值为1。一定要记住，当整数值的类型的有效范围由宽变窄时，只需在补码形式下截掉一定数量的高位二进制数即可。类似的快刀斩乱麻规则还有：当把一个浮点数类型的值转换为整数类型值时，前者的小数部分会被全部截掉。

- <strong>第二，虽然直接把一个整数值转换为一个<code>string</code>类型的值是可行的，但值得关注的是，被转换的整数值应该可以代表一个有效的 Unicode 代码点，否则转换的结果将会是<code>"�"</code>（仅由高亮的问号组成的字符串值）。</strong>

~~~go
fmt.PrintLn(string(-1))//�
~~~

![别名、类型再定义与潜在类型](../images/GO语言类型区别.png)



#### 数组和切片

数组类型的值长度是固定的，而切片类型的值是可变长的。数组的长度在声明的时候就必须给出，并且之后不会改变，而且数组的长度是其类型的一部分。[1]string 与 [2]string是两个不同的数组类型。

![sliceandarray](../images/sliceandarray.png)



Go语言切片类型属于引用类型，同属于引用类型的还有字典类型、通道类型、函数类型；数组属于值类型，同属于值类型的有基础数据类型及结构体类型。

~~~go
  slice1 := []int{0,1,2,3,4,5,6}
  slice3 := []int{7,5}
  slice1 = append(slice1,slice3...)//一个切片append到另一个切片
	slice2 := slice1[3:6]
	fmt.Println("slice2 length:",len(slice2),",slice2 cap:",cap(slice2))
  //输出：slice2 length: 3 ,slice2 cap: 4
~~~

更通用的规则是：一个切片的容量可以被看作是透过这个窗口最多可以看到的底层数组中元素的个数。S4是通过在S3shang施加切片操作得来的，所以S3d底层数组就是S4的底层数组。在底层数组不变的情况下，切片代表的窗口可以向右扩展，直至其底层数组的末尾。注意，**切片代表的窗口是无法向左扩展的**。

**一定要注意切片的扩容**

~~~go
var x []int
	//x := []int{}
	x = append(x, 0)
	x = append(x, 1)
	x = append(x, 2)
	y := append(x, 3)
	z := append(x, 4)
	fmt.Println(y, z)
	fmt.Println(len(x),cap(x))
  //[0 1 2 4] [0 1 2 4]
  //3 4

func doAppend(a []int) {
	_ = append(a, 0)
}


func main() {
	a := []int{1, 2, 3, 4, 5}
	doAppend(a[0:2])
	fmt.Println(a)//[1 2 0 4 5]
}

~~~

Go 语言字典的键类型不可以是**函数类型、字典类型和切片类型**。

#### 通道

通道类型的值是并发安全的，这也是Go语言自带的，唯一一个可以满足并发安全性的类型。声明并初始化通道的时候使用make关键字，第一个参数代表通道的类型、第二个可选参数表示该通道的容量。当容量为0时，也叫非缓冲通道。**一个通道相当于一个先进先出(FIFO)的队列**，通道中格格元素值都是严格地按照发送的顺序排列的，先被发送通道的元素一定会先被接收。

#### 通道发送和接收的特征：

- 对同一个通道，发送操作之间是互斥的，操作之间也是互斥的。
- 发送操作和接收操作中对元素值的处理都是不可分割的。
- 发送操作在完全完成之前会被阻塞，接收操作也是如此。

同一时刻，Go语言的运行时系统只会执行对同一个通道的任意个发送操作中的某一个。直到这个元素值被完全复制进该通道，针对该通道的其他操作才可能被执行。对接收操作也是。

对于缓冲通道，当通道已满时，对它的所有发送操作都会被阻塞，直到通道中有元素被接收走。如果通道已空，对它的所有接收操作都会被阻塞，直到通道中有新的元素出现。对于阻塞的goroutine，会顺序进入通道内的队列，公平的等待接收通知。对于非缓冲通道，只有发送端和接收端都准备好之后，才会继续传递。所以，非缓冲通道是用同步的方式传递数据。**当对已关闭的通道进行发送操作时或者关闭已关了的通道时**，会引发panic。

当用两个值来接收通道的值，第二个变量的类型则一定是bool，如果为false则说明通道已关闭，如果通道关闭，通道里还有值，那么第一个值仍会是通道中的某一个元素值，而第二个结果一定是true。所以如果通过第二个值来判断通道是否关闭可能会有延时。因此，要让发送方来关闭通道。

Select  case 中如果有多个case满足条件，会采用伪随机的情况选中一个case。

#### 结构体

结构体比较的时候，只有相同类型的结构体才可以用==比较，**结构体是否相同不但与属性个数有关，还与属性顺序相关**。如果结构体中有不可比较的类型，如map，slice也不能用==进行比较，但可以用reflect.DeepEqual()进行比较。

#### 接口

**如果我们使用一个变量给另外一个变量赋值，那么真正赋值给后者的，并不是前者持有的那个值，而是该值的一个副本。**例如，声明并初始化一个Dog类型的变量dog1，这时它的name是"little dog"，然后把dog1赋值给dog2，再接着修改了dog1的name字段的值。这时，dog2的name字段的值依旧是"little dog"。

~~~go
  var d *Dog
	 fmt.Println("The first dog is nil.",d == nil)
	 d1 := d
	fmt.Println("The second dog is nil.",d1 == nil)
	var p Pet=d1
	fmt.Println("The third dos is nil???",p==nil)
	//输出结果：true true false
~~~

那么，怎样才能让一个接口变量的值真正为nil?

要么只声明它但不做初始化，要么直接把字面量nil 赋值给它。当把一个有类型的nil赋值给它时，比如上边的例子，p的值实际上是一个*Dog类型的你来，此时Go语言会用一个iface的实例包装它，包装后就不是nil了。当我们给接口变量赋值时，接口变量会持有被赋予值的副本，而不是它本身。更重要的是，接口变量的值并不等同于这个可被称为动态值的副本。它会包含两个指针，一个指针指向动态值，一个指针指向类型信息。基于此，**即使我们把一个值为<code>nil</code>的某个实现类型的变量赋给了接口变量，后者的值也不可能是真正的<code>nil</code>。虽然这时它的动态值会为<code>nil</code>，但它的动态类型确是存在的。**

#### 指针

1、不可变的值不可寻址。常量、基本类型的值字面量、字符串变量的值、函数以及方法的字面量都是如此。

2、绝大多数被视为临时结果的值也都是不可寻址的。如算术操作的结果，针对值字面量的表达式结果值。

3、若拿到某值的指针可能会破坏程序的一致性、那么就是不安全的，该值不可寻址。比如字典。

#### Goroutine

调度器中三个主要元素：G(goroutine)、P(processor)、M(machine)。M指代的就是系统级线程，P指的是一种可以承载若干个G，并且能使G适时地与M进行对接。G和M存在多对多的关系。如图：

![goroutine](../images/goroutine.png)

以下代码输出结果为空：

~~~go
func main(){
  for i:= 0; i< 10; i++ {
    go funnc(){
      fmt.Println(i)
    }()
  }
}
~~~

类型字面量struct{}有些类似于空接口类型interface{}，表示既不包含任何字段也不拥有任何方法的空结构体类型。struct{}类型值的表示法只有一个，即struct{}{}，并且它占用的内存空间是0字节，这个值在整个Go程序中永远只存在一份。