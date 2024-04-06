# Go

[toc]

### 环境

##### 环境配置

1，Go开启代理：```go env -w GOPROXY=https://goproxy.cn,direct```，查看环境参数：```go env```，系统变量path:  ``` GoInstallPath\bin``` ，GOROOT: ```GoInstallPath```

2，GOPATH:  工作目录，下有：`src pkg bin`子目录，

* `src` 目录包含Go的源文件，它们被组织成包（每个目录都对应一个包），
* `pkg` 目录包含包对象，以包为单位进行构建，保留src下目录形式，main包不会在此构建。
* `bin` 目录包含可执行命令，以main包下的main函数为入口。

3，在控制面板->用户环境变量中修改 `GOPATH`,`GOPATH`可存在多个，优先级依次降低，`GOPATH`的 `bin` 子目录添加到系统变量中

4，找包顺序：```gomod =off : goroot/src->gopath0/src->gopath1/src->...```

​		    		   ``` gomod =on  : goroot/pkg/mod->gopath0/pkg/mod->gopath1/pkg/mod->...```

5，自动初始化目录：```sh ./create_go_proj.sh porject_name```

```shell
#!/bin/bash

#########################################################
# Useage : ./create_go_proj
# sh ./create_go_proj.sh porject_name
# Description: 创建一个go可编译的工程
#————————————–————————————–
# 默认情况下运行本程序，会生成如下目录和文件:
# test
# ├── bin
# ├── install.sh
# ├── pkg
# └── src
# ├── config
# │   └── config.go
# └── test
# └── main.go
#————————————–————————————–
# 5 directories, 3 files
#
# 其中:
# 1, install.sh为安装文件，
# 2, config.go为test项目的配置文件
# 3, main.go
#
# 生成完毕之后运行进入test目录，运行install.sh会生成如下文件和目录
# ├── bin
# │   └── test
# ├── install.sh
# ├── pkg
# │   └── darwin_amd64
# │   └── config.a
# └── src
# ├── config
# │   └── config.go
# └── test
# └── main.go
# 6 directories, 5 files
#
# 多了两个文件
# 1, bin目录下的test，这个是可执行文件
# 2, pkg/darwin_amd64下的config.a，这个是config编译后产生的文件
#
#########################################################

PWD=$(pwd)
cd $PWD

if [[ "$1" = "" ]]; then
echo "Useage: ./mk_go_pro.sh porject_name"
echo -ne "Please input the Porject Name[test]"
read Answer
if [ "$Answer" = "" ]; then
echo -e "test";
PRO_NAME=test;
else
PRO_NAME=$Answer;
fi
else
PRO_NAME=$1;
fi

#########################################################

#创建目录
echo "Init Directory …"
mkdir -p $PRO_NAME/bin
mkdir -p $PRO_NAME/pkg
mkdir -p $PRO_NAME/src/config
mkdir -p $PRO_NAME/src/$PRO_NAME


#########################################################
#创建 install.sh 文件
echo "Create install/install.sh …"
cd $PRO_NAME
echo "#!/bin/bash" > install.sh
echo "if [ ! -f install.sh ]; then" >> install.sh
echo "echo "install must be run within its container folder" 1>&2" >> install.sh
echo "exit 1" >> install.sh
echo "fi" >> install.sh
echo >> install.sh
echo "CURDIR=\`pwd\`" >> install.sh
echo "OLDGOPATH=\"\$GOPATH\"" >> install.sh
echo "export GOPATH=\"\$CURDIR\"" >> install.sh
echo >> install.sh
echo "gofmt -w src" >> install.sh
echo "go install $PRO_NAME" >> install.sh
echo "export GOPATH=\"\$OLDGOPATH\"" >> install.sh
echo >> install.sh
echo "echo "finished"" >>install.sh
chmod +x install.sh

#创建 config.go 文件
echo "Create src/config/config.go …"
cd src/config
echo package config > config.go
echo >> config.go
echo func LoadConfig\(\) { >> config.go
echo >> config.go
echo "}" >> config.go

#创建 main.go
echo "Create src/$PRO_NAME/main.go …"
cd ../$PRO_NAME/
echo "package main" > main.go
echo >> main.go
echo "import (" >> main.go
echo " \"config\"" >> main.go
echo " \"fmt\"" >> main.go
echo ")" >> main.go
echo >> main.go
echo "func main() {" >> main.go
echo " config.LoadConfig()" >> main.go
echo " fmt.Println(\"Hello $PRO_NAME!\")" >> main.go
echo "}" >> main.go
echo "All Done!"
```

##### IDE

1，Goland:将当前工程路径加入```project gopath```,```proj gopath ```优先于``` glob gopath```

2，vscode : setting.json

```json
// gopath支持多个路径，路径间用 ; 分割，src优先级依次降低。
// vscode中gopath会覆盖系统变量中GOPATH
// F:\\GoBasePath中安装vscode运行需要的包
"go.gopath": "F:\\GoBasePath;G:\\GolandProject\\LearnGo"
```

##### 包

1，包的构建：`go build packagePath`，若在该包的源码目录中:`go build`，如果构建的是库源码文件（库源码文件是不能被直接运行的源码文件，它仅用于存放程序实体，这些程序实体可以被其他代码使用），那么操作的结果文件只会存在于临时目录中。这里的构建的主要意义在于检查和验证；如果构建的是命令源码文件（即包含 main() 入口函数的源码文件”），在命令行执行目录内生成可执行文件，文件一般会与命令源码文件的直接父目录同名。

2，如果被当前包依赖的代码包的归档文件不存在，或者源码文件有了变化，那它会自动编译依赖包，否则默认不会编译目标代码包所依赖的那些代码包。

3，如果要强制编译依赖包：加入标记`-a`；如果不但要编译依赖的代码包，还要安装它们的归档文件：加入标记`-i`；查看`go build`命令具体都执行了哪些操作:	加入标记`-x`；只查看`go build`具体操作而不执行它们:	加入标记`-n`；查看`go build`命令编译的代码包的名称:	加入标记`-v`，（常与`-a`标记搭配使用）

4，包的构建并安装:`go install packagePath`，在包文件夹内简化为:`go install`，安装某个代码包而产生的归档文件是与这个代码包同名的。安装操作会先执行构建，然后还会进行链接操作，并且把结果文件搬运到指定目录。如果安装的是库源码文件，安装该包及他所用到的其他包，生成的静态链接库文件会被搬运到它所在工作区的 pkg 目录下的某个子目录中；如果安装的是命令源码文件，命令会构建产生一个以`main`函数为入口的可执行的二进制文件，文件一般会与命令源码文件的直接父目录同名，结果文件会被搬运到它所在工作区的 bin 目录中，或者环境变量`GOBIN`指向的目录中。并会自动编译所依赖的任何包。

5，构建和安装代码包的时候都会执行编译、打包等操作

6，包的获取：`go get` 就会自动地获取、 构建并安装远程包,在`src,pkg`下保存相应文件：

```shell
go get packageURL
    -u：下载并安装代码包，不论工作区中是否已存在它们。
    -d：只下载代码包，不安装代码包。
    -fix：在下载代码包后先运行一个用于根据当前 Go 语言版本修正代码的工具，然后再安装代码包。
    -t：同时下载测试所需的代码包。
    -insecure：允许通过非安全的网络协议下载和安装代码包。HTTP 就是这样的协议。
```

7，包的根路径为：`src`目录。

8，代码用圆括号组合了导入，这是“分组”形式的导入语句。  包名与导入路径的最后一个元素一致。如`"math/rand"` 包中的源码均以 `package rand` 语句开始。  

9，链接成单个二进制文件的所有包，其包名无需是唯一的，只有导入路径（它们的完整文件名） 才是唯一的。

10，可执行命令必须使用 `package main`，并包含一个无参数、无返回值的`main`函数，作为程序入口。

11，包应当以小写的单个单词来命名，且不应使用下划线或驼峰记法.

12，使用包结构可以帮助选择好的名称,通过文件名来判定使用的包，不会产生混淆的。如：`io.Reader,bufio.Reader`

14，代码包的名称可以和源码文件所在的目录不同名。如果报名与文件夹名不一致，要保证该文件夹下文件都为自定义的包名导包时以文件夹名导入，但以包名来调用包内容。构建与安装时使用文件夹名，生成归档文件也以文件夹命名。

15，如果导入的多个包的最末路径一致，且包名与文件夹名相同会产生冲突

```go
import alias packageName //使用别名
import _ packageName   	 //如果该包未被显示调用可以使用匿名导入
impoer . packageName	 //导入包内全部内容，直接使用包内内容，无需通过包名调用
```

16，内部包：代码包命名为`internal`的包，该代码包的直接父包及其子包中的代码可以引用内部包的大写可导出内容。其它包内不能访问内部包的内容。

### 编写

##### 注释

1，注释：`/* */ , //`

2，每个包都应包含一段包注释，即放置在包子句前的一个块注释。对于包含多个文件的包， 包注释只需出现在其中的任一文件中即可。包注释应在整体上对该包进行介绍，并提供包的相关信息。在包中，任何顶级声明前面的注释都将作为该声明的文档注释。在程序中，每个可导出（首字母大写）的名称都应该有文档注释。在`package, const, type, func`等`关键字`上面并且紧邻关键字

3，`type, const, func`以名称为注释的开头, `package`以`Package name`为注释的开头

```go
// Package banana ...
package banana

// Xyz ...
const Xyz = 1

// Abc ...
type Abc struct {}

// Bcd ...
func Bcd() {}

```

##### 空白标识符

1，空白标识符可被赋予或声明为任何类型的任何值，而其值会被无害地丢弃。

2，有时导入某个包只是为了其副作用， 而没有任何明确的使用。让编译器停止关于未使用导入的提示，需要空白标识符来引用已导入包中的符号。

3，将未使用的变量  赋予空白标识符也能关闭未使用变量错误。

```go
var _ io.Reader
 _ = fd
import _ "net/http/pprof"
```

##### 格式

1，无论如何，不应将一个控制结构（`if`、`else`、`for`、`switch` 或 `select`）的左大括号放在下一行。由于编译器会在代码中自动加入分号，如果这样做，就会在大括号前面插入一个分号，这可能引起不需要的效果。

### 数据类型

##### 基本类型

| 序号 | 类型和描述                                                   |
| ---- | ------------------------------------------------------------ |
| 1    | 布尔型: 布尔型的值只可以是常量 true 或者 false。             |
| 2    | 数字类型:整型 int 和浮点型 float32、float64.                 |
| 3    | 字符串类型:Go 的字符串是由单个字节连接起来的。字符串的字节使用 UTF-8 编码标识 Unicode 文本。 |
| 4    | 派生类型:  (a) 指针类型（Pointer）(b) 数组类型 (c) 结构化类型(struct) (d) Channel 类型 (e) 函数类型 (f) 切片类型 (g) 接口类型（interface） (h) Map 类型 |

| 序号 | 类型和描述                                                   |
| ---- | ------------------------------------------------------------ |
| 1    | **uint8** 无符号 8 位整型 (0 到 255)                         |
| 2    | **uint16** 无符号 16 位整型 (0 到 65535)                     |
| 3    | **uint32** 无符号 32 位整型 (0 到 4294967295)                |
| 4    | **uint64** 无符号 64 位整型 (0 到 18446744073709551615)      |
| 5    | **int8** 有符号 8 位整型 (-128 到 127)                       |
| 6    | **int16** 有符号 16 位整型 (-32768 到 32767)                 |
| 7    | **int32** 有符号 32 位整型 (-2147483648 到 2147483647)       |
| 8    | **int64** 有符号 64 位整型 (-9223372036854775808 到 9223372036854775807) |

| 序号 | 类型和描述                                     |
| ---- | ---------------------------------------------- |
| 1    | **float32** IEEE-754 32位浮点型数              |
| 2    | **float64** IEEE-754 64位浮点型数              |
| 3    | **complex64** 32 位实数和虚数:complex128(1+2i) |
| 4    | **complex128** 64 位实数和虚数                 |

| 序号 | 类型和描述                               |
| ---- | ---------------------------------------- |
| 1    | **byte** 类似 uint8                      |
| 2    | **rune** 类似 int32                      |
| 3    | **uint** 32 或 64 位                     |
| 4    | **int** 与 uint 一样大小                 |
| 5    | **uintptr** 无符号整型，用于存放一个指针 |

2，自定义数据类型

`type Seq []int`

3，表达式 `T(v)` 将值 `v` 转换为类型 `T`。 Go 在不同类型的项之间赋值时需要显式转换（上转、下转都要显示转换）。转换过程并不会创建新值，它只是值暂让现有的时看起来有个新类型而已。

```go
type Seq []int
a:=Seq{1,2,3}
b:=([]int(a))
```

4，转换会创建新值，如从整数转换为浮点数等。

```go
a:=1.2
b:=int(a)
```

##### 变量

1，Go中约定使用驼峰记法 `MixedCaps` 或 `mixedCaps`。名字以大写字母开头，那么它就是已导出的。任何“未导出”的名字在该包外均无法访问。

``` math.Pi ✔ math.pi ❌```

|                                  |                                                              |
| -------------------------------- | ------------------------------------------------------------ |
| C  :  基本类型  含变量名的表达式 | 如``` int *p => *p指向int => p为指向int的指针```,即求解以p为变量的表达式来获得p的类型，繁琐。 |
| Go: 变量名 基本类型/复合类型     | 如``` p *int```,p为指向int的指针，简单明了。                 |

2，`var` 语句用于声明一个变量列表:```var a,b,c int=1,2,3```如果初始化值已存在,则可以省略类型,变量会从初始值中获得类型:```var a,b,c=1,2,false```当右边包含未指明类型的数值常量时，新变量的类型取决于常量的精度：```f := 3.142 // float64```，变量声明也可以“分组”成一个语法块。  

```go
var (
	i   int       = -1
	j 	bool      = true
)
```

​	3，在函数中，简洁赋值语句 `:=` 可在类型明确的地方代替 `var` 声明。 `:=` 结构一般表示不太重要的变量，如方法的局部变量，所以不能在函数外使用。var x type 表示重要的变量，如全局变量。	```i:=0 == var i=0 == var i int =0```

4，在满足下列所有条件时，已被声明的变量 `v` 可被重声明可出现在`:=` 声明中：

* 本次声明与已声明的 `v` 处于同一作用域中，本次是对v的重新赋值（若 `v` 已在外层作用域中声明过，则此次声明会创建一个新的局部变量），

* 在初始化中与其类型相应的值才能赋予 `v`

* 在此次声明中至少另有一个变量是新声明的。

  ```go
  i := 1
  i, j := 2, 3
  ```

5，使用`var a=1,a:=1`这种声明时不指明变量类型，而使用类型推断来确定数据类型。Go 语言的类型推断可以明显提升程序的灵活性，可以随意改变右侧表达式返回值的类型，使得代码重构变得更加容易，同时因为Go语言为静态语言，类型检查在编译期间完成类型检查，所以不会给代码的维护带来额外负担，更不会损失程序的运行效率。

6，变量找寻顺序

`当前块 -> 函数域 -> 当前文件=当前包=导入的包(使用全导入方式导入)`

7，重声明与重名

* 变量重声明中的变量一定是在某一个代码块内的。

  可重名变量指的正是在多个代码块之间由相同的标识符代表的变量。

* 变量重声明是对同一个变量的多次声明，这里的变量只有一个，之后的声明只是赋值。

  可重名变量中涉及的变量肯定是有多个的。

* 不论对变量重声明多少次，其类型必须始终一致，具体遵从它第一次被声明时给定的类型。

  可重名变量之间不存在类似的限制，它们的类型可以是任意的。

* 如果可重名变量所在的代码块之间存在直接或间接的嵌套关系，那么它们之间一定会存在“屏蔽”的现象。

  但是这种现象绝对不会在变量重声明的场景下出现。

* 类型别名：`type aliasName typeName`,仍为同一种类型，方法属性都相同，只是名称不同而已。

  >大型重构应该支持一个过渡期：从旧位置和新位置获得的 API 都应该是可用的，而且可以混合使用这些 API 的引用。
  >
  >类型别名提供了一种机制，使得 oldpkg.OldType 和  newpkg.NewType 是相同的，并且引用旧名称的代码与引用新名称的代码可以互相操作。
  >
  >考虑将一个类型`oldTYpe`重构为`newType`,在之前引用`oldType`的代码中加入`type oldType newType`,以前的代码都无需修改既可运行。

* 类型再定义：`type name typeNmae` ,两种不同类型，即使两个类型的潜在类型相同，它们的值之间也不能进行判等或比较，它们的变量之间也不能赋值。

* 如果我们使用一个变量给另外一个变量赋值，那么真正赋给后者的，并不是前者持有的那个值，而是该值的一个副本。

##### 常量

1，使用 `const` 关键字。  常量不能用 `:=` 语法声明。它们在编译时创建。

2，常量只能是数字、字符、字符串或布尔值。由于编译时的限制， 定义它们的表达式必须也是可被编译器求值的常量表达式，只能使用字面值。

`const a int=1<<3 ✔  const b float=math.Sin(math.Pi/4)❌`

### 流程控制

##### 函数

1，当连续两个或多个函数的形参、已命名返回值类型相同时，除最后一个类型以外，其它都可以省略。``` f(x int, y int)(a,b string)==f(x ,y int)(a,b string)```

2，函数可以返回任意数量的返回值,函数声明时相应的也要指明多个返回值的类型。

```go
func add(a int ,b int )(int,int,int){
    return a,b，a+b
}
```

3，Go函数的返回值“形参”可被命名，它们会被视作定义在函数顶部的变量，就像传入的形参一样。命名后，一旦该函数开始执行，它们就会被初始化为与其类型相应的零值； 若该函数执行了一条不带实参的 `return` 语句，则结果形参的当前值将按返回值的声明顺序被返回。

```go
func split(sum int) (x, y int) {
	y = sum * 4 / 9
	x = sum - y
	return // return x,y
}
```

4，函数也是值。它们可以像其它值一样传递，函数值可以用作函数的参数或返回值。  

```go
var hypot = func(x, y float64) float64 {
	return math.Sqrt(x*x + y*y)
}

func compute(fn func(float64, float64) float64) float64 {
	return fn(3, 4)
}

hypot(5, 12)
compute(hypot)
```

5，Go 函数可以是一个闭包。闭包是一个函数值，它引用了其函数体之外的变量。该函数可以访问并赋予其引用的变量的值，换句话说，该函数被这些变量“绑定”在一起。 保证了函数内引用变量的生命周期与函数的活动时间相同。可以把闭包简单理解成"定义在一个函数内部的函数"，在本质上，闭包就是将函数内部和函数外部连接起来的一座桥梁。闭包最大用处有两个，一个是前可以读取函数内部的变量，另一个就是让这些变量的值始终保持在内存中。由于闭包会使得函数中的变量都被保存在内存中，内存消耗很大，所以不能滥用闭包，否则会性能问题，甚至可能导致IE6内存泄露（内存泄露是指用不到（访问不到）的变量，依然占居着内存空间，不能被再次利用起来,IE6自身问题，与闭包无关）。解决方法是，在退出函数之前，将不使用的局部变量全部删除。闭包实质上涉及的就是词法作用域和将函数作为值return，以及返回的函数不会立即执行，将函数作为值传递，实际上就是打开了一条访问内部变量的通道。

```go
func adder() func(int) int {
	sum := 0 //自由变量，非函数参数或者在匿名函数内部定义
    opt := func(x int) int {
		sum += x
        return sum
	}
	return opt 
    // 函数adder中的局部变量sum一直保存在内存中，因为opt通过一个指针指向它，而opt又一直存在于内存中，所以sum并没有在adder调用后被自动清除,不会被垃圾回收机制（garbage collection）回收。
}

func main() {
	pos, neg := adder(), adder() //每个闭包都被绑定在其各自的 sum 变量上。 
	for i := 0; i < 10; i++ {
		fmt.Println(pos(i),neg(-1*i))
	}
}

0    0
1 	 -1
3    -3
6 	 -6

```

内部函数可以引用外部函数的参数和局部变量，当外部函数返回内部函数时，相关参数和变量都通过指向变量的指针保存在返回的函数中，返回的函数并没有立刻执行，而是直到调用了`f()`才执行，所以变量不是在内部函数创建时就创建一个当前数值的快照保存到函数中，而是通过指针引用，直到调用时才通过指针去获取当前的值，这在父函数内部循环创建多个闭包，时如果闭包函数引用了循环变量要特别注意，可以通过在闭包函数外层再添加一层函数，并且立即调用，把变量保存在外部函数中，再在内部闭包内引用。

```js
function count() {
    var arr = [];
    for (var i = 1; i <= 3; i++) {
        arr.push(function () {
            return i * i;
        });
    }
    return arr;
}

var results = count();
console.log(results[0]()); //16
console.log(results[1]()); //16
console.log(results[2]()); //16


function count() {
    var arr = [];
    for (var i=1; i<=3; i++) {
        arr.push((function (n) {
            return function () {
                return n * n;
            }
        })(i));// 马上把当前循环项的item与事件回调相关联起来
    }
    return arr;
}

var results = count();
var f1 = results[0];
var f2 = results[1];
var f3 = results[2];

f1(); // 1
f2(); // 4
f3(); // 9
```

在没有`class`机制，只有函数的语言里，借助闭包，把父函数当作对象（object）使用，把闭包当作它的公用方法（Public Method），把内部变量当作它的私有属性（private value），返回的对象中，实现了一个闭包，该闭包携带了局部变量`x`，它的状态可以完全对外隐藏起来，从外部代码根本无法访问到变量`x`，只能通过闭包函数实现访问，并且多次调用父类方法创建多个闭包，相当于创建多个是实例，各个实例内变量各自独立，类似Java中的Record类型。

闭包会在父函数外部，改变父函数内部变量的值。 JavaScript 代码都是基于事件的 — 定义某种行为，然后将其添加到用户触发的事件之上，代码通常作为回调：为响应事件而执行的函数，将闭包函数绑定在特定行为上，实现触发特定行为。

```js
function makeSizer(size) {
  return function() {
    document.body.style.fontSize = size + 'px';
  };
}

var size12 = makeSizer(12);
var size14 = makeSizer(14);
var size16 = makeSizer(16);

document.getElementById('size-12').onclick = size12;
document.getElementById('size-14').onclick = size14;
document.getElementById('size-16').onclick = size16;
```

与OOP编程语言的区别

```
func makeFunc(){
    var a = 1
    var b = function(){
        console.log(a)
    }
    return b
}

Class A
{
    private int x = 1; 
    public int value()
    {
        return x;
    }
}

二者看似无区别，但实际，上面的代码是省略一部分状语的。编译器自动补全this,变量x其实是实例上的成员。成员函数的使用必须依附于类的实例。当然如果用闭包的设计来理解的话也可以对应的理解成 「实例是其拥有的成员函数的环境」。不过，一些以闭包为特性的语言比如JS里，函数可以捕获很多层作用域之外的变量，因此也较之有着更好的表达力。

Class A
{
    private int x = 1; 
    public int value()
    {  
        return this.x;
    }
}

```

6，匿名函数: 

```go
func(a int)int{
    a+=1
	return a
}(-1)
```

7，可变参

```go
func Min(a ...int) int {
    for _, i := range a {
            ops
        }
}
```

8，数据传递类型

| 值类型变量                           | 引用类型变量         |
| ------------------------------------ | -------------------- |
| 基本数据类型(bool int float string ) | 值传递类型的指针形式 |
| 结构体                               | 映射                 |
| 数组                                 | 切片                 |
| 函数接收者                           | 信道                 |
|                                      | 函数                 |

9，值类型的变量在赋值与函数参数传递时使用值传递，使用浅拷贝

10，引用类型的变量在赋值与函数参数传递时使用引用传递

* 函数形参为指针：实参必须为指针。函数形参为值：实参必须为值。

* 函数返回给调用方的结果值也会被复制。

* 使用指针接收者的原因：  方法能够修改其接收者指向的值；这样可以避免在每次调用方法时复制该值。若值的类型为大型结构体时，这样做会更加高效。  


11，函数式编程

  ```go
  type f func(int ,string)(int string) // 函数签名为参数和返回值类型，参数和返回值类型相同就可以认为是f类型
  ```

##### defer

1，``defer`` 语句会将函数推迟到外层函数返回之后执行。  推迟的函数调用会被压入一个栈中，位于当前函数之下。当外层函数返回时，被推迟的函数会按照后进先出的顺序`FILO`调用。

```go
i=0
for i := 0; i < 10; i++ {
    defer fmt.Println(i)
}
//输出： 9 8 7 6 5 4 3 2 1 0
```

2，推迟调用的函数其参数会立即求值即参数被固定，与闭包不同，但直到外层函数返回前该函数都不会被调用。  

```go
func main() {
    defer a(b("c")) // b("c")会立即执行,并把返回值最为a的参数，但a在main执行完毕后才执行
	d()
}
```

3，可以用于资源关闭等操纵，推迟诸如 `Close` 之类的函数调用：

* 能确保不会忘记关闭文件。
* 它意味着“关闭”离“打开”很近，比将它放在函数结尾处要清晰明了。

##### 输出

1，```fmt.Printf()```实现格式化输出，```fmt.Sprintf()```返回指定格式的字符串。

| 占位符 | 含义                                     |
| ------ | ---------------------------------------- |
| %T     | 变量的类型                               |
| %v     | 变量的值                                 |
| %+v    | 当打印结构体时，添加字段名的变量值       |
| %#v    | 当打印结构体时，相应值的Go语法表示       |
| %f     | 浮点数                                   |
| %s     | 输出字符串表示,解释转义符                |
| %q     | 双引号围绕的字符串，不解释转义符原样输出 |

##### 循环

1，`for` 循环由三部分组成，它们用分号隔开：  

* 初始化语句：在第一次迭代前执行（可选)

* 条件表达式：在每次迭代前求值

* 后置语句：在每次迭代的结尾执行

初始化语句通常为一句短变量声明，该变量声明仅在 `for` 语句的作用域中可见。  


```go
for i := 0; i < 10; i++ {
		ops
	}
```

```go
// 如同C的for循环
for init; condition; post { }

// 如同C的while循环
for condition { }

// 如同C的for(;;)循环
for { }
```

3，要在 `for` 中使用多个变量，应采用平行赋值的方式

```go
for i, j := 0, len(a)-1; i < j; i, j = i+1, j-1 {
	ops;
}
```

4，`for` 循环的 `range` 形式可遍历集合类数据如切片、映射、数组、字符串。  当使用 `for` 循环遍历切片时，每次迭代都会返回两个值。第一个值为当前元素的下标，第二个值为该下标所对应元素的一份副本。  `for i,j := range data`可以将下标或值赋予 `_` 来忽略它。  若只需要索引，忽略第二个变量即可。  `for i,_ := range data`

5，字符串使用unicode存储

```go
str := "we 我"
for i, j := range str {
    fmt.Printf("%d,%d,%d,%#U \n", i, j,str[i], str[i])
}
0,119,119,U+0077 'w' 
1,101,101,U+0065 'e' 
2,32,32,U+0020 ' ' 
3,25105,230,U+00E6 'æ' 
```

##### 判断

1，```if```表达式外无需小括号 `( )` ，而大括号 `{ }` 则是必须的。  

3，`if` 语句可以在条件表达式前执行一个简单的语句。  该语句声明的变量作用域仅在 `if` 和对应的 `else` 块中使用之内。 

```go
if v := math.Pow(i, j); v < lim {
    return v
} else {
    fmt.Printf("%g >= %g\n", v, lim)
}
```

###### switch

1，只运行选定的 case，而非之后所有的 case，Go 自动提供了在这些语言中每个 case 后面所需的 `break` 语句,没有穿透效应，除非以 `fallthrough` 语句结束分支，否则分支会自动终止。     加入 `break` 语句可以使 `switch` 在`break`处提前终止。

2，switch 的 case 无需为常量，且取值不必为整数。  

```go
switch os := runtime.GOOS; os {
    case "darwin":
    fmt.Println("OS X.")
    case "linux":
    fmt.Println("Linux.")
    default:
    fmt.Printf("%s.\n", os)
}
```

3，没有条件的 switch 同 `switch true` 一样。  这种形式能将一长串 if-then-else 写得更加清晰。  

 ```go
os := runtime.GOOS
switch  {
    case  os=="darwin":
    fmt.Println("OS X.")
    case  os=="linux":
    fmt.Println("Linux.")
    default:
    fmt.Printf("%s.\n", os)
}
 ```

4， `case` 可通过逗号分隔来列举相同的处理条件。

```go
switch c {
	case ' ', '?', '&', '=', '#', '+', '%':
		ops
	}
```

##### 指针

1，类型 `*T` 是指向 `T` 类型值的指针。其零值为 `nil`。  `&` 操作符会生成一个指向其操作数的指针。  `*` 操作符表示指针指向的底层值。  

2，Go 没有指针运算，即无法对指针加减。

```go
var (
    i int = -1
    p *int
)
p = &i
i++
(*p)++
```

### 内存分配

1，`new()`用来分配内存的内建函数，它不会初始化内存，只会将内存置零。 `new(T)` 会为类型为 `T` 的新项分配根据`T`的成员的类型置零的内存空间， 并返回它的地址，也就是一个类型为 `*T` 的值。它返回一个指针， 该指针指向新分配的，类型为 `T` 的成员对应类型的零值。

```go
type SyncedBuffer struct {
	lock    sync.Mutex
	buffer  bytes.Buffer
}
p := new(SyncedBuffer)  // type *SyncedBuffer ：{nil,nil}
p := &(SyncedBuffer{})  // type *SyncedBuffer : {nil,nil}
```

2，内建函数 `make(T, *arg)` 的目的不同于 `new(T)`。它只用于创建切片、映射和信道，并返回类型为 `T`（而非 `*T`）的一个已初始化（而非置零）的值。出现这种用差异的原因在于，这三种类型本质上为引用数据类型，它们在使用前必须初始化。 