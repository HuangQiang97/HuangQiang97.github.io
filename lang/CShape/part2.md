[toc]

### 25、异常

>C# 异常处理时建立在四个关键词之上的：**try**、**catch**、**finally** 和 **throw**。
>
>- **try**：一个 try 块标识了一个将被激活的特定的异常的代码块。后跟一个或多个 catch 块。
>
>- **catch**：程序通过异常处理程序捕获异常。catch 关键字表示异常的捕获。
>
>- **finally**：finally 块用于执行给定的语句，不管异常是否被抛出都会执行。
>
>- **throw**：当问题出现时，程序抛出一个异常。使用 throw 关键字来完成。
>
>以列出多个 catch 语句捕获不同类型的异常，以防 try 块在不同的情况下生成多个异常。
>
>```
>namespace ErrorHandlingApplication
>
>​```
>
>try
>{
> // 引起异常的语句
>}
>catch( ExceptionName e1 )
>{
> // 错误处理代码
> Console.WriteLine(e1.Message);
> throw (exceptionObj);//如果异常是直接或间接派生自 System.Exception 类,可以抛出一个对象。
>}
>catch( ExceptionName eN )
>{
> // 错误处理代码
> Console.WriteLine(eN.Message);
>  throw (exceptionObj);//如果异常是直接或间接派生自 System.Exception 类,可以抛出一个对象。
>}
>finally
>{
> // 要执行的语句
>}
>```
>
>

>C# 异常是使用类来表示的。C# 中的异常类主要是直接或间接地派生于 **System.Exception** 类。
>
>**System.ApplicationException** 和 **System.SystemException** 类是派生于 System.Exception 类的异常类。
>
>**System.ApplicationException** 类支持由应用程序生成的异常。所以自定义的异常都应派生自该类。
>
>**System.SystemException** 类是所有预定义的系统异常的基类。

**常见异常**

| 异常类                            | 描述                                           |
| :-------------------------------- | :--------------------------------------------- |
| System.IO.IOException             | 处理 I/O 错误。                                |
| System.IndexOutOfRangeException   | 处理当方法指向超出范围的数组索引时生成的错误。 |
| System.ArrayTypeMismatchException | 处理当数组类型不匹配时生成的错误。             |
| System.NullReferenceException     | 处理当依从一个空对象时生成的错误。             |
| System.DivideByZeroException      | 处理当除以零时生成的错误。                     |
| System.InvalidCastException       | 处理在类型转换期间生成的错误。                 |
| System.OutOfMemoryException       | 处理空闲内存不足生成的错误。                   |
| System.StackOverflowException     | 处理栈溢出生成的错误。                         |

> 以定义自己的异常。用户自定义的异常类是派生自 **ApplicationException** 类。
>
> ```
> public class MyException: ApplicationException
> {
> 	public TMyException(string message): base(message)
>         {
>             ops;
>         }
> }
> ```
>
> 

### 26、文件IO

>一个 **文件** 是一个存储在磁盘中带有指定名称和目录路径的数据集合。当打开文件进行读写时，它变成一个 **流**。从根本上说，流是通过通信路径传递的字节序列。

| I/O 类         | 描述                               |
| :------------- | :--------------------------------- |
| BinaryReader   | 从二进制流读取原始数据。           |
| BinaryWriter   | 以二进制格式写入原始数据。         |
| BufferedStream | 字节流的临时存储。                 |
| Directory      | 有助于操作目录结构。               |
| DirectoryInfo  | 用于对目录执行操作。               |
| DriveInfo      | 提供驱动器的信息。                 |
| File           | 有助于处理文件。                   |
| FileInfo       | 用于对文件执行操作。               |
| FileStream     | 用于文件中任何位置的读写。         |
| MemoryStream   | 用于随机访问存储在内存中的数据流。 |
| Path           | 对路径信息执行操作。               |
| StreamReader   | 用于从字节流中读取字符。           |
| StreamWriter   | 用于向一个流中写入字符。           |
| StringReader   | 用于读取字符串缓冲区。             |
| StringWriter   | 用于写入字符串缓冲区。             |

> System.IO 命名空间中的 **FileStream** 类有助于文件的读写与关闭。该类派生自抽象类 Stream。
>
> 创建一个 **FileStream** 对象来创建一个新的文件，或打开一个已有的文件。
>
> ```
> using System.IO;
> ​```
> FileStream <object_name> = new FileStream( <file_name>,<FileMode Enumerator>, <FileAccess Enumerator>, <FileShare Enumerator>);
> 例如：FileStream F = new FileStream("sample.txt", FileMode.Open, FileAccess.Read, FileShare.Read);
> ```

| 参数       | 描述                                                         |
| :--------- | :----------------------------------------------------------- |
| FileMode   | **FileMode** 枚举定义了各种打开文件的方法。FileMode 枚举的成员有：                                                                                                                                                     **Append**：打开一个已有的文件，并将光标放置在文件的末尾。如果文件不存在，则创建文件。                                                                                                                                      **Create**：创建一个新的文件。如果文件已存在，则删除旧文件，然后创建新文件。                                                                                                                                            **CreateNew**：指定操作系统应创建一个新的文件。如果文件已存在，则抛出异常。                                                                                                                           **Open**：打开一个已有的文件。如果文件不存在，则抛出异常。                                                                                                                                                                             **OpenOrCreate**：指定操作系统应打开一个已有的文件。如果文件不存在，则用指定的名称创建一个新的文件打开。                                                                                                                                                    **Truncate**：打开一个已有的文件，文件一旦打开，就将被截断为零字节大小。然后我们可以向文件写入全新的数据，但是保留文件的初始创建日期。如果文件不存在，则抛出异常。 |
| FileAccess | **FileAccess** 枚举的成员有：**Read**、**ReadWrite** 和 **Write**。 |
| FileShare  | **FileShare** 枚举的成员有：                                                                                                                              **Inheritable**：允许文件句柄可由子进程继承。Win32 不直接支持此功能。                                     **None**：谢绝共享当前文件。文件关闭前，打开该文件的任何请求（由此进程或另一进程发出的请求）都将失败。                                                                                                                                                            **Read**：允许随后打开文件读取。如果未指定此标志，则文件关闭前，任何打开该文件以进行读取的请求（由此进程或另一进程发出的请求）都将失败。但是，即使指定了此标志，仍可能需要附加权限才能够访问该文件。                                                                                                                                   **ReadWrite**：允许随后打开文件读取或写入。如果未指定此标志，则文件关闭前，任何打开该文件以进行读取或写入的请求（由此进程或另一进程发出）都将失败。但是，即使指定了此标志，仍可能需要附加权限才能够访问该文件。                                                                                                         **Write**：允许随后打开文件写入。如果未指定此标志，则文件关闭前，任何打开该文件以进行写入的请求（由此进程或另一进过程发出的请求）都将失败。但是，即使指定了此标志，仍可能需要附加权限才能够访问该文件。                                                                                                                            **Delete**：允许随后删除文件。 |

>| 主题                                                         | 描述                                                         |
>| :----------------------------------------------------------- | :----------------------------------------------------------- |
>| [文本文件的读写](https://www.runoob.com/csharp/csharp-text-files.html) | 它涉及到文本文件的读写。**StreamReader** 和 **StreamWriter** 类有助于完成文本文件的读写。 |
>| [二进制文件的读写](https://www.runoob.com/csharp/csharp-binary-files.html) | 它涉及到二进制文件的读写。**BinaryReader** 和 **BinaryWriter** 类有助于完成二进制文件的读写。 |
>| [Windows 文件系统的操作](https://www.runoob.com/csharp/csharp-windows-file-system.html) | 它让 C# 程序员能够浏览并定位 Windows 文件和目录。            |

**读写文本**

> **StreamReader** 和 **StreamWriter** 类用于文本文件的数据读写。这些类从抽象基类 Stream 继承，Stream 支持文件流的字节读写。
>
> --------
>
> 
>
> **StreamReader** 类继承自抽象基类 TextReader，表示阅读器读取一系列字符。
>
> | 序号 | 方法 & 描述                                                  |
> | :--- | :----------------------------------------------------------- |
> | 1    | **public override void Close()** 关闭 StreamReader 对象和基础流，并释放任何与读者相关的系统资源。 |
> | 2    | **public override int Peek()** 返回下一个可用的字符，但不使用它。 |
> | 3    | **public override int Read()** 从输入流中读取下一个字符，并把字符位置往前移一个字符。 |
> | 4    | **public override string ReadLine()** 从输入流中读取一行字符，并把字符位置移到下一行开头。 |
>
> ```
> using (StreamReader sr = new StreamReader(filePath))
> {
>  //读
>  while ((line = sr.ReadLine()) != null)
>  {
>  	Console.WriteLine(line);
>  }      
> }
> 这里using与对象绑定，类似于python的with，不用手动关闭文件流。
> ```
>
> ---
>
> **StreamWriter** 类继承自抽象类 TextWriter，表示编写器写入一系列字符。
>
> | 序号 | 方法 & 描述                                                  |
> | :--- | :----------------------------------------------------------- |
> | 1    | **public override void Close()** 关闭当前的 StreamWriter 对象和基础流。 |
> | 2    | **public override void Flush()** 清理当前编写器的所有缓冲区，使得所有缓冲数据写入基础流。 |
> | 3    | **public virtual void Write(bool value)** 把一个布尔值的文本表示形式写入到文本字符串或流。（继承自 TextWriter。） |
> | 4    | **public override void Write( char value )** 把一个字符写入到流。 |
> | 5    | **public virtual void Write( decimal value )** 把一个十进制值的文本表示形式写入到文本字符串或流。 |
> | 6    | **public virtual void Write( double value )** 把一个 8 字节浮点值的文本表示形式写入到文本字符串或流。 |
> | 7    | **public virtual void Write( int value )** 把一个 4 字节有符号整数的文本表示形式写入到文本字符串或流。 |
> | 8    | **public override void Write( string value )** 把一个字符串写入到流。 |
> | 9    | **public virtual void WriteLine(string value)** 把字符串写入到文本字符串或流。 |
>
> ```
> using (StreamWriter sw = new StreamWriter(filePath))
> {
> 		ops;          
> }
> 这里using与对象绑定，类似于python的with，不用手动关闭文件流。
> ```
>
> 

**读写二进制数据**

> **BinaryReader** 和 **BinaryWriter** 类用于二进制文件的读写。
>
> ---
>
> **BinaryReader** 类用于从文件读取二进制数据。一个 **BinaryReader** 对象通过向它的构造函数传递 **FileStream** 对象而被创建。
>
> | 序号 | 方法 & 描述                                                  |
> | :--- | :----------------------------------------------------------- |
> | 1    | **public override void Close()** 关闭 BinaryReader 对象和基础流。 |
> | 2    | **public virtual int Read()** 从基础流中读取字符，并把流的当前位置往前移。 |
> | 3    | **public virtual bool ReadBoolean()** 从当前流中读取一个布尔值，并把流的当前位置往前移一个字节。 |
> | 4    | **public virtual byte ReadByte()** 从当前流中读取下一个字节，并把流的当前位置往前移一个字节。 |
> | 5    | **public virtual byte[] ReadBytes( int count )** 从当前流中读取指定数目的字节到一个字节数组中，并把流的当前位置往前移指定数目的字节。 |
> | 6    | **public virtual char ReadChar()** 从当前流中读取下一个字节，并把流的当前位置按照所使用的编码和从流中读取的指定的字符往前移。 |
> | 7    | **public virtual char[] ReadChars( int count )** 从当前流中读取指定数目的字节，在一个字符数组中返回数组，并把流的当前位置按照所使用的编码和从流中读取的指定的字符往前移。 |
> | 8    | **public virtual double ReadDouble()** 从当前流中读取一个 8 字节浮点值，并把流的当前位置往前移八个字节。 |
> | 9    | **public virtual int ReadInt32()** 从当前流中读取一个 4 字节有符号整数，并把流的当前位置往前移四个字节。 |
> | 10   | **public virtual string ReadString()** 从当前流中读取一个字符串。字符串以长度作为前缀，同时编码为一个七位的整数。 |
>
> ```
> BinaryReader br =new BinaryReader(new FileStream("mydata",FileMode.Open));
> 
> ```
>
> 
>
> ---
>
> **BinaryWriter** 类用于向文件写入二进制数据。一个 **BinaryWriter** 对象通过向它的构造函数传递 **FileStream** 对象而被创建。
>
> | 序号 | 方法 & 描述                                                  |
> | :--- | :----------------------------------------------------------- |
> | 1    | **public override void Close()** 关闭 BinaryWriter 对象和基础流。 |
> | 2    | **public virtual void Flush()** 清理当前编写器的所有缓冲区，使得所有缓冲数据写入基础设备。 |
> | 3    | **public virtual long Seek( int offset, SeekOrigin origin )** 设置当前流内的位置。 |
> | 4    | **public virtual void Write( bool value )** 把一个单字节的布尔值写入到当前流中，0 表示 false，1 表示 true。 |
> | 5    | **public virtual void Write( byte value )** 把一个无符号字节写入到当前流中，并把流的位置往前移一个字节。 |
> | 6    | **public virtual void Write( byte[] buffer )** 把一个字节数组写入到基础流中。 |
> | 7    | **public virtual void Write( char ch )** 把一个 Unicode 字符写入到当前流中，并把流的当前位置按照所使用的编码和要写入到流中的指定的字符往前移。 |
> | 8    | **public virtual void Write( char[] chars )** 把一个字符数组写入到当前流中，并把流的当前位置按照所使用的编码和要写入到流中的指定的字符往前移。 |
> | 9    | **public virtual void Write( double value )** 把一个 8 字节浮点值写入到当前流中，并把流位置往前移八个字节。 |
> | 10   | **public virtual void Write( int value )** 把一个 4 字节有符号整数写入到当前流中，并把流位置往前移四个字节。 |
> | 11   | **public virtual void Write( string value )** 把一个以长度为前缀的字符串写入到 BinaryWriter 的当前编码的流中，并把流的当前位置按照所使用的编码和要写入到流中的指定的字符往前移。 |
>
> ```
> BinaryWriter bw = new BinaryWriter(new FileStream("mydata",FileMode.Create));
> //读
> while (br.PeekChar() != -1)
>  {
>      line = br.ReadString();
>  }
> ```
>
> 

**文件系统操作**

>**DirectoryInfo** 类派生自 **FileSystemInfo** 类。它提供了各种用于创建、移动、浏览目录和子目录的方法。该类不能被继承。
>
>| 序号 | 属性 & 描述                                             |
>| :--- | :------------------------------------------------------ |
>| 1    | **Attributes** 获取当前文件或目录的属性。               |
>| 2    | **CreationTime** 获取当前文件或目录的创建时间。         |
>| 3    | **Exists** 获取一个表示目录是否存在的布尔值。           |
>| 4    | **Extension** 获取表示文件存在的字符串。                |
>| 5    | **FullName** 获取目录或文件的完整路径。                 |
>| 6    | **LastAccessTime** 获取当前文件或目录最后被访问的时间。 |
>| 7    | **Name** 获取该 DirectoryInfo 实例的名称。              |
>
>| 序号 | 方法 & 描述                                                  |
>| :--- | :----------------------------------------------------------- |
>| 1    | **public void Create()** 创建一个目录。                      |
>| 2    | **public DirectoryInfo CreateSubdirectory( string path )** 在指定的路径上创建子目录。指定的路径可以是相对于 DirectoryInfo 类的实例的路径。 |
>| 3    | **public override void Delete()** 如果为空的，则删除该 DirectoryInfo。 |
>| 4    | **public DirectoryInfo[] GetDirectories()** 返回当前目录的子目录。 |
>| 5    | **public FileInfo[] GetFiles()** 从当前目录返回文件列表。    |
>
>```
>DirectoryInfo mydir = new DirectoryInfo(@"path");
>```
>
>
>
>---
>
>**FileInfo** 类派生自 **FileSystemInfo** 类。它提供了用于创建、复制、删除、移动、打开文件的属性和方法，且有助于 FileStream 对象的创建。该类不能被继承。
>
>| 序号 | 属性 & 描述                                       |
>| :--- | :------------------------------------------------ |
>| 1    | **Attributes** 获取当前文件的属性。               |
>| 2    | **CreationTime** 获取当前文件的创建时间。         |
>| 3    | **Directory** 获取文件所属目录的一个实例。        |
>| 4    | **Exists** 获取一个表示文件是否存在的布尔值。     |
>| 5    | **Extension** 获取表示文件存在的字符串。          |
>| 6    | **FullName** 获取文件的完整路径。                 |
>| 7    | **LastAccessTime** 获取当前文件最后被访问的时间。 |
>| 8    | **LastWriteTime** 获取文件最后被写入的时间。      |
>| 9    | **Length** 获取当前文件的大小，以字节为单位。     |
>| 10   | **Name** 获取文件的名称。                         |
>
>| 序号 | 方法 & 描述                                                  |
>| :--- | :----------------------------------------------------------- |
>| 1    | **public StreamWriter AppendText()** 创建一个 StreamWriter，追加文本到由 FileInfo 的实例表示的文件中。 |
>| 2    | **public FileStream Create()** 创建一个文件。                |
>| 3    | **public override void Delete()** 永久删除一个文件。         |
>| 4    | **public void MoveTo( string destFileName )** 移动一个指定的文件到一个新的位置，提供选项来指定新的文件名。 |
>| 5    | **public FileStream Open( FileMode mode )** 以指定的模式打开一个文件。 |
>| 6    | **public FileStream Open( FileMode mode, FileAccess access )** 以指定的模式，使用 read、write 或 read/write 访问，来打开一个文件。 |
>| 7    | **public FileStream Open( FileMode mode, FileAccess access, FileShare share )** 以指定的模式，使用 read、write 或 read/write 访问，以及指定的分享选项，来打开一个文件。 |
>| 8    | **public FileStream OpenRead()** 创建一个只读的 FileStream。 |
>| 9    | **public FileStream OpenWrite()** 创建一个只写的 FileStream。 |

**路径操作**

>
>
>```
>string dirPath = @"D:\TestDir";
>string filePath = @"D:\TestDir\TestFile.txt";
>Console.WriteLine("<<<<<<<<<<<{0}>>>>>>>>>>", "文件路径");
>//获得当前路径
>Console.WriteLine(Environment.CurrentDirectory);
>//文件或文件夹所在目录
>Console.WriteLine(Path.GetDirectoryName(filePath));     //D:\TestDir
>Console.WriteLine(Path.GetDirectoryName(dirPath));      //D:\
>//文件扩展名
>Console.WriteLine(Path.GetExtension(filePath));         //.txt
>//文件名
>Console.WriteLine(Path.GetFileName(filePath));          //TestFile.txt
>Console.WriteLine(Path.GetFileName(dirPath));           //TestDir
>Console.WriteLine(Path.GetFileNameWithoutExtension(filePath)); //TestFile
>//绝对路径
>Console.WriteLine(Path.GetFullPath(filePath));          //D:\TestDir\TestFile.txt
>Console.WriteLine(Path.GetFullPath(dirPath));           //D:\TestDir  
>//更改扩展名
>Console.WriteLine(Path.ChangeExtension(filePath, ".jpg"));//D:\TestDir\TestFile.jpg
>//根目录
>Console.WriteLine(Path.GetPathRoot(dirPath));           //D:\      
>//生成路径
>Console.WriteLine(Path.Combine(new string[] { @"D:\", "BaseDir", "SubDir", "TestFile.txt" })); //D:\BaseDir\SubDir\TestFile.txt
>//生成随即文件夹名或文件名
>Console.WriteLine(Path.GetRandomFileName());
>//创建磁盘上唯一命名的零字节的临时文件并返回该文件的完整路径
>Console.WriteLine(Path.GetTempFileName());
>//返回当前系统的临时文件夹的路径
>Console.WriteLine(Path.GetTempPath());
>//文件名中无效字符
>Console.WriteLine(Path.GetInvalidFileNameChars());
>//路径中无效字符
>Console.WriteLine(Path.GetInvalidPathChars()); 
>```
>
>

**文件属性操作**

>
>
>```
>//use File class
>Console.WriteLine(File.GetAttributes(filePath));
>File.SetAttributes(filePath,FileAttributes.Hidden | FileAttributes.ReadOnly);
>Console.WriteLine(File.GetAttributes(filePath));
>
>//user FilInfo class
>FileInfo fi = new FileInfo(filePath);
>Console.WriteLine(fi.Attributes.ToString());
>fi.Attributes = FileAttributes.Hidden | FileAttributes.ReadOnly; //隐藏与只读
>Console.WriteLine(fi.Attributes.ToString());
>
>//只读与系统属性，删除时会提示拒绝访问
>fi.Attributes = FileAttributes.Archive;
>Console.WriteLine(fi.Attributes.ToString());
>```
>
>

### 27、访问器

> 使用 **访问器（accessors）** 让私有域的值可被读写或操作。
>
> 属性（Property）的**访问器（accessor）**包含有助于获取（读取或计算）或设置（写入）属性的可执行语句。访问器（accessor）声明可包含一个 get 访问器、一个 set 访问器，或者同时包含二者。只有get为只读属性，只有set为只写属性，加入访问器可以对参数作预处理，防止错误的参数传入。

```
 private int age = -1;
 public int Age
 {
     get
     {
     	return age;
     }
     set
     {
    	 age = value>=0? value:0;
     }
 }
```

### 28、索引器

> **索引器（Indexer）** 允许一个对象可以像数组一样被索引。当您为类定义一个索引器时，该类的行为就会像一个 **虚拟数组（virtual array）** 一样。您可以使用数组访问运算符（[ ]）来访问该类的实例。
>
> 把实例数据分为更小的部分，并索引每个部分，获取或设置每个部分。
>
> 定义一个属性（property）包括提供属性名称。索引器定义的时候不带有名称，但带有 **this** 关键字，它指向对象实例。
>
> 索引器（Indexer）可被重载。索引器声明的时候也可带有多个参数，且每个参数可以是不同的类型。没有必要让索引器必须是整型的。C# 允许索引器可以是其他类型，例如，字符串类型。

```
private string[] namelist = new string[3]{"str","str","str"};

public string this[int index]
{
    get
    {
        if( index >= 0 && index <= size-1 )
        {
            return( namelist[index]);
        }
        return ( "");
    }
    set
    {
        if( index >= 0 && index <3 )
        {
       	 	namelist[index] = value;
        }
    }
}
public int this[string name]
{
    get
    {
        int index = 0;
        while(index < 3)
        {
            if (namelist[index] == name)
            {
            	return index;
            }
        index++;
        }
        return -1;
    }

}

​```

obj[index]="string";
Console.WriteLine(obj[index]);
```

### 29、委托

>**委托（Delegate）** 是存有对某个方法的引用的一种引用类型变量。引用可在运行时被改变。
>
>委托（Delegate）特别用于实现事件和回调方法。所有的委托（Delegate）都派生自 **System.Delegate** 类。

> 委托声明决定了可由该委托引用的方法。委托可指向一个与其具有相同返回值、参数列表的方法。delegate 可以视为一个类，只有在可以定义类的地方才能定义委托。
>
> ```
> delegate <return type> <delegate-name> (<parameter list>)
> ```
>
> 一旦声明了委托类型，委托对象必须使用 **new** 关键字来创建，且与一个特定的方法有关。当创建委托时，传递到 **new** 语句的参数就像方法调用一样书写，但是不带有参数。
>
> ```
> delegate-name delegateName=new delegate-name(func);
> ```
>
> 可以像原函数一样调用委托。

>委托对象可使用 "+" 运算符进行合并。一个合并委托调用它所合并的两个委托。只有相同类型的委托可被合并。"-" 运算符可用于从合并的委托中移除组件委托。可以创建一个委托被调用时要调用的方法的调用列表。这被称为委托的 **多播（multicasting）**，也叫组播。
>
>```
>delegate-name delegateName1=new delegate-name(func1);
>delegate-name delegateName2=new delegate-name(func2);
>delegate-name delegateName3=delegateName1+delegateName2;
>delegateName3(parameter)//等价于先调用 func1(parameter)再func2(parameter)
>```

### 30、事件

> **事件（Event）** 基本上说是一个用户操作，如按键、点击、鼠标移动等等，或者是一些提示信息，如系统生成的通知。应用程序需要在事件发生时响应事件。
>
> 事件在类中声明且生成，且通过使用同一个类或其他类中的委托与事件处理程序关联。包含事件的类用于发布事件。这被称为 **发布器（publisher）** 类。其他接受该事件的类被称为 **订阅器（subscriber）** 类。事件使用 **发布-订阅（publisher-subscriber）** 模型。
>
> **发布器（publisher）** 是一个包含事件和委托定义的对象。事件和委托之间的联系也定义在这个对象中。发布器（publisher）类的对象调用这个事件，并通知其他的对象。
>
> **订阅器（subscriber）** 是一个接受事件并提供事件处理程序的对象。在发布器（publisher）类中的委托调用订阅器（subscriber）类中的方法（事件处理程序）。

>
>
>```
>public delegate return_type BoilerLogHandler(params);
>public event BoilerLogHandler BoilerEventLog;
>```
>
>```
>using System;
>/***********发布器类***********/
>public class EventTest
>{
>private int value;
>	// 委托声明。
>public delegate void NumManipulationHandler();
>
>	// 事件声明，可将ChangeNum视为NumManipulationHandler类型的一个变量,此时ChangeNum为null。
>public event NumManipulationHandler ChangeNum;
>protected virtual void OnNumChanged()
>{
> if ( ChangeNum != null )
> { //执行与事件相关联的委托。
>   ChangeNum(); 
> }else {
>   // 事件未关联委托，无法执行。
>   Console.WriteLine( "event not fire" );
> }
>}
>
>public EventTest()
>{
> int n = 5;
> SetValue( n );
>}
>
>
>public void SetValue( int n )
>{
> if ( value != n )
> {
>   value = n;
>   OnNumChanged();
> }
>}
>}
>
>
>/***********订阅器类***********/
>
>public class subscribEvent
>{
>//将于事件相关联。负责具体处理。
>public void printf()
>{
> Console.WriteLine( "event fire" );
>}
>}
>
>/***********触发***********/
>public class MainClass
>{
>public static void Main()
>{
>// 实例化对象,第一次ChangeNum为null没有触发事件
> EventTest e = new EventTest(); 
> //  实例化对象
> subscribEvent v = new subscribEvent();
> //将事件与具体委托关联，ChangeNum不为null.
> e.ChangeNum += new EventTest.NumManipulationHandler( v.printf ); 
> // ChangeNum不为null，将触发事件。
> e.SetValue( 7 );
> e.SetValue( 11 );
>}
>}
>/////////////////////////////////////////////////////
>event not fire
>event fire
>event fire
>```
>
>

### 31、集合

> 集合（Collection）类是专门用于数据存储和检索的类。这些类提供了对栈（stack）、队列（queue）、列表（list）和哈希表（hash table）的支持。大多数集合类实现了相同的接口。

| 类                                                           | 描述和用法                                                   |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [动态数组（ArrayList）](https://www.runoob.com/csharp/csharp-arraylist.html) | 它代表了可被单独**索引**的对象的有序集合。它基本上可以替代一个数组。但是，与数组不同的是，您可以使用**索引**在指定的位置添加和移除项目，动态数组会自动重新调整它的大小。它也允许在列表中进行动态内存分配、增加、搜索、排序各项。 |
| [哈希表（Hashtable）](https://www.runoob.com/csharp/csharp-hashtable.html) | 它使用**键**来访问集合中的元素。当您使用键访问元素时，则使用哈希表，而且您可以识别一个有用的键值。哈希表中的每一项都有一个**键/值**对。键用于访问集合中的项目。 |
| [排序列表（SortedList）](https://www.runoob.com/csharp/csharp-sortedlist.html) | 它可以使用**键**和**索引**来访问列表中的项。排序列表是数组和哈希表的组合。它包含一个可使用键或索引访问各项的列表。如果您使用索引访问各项，则它是一个动态数组（ArrayList），如果您使用键访问各项，则它是一个哈希表（Hashtable）。集合中的各项总是按键值排序。 |
| [堆栈（Stack）](https://www.runoob.com/csharp/csharp-stack.html) | 它代表了一个**后进先出**的对象集合。当您需要对各项进行后进先出的访问时，则使用堆栈。当您在列表中添加一项，称为**推入**元素，当您从列表中移除一项时，称为**弹出**元素。 |
| [队列（Queue）](https://www.runoob.com/csharp/csharp-queue.html) | 它代表了一个**先进先出**的对象集合。当您需要对各项进行先进先出的访问时，则使用队列。当您在列表中添加一项，称为**入队**，当您从列表中移除一项时，称为**出队**。 |
| [点阵列（BitArray）](https://www.runoob.com/csharp/csharp-bitarray.html) | 它代表了一个使用值 1 和 0 来表示的**二进制**数组。当您需要存储位，但是事先不知道位数时，则使用点阵列。您可以使用**整型索引**从点阵列集合中访问各项，索引从零开始。 |

**ArrayList**

下表列出了 **ArrayList** 类的一些常用的 **属性**：

| 属性           | 描述                                                  |
| :------------- | :---------------------------------------------------- |
| Capacity       | 获取或设置 ArrayList 可以包含的元素个数。             |
| Count          | 获取 ArrayList 中实际包含的元素个数。                 |
| IsFixedSize    | 获取一个值，表示 ArrayList 是否具有固定大小。         |
| IsReadOnly     | 获取一个值，表示 ArrayList 是否只读。                 |
| IsSynchronized | 获取一个值，表示访问 ArrayList 是否同步（线程安全）。 |
| Item[Int32]    | 获取或设置指定索引处的元素。                          |
| SyncRoot       | 获取一个对象用于同步访问 ArrayList。                  |

下表列出了 **ArrayList** 类的一些常用的 **方法**：

| 序号 | 方法名 & 描述                                                |
| :--- | :----------------------------------------------------------- |
| 1    | **public virtual int Add( object value );** 在 ArrayList 的末尾添加一个对象。 |
| 2    | **public virtual void AddRange( ICollection c );** 在 ArrayList 的末尾添加 ICollection 的元素。 |
| 3    | **public virtual void Clear();** 从 ArrayList 中移除所有的元素。 |
| 4    | **public virtual bool Contains( object item );** 判断某个元素是否在 ArrayList 中。 |
| 5    | **public virtual ArrayList GetRange( int index, int count );** 返回一个 ArrayList，表示源 ArrayList 中元素的子集。 |
| 6    | **public virtual int IndexOf(object);** 返回某个值在 ArrayList 中第一次出现的索引，索引从零开始。 |
| 7    | **public virtual void Insert( int index, object value );** 在 ArrayList 的指定索引处，插入一个元素。 |
| 8    | **public virtual void InsertRange( int index, ICollection c );** 在 ArrayList 的指定索引处，插入某个集合的元素。 |
| 9    | **public virtual void Remove( object obj );** 从 ArrayList 中移除第一次出现的指定对象。 |
| 10   | **public virtual void RemoveAt( int index );** 移除 ArrayList 的指定索引处的元素。 |
| 11   | **public virtual void RemoveRange( int index, int count );** 从 ArrayList 中移除某个范围的元素。 |
| 12   | **public virtual void Reverse();** 逆转 ArrayList 中元素的顺序。 |
| 13   | **public virtual void SetRange( int index, ICollection c );** 复制某个集合的元素到 ArrayList 中某个范围的元素上。 |
| 14   | **public virtual void Sort();** 对 ArrayList 中的元素进行排序。 |
| 15   | **public virtual void TrimToSize();** 设置容量为 ArrayList 中元素的实际个数。 |

```
 // 定义元素排序依据
 class MyComparer : IComparer
    {
        public int Compare(object x, object y)
        {
            //return -1：x<y;0:x=y;1:x<y
            ops;
        }
    }
```

    list.Sort(new MyComparer());
    ```
     var arrayList = new ArrayList();
     arrayList.Add(0);
     arrayList.Add(100);
     arrayList.Remove(1);
     arrayList[0] = -1;

**哈希表**

>Hashtable 类代表了一系列基于键的哈希代码组织起来的**键/值**对。它使用**键**来访问集合中的元素。
>
>当使用**键**访问元素时，则使用哈希表，而且可以识别一个有用的键值。哈希表中的每一项都有一个**键/值**对。键用于访问集合中的项目。



| 属性        | 描述                                          |
| :---------- | :-------------------------------------------- |
| Count       | 获取 Hashtable 中包含的键值对个数。           |
| IsFixedSize | 获取一个值，表示 Hashtable 是否具有固定大小。 |
| IsReadOnly  | 获取一个值，表示 Hashtable 是否只读。         |
| Item        | 获取或设置与指定的键相关的值。                |
| Keys        | 获取一个 ICollection，包含 Hashtable 中的键。 |
| Values      | 获取一个 ICollection，包含 Hashtable 中的值。 |

| 序号 | 方法名 & 描述                                                |
| :--- | :----------------------------------------------------------- |
| 1    | **public virtual void Add( object key, object value );** 向 Hashtable 添加一个带有指定的键和值的元素。 |
| 2    | **public virtual void Clear();** 从 Hashtable 中移除所有的元素。 |
| 3    | **public virtual bool ContainsKey( object key );** 判断 Hashtable 是否包含指定的键。 |
| 4    | **public virtual bool ContainsValue( object value );** 判断 Hashtable 是否包含指定的值。 |
| 5    | **public virtual void Remove( object key );** 从 Hashtable 中移除带有指定的键的元素。 |

```
Hashtable hashtable=new Hashtable();
hashtable.Add(111,"111");
hashtable.Add(222,"222");
hashtable.Remove(222);
hashtable[111] = 111;
```

**排序列表**

>SortedList 类代表了一系列按照键来排序的**键/值**对，这些键值对可以通过键和索引来访问。
>
>排序列表是数组和哈希表的组合。它包含一个可使用键或索引访问各项的列表。如果您使用索引访问各项，则它是一个动态数组（ArrayList），如果您使用键访问各项，则它是一个哈希表（Hashtable）。集合中的各项总是按键值排序。

| 属性        | 描述                                           |
| :---------- | :--------------------------------------------- |
| Capacity    | 获取或设置 SortedList 的容量。                 |
| Count       | 获取 SortedList 中的元素个数。                 |
| IsFixedSize | 获取一个值，表示 SortedList 是否具有固定大小。 |
| IsReadOnly  | 获取一个值，表示 SortedList 是否只读。         |
| Item        | 获取或设置与 SortedList 中指定的键相关的值。   |
| Keys        | 获取 SortedList 中的键。                       |
| Values      | 获取 SortedList 中的值。                       |

| 序号 | 方法名 & 描述                                                |
| :--- | :----------------------------------------------------------- |
| 1    | **public virtual void Add( object key, object value );** 向 SortedList 添加一个带有指定的键和值的元素。 |
| 2    | **public virtual void Clear();** 从 SortedList 中移除所有的元素。 |
| 3    | **public virtual bool ContainsKey( object key );** 判断 SortedList 是否包含指定的键。 |
| 4    | **public virtual bool ContainsValue( object value );** 判断 SortedList 是否包含指定的值。 |
| 5    | **public virtual object GetByIndex( int index );** 获取 SortedList 的指定索引处的值。 |
| 6    | **public virtual object GetKey( int index );** 获取 SortedList 的指定索引处的键。 |
| 7    | **public virtual IList GetKeyList();** 获取 SortedList 中的键。 |
| 8    | **public virtual IList GetValueList();** 获取 SortedList 中的值。 |
| 9    | **public virtual int IndexOfKey( object key );** 返回 SortedList 中的指定键的索引，索引从零开始。 |
| 10   | **public virtual int IndexOfValue( object value );** 返回 SortedList 中的指定值第一次出现的索引，索引从零开始。 |
| 11   | **public virtual void Remove( object key );** 从 SortedList 中移除带有指定的键的元素。 |
| 12   | **public virtual void RemoveAt( int index );** 移除 SortedList 的指定索引处的元素。 |
| 13   | **public virtual void TrimToSize();** 设置容量为 SortedList 中元素的实际个数。 |

```
ortedList sortedList=new SortedList();
sortedList[key]; //键访问
sortedList.GetByIndex(index); //索引访问
```

**堆栈**

>堆栈（Stack）代表了一个**后进先出**的对象集合。当您需要对各项进行后进先出的访问时，则使用堆栈。当您在列表中添加一项，称为**推入**元素，当您从列表中移除一项时，称为**弹出**元素。

| 属性  | 描述                          |
| :---- | :---------------------------- |
| Count | 获取 Stack 中包含的元素个数。 |

| 序号 | 方法名 & 描述                                                |
| :--- | :----------------------------------------------------------- |
| 1    | **public virtual void Clear();** 从 Stack 中移除所有的元素。 |
| 2    | **public virtual bool Contains( object obj );** 判断某个元素是否在 Stack 中。 |
| 3    | **public virtual object Peek();** 返回在 Stack 的顶部的对象，但不移除它。 |
| 4    | **public virtual object Pop();** 移除并返回在 Stack 的顶部的对象。 |
| 5    | **public virtual void Push( object obj );** 向 Stack 的顶部添加一个对象。 |
| 6    | **public virtual object[] ToArray();** 复制 Stack 到一个新的数组中。 |

```
Stack stack=new Stack();
stack.Push(1);
stack.Push(2);
while (stack.Count!=0)
{
	Console.WriteLine(stack.Pop());
}

```

**队列**

> 队列（Queue）代表了一个**先进先出**的对象集合。当您需要对各项进行先进先出的访问时，则使用队列。当您在列表中添加一项，称为**入队**，当您从列表中移除一项时，称为**出队**。

| 属性  | 描述                          |
| :---- | :---------------------------- |
| Count | 获取 Queue 中包含的元素个数。 |

| 序号 | 方法名 & 描述                                                |
| :--- | :----------------------------------------------------------- |
| 1    | **public virtual void Clear();** 从 Queue 中移除所有的元素。 |
| 2    | **public virtual bool Contains( object obj );** 判断某个元素是否在 Queue 中。 |
| 3    | **public virtual object Dequeue();** 移除并返回在 Queue 的开头的对象。 |
| 4    | **public virtual void Enqueue( object obj );** 向 Queue 的末尾添加一个对象。 |
| 5    | **public virtual object[] ToArray();** 复制 Queue 到一个新的数组中。 |
| 6    | **public virtual void TrimToSize();** 设置容量为 Queue 中元素的实际个数。 |

```
Queue q = new Queue();
q.Enqueue('A');
q.Enqueue('M');
char ch = (char)q.Dequeue();
Console.WriteLine(ch);
foreach (char c in q)
Console.WriteLine(c);
```

### 32、泛型

>**泛型（Generic）** 允许您延迟编写类或方法中的编程元素的数据类型的规范，直到实际在程序中使用它的时候。
>
>可以通过数据类型的替代参数编写类或方法的规范。当编译器遇到类的构造函数或方法的函数调用时，它会生成代码来处理指定的数据类型。
>
>- 它有助于您最大限度地重用代码、保护类型的安全以及提高性能。
>- 您可以创建泛型集合类。.NET 框架类库在 *System.Collections.Generic* 命名空间中包含了一些新的泛型集合类。您可以使用这些泛型集合类来替代 *System.Collections* 中的集合类。
>- 您可以创建自己的泛型接口、泛型类、泛型方法、泛型事件和泛型委托。
>- 您可以对泛型类进行约束以访问特定数据类型的方法。
>- 关于泛型数据类型中使用的类型的信息可在运行时通过使用反射获取。



>**泛型类**
>
>```
>public class MyGenericArray<T>
>{
>   private T[] array;
>   public MyGenericArray(int size)
>   {
>       array = new T[size + 1];
>   }
>   public T getItem(int index)
>   {
>       return array[index];
>   }
>   public void setItem(int index, T value)
>   {
>       array[index] = value;
>   }
>}
>---------------------------------------------------------
>MyGenericArray<int> intArray = new MyGenericArray<int>(5);
>```
>
>**泛型方法**
>
>```
>void Swap<T>(ref T lhs, ref T rhs)
>{
>T temp;
>temp = lhs;
>lhs = rhs;
>rhs = temp;
>}
>-------------------------------------------------------------
>Swap<int>(ref a, ref b);
>```
>
>**泛型委托**
>
>```
>delegate T NumberChanger<T>(T n);
>public  int func1(int p)
>{
>return p+1;
>}
>public  T func2(T p)
>{
>return p;
>}
>NumberChanger<int> nc1 = new NumberChanger<int>(func1);
>nc1(25);
>NumberChanger<int> nc2 = new NumberChanger<int>(func2<int>);
>nc2(25);
>```
>
>

>在声明泛型方法/泛型类的时候，可以给泛型加上一定的约束来满足我们特定的一些条件。
>
>```
>class CacheHelper<T> where T:new()
>```
>
>- T：结构（类型参数必须是值类型。可以指定除 Nullable 以外的任何值类型）
>- T：类 （类型参数必须是引用类型，包括任何类、接口、委托或数组类型）
>- T：new() （类型参数必须具有无参数的公共构造函数。当与其他约束一起使用时new() 约束必须最后指定）
>- T：<基类名> 类型参数必须是指定的基类或派生自指定的基类
>- T：<接口名称> 类型参数必须是指定的接口或实现指定的接口。可以指定多个接口约束。约束接口也可以是泛型的。

### 33、匿名方法

>委托是用于引用与其具有相同标签的方法。可以使用委托对象调用可由委托引用的方法。
>
>匿名方法（Anonymous methods）提供了一种传递代码块作为委托参数的技术。匿名方法是没有名称只有主体的方法。
>
>在匿名方法中您不需要指定返回类型，它是从方法主体内的 return 语句推断的。
>
>

>
>
>匿名方法是通过使用 **delegate** 关键字创建委托实例来声明的。
>
>```
>delegate void NumberChanger(int n);
>...
>NumberChanger nc = delegate(int x)
>{
>Console.WriteLine("Anonymous Method: {0}", x);
>};
>nc(10);
>```
>
>

### 34、不安全的代码

>当一个代码块使用 **unsafe** 修饰符标记时，C# 允许在函数中使用指针变量。
>
>**指针** 是值为另一个变量的地址的变量，即，内存位置的直接地址。
>
>

| 实例        | 描述                             |
| :---------- | :------------------------------- |
| `int* p`    | `p` 是指向整数的指针。           |
| `double* p` | `p` 是指向双精度数的指针。       |
| `float* p`  | `p` 是指向浮点数的指针。         |
| `int** p`   | `p` 是指向整数的指针的指针。     |
| `int*[] p`  | `p` 是指向整数的指针的一维数组。 |
| `char* p`   | `p` 是指向字符的指针。           |
| `void* p`   | `p` 是指向未知类型的指针。       |

> ```
> // 不安全的函数 
> unsafe void func(string[] args)
> {
>   int var = 20;
>   int* p = &var;
> }
> //不安全的代码块
> void func(string[] args)
> {
> 	 unsafe
> 	 {
>       int var = 20;
>       int* p = &var;
>    }
> }
> //可以向方法传递指针变量作为方法的参数。
> unsafe void swap(int* p, int *q)
> {
>       int temp = *p;
>       *p = *q;
>       *q = temp;
> }
> swap(&a,&b);
> ```
>
> 

> 数组名称和一个指向与数组数据具有相同数据类型的指针是不同的变量类型。例如:int *p 和 int[] p 是不同的类型。可以增加指针变量 p，因为它在内存中不是固定的，但是数组地址在内存中是固定的，所以不能增加数组 p。
>
> 由于C#中声明的变量在内存中的存储受垃圾回收器管理；因此一个变量（例如一个大数组）有可能在运行过程中被移动到内存中的其他位置。如果一个变量的内存地址会变化，那么指针也就没有意义了。
>
> 如果需要使用指针变量访问数组数据，可以使用 **fixed** 关键字来固定指针。
>
> ```
> int[]  list = {10, 100, 200};
> fixed(int *ptr = list)
> 
> for ( int i = 0; i < 3; i++)
> {
>  Console.WriteLine("Address of list[{0}]={1}",i,(int)(ptr + i));
>  Console.WriteLine("Value of list[{0}]={1}", i, *(ptr + i));
> }
> ```
>
> 

### 35、多线程

> 
>
> **线程** 被定义为程序的执行路径。每个线程都定义了一个独特的控制流。如果应用程序涉及到复杂的和耗时的操作，那么设置不同的线程执行路径往往是有益的，每个线程执行特定的工作。
>
> 线程是**轻量级进程**。一个使用线程的常见实例是现代操作系统中并行编程的实现。使用线程节省了 CPU 周期的浪费，同时提高了应用程序的效率。

>## 线程生命周期
>
>线程生命周期开始于 System.Threading.Thread 类的对象被创建时，结束于线程被终止或完成执行时。
>
>下面列出了线程生命周期中的各种状态：
>
>- **未启动状态**：当线程实例被创建但 Start 方法未被调用时的状况。
>
>- **就绪状态**：当线程准备好运行并等待 CPU 周期时的状况。
>
>- 不可运行状态
>
> ：下面的几种情况下线程是不可运行的：
>
>
>
> - 已经调用 Sleep 方法
> - 已经调用 Wait 方法
> - 通过 I/O 操作阻塞
>
>- **死亡状态**：当线程已完成执行或已中止时的状况。
>
>可以使用 Thread 类的 **CurrentThread** 属性访问线程。

>下表列出了 **Thread** 类的一些常用的 **属性**：
>
>| 属性               | 描述                                                         |
>| :----------------- | :----------------------------------------------------------- |
>| CurrentContext     | 获取线程正在其中执行的当前上下文。                           |
>| CurrentCulture     | 获取或设置当前线程的区域性。                                 |
>| CurrentPrincipal   | 获取或设置线程的当前负责人（对基于角色的安全性而言）。       |
>| CurrentThread      | 获取当前正在运行的线程。                                     |
>| CurrentUICulture   | 获取或设置资源管理器使用的当前区域性以便在运行时查找区域性特定的资源。 |
>| ExecutionContext   | 获取一个 ExecutionContext 对象，该对象包含有关当前线程的各种上下文的信息。 |
>| IsAlive            | 获取一个值，该值指示当前线程的执行状态。                     |
>| IsBackground       | 获取或设置一个值，该值指示某个线程是否为后台线程。           |
>| IsThreadPoolThread | 获取一个值，该值指示线程是否属于托管线程池。                 |
>| ManagedThreadId    | 获取当前托管线程的唯一标识符。                               |
>| Name               | 获取或设置线程的名称。                                       |
>| Priority           | 获取或设置一个值，该值指示线程的调度优先级。                 |
>| ThreadState        | 获取一个值，该值包含当前线程的状态。                         |
>
>
>
>| 序号 | 方法名 & 描述                                                |
>| :--- | :----------------------------------------------------------- |
>| 1    | **public void Abort()** 在调用此方法的线程上引发 ThreadAbortException，以开始终止此线程的过程。调用此方法通常会终止线程。 |
>| 2    | **public static LocalDataStoreSlot AllocateDataSlot()** 在所有的线程上分配未命名的数据槽。为了获得更好的性能，请改用以 ThreadStaticAttribute 属性标记的字段。 |
>| 3    | **public static LocalDataStoreSlot AllocateNamedDataSlot( string name)** 在所有线程上分配已命名的数据槽。为了获得更好的性能，请改用以 ThreadStaticAttribute 属性标记的字段。 |
>| 4    | **public static void BeginCriticalRegion()** 通知主机执行将要进入一个代码区域，在该代码区域内线程中止或未经处理的异常的影响可能会危害应用程序域中的其他任务。 |
>| 5    | **public static void BeginThreadAffinity()** 通知主机托管代码将要执行依赖于当前物理操作系统线程的标识的指令。 |
>| 6    | **public static void EndCriticalRegion()** 通知主机执行将要进入一个代码区域，在该代码区域内线程中止或未经处理的异常仅影响当前任务。 |
>| 7    | **public static void EndThreadAffinity()** 通知主机托管代码已执行完依赖于当前物理操作系统线程的标识的指令。 |
>| 8    | **public static void FreeNamedDataSlot(string name)** 为进程中的所有线程消除名称与槽之间的关联。为了获得更好的性能，请改用以 ThreadStaticAttribute 属性标记的字段。 |
>| 9    | **public static Object GetData( LocalDataStoreSlot slot )** 在当前线程的当前域中从当前线程上指定的槽中检索值。为了获得更好的性能，请改用以 ThreadStaticAttribute 属性标记的字段。 |
>| 10   | **public static AppDomain GetDomain()** 返回当前线程正在其中运行的当前域。 |
>| 11   | **public static AppDomain GetDomainID()** 返回唯一的应用程序域标识符。 |
>| 12   | **public static LocalDataStoreSlot GetNamedDataSlot( string name )** 查找已命名的数据槽。为了获得更好的性能，请改用以 ThreadStaticAttribute 属性标记的字段。 |
>| 13   | **public void Interrupt()** 中断处于 WaitSleepJoin 线程状态的线程。 |
>| 14   | **public void Join()** 在继续执行标准的 COM 和 SendMessage 消息泵处理期间，阻塞调用线程，直到某个线程终止为止。此方法有不同的重载形式。 |
>| 15   | **public static void MemoryBarrier()** 按如下方式同步内存存取：执行当前线程的处理器在对指令重新排序时，不能采用先执行 MemoryBarrier 调用之后的内存存取，再执行 MemoryBarrier 调用之前的内存存取的方式。 |
>| 16   | **public static void ResetAbort()** 取消为当前线程请求的 Abort。 |
>| 17   | **public static void SetData( LocalDataStoreSlot slot, Object data )** 在当前正在运行的线程上为此线程的当前域在指定槽中设置数据。为了获得更好的性能，请改用以 ThreadStaticAttribute 属性标记的字段。 |
>| 18   | **public void Start()** 开始一个线程。                       |
>| 19   | **public static void Sleep( int millisecondsTimeout )** 让线程暂停一段时间。 |
>| 20   | **public static void SpinWait( int iterations )** 导致线程等待由 iterations 参数定义的时间量。 |
>| 21   | **public static byte VolatileRead( ref byte address ) public static double VolatileRead( ref double address ) public static int VolatileRead( ref int address ) public static Object VolatileRead( ref Object address )** 读取字段值。无论处理器的数目或处理器缓存的状态如何，该值都是由计算机的任何处理器写入的最新值。此方法有不同的重载形式。这里只给出了一些形式。 |
>| 22   | **public static void VolatileWrite( ref byte address, byte value ) public static void VolatileWrite( ref double address, double value ) public static void VolatileWrite( ref int address, int value ) public static void VolatileWrite( ref Object address, Object value )** 立即向字段写入一个值，以使该值对计算机中的所有处理器都可见。此方法有不同的重载形式。这里只给出了一些形式。 |
>| 23   | **public static bool Yield()** 导致调用线程执行准备好在当前处理器上运行的另一个线程。由操作系统选择要执行的线程。 |
>
>

>
>
>```
>// 创建线程
>ThreadStart childref = new ThreadStart(targetFunc);
>Thread childThread = new Thread(childref);
>childThread.Start();
>// 线程名称
>childThread.name
>// 获得当前线程
>Thread.CurrentThread;
>// 休眠线程
>childThread.sleep(milliseconds)
>//销毁线程，通过抛出 threadabortexception在运行时中止线程。这个异常不能被捕获，如果有 *finally* 块，控制会被送至 *finally* 块。
>childThread.Abort()
>```

>线程函数通过委托传递，可以不带参数，也可以带参数（只能有一个参数），可以用一个类或结构体封装参数
>
>```
>childThread.Start(param);
>```
>
>