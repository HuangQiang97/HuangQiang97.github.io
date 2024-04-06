[toc]

# Maven

### Maven介绍

[IDEA配置Maven](https://blog.csdn.net/qq_42057154/article/details/106114515)

1，结构

```ascii
a-maven-project
├── pom.xml
├── src
│   ├── main
│   │   ├── java	// Java源码目录
│   │   └── resources	// 资源文件的目录
│   └── test
│       ├── java	// 测试源码
│       └── resources	// 测试资源
└── target	// 所有编译、打包生成的文件
```

2，pom.xml

Maven工程就是由`groupId`，`artifactId`和`version`作为唯一标识。我们在引用其他第三方库的时候，也是通过这3个变量确定。Maven通过对jar包进行PGP签名确保任何一个jar包一经发布就无法修改。修改已发布jar包的唯一方法是发布一个新版本。因此，某个jar包一旦被Maven下载过，即可永久地安全缓存在本地。

```
<project ...>
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.itranswarp.learnjava</groupId>	// 类似于Java的包名，通常是公司或组织名称
	<artifactId>hello</artifactId>	// 类似于Java的类名，通常是项目名称
	<version>1.0</version>	// 版本号
	<packaging>jar</packaging>
	<properties>
        ...
	</properties>
	<dependencies>	// 使用<dependency>声明一个依赖后，Maven就会自动下载这个依赖包并把它放到classpath中
        <dependency>	
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
            <version>1.2</version>
        </dependency>
        <dependency>
          <groupId>org.junit.jupiter</groupId>
          <artifactId>junit-jupiter-engine</artifactId>
          <version>5.5.2</version>
          <scope>test</scope>// 该依赖类型，不写默认为compile
        </dependency>
	</dependencies>
</project>
```

3，依赖管理

当声明了自己的项目需要`abc`，Maven会自动导入`abc`的jar包，再判断出`abc`需要`xyz`，又会自动导入`xyz`的jar包，不断循环自动导入所有依赖包。

Maven定义了几种依赖关系，分别是`compile`、`test`、`runtime`和`provided`：

| scope    | 说明                                                         | 示例            |
| :------- | :----------------------------------------------------------- | :-------------- |
| compile  | 编译时需要用到该jar包（默认），Maven会把这种类型的依赖直接放入classpath | commons-logging |
| test     | 编译Test时需要用到该jar包                                    | junit           |
| runtime  | 编译时不需要，但运行时需要用到                               | mysql           |
| provided | 编译时需要用到，但运行时由JDK或某个服务器提供                | servlet-api     |

### 构建

Maven包含多种生命周期（lifecycle：default：构建的核心部分，编译，测试，打包，部署等，clean：在进行真正的构建之前进行一些清理工作，Site：生成项目报告，站点，发布站点），每个生命周期由一系列阶段（phase）构成，执行一个phase又会触发一个或多个goal，具体任务由一系列goal来完成，通常情况，我们总是执行phase默认绑定的goal，因此不必指定goal。。

mvn命令后面跟的是phase, maven自动根据phase确定对应的lifecycle（没有重名的phase），然后运行对应lifecycle中的第一个phase直到指定的phase

*   `mv phase_i `：单个phase，在默认生命周期中运行到指定的phase

*   `mvn clean package`:指定多个phase，Maven先执行`clean`生命周期并运行到`clean`这个phase，然后执行`default`生命周期并运行到`package`这个phase。

经常使用的命令有：

`mvn clean`：清理所有生成的class和jar；

`mvn clean compile`：先清理，再执行到`compile`；

`mvn clean test`：先清理，再执行到`test`，因为执行`test`前必须执行`compile`，所以这里不必指定`compile`；

`mvn clean package`：先清理，再执行到`package`。

### 插件

1，Maven执行`compile`这个phase，这个phase会调用`compiler`插件执行关联的`compiler:compile`这个goal。实际上，执行每个phase，都是通过某个插件（plugin）来执行的，Maven本身其实并不知道如何执行`compile`，它只是负责找到对应的`compiler`插件，然后执行默认的`compiler:compile`这个goal来完成编译。所以，使用Maven，实际上就是配置好需要使用的插件，然后通过phase调用它们。

### 模块

1，在软件开发中，把一个大项目分拆为多个模块是降低软件复杂度的有效方法，Maven可以有效地管理多个模块，我们只需要把每个模块当作一个独立的Maven项目，它们有各自独立的`pom.xml`

```ascii
multiple-project
├── pom.xml //根目录使用一个pom.xml统一编译：
├── parent
│   └── pom.xml //各个子模块的公共部分
├── module-a
│   ├── pom.xml //各个子模块部分
│   └── src
├── module-b
│   ├── pom.xml //各个子模块部分
│   └── src
└── module-c
    ├── pom.xml //各个子模块部分
    └── src
```

*   parent pom.xml

    ```
    <project xmlns="http://maven.apache.org/POM/4.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>
    
        <groupId>org.huangqiang.learnjava</groupId>
        <artifactId>parent</artifactId>
        <version>1.0</version>
        <packaging>pom</packaging> // <packaging>是pom而不是jar，因为parent本身不含任何Java代码。编写parent的pom.xml只是为了在各个模块中减少重复的配置
        <name>parent</name>
    
        <properties>
            <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
            <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
            <maven.compiler.source>16</maven.compiler.source>
            <maven.compiler.target>16</maven.compiler.target>
            <java.version>16</java.version>
        </properties>
    
        <dependencies>
            <dependency>
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-api</artifactId>
                <version>1.7.28</version>
            </dependency>
            <dependency>
                <groupId>ch.qos.logback</groupId>
                <artifactId>logback-classic</artifactId>
                <version>1.2.3</version>
                <scope>runtime</scope>
            </dependency>
            <dependency>
                <groupId>org.junit.jupiter</groupId>
                <artifactId>junit-jupiter-engine</artifactId>
                <version>5.5.2</version>
                <scope>test</scope>
            </dependency>
        </dependencies>
    </project>
    ```

*   module-a

    ```
    <project xmlns="http://maven.apache.org/POM/4.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>
    
        <parent> //引用parent
            <groupId>org.huangqiang.learnjava</groupId>
            <artifactId>parent</artifactId>
            <version>1.0</version>
            <relativePath>../parent/pom.xml</relativePath>
        </parent>
    
        <artifactId>module-a</artifactId>
        <packaging>jar</packaging>
        <name>module-a</name>
        
         <dependencies>
            <dependency> // 如果模块间相互依赖同样可以导入相邻模块
                <groupId>org.huangqiang.learnjava</groupId>
                <artifactId>module-b</artifactId>
                <version>1.0</version>
            </dependency>
        </dependencies>
        
    </project>
    ```

    

*   在编译的时候，需要在根目录创建一个`pom.xml`统一编译，在根目录执行`mvn clean package`时，Maven根据根目录的`pom.xml`找到包括`parent`在内的共4个`<module>`，一次性全部编译

    ```
    <project xmlns="http://maven.apache.org/POM/4.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    
        <modelVersion>4.0.0</modelVersion>
        <groupId>org.huangqiang.learnjava</groupId>
        <artifactId>build</artifactId>
        <version>1.0</version>
        <packaging>pom</packaging>
        <name>build</name>
    
        <modules> //各个模块
            <module>parent</module>
            <module>module-a</module>
            <module>module-b</module>
            <module>module-c</module>
        </modules>
    </project>
    ```

    

### mvnw

1，使用Maven Wrapper，它可以负责给这个特定的项目安装指定版本的Maven，而其他项目不受影响。给一个项目提供一个独立的，指定版本的Maven给它使用

2，

在项目的根目录（即`pom.xml`所在的目录）下：

```
mvn -N io.takari:maven:0.7.6:wrapper //使用最新版本的Maven
mvn -N io.takari:maven:0.7.6:wrapper -Dmaven=3.3.3 //指定使用的Maven版本
```

安装后，查看项目结构,把项目的`mvnw`、`mvnw.cmd`和`.mvn`提交到版本库中，可以使所有开发人员使用统一的Maven版本。：

```ascii
my-project
├── .mvn
│   └── wrapper
│       ├── MavenWrapperDownloader.java
│       ├── maven-wrapper.jar
│       └── maven-wrapper.properties
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   └── resources
    └── test
        ├── java
        └── resources
```

只需要把`mvn`命令改成`mvnw`就可以使用跟项目关联的Maven

# 网络

### 基础

-   计算机网络：由两台或更多计算机组成的网络；
-   互联网：连接网络的网络；
-   IP地址：计算机的网络接口（通常是网卡）在网络中的唯一标识；
-   网关：负责连接多个网络，并在多个网络之间转发数据的计算机，通常是路由器或交换机；
-   网络协议：互联网使用TCP/IP协议，它泛指互联网协议簇；
-   IP协议：一种分组交换传输协议；
-   TCP协议：一种面向连接，可靠传输的协议；
-   UDP协议：一种无连接，不可靠传输的协议。

### TCP

1，Socket是一个抽象概念，一个应用程序通过一个Socket来建立一个远程连接，而Socket内部通过TCP/IP协议把数据传输到网络。Socket、TCP和部分IP的功能都是由操作系统提供的，不同的编程语言只是提供了对操作系统调用的简单的封装。Java提供的几个Socket相关的类就封装了操作系统提供的接口。每个应用程序需要各自对应到不同的Socket，数据包才能根据Socket正确地发到对应的应用程序。

```ascii
┌───────────┐                              
│Application   │                                 
├───────────┤                                   
│  Socket      │                                   
├───────────┤                                  
│    TCP       │                                 
├───────────┤      ┌──────┐      
│    IP        │<───>│Router  │<────>
└───────────┘      └──────┘       
```

2，使用Socket进行网络编程时，本质上就是两个进程之间的网络通信。其中一个进程必须充当服务器端，它会主动监听某个指定的端口，另一个进程必须充当客户端，它必须主动连接服务器的IP地址和指定端口，如果连接成功，服务器端和客户端就成功地建立了一个TCP连接，双方后续就可以随时发送和接收数据。对服务器端来说，它的Socket是指定的IP地址和指定的端口号；对客户端来说，它的Socket是它所在计算机的IP地址和一个由操作系统分配的随机端口号。

3，使用Java进行TCP编程时，需要使用Socket模型：

-   服务器端用`ServerSocket`监听指定端口；
-   客户端使用`Socket(InetAddress, port)`连接服务器；
-   服务器端用`accept()`接收连接并返回`Socket`；如果没有客户端连接进来，`accept()`方法会阻塞并一直等待。如果有多个客户端同时连接进来，`ServerSocket`会把连接扔到队列里，然后一个一个处理
-   双方通过`Socket`打开`InputStream`/`OutputStream`读写数据；当Socket连接创建成功后，无论是服务器端，还是客户端，我们都使用`Socket`实例进行网络通信。因为TCP是一种基于流的协议
-   服务器端通常使用多线程同时处理多个客户端连接，利用线程池可大幅提升效率；
-   `flush()`用于强制输出缓冲区到网络。以流的形式写入数据的时候，并不是一写入就立刻发送到网络，而是先写入内存缓冲区，直到缓冲区满了以后，才会一次性真正发送到网络，这样设计的目的是为了提高传输效率。如果缓冲区的数据很少，而我们又想强制把这些数据发送到网络，就必须调用`flush()`强制把缓冲区数据发送出去。

```java
public class Server {
	public static void main(String[] args) throws IOException {
		ServerSocket ss = new ServerSocket(6666);
		ThreadPoolExecutor threadPoolExecutor=  new ThreadPoolExecutor(4,32,30, TimeUnit.SECONDS,new ArrayBlockingQueue<>(16), Executors.defaultThreadFactory(),new ThreadPoolExecutor.DiscardPolicy());
		for (;;) {
			Socket sock = ss.accept();
			threadPoolExecutor.execute(new Handler(sock));
		}
	}
}

class Handler implements  Runnable {
	Socket sock;
	public Handler(Socket sock) {
		this.sock = sock;
	}
	@Override
	public void run() {
		try (InputStream input = this.sock.getInputStream()) {
			try (OutputStream output = this.sock.getOutputStream()) {
				handle(input, output);
			}
		} catch (Exception e) {
			try {
				this.sock.close();
			} catch (IOException ignored) {
			}
		}
	}
}
```

```java
public class Client {
   public static void main(String[] args) throws IOException {
      Socket sock = new Socket("localhost", 6666);
      try (InputStream input = sock.getInputStream()) {
         try (OutputStream output = sock.getOutputStream()) {
            handle(input, output);
         }
      }
      sock.close();
   }
}
```

### UDP

1，UDP没有创建连接，数据包也是一次收发一个，所以没有流的概念。

2，UDP端口和TCP端口虽然都使用0~65535，但他们是两套独立的端口，即一个应用程序用TCP占用了端口1234，不影响另一个应用程序用UDP占用端口1234。

3，服务器端

```java
DatagramSocket ds = new DatagramSocket(6666); // 监听指定端口
for (;;) { // 无限循环
    // 数据缓冲区:
    byte[] buffer = new byte[1024];
    DatagramPacket packet = new DatagramPacket(buffer, buffer.length); // udp没有IO流接口，数据被直接写入byte[]缓冲区。
    // packet.getSocketAddress() // 获得客户端IP和端口
    ds.receive(packet); // 收取一个UDP数据包，没收到数据时将会阻塞，
    // 收取到的数据存储在buffer中，由packet.getOffset(), packet.getLength()指定起始位置和长度
    // 将其按UTF-8编码转换为String:
    String s = new String(packet.getData(), packet.getOffset(), packet.getLength(), StandardCharsets.UTF_8);
    // 发送数据:当服务器收到一个DatagramPacket后，通常必须立刻回复一个或多个UDP包，因为客户端地址在DatagramPacket中，每次循环处理一个客户，所以下一次收到的DatagramPacket可能是不同的客户端，如果不回复，将于客户端失去联系，客户端就收不到任何UDP包。
    byte[] data = "ACK".getBytes(StandardCharsets.UTF_8);
    packet.setData(data);
    ds.send(packet);
}
```

客户端
```java
DatagramSocket ds = new DatagramSocket(); // 客户端创建DatagramSocket实例时并不需要指定端口，而是由操作系统自动指定一个当前未使用的端口。
ds.setSoTimeout(1000); // 后续接收UDP包时，等待时间最多不会超过1秒，否则在没有收到UDP包时，客户端会无限等待下去。
ds.connect(InetAddress.getByName("localhost"), 6666); // 连接指定服务器和端口，connect()方法不是真连接，它是为了在客户端的DatagramSocket实例中保存服务器端的IP和端口号，确保这个DatagramSocket实例只能往指定的地址和端口发送UDP包，不能往其他地址和端口发送。这么做不是UDP的限制，而是Java内置了安全检查。如果客户端希望向两个不同的服务器发送UDP包，那么它必须创建两个DatagramSocket实例。
// 发送:
byte[] data = "Hello".getBytes();
DatagramPacket packet = new DatagramPacket(data, data.length);
ds.send(packet); // 通常来说，客户端必须先发UDP包，因为客户端不发UDP包，服务器端就根本不知道客户端的地址和端口号。
// 接收:
byte[] buffer = new byte[1024];
packet = new DatagramPacket(buffer, buffer.length);
ds.receive(packet); // 收不到数据时阻塞，等待Timeout毫秒后跳出阻塞。
String resp = new String(packet.getData(), packet.getOffset(), packet.getLength());
ds.disconnect(); // disconnect()也不是真正地断开连接，它只是清除了客户端DatagramSocket实例记录的远程服务器地址和端口号，这样，DatagramSocket实例就可以连接另一个服务器端。
```

### HTTP

1，HTTP请求的格式是固定的，它由HTTP Header和HTTP Body两部分构成。第一行总是`请求方法 路径 HTTP版本`，例如，`GET / HTTP/1.1`表示使用`GET`请求，路径是`/`，版本是`HTTP/1.1`。

如果是`GET`请求，那么该HTTP请求只有HTTP Header，没有HTTP Body，`GET`请求的参数必须附加在URL上，并以URLEncode方式编码，因为URL的长度限制，`GET`请求的参数不能太多。如果是`POST`请求，那么该HTTP请求带有Body，以一个空行分隔，body中就是请求的参数，``POST`请求的参数就没有长度限制，因为`POST`请求的参数必须放到Body中。

后续的每一行都是固定的`Header: Value`格式，我们称为HTTP Header，服务器依靠某些特定的Header来识别客户端请求，例如：

-   Host：表示请求的域名，因为一台服务器上可能有多个网站，因此有必要依靠Host来识别请求是发给哪个网站的；
-   User-Agent：表示客户端自身标识信息，不同的浏览器有不同的标识，服务器依靠User-Agent判断客户端类型是IE还是Chrome，是Firefox还是一个Python爬虫；
-   Accept：表示客户端能处理的HTTP响应格式，`*/*`表示任意格式，`text/*`表示任意文本，`image/png`表示PNG格式的图片；
-   Accept-Language：表示客户端接收的语言，多种语言按优先级排序，服务器依靠该字段给用户返回特定语言的网页版本。

2，HTTP响应也是由Header和Body两部分组成，响应的第一行总是`HTTP版本 响应代码 响应说明`，例如，`HTTP/1.1 200 OK`表示版本是`HTTP/1.1`，响应代码是`200`。客户端只依赖响应代码判断HTTP响应是否成功。HTTP有固定的响应代码：

-   1xx：表示一个提示性响应，例如101表示将切换协议，常见于WebSocket连接；
-   2xx：表示一个成功的响应，例如200表示成功，206表示只发送了部分内容；
-   3xx：表示一个重定向的响应，例如301表示永久重定向，303表示客户端应该按指定路径重新发送请求；
-   4xx：表示一个因为客户端问题导致的错误响应，例如400表示因为Content-Type等各种原因导致的无效请求，404表示指定的路径不存在；
-   5xx：表示一个因为服务器问题导致的错误响应，例如500表示服务器内部故障，503表示服务器暂时无法响应。

当浏览器收到第一个HTTP响应后，它解析HTML后，根据解析的内容又会发送一系列HTTP请求，请求需要的资源，例如，`GET /logo.jpg HTTP/1.1`请求一个图片，服务器响应图片请求后，会直接把二进制内容的图片发送给浏览器。

服务器总是被动地接收客户端的一个HTTP请求，然后响应它。客户端则根据需要发送若干个HTTP请求。

3，客户端的HTTP编程。

Get请求

```java
import java.net.URI;
import java.net.http.*;
import java.net.http.HttpClient.Version;
import java.time.Duration;
import java.util.*;

public class Main {
    // 全局HttpClient:
    static HttpClient httpClient = HttpClient.newBuilder().build(); // 创建一个全局HttpClient实例，因为HttpClient内部使用线程池优化多个HTTP连接，可以复用，使用链式调用的API，能大大简化HTTP的处理

    public static void main(String[] args) throws Exception {
        String url = "https://www.sina.com.cn/";
        HttpRequest request = HttpRequest.newBuilder(new URI(url))
            // 设置Header:
            .header("User-Agent", "Java HttpClient").header("Accept", "*/*")
            // 设置超时:
            .timeout(Duration.ofSeconds(5))
            // 设置版本:
            .version(Version.HTTP_2).build();
        // 通过内置的BodyHandlers来更方便地处理数据。
        // 如果要获取图片这样的二进制内容，只需要把HttpResponse.BodyHandlers.ofString()换成HttpResponse.BodyHandlers.ofByteArray()，就可以获得一个HttpResponse<byte[]>对象。如果响应的内容很大，不希望一次性全部加载到内存，可以使用HttpResponse.BodyHandlers.ofInputStream()获取一个InputStream流。
        HttpResponse<String> response = httpClient.send(request, HttpResponse.BodyHandlers.ofString()); 
        // HTTP允许重复的Header，因此一个Header可对应多个Value:
        Map<String, List<String>> headers = response.headers().map();
        for (String header : headers.keySet()) {
            System.out.println(header + ": " + headers.get(header).get(0));
        }
        System.out.println(response.body().substring(0, 1024) + "...");
    }
}
```

Post请求：要准备好发送的Body数据并正确设置`Content-Type

```java
String url = "http://www.example.com/login";
String body = "username=bob&password=123456";
HttpRequest request = HttpRequest.newBuilder(new URI(url))
    // 设置Header:
    .header("Accept", "*/*")
    .header("Content-Type", "application/x-www-form-urlencoded")
    // 设置超时:
    .timeout(Duration.ofSeconds(5))
    // 设置版本:
    .version(Version.HTTP_2)
    // 使用POST并设置Body,通过内置的BodyPublishers来更方便地处理数据
    .POST(BodyPublishers.ofString(body, StandardCharsets.UTF_8)).build();
HttpResponse<String> response = httpClient.send(request, HttpResponse.BodyHandlers.ofString());
String s = response.body();
```

### RMI

1，RMI：一个JVM中的代码可以通过网络实现远程调用另一个JVM的某个方法。提供服务的一方我们称之为服务器，而实现远程调用的一方我们称之为客户端。

2，服务器和客户端必须共享同一个接口。Java的RMI规定此接口必须派生自`java.rmi.Remote`，并在每个方法声明抛出`RemoteException`

*   接口

```java
public interface WorldClock extends Remote {
    LocalDateTime getLocalDateTime(String zoneId) throws RemoteException;
}
```

*   服务器端

```java
// 客户端请求的调用方法getLocalDateTime()最终会通过这个实现类返回结果
public class WorldClockService implements WorldClock {
    @Override
    public LocalDateTime getLocalDateTime(String zoneId) throws RemoteException {
        return LocalDateTime.now(ZoneId.of(zoneId)).withNano(0);
    }
}
```

通过Java RMI提供的一系列底层支持接口，把上面编写的服务以RMI的形式暴露在网络上，客户端才能调用：

```java
public class Server {
    public static void main(String[] args) throws RemoteException {
        System.out.println("create World clock remote service...");
        // 实例化一个WorldClock:
        WorldClock worldClock = new WorldClockService();
        // 将此服务转换为远程服务接口:
        WorldClock skeleton = (WorldClock) UnicastRemoteObject.exportObject(worldClock, 0);
        // RMI的默认端口是1099,将RMI服务注册到1099端口:
        Registry registry = LocateRegistry.createRegistry(1099);
        // 注册此服务，指定服务名为"WorldClock":
        registry.rebind("WorldClock", skeleton);
    }
```

*   客户端

```java
public class Client {
    public static void main(String[] args) throws RemoteException, NotBoundException {
        // 连接到服务器localhost，端口1099:
        Registry registry = LocateRegistry.getRegistry("localhost", 1099);
        // 查找名称为"WorldClock"的服务并强制转型为WorldClock接口:
        WorldClock worldClock = (WorldClock) registry.lookup("WorldClock");
        // 正常调用接口方法:
        LocalDateTime now = worldClock.getLocalDateTime("Asia/Shanghai");
        // 打印调用结果:
        System.out.println(now);
    }
}
```

3，先运行服务器，再运行客户端。RMI通过自动生成stub和skeleton实现网络调用，客户端只需要查找服务并获得接口实例，服务器端只需要编写实现类并注册为服务；从运行结果可知，因为客户端只有接口，并没有实现类，因此，客户端获得的接口方法返回值实际上是通过网络从服务器端获取的。对客户端来说，客户端持有的`WorldClock`接口实际上对应了一个“实现类”，它是由`Registry`内部动态生成的，并负责把方法调用通过网络传递到服务器端。而服务器端接收网络调用的服务并不是我们自己编写的`WorldClockService`，而是`Registry`自动生成的代码。我们把客户端的“实现类”称为`stub`，而服务器端的网络服务类称为`skeleton`，它会真正调用服务器端的`WorldClockService`，获取结果，然后把结果通过网络传递给客户端。整个过程由RMI底层负责实现序列化和反序列化，Java的RMI严重依赖序列化和反序列化，而这种情况下可能会造成严重的安全漏洞，因为Java的序列化和反序列化不但涉及到数据，还涉及到二进制的字节码，即使使用白名单机制也很难保证100%排除恶意构造的字节码。因此，使用RMI时，双方必须是内网互相信任的机器，不要把1099端口暴露在公网上作为对外服务。Java的RMI调用机制决定了双方必须是Java程序，其他语言很难调用Java的RMI。如果要使用不同语言进行RPC调用，可以选择更通用的协议，例如[gRPC](https://grpc.io/)。

![image-20210504122055342](C:\Users\huangqiang\AppData\Roaming\Typora\typora-user-images\image-20210504122055342.png)

# XML

### 结构

1，XML有几个特点：一是纯文本，默认使用UTF-8编码，二是可嵌套，适合表示结构化数据。

2，XML有固定的结构，首行必定是`<?xml version="1.0"?>`，可以加上可选的编码。紧接着，如果以类似`<!DOCTYPE note SYSTEM "book.dtd">`声明的是文档定义类型（DTD：Document Type Definition），DTD是可选的。接下来是XML的文档内容，一个XML文档有且仅有一个根元素，根元素可以包含任意个子元素，元素可以包含属性，例如，`<isbn lang="CN">1234567</isbn>`包含一个属性`lang="CN"`，且元素必须正确嵌套。如果是空元素，可以用`<tag/>`表示。

由于使用了`<`、`>`以及引号等标识符，如果内容出现了特殊符号，需要使用`&???;`表示转义。例如，`Java<tm>`必须写成：

```
<name>Java&lt;tm&gt;</name>
```

常见的特殊字符如下：

| 字符 | 表示   |
| ---- | ------ |
| <    | &lt;   |
| >    | &gt;   |
| &    | &amp;  |
| "    | &quot; |
| '    | &apos; |

格式正确的XML（Well Formed）是指XML的格式是正确的，可以被解析器正常读取。而合法的XML是指，不但XML格式正确，而且它的数据结构可以被DTD或者XSD验证。

3，XML是一种树形结构的文档，它有两种标准的解析API：

-   DOM：一次性读取XML，并在内存中完整表示为树形结构，把XML结构作为一个树形结构处理，从根节点开始，每个节点都可以包含任意个子节点。DOM解析速度慢，内存占用大。
-   SAX：以流的形式读取XML，使用事件回调，边读取XML边解析，并以事件回调的方式让调用者获取数据。因为是一边读一边解析，所以无论XML有多大，占用的内存都很小。

4，DOM，顶层的是document代表XML文档，它是真正的“根”

XML的内容：

-   Document：代表整个XML文档；
-   Element：代表一个XML元素；
-   Attribute：代表一个元素的某个属性。

```java
InputStream input = Main.class.getResourceAsStream("/book.xml");
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
DocumentBuilder db = dbf.newDocumentBuilder();
Document doc = db.parse(input); // 解析一个XML，它可以接收InputStream，File或者URL，如果解析无误，我们将获得一个Document对象，这个对象代表了整个XML文档的树形结构，需要遍历以便读取指定元素的值：
```

5，SAX解析会触发一系列事件：

-   startDocument：开始读取XML文档；
-   startElement：读取到了一个元素，例如`<book>`；
-   characters：读取到了字符；
-   endElement：读取到了一个结束的元素，例如`</book>`；
-   endDocument：读取XML文档结束。

```java
InputStream input = Main.class.getResourceAsStream("/book.xml");
SAXParserFactory spf = SAXParserFactory.newInstance();
SAXParser saxParser = spf.newSAXParser();
saxParser.parse(input, new MyHandler());
```

关键代码`SAXParser.parse()`除了需要传入一个`InputStream`外，还需要传入一个回调对象，这个对象要继承自`DefaultHandler`

由于SAX没有文件结构，如果要读取`<name>`节点的文本，我们就必须在解析过程中根据`startElement()`和`endElement()`定位当前正在读取的节点，可以使用栈结构保存，每遇到一个`startElement()`入栈，每遇到一个`endElement()`出栈，这样，读到`characters()`时我们才知道当前读取的文本是哪个节点的

6，XML文件结构可以对应到一个定义好的JavaBean，Jackson可以实现XML到JavaBean的转换。

Maven的依赖：

-   com.fasterxml.jackson.dataformat:jackson-dataformat-xml:2.10.1
-   org.codehaus.woodstox:woodstox-core-asl:4.4.1

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<book id="1">
    <name>Java核心技术</name>
    <author>Cay S. Horstmann</author>
    <isbn lang="CN">1234567</isbn>
    <tags>
        <tag>Java</tag>
        <tag>Network</tag>
    </tags>
    <pubDate/>
</book>
```

```java
public class Book {
    public long id;
    public String name;
    public String author;
    public String isbn;
    public List<String> tags;
    public String pubDate;
}

```

```java
InputStream input = Main.class.getResourceAsStream("/book.xml");
JacksonXmlModule module = new JacksonXmlModule();
XmlMapper mapper = new XmlMapper(module);
Book book = mapper.readValue(input, Book.class);
```

7，JSON是JavaScript Object Notation的缩写，它去除了所有JavaScript执行代码，只保留JavaScript的对象格式

```json
{
    "id": 1,
    "name": "Java核心技术",
    "author": {
        "firstName": "Abc",
        "lastName": "Xyz"
    },
    "isbn": "1234567",
    "tags": ["Java", "Network"]
}
```

几个显著的优点：

-   JSON只允许使用UTF-8编码，不存在编码问题；
-   JSON只允许使用双引号作为key，特殊字符用`\`转义，格式简单；键值对：`{"key": value}`，数组：`[1, 2, 3]`，字符串：`"abc"`，数值（整数和浮点数）：`12.34`，布尔值：`true`或`false`，空值：`null`
-   浏览器内置JSON支持，如果把数据用JSON发送给浏览器，可以用JavaScript直接处理。

JSON也可以转换为Java Bean，依赖：com.fasterxml.jackson.core:jackson-databind:2.10.0

```java
InputStream input = Main.class.getResourceAsStream("/book.json");
ObjectMapper mapper = new ObjectMapper();
// 反序列化时忽略不存在的JavaBean属性,使得解析时如果JavaBean不存在该属性时解析不会报错。
mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
// f反序列化：Json->JavaBean
Book book = mapper.readValue(input, Book.class);
// 序列化：JavaBean->Json
String json = mapper.writeValueAsString(book);
```

JSON还支持自定义序列化与反序列化，例如isbn类型不匹配，需要自定义序列化与反序列化器，

```java
// JSON 文件
{
    "name": "Java核心技术",
    "isbn": "978-7-111-54742-6"
}
// JavaBean
public class Book {
	public String name;
    // 表示反序列化isbn时使用自定义的IsbnDeserializer:
    @JsonDeserialize(using = IsbnDeserializer.class)
	public BigInteger isbn; 
}
```

```
public class IsbnDeserializer extends JsonDeserializer<BigInteger> {
    public BigInteger deserialize(JsonParser p, DeserializationContext ctxt) throws IOException, JsonProcessingException {
        // 读取原始的JSON字符串内容:
        String s = p.getValueAsString();
        if (s != null) {
            try {
                return new BigInteger(s.replace("-", ""));
            } catch (NumberFormatException e) {
                throw new JsonParseException(p, s, e);
            }
        }
        return null;
    }
}
```

# JDBC

### 简介

1,使用Java程序访问数据库时，Java代码通过JDBC接口来访问，而JDBC接口则通过JDBC驱动来实现真正对数据库的访问。注意到JDBC接口是Java标准库自带的，所以可以直接编译。而具体的JDBC驱动是由数据库厂商提供的，通过引入该厂商提供的JDBC驱动，就可以通过JDBC接口来访问，这样保证了Java程序编写的是一套数据库访问代码，却可以访问各种不同的数据库。Java程序编译期仅依赖java.sql包，不依赖具体数据库的jar包，可随时替换底层数据库，访问数据库的Java代码基本不变。

2，

```java
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.24</version>
    <scope>runtime</scope> //编译Java程序并不需要MySQL的这个jar包，只有在运行期才需要使用。如果把runtime改成compile，虽然也能正常编译，但是在IDE里写程序的时候，会多出来一大堆类似com.mysql.jdbc.Connection这样的类，非常容易与Java标准库的JDBC接口混淆，所以坚决不要设置为compile。
</dependency>
```

3，类型转换

| SQL数据类型   | Java数据类型             |
| :------------ | :----------------------- |
| BIT, BOOL     | boolean                  |
| INTEGER       | int                      |
| BIGINT        | long                     |
| REAL          | float                    |
| FLOAT, DOUBLE | double                   |
| CHAR, VARCHAR | String                   |
| DECIMAL       | BigDecimal               |
| DATE          | java.sql.Date, LocalDate |
| TIME          | java.sql.Time, LocalTime |

6，插入

### 查询

1，第一步，通过`Connection`提供的`createStatement()`方法创建一个`Statement`对象，用于执行一个查询，`Statment`是需要关闭的资源；

第二步，执行`Statement`对象提供的`executeQuery("SELECT * FROM students")`并传入SQL语句，执行查询并获得返回的结果集，使用`ResultSet`来引用这个结果集，`ResultSet`是需要关闭的资源，无论是什么查询语句JDBC查询的返回值总是`ResultSet`，即使使用聚合查询也不例外。

第三步，反复调用`ResultSet`的`next()`方法并读取每一行结果。`ResultSet`获取列时，索引从`1`开始而不是`0`；必须根据`SELECT`的列的对应位置来调用`getLong(1)`，`getString(2)`这些方法，否则对应位置的数据类型不对，将报错。

```java
// JDBC连接的URL, 不同数据库有不同的格式:
static final String JDBC_URL = "jdbc:mysql://localhost:3306/test";
static final String JDBC_USER = "root";
static final String JDBC_PASSWORD = "password";
// 获取连接:
try (Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD)) {
    try (Statement stmt = conn.createStatement()) {
        try (ResultSet rs = stmt.executeQuery("SELECT id, grade, name, gender FROM students WHERE gender=1")) {
            while (rs.next()) {
           		
                long id = rs.getLong(1); // 注意：索引从1开始，数据类型要和Selet中国类型顺序预要对应
                long grade = rs.getLong(2);
                String name = rs.getString(3);
                int gender = rs.getInt(4);
            }
        }
    }
}
```

使用`Statement`拼字符串非常容易引发SQL注入的问题，这是因为SQL参数往往是从方法参数传入的。

```java
// 参数：name = "bob' OR pass=", pass = " OR pass='"：，将导致直接跳过认证
stmt.executeQuery("SELECT * FROM user WHERE login='" + name + "' AND pass='" + pass + "'");
```

使用`PreparedStatement`可以*完全避免SQL注入*的问题，因为`PreparedStatement`始终使用`?`作为占位符，并且把数据连同SQL本身传给数据库，这样可以保证每次传给数据库的SQL语句是相同的，只是占位符的数据不同，还能高效利用数据库本身对查询的缓存。使用Java对数据库进行操作时，必须使用PreparedStatement，严禁任何通过参数拼字符串的代码！

```java
try (Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD)) {
    try (PreparedStatement ps = conn.prepareStatement("SELECT id, grade, name, gender FROM students WHERE gender=? AND grade=?")) {
        ps.setObject(1, "M"); // 必须首先调用setObject()设置每个占位符?的值，注意：索引从1开始
        ps.setObject(2, 3);
        try (ResultSet rs = ps.executeQuery()) {
            while (rs.next()) {
                long id = rs.getLong("id");// 使用String类型的列名比索引要易读，而且不易出错,也可以是使用下标读取。
                long grade = rs.getLong("grade");
                String name = rs.getString("name");
                String gender = rs.getString("gender");
            }
        }
    }
}
```

### 更新

1，插入

```java
try (PreparedStatement ps = conn.prepareStatement( "INSERT INTO students (id, grade, name, gender) VALUES (?,?,?,?)")) {
    ps.setObject(1, 999); // 注意：索引从1开始
    ps.setObject(2, 1); // grade
    ps.setObject(3, "Bob"); // name
    ps.setObject(4, "M"); // gender
    int n = ps.executeUpdate(); //执行更新，返回受影响的记录数量 
}
// 获取自增主键,指定一个RETURN_GENERATED_KEYS标志位，表示JDBC驱动必须返回插入的自增主键
try (PreparedStatement ps = conn.prepareStatement( "INSERT INTO students (grade, name, gender) VALUES (?,?,?)",
    Statement.RETURN_GENERATED_KEYS)) {
    ps.setObject(1, 1); // grade
    ps.setObject(2, "Bob"); // name
    ps.setObject(3, "M"); // gender
    int n = ps.executeUpdate(); 
    // rs对象包含了数据库自动生成的主键的值，读取该对象的每一行来获取每次插入自增主键的值。如果一次插入多条记录，那么这个ResultSet对象就会有多行返回值。如果插入时有多列自增，那么ResultSet对象的每一行都会对应多个自增值（自增列不一定必须是主键）。
    try (ResultSet rs = ps.getGeneratedKeys()) {
        if (rs.next()) {
            long id = rs.getLong(1); // 注意：索引从1开始
        }
    }
}

```

2，删除，更新

```java
// 除了SQL语句不同外其余完全相同
try (Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD)) {
    try (PreparedStatement ps = conn.prepareStatement("XXX")) {
        ps.setObject(1, 999); // 注意：索引从1开始
        int n = ps.executeUpdate(); // 影响的行数
    }
}
```

### 事务

1，数据库事务（Transaction）是由若干个SQL语句构成的一个操作序列，有点类似于Java的`synchronized`同步。数据库系统保证在一个事务中的所有SQL要么全部执行成功，要么全部不执行，ACID准则，Atomicity：原子性，事务的全部修改全部执行完成，或者全部修改失败。，Consistency：一致性，从一个合理的状态转移到另一个合理的状态，Isolation：隔离性，事务感知不到其它事务的存在，Durability：持久性，修改永久反映在DB中。

2，SQL标准定义了4种隔离级别，分别对应可能出现的数据不一致的情况：

```java
// 设定隔离级别为READ COMMITTED,MySQL的默认隔离级别是REPEATABLE READ。
conn.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);
```

| Isolation Level  | 脏读（Dirty Read） | 不可重复读（Non Repeatable Read） | 幻读（Phantom Read） |
| :--------------- | :----------------- | :-------------------------------- | :------------------- |
| Read Uncommitted | Yes                | Yes                               | Yes                  |
| Read Committed   | -                  | Yes                               | Yes                  |
| Repeatable Read  | -                  | -                                 | Yes                  |
| Serializable     | -                  | -                                 | -                    |

Read Uncommitted：可以读取其它事务已经修改但未提交的数据。不存在视图，直接到数据库中读取。事务B修改数据但未提交，事务A读取到B修改的数据，如果B事务回滚或者继续修改，那么之前事务A读到的数据就是脏数据是不存在的，这就是脏读（Dirty Read）。

Read Committed：可以读取其它事务已经提交的数据，每次执行语句前都要重新构建视图。不可重复读是指，在一个事务A内，第一次读一数据，在这个事务A还没有结束时，如果另一个事务B恰好修改并提交了这个数据，第二次A读取的是新的数据，与第一次不一致。通过对读取的行数据加锁解决问题。

Repeatable Read：事务一开始就构建要用到的数据的视图，即使其他事务更改了数据，也不会对视图产生影响，事务类读取的数据始终一致。幻读是指，在一个事务A中，第一次查询（select)某条记录发现没有，之后如果其它事务B添加了该数据并提交，此时A第二次查询（select)某条记录发现还是没有，但当A试图更新这条不存在的记录时竟然能成功，并且A第三次读取同一条记录，它就神奇地出现了。可以通过对表加锁解决问题。

Serializable：所有事务按照次序依次串行执行，因此，脏读、不可重复读、幻读都不会出现。

3，事务的思想

```java
Connection conn = openConnection();
try {
    // 关闭自动提交，提交事务的代码在执行完指定的若干条SQL语句后，调用conn.commit()
    conn.setAutoCommit(false);
    // 执行多条SQL语句，使用PreparedStatement执行。
    insert(); update(); delete();
    // 提交事务:
    conn.commit();
    // 如果事务提交失败，会抛出SQL异常，回滚事务，只可以自定义条件出发回滚。
} catch (SQLException e) {
    conn.rollback();
} finally {
	// 把Connection对象的状态恢复到初始值
    conn.setAutoCommit(true);
    conn.close();
}
```

默认情况下，我们获取到`Connection`连接后，总是处于“自动提交”模式，也就是每执行一条SQL都是作为事务自动执行的。

### 批处理

1，SQL数据库对SQL语句相同，但只有参数不同的若干语句可以作为batch执行，把同一个SQL但参数不同的若干次操作合并为一个batch执行，即批量执行，这种操作有特别优化，速度远远快于循环执行每个SQL。

```java
try (PreparedStatement ps = conn.prepareStatement("INSERT INTO students (name, gender, grade, score) VALUES (?, ?, ?, ?)")) {
    // 对同一个PreparedStatement反复设置参数并调用addBatch():
    for (Student s : students) {
        ps.setString(1, s.name);
        ps.setBoolean(2, s.gender);
        ps.setInt(3, s.grade);
        ps.setInt(4, s.score);
        ps.addBatch(); // 添加到batch，需要对同一个PreparedStatement反复设置参数并调用addBatch()，这样就相当于给一个SQL加上了多组参数，相当于变成了“多行”SQL。
    }
    // 执行batch,返回batch中每个SQL执行的结果数量
    int[] ns = ps.executeBatch();
    for (int n : ns) {
        System.out.println(n + " inserted."); 
    }
}
```

### 连接池

1，在执行JDBC的增删改查的操作时，如果每一次操作都来一次打开连接，操作，关闭连接，那么创建和销毁JDBC连接的开销就太大了。为了避免频繁地创建和销毁JDBC连接，我们可以通过连接池（Connection Pool）复用已经创建好的连接。JDBC连接池有一个标准的接口`javax.sql.DataSource`，要使用JDBC连接池，必须选择一个JDBC连接池的实现。常用的JDBC连接池有：HikariCP，C3P0等。

```xml
<!-- https://mvnrepository.com/artifact/com.zaxxer/HikariCP -->
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <version>3.3.1</version>
</dependency>

```

```java
//创建连接池，注意创建DataSource也是一个非常昂贵的操作，所以通常DataSource实例总是作为一个全局变量存储，并贯穿整个应用程序的生命周期。
HikariConfig config = new HikariConfig();
config.setJdbcUrl("jdbc:mysql://localhost:3306/test");
config.setUsername("root");
config.setPassword("password");
config.addDataSourceProperty("connectionTimeout", "1000"); // 连接超时：1秒
config.addDataSourceProperty("idleTimeout", "60000"); // 空闲超时：60秒，一个连接在空闲一段时间后自动关闭等
config.addDataSourceProperty("maximumPoolSize", "10"); // 最大连接数：10
DataSource ds = new HikariDataSource(config);
// 在此获取连接
try (Connection conn = ds.getConnection()) { 
    ...
} 
```

一开始，连接池内部并没有连接，所以，第一次调用`ds.getConnection()`，会迫使连接池内部先创建一个`Connection`，再返回给客户端使用。当我们调用`conn.close()`方法时（`在try(resource){...}`结束处），不是真正“关闭”连接，而是释放到连接池中，以便下次获取连接时能直接返回。因此，连接池内部维护了若干个`Connection`实例，如果调用`ds.getConnection()`，就选择一个空闲连接，并标记它为“正在使用”然后返回，如果对`Connection`调用`close()`，那么就把连接再次标记为“空闲”从而等待下次调用。

# 函数式编程

### 基础

1，Java不支持单独定义函数，但可以把静态方法视为独立的函数，把实例方法视为自带`this`参数的函数。

2，函数式编程就是一种抽象程度很高的编程范式，纯粹的函数式编程语言编写的函数没有变量，因此，任意一个函数，只要输入是确定的，输出就是确定的，这种纯函数我们称之为没有副作用。而允许使用变量的程序设计语言，由于函数内部的变量状态不确定，同样的输入，可能得到不同的输出，因此，这种函数是有副作用的。

3，函数式编程的一个特点就是，允许把函数本身作为参数传入另一个函数，还允许返回一个函数！函数式编程（Functional Programming）是把函数作为基本运算单元，函数可以作为变量，可以接收函数，还可以返回函数。

4，用Lambda表达式替换单方法接口

```java
Arrays.sort(array, new Comparator<String>() {
    public int compare(String s1, String s2) {
        return s1.compareTo(s2);
    }
});
// Lambda方式
// 参数是(s1, s2)，参数类型可以省略，因为编译器可以自动推断出类型，返回值的类型也是由编译器自动推断的。
Arrays.sort(array, (s1, s2) -> {
            return s1.compareTo(s2);
        });
// 简化版
Arrays.sort(array, (s1, s2) -> s1.compareTo(s2));
```

5，只定义了单抽象方法(`default`方法或`static`方法不算入接口定义的抽象方法）的接口称之为`FunctionalInterface`，用注解`@FunctionalInterface`标记

```java
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```

### 方法引用

1，可以向方法传入Lambda表达式和方法引用。

所谓方法引用，是指如果某个方法签名(参数列表和返回值列表）和接口恰好一致，就可以直接传入方法引用。（鸭子类型）

```java
public static void main(String[] args) {
    String[] array = new String[] { "Apple", "Orange", "Banana", "Lemon" };
    Arrays.sort(array, Main::cmp);
    Arrays.sort(array, (s1, s2) -> {return s1.compareTo(s2);});
}
// 因为Comparator<String>接口定义的方法是int compare(String, String)和cmp拥有相同的方法签名(参数列表和返回值列表），可以直接把方法名作为Lambda表达式传入
static int cmp(String s1, String s2) {
    return s1.compareTo(s2);
}
```

2，某些方法要求传入一个FunctionalInterface的实现类，可以向他传递该FunctionalInterface中定义方法的方法引用。`FunctionalInterface`不强制继承关系，不需要方法名称相同，只要求方法参数（类型和数量）与方法返回类型相同，即认为方法签名相同。

可以传入静态方法，实例方法，构造方法，

```java
// 期望方法：int Comparator<String>.compare(String, String)
// 传入静态方法，cmp完全满足该要求。
Arrays.sort(array, Main::cmp);
// 传入实例方法，string 中方法签名：int compareTo(String o)，但实例方法的第一个隐含参数总是传入this,即是int compareTo(this, String o);this属于String类型，所以满足要求。
Arrays.sort(array, String::compareTo);
// 构造方法，ClassName::new,构造方法虽然没有return语句，但它会隐式地返回this实例，map接受的方法 R apply(T t);与Persion构造方法：public Person(String name) {this.name = name;}，此处T为String，R为Persion,apply为构造方法，隐式地返回Persion实例,与要求相匹配。
List<Person> persons = List.of("Bob", "Alice", "Tim").stream().map(Person::new).collect(Collectors.toList());
```

### Stream

1，和文件流区别

|      | java.io                  | java.util.stream           |
| :--- | :----------------------- | -------------------------- |
| 存储 | 顺序读写的`byte`或`char` | 顺序输出的任意Java对象实例 |
| 用途 | 序列化至文件或网络       | 内存计算／业务逻辑         |

2，和集合区别

`List`存储的每个元素都是已经存储在内存中的某个Java对象，而`Stream`输出的元素可能并没有预先存储在内存中，而是实时计算出来的。`List`的用途是操作一组已存在的Java对象，而`Stream`实现的是惰性计算，真正的计算通常发生在最后结果的获取，也就是惰性计算。一个`Stream`转换为另一个`Stream`时，实际上只存储了转换规则，并没有任何计算发生。它可以“存储”有限个或无限个元素。这里的存储打了个引号，是因为元素有可能已经全部存储在内存中，也有可能是根据需要实时计算出来的。

| java.util.List | java.util.stream         |                      |
| :------------- | :----------------------- | -------------------- |
| 元素           | 已分配并存储在内存       | 可能未分配，实时计算 |
| 用途           | 操作一组已存在的Java对象 | 惰性计算             |

```java
Stream<BigInteger> naturals = createNaturalStream(); // 不计算
Stream<BigInteger> s2 = naturals.map(BigInteger::multiply); // 不计算
Stream<BigInteger> s3 = s2.limit(100); // 不计算
s3.forEach(System.out::println); // 计算
// Stream API支持函数式编程和链式操作；
createNaturalStream().map(BigInteger::multiply).limit(100).forEach(System.out::println);
```

3，创建Stream

* Stream.of()静态方法:传入可变参数即创建了一个能输出确定元素的`Stream`,

    ```java
    Stream<String> stream = Stream.of("A", "B", "C", "D");
    ```

* 基于一个数组或者`Collection`:

    ```java
    // 把数组变成Stream使用Arrays.stream()方法
    Stream<String> stream1 = Arrays.stream(new String[] { "A", "B", "C" });
    // 对于Collection（List、Set、Queue等），直接调用stream()方法
    Stream<String> stream2 = List.of("X", "Y", "Z").stream();
    ```

    Java的范型不支持基本类型,如果使用包装类会产生频繁的装箱、拆箱操作。为了提高效率，Java标准库提供了`IntStream`、`LongStream`和`DoubleStream`这三种使用基本类型的`Stream`，它们的使用方法和范型`Stream`没有大的区别，设计这三个`Stream`的目的是提高运行效率：

    ```java
    // 将int[]数组变为IntStream:
    IntStream is = Arrays.stream(new int[] { 1, 2, 3 });
    // 将Stream<String>转换为LongStream:
    LongStream ls = List.of("1", "2", "3").stream().mapToLong(Long::parseLong);
    ```

* Supplier:基于`Supplier`创建的`Stream`会不断调用`Supplier.get()`方法来不断产生下一个元素，这种`Stream`保存的不是元素，而是算法，它可以用来表示无限序列，`Stream`几乎不占用空间，因为每个元素都是实时计算出来的，用的时候再算。

    ```java
    Stream<String> s = Stream.generate(Supplier<String> sp);
    // sp要实现Supplier<T>接口，同时提供LongSuppliers等对基本数据内心的支持。
    class NatualSupplier implements Supplier<Integer> {
        int n = 0;
        public Integer get() {
            n++;
            return n;
        }
    }
    ```

* `Files`类：`Files`类的`lines()`方法可以把一个文件变成一个`Stream`，每个元素代表文件的一行内容

    ```java
    try (Stream<String> lines = Files.lines(Paths.get("/path/to/file.txt"))) {
        ...
    }
    ```

* 正则，`plitAsStream()`方法把一个长字符串分割成`Stream`序列而不是数组

    ```java
    Pattern p = Pattern.compile("\\s+");
    Stream<String> s = p.splitAsStream("The quick brown fox jumps over the lazy dog");
    s.forEach(System.out::println);
    ```

### Map

1，通过吧一个Steam中的每个元素通过某种运算得到结果，把结果加入另一个Stream，实现把一个`Stream`转换为另一个`Stream`。`map()`方法接收的对象是`Function`接口对象，它定义了一个`apply()`方法，负责把一个`T`类型转换成`R`类型，可以传入现有方法，或者是Lambda表达式：

```java
<R> Stream<R> map(Function<? super T, ? extends R> mapper); //T只读，R只写
// Function定义
@FunctionalInterface
public interface Function<T, R> {
    // 将T类型转换为R:
    R apply(T t);
}
```

### Filter

1，`filter()`操作，就是对一个`Stream`的所有元素一一进行测试，不满足条件的就被“滤掉”了，剩下的满足条件的元素就构成了一个新的`Stream`。

2，`filter()`方法接收的对象是`Predicate`接口对象，它定义了一个`test()`方法，负责判断元素是否符合条件。传入lambda表达式或者满足满足该方法签名的方法或者该接口的实现类。

```java
@FunctionalInterface
public interface Predicate<T> {
    // 判断元素t是否符合条件:
    boolean test(T t);
}
```

### Reduce

1,`Stream.reduce()`则是`Stream`的一个聚合方法，它可以把一个`Stream`的所有元素按照聚合函数聚合成一个结果。

2,`reduce()`方法传入的对象是`BinaryOperator`接口，它定义了一个`apply()`方法，负责把上次累加的结果和本次的元素 进行运算，并返回累加的结果，`reduce()`方法将一个`Stream`的每个元素依次作用于`BinaryOperator`，并将结果合并。`reduce()`是聚合方法，聚合方法会立刻对`Stream`进行计算。流的特点是每个元素只遍历一次，聚合操作无非就是一个遍历+累加计算的过程，所以使用reduce后流就已经关闭，无法再次遍历流。

```java
@FunctionalInterface
public interface BinaryOperator<T> {
    // Bi操作：两个输入，一个输出
    T apply(T t, T u);
}
// initValue为初始值，初始acc=initValue;n为Stream中元素，acc=acc*n;然后再次调用lambda表达式不断处理Stream中元素，最后返回acc
reduce(initValue, (acc, n) -> acc * n);
// reduce 也可以操作复杂Java对象，其中原始stream 为Map构成的Stream。
reduce(new HashMap<String, String>(), (m, kv) -> {m.putAll(kv);return m;})
```

### 导出Stream

1，对于`Stream`来说，对其进行转换操作（map()`，`filter()`，`sorted()`，`distinct()）并不会触发任何计算，转换操作只是保存了转换规则，无论我们对一个`Stream`转换多少次，都不会有任何实际计算发生。聚合操作（reduce()`，`collect()`，`count()`，`max()`，`min()`，`sum()`，`average())会立刻促使`Stream`输出它的每一个元素，并依次纳入计算，以获得最终结果。所以，对一个`Stream`进行聚合操作，会触发一系列连锁反应。聚合方法会立刻对`Stream`进行计算。流的特点是每个元素只遍历一次，聚合操作无非就是一个遍历+累加计算的过程，所以使用聚合操作后流就已经关闭，无法再次遍历流。

2，把`Stream`变为`List`/`Array`不是一个转换操作，而是一个聚合操作，它会强制`Stream`输出每个元素。

```java
// 调用collect()并传入Collectors.toList()对象，它实际上是一个Collector实例，通过类似reduce()的操作，把每个元素添加到一个收集器中（实际上是ArrayList）。
collect(Collectors.toList());
// 把Stream的每个元素收集到Set中
collect(Collectors.toSet());
// 把Stream的每个元素收集到数组中，要传入数组的“构造方法
toArray(String[]::new);
```

3，输出为Map，方法接受两个方法参数，第一个返回key，第二个返回value。

```jAVA
collect(Collectors.toMap(
                        // 把元素s映射为key:
                        s -> s.substring(0, s.indexOf(':')),
                        // 把元素s映射为value:
                        s -> s.substring(s.indexOf(':') + 1)));
```

4，分组输出,需要提供两个函数：一个是返回单个元素用于分组的key，第二个是如何处理聚集到一起的多个元素，可以再次对他们分组。

```java
Map<String, List<String>> groups =collect(Collectors.groupingBy(s -> s.substring(0, 1), Collectors.toList()));
Map<Integer, Map<Integer, List<Student>>> groups = studentStream.collect( Collectors.groupingBy(s -> s.gradeId, Collectors.groupingBy(s -> s.classId, Collectors.toList())));
```

### 常用方法

1，

* sorted()：排序，要求`Stream`的每个元素必须实现`Comparable`接口。如果要自定义排序，传入指定的`Comparator`即可。转换操作，它会返回一个新的`Stream`

    ```java
    sort() //每个元素必须实现`Comparable`接口
    sorted(String::compareToIgnoreCase) // 传入指定的Comparator
    ```

* distinct()：去重，正确实现equals，或者传入只定义比较器。截取操作也是一个转换操作，将返回新的`Stream`

* concat()：合并两个流，`Stream.concat(s1, s2);`

* flatMap()：把`Stream`的每个元素映射为`Stream`，然后合并成一个新的`Stream`

    ```java
    Stream<Integer> i = Stream.of(
                                    Arrays.asList(1, 2, 3),
                                    Arrays.asList(4, 5, 6),
                                    Arrays.asList(7, 8, 9)
                                   ).flatMap(list -> list.stream());
    ```

* parallel()：一个普通`Stream`转换为可以并行处理的`Stream`，通常情况下，对`Stream`的元素进行处理是单线程的，即一个一个元素进行处理。但是很多时候，我们希望可以并行处理`Stream`的元素，因为在元素数量非常大的情况，并行处理可以大大加快处理速度。经过`parallel()`转换后的`Stream`只要可能，就会对后续操作进行并行处理。我们不需要编写任何多线程代码就可以享受到并行处理带来的执行效率的提升。注意对线程不安全（如hashMap)的集合的操作的并发安全性问题。

* forEach()：相当于内部循环调用，它可以循环处理`Stream`的每个元素，可传入符合Consumer接口的``void accept(T t)``的方法引用，如`stream.forEach(System.out::println);`

* limit(n)：截断

* count()：计数

* max(Comparator<? super T> cp)：找出最大元素；

* min(Comparator<? super T> cp)：找出最小元素。

* sum()：对所有元素求和；

* average()：对所有元素求平均数

* boolean allMatch(Predicate<? super T>)：测试是否所有元素均满足测试条件；

* boolean anyMatch(Predicate<? super T>)：测试是否至少有一个元素满足测试条件。

* summaryStatistics()一次性终端操作获得{`count`, `sum`, `min` ,`average`, `max`}等统计数据。

2，分类

转换操作：`map()`，`filter()`，`sorted()`，`distinct()`；

合并操作：`concat()`，`flatMap()`；

并行处理：`parallel()`；

聚合操作：`reduce()`，`collect()`，`count()`，`max()`，`min()`，`sum()`，`average()`；

其他操作：`allMatch()`, `anyMatch()`, `forEach()`。

### 其他

可被传入当作满足函数接口要求的参数可分为四类

* 接口的实现类

* 接口的匿名实现类

* 静态方法

* 实例方法，注意this参数

* Lambda表达式

    