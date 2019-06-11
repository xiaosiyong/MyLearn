### 关于锁

最次：public volatile

其次  lock（）效率比较低

最优  Interlocked.Increment(ref this.counter);

连接：https://stackoverflow.com/questions/154551/volatile-vs-interlocked-vs-lock

### CLR

所有类型均派生字System.Object 所以所有类型至少有四种方法：

1、Equals

2、GetHashCode

3、Tostring

4、GetType

类型转换is和as  is永远不会抛异常判断2次   as也不会抛出异常 判断转型后的对象是否为null，判断一次 效率较is高

堆上的所有对象都加载两个额外的成员，类型对象指针和同步块儿索引

int i=5;

如果

Console.WriteLine(i);

Console.WriteLine(i.toString())的效率 

int32 v=5；

Cons.WriteLine("{0},{1},{2}",v,v,v);效率比int32 v=5；Object  o=v;Cons.WriteLine("{0},{1},{2}",o,o,o);低

引用类型、值类型、装箱，拆箱114

当某个字段是ReadOnly时，不可改变的是引用，而不是字段引用的对象。

List<T> 实现了IList<T>,ICollection<T>,IEnumerable<T>接口

115精华

泛型推断时，C#使用变量的数据类型，而不是由变量引用的对象的实际类型。

基类：is—a，接口：can-do

CLR的核心功能：**内存管理，程序集加载，安全性，异常处理和线程同步**

### 深入理解C#笔记

5.1.3  

P71  扩展方法  （接口类似）

public static class MyExtentionMethods{

public static decimal TotalPrices(this ShoppingCart cartParam){

decimal total=0;

foreach(Product p in cartParam.Products){

total+=p.Prices;

}

return total;

}

}

5.15  过滤扩展方法

public static IEnumerable<Product> FilterByCategory(this IEnumerable<Product> productEnum,string categoryParam){

  foreach(Product p in productEnum){

  if(p.Category==categoryParam){

  yied return p;

  }

  }

}

扩展方法采用了一个附加参数，允许在调用这个方法的时候注入一个过滤条件。

P82 LING的非延迟查询与非延迟查询