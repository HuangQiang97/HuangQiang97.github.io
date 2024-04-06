[toc]

# 测试



1：JUnit编写单元测试的好处在于，我们可以非常简单地组织测试代码，并随时运行它们，JUnit就会给出成功的测试和失败的测试，还可以生成测试报告，不仅包含测试的成功率，还可以统计测试的代码覆盖率，即被测试的代码本身有多少经过了测试。对于高质量的代码来说，测试覆盖率应该在80%以上。

所谓测试驱动开发，是指先编写接口，紧接着编写测试。编写完测试后，我们才开始真正编写实现代码。在编写实现代码的过程中，一边写，一边测，什么时候测试全部通过了，那就表示编写的实现完成了：

```ascii
    编写接口
     │
     ▼
    编写测试
     │
     ▼
┌─> 编写实现
│    │
│ N  ▼
└── 运行测试
     │ Y
     ▼
    任务完成
```

### 编写测试

```java
public class Factorial {
    public static long fact(long n) {
        long r = 1;
        for (long i = 1; i <= n; i++) {
            r = r * i;
        }
        return r;
    }
}
```

当我们已经编写了一个`Factorial.java`文件后，我们想对其进行测试，需要编写一个对应的`FactorialTest.java`文件，以`Test`为后缀是一个惯例，并分别将其放入`src`和`test`目录中，并将test目录标记为test-src目录。

```java
import static org.junit.jupiter.api.Assertions.*;
import org.junit.jupiter.api.Test;

public class FactorialTest {
	// 把带有@Test的方法识别为测试方法
    @Test
    void testFact() {
        assertEquals(1, Factorial.fact(1)); //assertEquals(expected, actual)
    }
}
```

断言方法

* `assertEquals(expected, actual)`:期待相等,使用浮点数时，由于浮点数无法精确地进行比较，需要调用`assertEquals(double expected, double actual, double delta)`指定误差

- `assertTrue(x)`: 期待结果为`true`

- `assertFalse(x)`: 期待结果为`false`

- `assertNotNull(x)`: 期待结果为非`null`

- `assertArrayEquals(expected, actual)`: 期待结果为数组并与期望数组每个元素的值均相等

- `assertThrows()`:期待返回指定的异常，第一个参数、指定的异常。第二个参数`Executable`封装了我们要执行的会产生异常的代码，捕获到指定异常时表示通过测试，未捕获到异常，或者捕获到的异常类型不对，均表示测试失败。对可能发生的每种类型的异常都必须进行测试

    ```java
    @Test
    void testNegative() {
        assertThrows(IllegalArgumentException.class, new Executable() {
            @Override
            public void execute() throws Throwable {
                Factorial.fact(-1);
            }
        });
    }
    
    @Test
    void testNegative() {
        assertThrows(IllegalArgumentException.class, () -> {
            Factorial.fact(-1);
        });
    }
    ```



单元测试可以确保单个方法按照正确预期运行，如果修改了某个方法的代码，只需确保其对应的单元测试通过，即可认为改动正确。此外，测试代码本身就可以作为示例代码，用来演示如何调用该方法。

在编写单元测试的时候，我们要遵循一定的规范：

* 是单元测试代码本身必须非常简单，能一下看明白，决不能再为测试代码编写测试；

* 是每个单元测试应当互相独立，不依赖运行的顺序；

* 是测试时不但要覆盖常用测试用例，还要特别注意测试边界条件，例如输入为`0`，`null`，空字符串`""`等情况。

### Fixture

标记为`@BeforeEach`和`@AfterEach`的方法，它们会在运行每个`@Test`方法前后自动运行。通过`@BeforeEach`来初始化，通过`@AfterEach`来清理资源。

`@BeforeAll`和`@AfterAll`，它们在运行所有@Test前后运行，当一些资源初始化和清理可能更加繁琐，而且会耗费较长的时间，例如初始化数据库。在所有`@Test`方法运行前后仅运行一次，因此它们只能初始化静态变量。并且注解也只能标注在静态方法上。

```java
// 运行顺序
invokeBeforeAll(CalculatorTest.class);
for (Method testMethod : findTestMethods(CalculatorTest.class)) {
	// 每次运行一个`@Test`方法前，JUnit首先创建一个`XxxTest`实例，因此，每个`@Test`方法内部的成员变量都是独立的
    var test = new CalculatorTest(); 
    invokeBeforeEach(test);
    invokeTestMethod(test, testMethod);
    invokeAfterEach(test);
}
invokeAfterAll(CalculatorTest.class);
```

```
@BeforeEach
public void beforeEach() throws Exception { 
// 对于实例变量，在@BeforeEach中初始化，在@AfterEach中清理，它们在各个@Test方法中互不影响，因为是不同的实例；
} 

@AfterEach
public void afterEach() throws Exception { 
} 

@BeforeAll
static public void beforeAll() throws Exception { 
// 对于静态变量，在@BeforeAll中初始化，在@AfterAll中清理，它们在各个@Test方法中均是唯一实例，会影响各个@Test方法。
} 

@AfterAll
static public void afterAll() throws Exception { 
} 

```

注意到每次运行一个`@Test`方法前，JUnit首先创建一个`XxxTest`实例，因此，每个`@Test`方法内部的成员变量都是独立的，不能也无法把成员变量的状态从一个`@Test`方法带到另一个`@Test`方法。

* 运行``BeforeAll`初始化静态字段，静态字段属于类，所有对象共享相同的成员变量。
* 创建`XxxTest`实例`obj1`,运行``obj1.BeforeEach,obj1.Test1,obj1.AfterEach`,三个方法操作的是`obj1`,同时可以访问静态变量。
* 创建`XxxTest`实例`obj2`,运行``obj2.BeforeEach,obj2.Test2,obj2.AfterEach`,三个方法操作的是`obj2`,同时可以访问静态变量。
* 、、、
* 运行`AfterAll`处理静态字段

### 条件测试

JUnit根据不同的条件注解，决定是否运行当前的`@Test`方法

在运行测试的时候，有些时候，我们需要排出某些`@Test`方法，不要让它运行，这时，我们就可以给它标记一个`@Disabled`：

```java
@Disabled("被禁止的理由")
@Test
void testBug101() {
    // 这个测试不会运行，JUnit仍然识别出这是个测试方法，只是暂时不运行
}
```

```java
@Test
@EnabledOnOs({ OS.LINUX, OS.MAC })
void testLinuxAndMac() {
    // 当系统为linux或者mac时才运行本测试
}

@Test
@EnabledOnOs(OS.WINDOWS)
void testWindows() {
    // 当系统为win时才运行本测试
}

@DisabledOnOs(OS.WINDOWS)
@Test
void testNotOnWindows() {
    // 当系统不是为win时才运行本测试
}
```

```java
@Test
@DisabledOnJre(JRE.JAVA_8) 
void testOnJava9OrAbove() {
    // java8下不执行
}

@Test
@EnabledIfSystemProperty(named = "os.arch", matches = ".*64.*")
void testOnlyOn64bitSystem() {
    //只在64位系统下执行
}
```

### 参数化测试

一个测试方法需要接收至少一个参数，然后，传入一组参数反复运行。

向测试方法传递测试参数：

* 数组传入

    ```java
    @ParameterizedTest
    @ValueSource(ints = {0, 1, 5, 100})
    void testAbs(int x) {
    assertEquals(x, Math.abs(x));
    }
    ```

    

* 通过`@MethodSource`注解，它允许我们编写一个同名的静态方法来提供测试参数：

    ```java
    @ParameterizedTest
    @MethodSource
    void testCapitalize(String input, String result) {
        assertEquals(result, StringUtils.capitalize(input));
    }
    
    //  如果静态方法和测试方法的名称不同，@MethodSource也允许指定方法名
    static List<Arguments> testCapitalize() {
        return List.of( Arguments.arguments("abc", "Abc"), Arguments.arguments("APPLE", "Apple"));
    }
    ```

* 使用`@CsvSource`，它的每一个字符串表示一行，一行包含的若干参数用`,`分隔

    ```java
    @ParameterizedTest
    @CsvSource({ "abc, Abc", "APPLE, Apple", "gooD, Good" })
    void testCapitalize(String input, String result) {
        assertEquals(result, StringUtils.capitalize(input));
    }
    ```

* 当有大量的测试数据使用文件存储测试数据，使用``@CsvFileSource`指定文件，并且只在classpath(src目录下）中查找指定的CSV文件如果放在src目录下，路径就是：`test-capitalize.csv`，如果文件要放到`test`文件夹里面,路径就是`../test/test-capitalize.csv`，文件中一行表示一组参数，用逗号分隔，String不用加引号：`apple, Apple \n HELLO, Hello `。

    ```java
    @ParameterizedTest
    @CsvFileSource(resources = { "/test-capitalize.csv" })
    void testCapitalizeUsingCsvFile(String input, String result) {
        assertEquals(result, StringUtils.capitalize(input));
    }
    ```

    ```java
    @ParameterizedTest
    @CsvFileSource(resources = { "/test-capitalize.csv" })
    void testCapitalizeUsingCsvFile(String input, String result) {
        assertEquals(result, StringUtils.capitalize(input));
    }
    ```

# 正则

### 简介

1：

```java
// \d在String中要将\转义：\d -> "\\d"
str.matches(regexStr)
```

### 规则

单个字符的匹配规则如下：

| 正则表达式 | 规则                             | 可以匹配                       |
| :--------- | :------------------------------- | :----------------------------- |
| `A`        | 指定字符                         | `A`                            |
| `\u548c`   | 指定Unicode字符                  | `和`                           |
| `.`        | 任意字符                         | `a`，`b`，`&`，`0`             |
| `\d`       | 数字0~9                          | `0`~`9`                        |
| `\w`       | 大小写字母，数字和下划线         | `a` ~ `z`，`A` ~ `Z`，`0`~`9`，`_` |
| `\s`       | 空格、Tab键                      | 空格，Tab                      |
| `\D`       | 非数字                           | `a`，`A`，`&`，`_`，……         |
| `\W`       | 非\w，即非字母、非数字，非下划线 | `&`，`@`，`中`，……             |
| `\S`       | 非\s，即非空格非Tab              | `a`，`A`，`&`，`_`，……         |

多个字符的匹配规则如下：

| 正则表达式 | 规则                                      | 可以匹配                        |
| :--------- | :---------------------------------------- | :------------------------------ |
| `*`        | 任意个数字符，包括0个字符                 | A* : 空，`A`，`AA`，`AAA`，……   |
| `+`        | 至少1个字符                               | A+ : `A`，`AA`，`AAA`，……       |
| `?`        | 0个或1个字符                              | A? : 空，`A`                    |
| `{3}`      | 指定个数字符                              | A{3} : `AAA`                    |
| `{2,3}`    | 指定范围个数字符,匹配[n,m]个字符,包括n和m | A{2,3} : `AA`，`AAA`            |
| `{2,}`     | 至少n个字符,无上限                        | A{2,} : `AA`，`AAA`，`AAAA`，…… |
| `{0,3}`    | 最多n个字符，包含n                        | A{0,3 ：空，`A`，`AA`，`AAA`    |

### 复杂匹配

| 正则表达式     | 规则                 | 可以匹配                                         |
| :------------- | :------------------- | :----------------------------------------------- |
| ^              | 开头                 | 字符串开头，`^A\d{3}$`匹配：`"A001"`、`"A380"`。 |
| $              | 结尾                 | 字符串结束，`^A\d{3}$`匹配：`"A001"`、`"A380"`。 |
| [ABC]          | […]内任意字符        | A，B，C                                          |
| [A-F0-9xy]     | 指定范围的字符       | `A`，……，`F`，`0`，……，`9`，`x`，`y`             |
| [^A-F]         | 指定范围外的任意字符 | 非`A`~`F`                                        |
| AB\|CD\|EF     | AB或CD或EF           | `AB`，`CD`，`EF`                                 |
| (AB\|CD\|EF)GH | 组合                 | ABGH或CDGH或EFGH                                 |

`|`以括号或者另一个`|`为界，进行片段划分，不是只匹配两边的单个字符，并且不要随意加括号，因为括号还是分组匹配的界限符。

### 分组匹配

```java
 // 创建出一个Pattern对象，然后反复使用，就可以实现编译一次，多次匹配
 // 表达式用(...)分组可以通过Matcher对象快速提取子串
 Pattern pattern = Pattern.compile("(\\d{3,4})\\-(\\d{7,8})");
 pattern.matcher("010-12345678").matches(); // true
 // 获得Matcher对象:
 Matcher matcher = pattern.matcher("010-12345678");
 // 使用Matcher时，必须首先调用matches()判断是否匹配成功，匹配成功后，才能调用group()提取子串
 if (matcher.matches()) {
     String whole = matcher.group(0); // "010-12345678", 0表示匹配的整个字符串
     String area = matcher.group(1); // "010", 1表示匹配的第1个子串
     String tel = matcher.group(2); // "12345678", 2表示匹配的第2个子串
 }
```

### 非贪婪匹配

则表达式默认使用贪婪匹配：任何一个规则，它总是尽可能多地向后匹配：

```java
Pattern.compile("(\\d+)(0*)").matcher("1230000"); // ["1230000","1230000",""]
```

在规则后面加个`?`即可表示非贪婪匹配

```java
Pattern.compile("(\\d+?)(0*)").matcher("1230000"); // ["1230000","123","0000"]
```

`\d??` : 表示零个或者一个数字的非贪婪搜索模式

### 分割字符串

`String.split(strReg)`方法传入的正是正则表达式,将`strReg`作为分隔符，风格字符串。

```java
"a b c".split("\\s"); // { "a", "b", "c" }
"a b  c".split("\\s"); // { "a", "b", "", "c" }
"a, b ;; c".split("[\\,\\;\\s]+"); // { "a", "b", "c" }
```

### 查找字符串

```java
String s = "the quick brown fox jumps over the lazy dog.";
Pattern p = Pattern.compile("\\wo\\w");
Matcher m = p.matcher(s);
// 不需要调用matches()方法（因为匹配整个串肯定返回false），而是反复调用find()方法，在整个串中搜索能匹配上规则的子串
while (m.find()) {
	// 此时m就指向匹配到子串
	// 此时调用 m.group(1)就是对匹配到的子串按规则进行分组，由于匹配到的子串必然符合规则，所以不用matches()判断。
    String sub = s.substring(m.start(), m.end());
    System.out.println(sub);
}
```

### 替换字符串

替换字符串可以直接调用`String.replaceAll()`，它的第一个参数是正则表达式，第二个参数是待替换的字符串。

```java
String s = "The     quick\t\t brown   fox  jumps   over the  lazy dog.";
String r = s.replaceAll("\\s+", " "); //"The quick brown fox jumps over the lazy dog."
```
7：

Matcher（字符串+规则）的方法

* `matches()`：字符串是否符合规则
* `group()`：在字符串符合规则的前提下，返回按规则分组的字符串
* `find()` ：查找字符串中符合规则的子串，此时matcher指向子串

# 加密

### 编码

1：URL编码

非ASCII字符转ASCII字符后发送给服务器

- 如果字符是`A`~ `Z`，`a` ~ `z`，`0` ~ `9`以及`-`、`_`、`.`、`*`，则保持不变；
- 如果是其他字符，先转换为UTF-8编码，然后对每个字节（两位16进制数）以`%XX`表示。

服务器收到URL编码的字符串，对其进行解码，还原成原始字符串

2：Base64

对二进制数据进行编码，表示成文本格式。只包含`A`~ `Z`、`a` ~ `z`、`0`~`9`、`+`、`/`、`=`

原理是把3字节(6位16进制数）的二进制数据按6bit一组，用4个int整数表示，然后查表，把int整数用索引对应到字符，得到编码后的字符串。6位二进制的范围总是`0`~ `63`，查表对应到`A`~ `Z`、`a` ~ `z`、`0`~`9`、`+`、`/`、`=`

Base64编码的缺点是传输效率会降低，因为它把原始数据的长度增加了1/3。

3：URLBase64

标准的Base64编码会出现`+`、`/`和`=`，所以不适合把Base64编码后的字符串放到URL中。URLBase64把`+`变成`-`，`/`变成`_`

### 哈希算法

1：对任意一组输入数据进行计算，得到一个固定长度的输出摘要。

2：哈希碰撞是指，两个不同的输入得到了相同的输出，碰撞是一定会出现的，因为输入的数据长度是不固定的，有无数种输入，输出却是有限的，本质为数据压缩。哈希算法是把一个无限的输入集合映射到一个有限的输出集合，必然会产生碰撞。哈希算法的输出长度越长，就越难产生碰撞，也就越安全。

3：用途：文件校验，登录密码验证

4：彩虹表：预先计算好的常用口令和它们的MD5的对照表。

对策：对每个口令额外添加随机数，这个方法称之为加盐（salt），使黑客的彩虹表失效，即使用户使用常用口令，也无法从MD5反推原始口令。

### Hmac算法

1：Hmac算法总是和某种哈希算法配合起来用的。例如，使用MD5算法，对应的就是HmacMD5算法，它相当于“加盐”的MD5：``HmacMD5 ≈ md5(secure_random_key, input)``

2: 本质上就是把key混入摘要的算法。验证此哈希时，除了原始的输入数据，还要提供key。

### 对称加密算法

1: 对称加密算法就是传统的用一个密码进行加密和解密,常用的对称加密算法有：`DES,AES,IDEA`

### 口令加密算法

1: 用户输入的口令，由于位数不够或者不够随机，通常还需要使用PBE算法，采用随机数杂凑计算出真正的密钥，再进行加密。

```java
// 把用户输入的口令和一个安全随机的口令采用杂凑后计算出真正的密钥
key = generate(userPassword, secureRandomPassword);
```

2：把随机生成的salt（secureRandomPassword）存储在U盘，就得到了一个“口令”加USB Key的加密软件，它的好处在于，即使用户使用了一个非常弱的口令，没有USB Key仍然无法解密，因为USB Key存储的随机数密钥安全性非常高。

### 密钥交换算法

1：解决安全传递密钥的问题，DH算法解决了密钥在双方不直接传递密钥的情况下完成密钥交换，DH算法是一个密钥协商算法，双方最终协商出一个共同的密钥，而这个密钥不会通过网络传输。

2：DH算法的本质就是双方各自生成自己的私钥和公钥，私钥仅对自己可见，然后交换公钥，并根据自己的私钥和对方的公钥，生成最终的密钥`secretKey`

3：但是DH算法并未解决中间人攻击，即甲乙双方并不能确保与自己通信的是否真的是对方。

### 非对称加密算法

1：非对称加密就是加密和解密使用的不是相同的密钥：只有同一个公钥-私钥对才能正常加解密。

2：对称加密需要协商密钥，而非对称加密可以安全地公开各自的公钥，在N个人之间通信的时候：使用非对称加密只需要N个密钥对，每个人只管理自己的密钥对。而使用对称加密需要则需要`N*(N-1)/2`个密钥，因此每个人需要管理`N-1`个密钥，

3：为非对称加密的缺点就是运算速度非常慢，比对称加密要慢很多

4：常见用法是用非对称加密对密钥进行加密，然后使用的对称加密算法传输数据，双方使用之前传输的密钥进行加密，解密。HTTPS就使用这种加密方式。

5：只使用非对称加密算法不能防止中间人攻击。

### 签名算法

1：通常是用公钥加密，私钥解密。如果使用私钥加密，公钥解密，由于私钥是保密的，而公钥是公开的，用私钥加密，那相当于所有人都可以用公钥解密，这个加密的意义在于，如果A用自己的私钥加密了一条消息，然后他公开了加密消息，由于任何人都可以用A的公钥解密，从而使得任何人都可以确认这条消息肯定是A发出的，其他人不能伪造这个消息，A也不能抵赖这条消息不是自己写的。

2:私钥加密得到的密文实际上就是数字签名，要验证这个签名是否正确，只能用私钥持有者的公钥进行解密验证。使用数字签名的目的是为了确认某个信息确实是由某个发送方发送的，任何人都不可能伪造消息，并且，发送方也不能抵赖。私钥就相当于用户身份。而公钥用来给外部验证用户身份。使用其他公钥，或者验证签名的时候修改原始信息，都无法验证成功。

3:数字签名用于：防止伪造；防止抵赖；检测篡改。

### 数字证书

1：数字证书=摘要算法用来确保数据没有被篡改+非对称加密算法可以对数据进行加解密+签名算法可以确保数据完整性和抗否认性。用于实现数据加解密、身份认证、签名等多种功能的一种安全标准。

2：数字证书可以防止中间人攻击，因为它采用链式签名认证，即通过根证书（Root CA）去签名下一级证书，这样层层签名，直到最终的用户证书。

3：以HTTPS协议为例，浏览器和服务器建立安全连接的步骤如下：

1. 浏览器向服务器发起请求，服务器向浏览器发送自己的数字证书；
2. 浏览器用操作系统内置的Root CA来验证服务器的证书是否有效，如果有效，就使用该证书加密一个随机的AES口令并发送给服务器；
3. 服务器用自己的私钥解密获得AES口令，并在后续通讯中使用AES加密。
4. 上述流程只是一种最常见的单向验证，如果服务器还要验证客户端，那么客户端也需要把自己的证书发送给服务器验证，这种场景常见于网银等

3：数字证书存储的是公钥，以及相关的证书链和算法信息。私钥必须严格保密，如果数字证书对应的私钥泄漏，就会造成严重的安全威胁。

