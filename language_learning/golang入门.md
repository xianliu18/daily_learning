### Golang 入门

#### 1. 程序开发注意事项
1，go 源文件以 `.go` 为扩展名；
2，go 语言严格区分大小写；
3，go 编译器是一行行进行编译的，一行只能写一条语句，不能把多条语句写在同一个；每个语句后不需要分号；
4，go 语言定义的**变量**或者 **import 的包**如果没有使用到，代码不能编译通过；

#### 1.1 编译和运行
- 编译程序：`go build hello.go`
- 运行程序：`go run hello.go`


<details>
<summary>hello.go<summary>

```go
package main

import "fmt"

func main() {
    var a int = 20
    // Println 自动增加换行
	fmt.Println("a = ", a)
    // Printf 格式化输出
	fmt.Printf("a 的类型: %T\n", a) 
}
```
</details>

#### 2. 数据类型

#### 2.1 变量与常量
- 数据类型作用：告诉编译器这个变量应该以多大的内存存储；
- 变量：`var 变量名称 变量类型`；
- 常量：`const 常量名称 常量类型`；
- 匿名变量：`_` 配合函数返回值使用；

<details>
<summary>变量示例</summary>

```go
func main() {
    // 变量声明
    var score int

    // 自动推导类型 :=
    // := 只能在声明“局部变量”的时候使用，而 var 没有这个限制
    name := "关羽"
    fmt.Printf("name 的类型: %T\n", name)

    // 同一类型，多变量声明
    var name, desc string
    name, desc = "阿离", "射手"

    // 常量自动推导，不用 :=
    const age = 20

    // 多个不同类型的变量或常量定义
    var (
        a int
        b float64
    )

    const (
        i int = 10
        j float64 = 3.14
    )

    // iota 常量计数器，在 const 中每新增一行，iota 自增 1，知道遇到下一个 const 关键字
    const (
        a int = iota    // 0
        b               // 1
        c               // 2
        d               // 3
    )
}
```
</details>

#### 2.2 数据类型
- 字符串：
  - go 字符串结尾都隐藏了一个结束符， '\0'；
- 类型转换：
  - go 中数据类型不能自动转换，在不同类型的变量之间赋值时，需要显式转换；
  - bool 类型不能转换为 int，整型不能转换为 bool；
- 类型别名：
  - 结构：`type bigint int64`

#### 2.3 流程控制
- `if` 条件判断语句里面允许声明一个变量；
- `switch` 默认每个 case 最后带有 `break`，匹配成功后不会自动向下执行其他 case；

```go
// if 条件判断语句
if age := 20; age > 18 {
    fmt.Println("哈哈哈")
    fmt.Println(age)
}

// for 循环
sum := 0
for i := 1; i <= 100; i++ {
    sum += i
}
fmt.Printf("sum = %d\n", sum)

// 关键字 range 会返回两个值，第一个返回值是元素的数组下标，第二个返回值是元素的值
var str string = "欢迎来到 golang world"
for index, value := range str {
    fmt.Printf("str[%d] = %c\n", index, value)
}
```

#### 3. 函数
1，函数的形参列表可以是多个，返回值列表也可以是多个；
2，函数也是一种数据类型，可以赋值给一个变量，也可以作为形参；
3，函数由关键词 `func` 开始声明；
4，函数名首字母小写即为 `private`，大写即为 `public`；

<details>
<summary>函数示例</summary>

```go
// 函数定义格式
func FuncName(/* 形参列表 */) (/* 返回类型 */) {
    // 函数体

    return v1, v2   // 可以返回多个值
}

// 可变参数的声明方式
func Function(args ...Type) {

}

// 当函数的形参类型一样时，前面的形参数据类型可以省略
func num(n1, n2 float32) float32 {
    return n1 + n2
}

// 函数也是一种数据类型
func Add(a, b int) int {
    return a + b
}

func minus(a, b int) int {
    return a - b
}

// 通过 type 给一个函数类型起名
type FuncType func(int, int) int

func main() {
    // 声明一个函数类型的变量，变量名 fTest
    var fTest FuncType
    fTest = Add
    result := fTest(10, 20) // 等价于 Add(10, 20)
    fmt.Printf("结果: %d\n", result)
}

// 函数别名应用场景：回调函数
// 回调函数：函数有一个参数是函数类型
// 多态，fTest 既传入 Add，也可以传入 minus
func Calc(a, b int, fTest FuncType) (result int) {
    // 支持对函数返回值命名
    result = fTest(a, b)
    return   // 此处 result 已与返回值建立对应关系，无需指明返回值
}

// 匿名函数与闭包
// 匿名函数：函数没有名称
mtFunc := func (n1 int, n2 int) int {   // := 自动推导
    return n1 + n2
}

// 闭包
// add 函数返回值为一个 匿名函数
func add() func(int) int {
    var n int = 10  // 上下文 n，受每次执行返回函数的影响
    return func(x int) int {
        n = n + x
        return n
    }
}

func main() {
    f := add()          
    fmt.Println(f(1))   // 11
    fmt.Println(f(2))   // 13
    fmt.Println(f(3))   // 16
}

// defer 延时机制
// 为了在函数执行完毕后，及时的释放资源（比如：数据库连接、文件句柄、锁等）
func sum(n1 int, n2 int) int {
    // 当执行到 defer 时候，暂时不执行，会将 defer 后面的语句“压入到独立的栈中”，然后继续执行函数下一个语句
    // 当函数执行反比后，再从 defer 栈，按照 “先入后出” 的方式出栈执行
    // defer 将语句放入到栈时，也会将相关的值拷贝入栈
    defer fmt.Println("n1 = ", n1)      // n1 值不受下面修改影响 10
    defer fmt.Println("n2 = ", n2)      // n2 值不受下面修改影响 20

    n1++
    n2++
    fmt.Println("结果为：", n1 + n2)
    return n1 + n2
}

func main() {
    res := sum(10, 20)
    fmt.Println("res = ", res)
}

// init 函数
// 每个源文件都可以包含一个 init 函数，该函数会在 main 函数执行前，被 go 运行框架调用

func init() {
    fmt.Println("init 执行...")
}

func main() {
    fmt.Println("main 执行....")
}
```
</details>

#### 4. 复杂数据类型
- 数组
- 切片
  - 切片是一个拥有相同类型元素的可变长度序列，是基于数组类型的一层封装；
  - 切片是一个引用类型，它的内部结构包含`地址`、`长度`和`容量`；
- map
- 指针
- 结构体

```go
// 数组声明
var a [5]int
// 初始化
a = [5]int{1, 2, 3, 4}
fmt.Println(a)

var c = [3]string{"北京", "上海", "深圳"}

// 多维数组
var c = [...][2] int {
    {1, 2},
    {3, 4},
    {5, 6},     // 末尾逗号不能省略
}

// 数组：
// 基本语法：var 数组变量名 [元素数量]T

// 切片(slice)：
// 基本语法： var name []T
// 源码：runtime/slice.go
type slice struct {
    array unsafe.Pointer    // 元素指针
    len   int               // 长度
    cap   int               // 容量
}

func main() {
    slice := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
    s1 := slice[2:5]
    s2 := s1[2:6:7]

    s2 = append(s2, 100)
    s2 = append(s2, 200)

    s1[2] = 20

    fmt.Println(s1)
    fmt.Println(s2)
    fmt.Println(slice)
}

// map
// 声明但未初始化map，此时是map的零值状态
map1 := make(map[string]string, 5)

map2 := make(map[string]string)

// 创建了初始化了一个空的的map，这个时候没有任何元素
map3 := map[string]string{}

// map中有三个值
map4 := map[string]string{"a": "1", "b": "2", "c": "3"}
```

#### 5. 异常处理
- `errors.New`：普通的错误；
  - `errors.New("描述信息")`
- `panic()`：标识非常严重的不可恢复的错误，必须先声明 defer，否则不能捕获到异常；
- `recover()`：用于从异常或错误场景中恢复；

```go
func Sqrt(f float64)(float64, error) {
    if f < 0 {
        return 0, errors.New("math - square root of negative number")
    }
}
```


#### 6. 接口

#### 7. go反射

#### 8. goroutine 和 select

#### 9. go 模板语法




<br/>

**参考资料：**
- [Golang基础知识点整理](https://www.cnblogs.com/bndong/p/16731102.html)
- [深度解密 Go 语言之 Slice](https://www.cnblogs.com/qcrao-2018/tag/Golang/)