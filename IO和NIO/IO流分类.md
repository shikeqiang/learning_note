# IO流分类

这里要先区分一个概念，输入和输出流：==划分输入/输出流时是从程序运行所在的内存的角度来考虑的==，如图，是输出流。![1544082807835](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/1544082807835.png)![1544083124658](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/1544083124658.png)

数据从服务器通过网络流向客户端，在这种情况下,Server端的内存负责将数据输出到网络里，因此Server端的程序使用输出流；Client端的内存负责从网络中读取数据，因此Client端的程序应该使用输入流。

==java的输入流主要是InputStream和Reader作为基类，而输出流则是主要由outputStream和Writer作为基类。它们都是一些抽象基类，无法直接创建实例==。

# 一、分类

**（1） 按操作方式分类结构图：**

[![按操作方式分类结构图：](https://camo.githubusercontent.com/50f105c85f6b42d643d46e1ac7bb0f855b92cd9d/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f352f31362f313633363764346664316365316234363f773d37323026683d3130383026663d6a70656726733d3639353232)](https://camo.githubusercontent.com/50f105c85f6b42d643d46e1ac7bb0f855b92cd9d/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f352f31362f313633363764346664316365316234363f773d37323026683d3130383026663d6a70656726733d3639353232)

#### 1.1 字节流的概念

在Java API中，可以从其中读入一个字节序列的对象称作输入流，而可以向其中写入一个字节序列的对象称作输出流。抽象类**InputStream和OutputStream**构成了输入/输出(I/O)类层次结构的基础。

#### 1.2 InputStream和OutputStream相关方法

![1544078605855](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/1544078605855.png)

![1544078626789](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/1544078626789.png)

![1544078648580](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/1544078648580.png)

#### 1.3 分类说明

**1） 输入字节流InputStream**（要输入首先就先要读取数据）：

- **ByteArrayInputStream、StringBufferInputStream、FileInputStream** 是三种基本的介质流，它们分别从Byte 数组、StringBuffer、和本地文件中**读取**数据。
- **PipedInputStream** 是从与其它线程共用的管道中读取数据。PipedInputStream的一个实例要和PipedOutputStream的一个实例共同使用，共同完成管道的读取写入操作。主要用于线程操作。
- **DataInputStream**： 将基础数据类型读取出来
- **ObjectInputStream** 和所有 **FilterInputStream** 的子类都是装饰流（装饰器模式的主角）。

**2）输出字节流OutputStream(要输出首先就要先写入数据)：**

**ByteArrayOutputStream**、**FileOutputStream**： 是两种基本的介质流，它们分别向- Byte 数组、和本地文件中**写入数据**。

**PipedOutputStream** 是向与其它线程共用的管道中写入数据。

**DataOutputStream** 将基础数据类型写入到文件中

**ObjectOutputStream** 和所有 **FilterOutputStream** 的子类都是装饰流。

**3）字符输入流Reader：**

- **FileReader、CharReader、StringReader** 是三种基本的介质流，它们分在本地文件、Char 数组、String中读取数据。
- **PipedReader**：是从与其它线程共用的管道中读取数据
- **BufferedReader** ：加缓冲功能，避免频繁读写硬盘
- **InputStreamReader**： 是一个连接字节流和字符流的桥梁，它将字节流转变为字符流。

**4）字符输出流Writer：**

- **StringWriter**:向String 中写入数据。
- **CharArrayWriter**：实现一个可用作字符输入流的字符缓冲区
- **PipedWriter**:是向与其它线程共用的管道中写入数据
- **BufferedWriter** ： 增加缓冲功能，避免频繁读写硬盘。
- **PrintWriter** 和**PrintStream** 将对象的格式表示打印到文本输出流。 极其类似，功能和使用也非常相似
- **OutputStreamWriter**： 是OutputStream 到Writer 转换的桥梁，它的子类FileWriter 其实就是一个实现此功能的具体类（具体可以研究一SourceCode）。功能和使用和OutputStream 极其类似，后面会有它们的对应图。

**（2）按操作对象分类结构图**

[![按操作对象分类结构图](https://camo.githubusercontent.com/8957efacdf1cd4eac15d844da8353a7f77a3c863/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f352f31362f313633363764363733623065323638643f773d37323026683d35333526663d6a70656726733d3436303831)](https://camo.githubusercontent.com/8957efacdf1cd4eac15d844da8353a7f77a3c863/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f352f31362f313633363764363733623065323638643f773d37323026683d35333526663d6a70656726733d3436303831)

#### 分类说明：

**对文件进行操作（节点流）：**

- FileInputStream（字节输入流，读取数据），
- FileOutputStream（字节输出流，写入数据），
- FileReader（字符输入流），
- FileWriter（字符输出流）

**对管道进行操作（节点流）：**

- PipedInputStream（字节输入流）,
- PipedOutStream（字节输出流），
- PipedReader（字符输入流），
- PipedWriter（字符输出流）。
- PipedInputStream的一个实例要和PipedOutputStream的一个实例共同使用，共同完成管道的读取写入操作。主要用于线程操作。

**字节/字符数组流（节点流）：**

- ByteArrayInputStream，
- ByteArrayOutputStream，
- CharArrayReader，
- CharArrayWriter；

**除了上述三种是节点流，其他都是处理流，需要跟节点流配合使用。**

**Buffered缓冲流（处理流）：**<u>带缓冲区的处理流，缓冲区的作用的主要目的是：避免每次和硬盘打交道，提高数据访问的效率。</u>

- BufferedInputStream，
- BufferedOutputStream，
- BufferedReader,
- BufferedWriter,

**转化流（处理流）：**

- InputStreamReader：把字节转化成字符；
- OutputStreamWriter：把字节转化成字符。

**基本类型数据流（处理流）：用于操作基本数据类型值。**

因为平时若是我们输出一个8个字节的long类型或4个字节的float类型，那怎么办呢？可以一个字节一个字节输出，也可以把转换成字符串输出，但是这样转换费时间，若是直接输出该多好啊，因此这个数据流就解决了我们输出数据类型的困难。**数据流可以直接输出float类型或long类型，提高了数据读写的效率。**

- DataInputStream，
- DataOutputStream。

**打印流（处理流）：**

一般是打印到控制台，可以进行控制打印的地方。

- PrintStream，
- PrintWriter，

**对象流（处理流）：**

把封装的对象直接输出，而不是一个个在转换成字符串再输出。

- ObjectInputStream，对象反序列化；
- ObjectOutputStream，对象序列化；

**合并流（处理流）：**

- SequenceInputStream：可以认为是一个工具类，将两个或者多个输入流当成一个输入流依次读取。

### 其他类：File（已经被Java7的Path取代）

File类是对文件系统中文件以及文件夹进行封装的对象，可以通过对象的思想来操作文件和文件夹。 File类保存文件或目录的各种元数据信息，包括文件名、文件长度、最后修改时间、是否可读、获取当前文件的路径名，判断指定文件是否存在、获得当前目录中的文件列表，创建、删除文件和目录等方法。

### 其他类：RandomAccessFile

该对象并不是流体系中的一员，其封装了字节流，同时还封装了一个==缓冲区（字符数组），通过内部的指针来操作字符数组中的数据。== 该对象特点：

1. 该对象只能操作文件，所以构造函数接收两种类型的参数：a.字符串文件路径；b.File对象。
2. 该对象既可以对文件进行读操作，也能进行写操作，在进行对象实例化时可指定操作模式(r,rw)。

**注意：** IO中的很多内容都可以使用NIO完成，这些知识点大家知道就好，使用的话还是尽量使用NIO/AIO。![1544083896118](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/1544083896118.png)

**==注：表中粗体字所标出的类代表节点流，必须直接与指定的物理节点关联：斜体字标出的类代表抽象基类，无法直接创建实例。==**

例子：

```java
public class FileOutputStreamTest {
    public  static void main(String[] args)throws IOException {
        FileInputStream fis=null;
        FileOutputStream fos=null;
        try {
            //创建字节输入流
            fis=new FileInputStream("E:\\learnproject\\Iotest\\lib\\src\\main\\java\\com\\Test.txt");
            //创建字节输出流
            fos=new FileOutputStream("E:\\learnproject\\Iotest\\lib\\src\\main\\java\\com\\newTest.txt");

            byte[] b=new byte[1024];
            int hasRead=0;

            //循环从输入流中取出数据
            while((hasRead=fis.read(b))>0){
                //每读取一次，即写入文件输入流，读了多少，就写多少。
                fos.write(b,0,hasRead);
            }
        }catch (IOException e){
            e.printStackTrace();
        }finally {
            fis.close();
            fos.close();
        }
    }
}
```

这里是通过定义了两个继承了输入流InputStream和输出流OutputStream的类：FileInputStream和FileOutputStream，然后通过读取文件Test.txt，此时程序应该用输入流FileInputStream的read方法，然后将读取到数据存到newTest.txt，用FileOutputStream的write方法，注意最后两个流都要用close关闭。

注： 使用java的io流执行输出时，不要忘记关闭输出流，关闭输出流除了可以保证流的物理资源被回收之外，可能==还可以将输出流缓冲区中的数据flush到物理节点中里（因为在执行close（）方法之前，自动执行输出流的flush（）方法）==。java很多输出流默认都提供了缓存功能，其实我们没有必要刻意去记忆哪些流有缓存功能，哪些流没有，只有正常关闭所有的输出流即可保证程序正常。

下面介绍字节缓存流的用法（字符缓存流的用法和字节缓存流一致就不介绍了）：

```Java
public class BufferedStreamTest {
    public  static void main(String[] args)throws IOException {
        FileInputStream fis=null;
        FileOutputStream fos=null;
        BufferedInputStream bis=null;
        BufferedOutputStream bos=null;
        try {
            //创建字节输入流
            fis=new FileInputStream("E:\\learnproject\\Iotest\\lib\\src\\main\\java\\com\\Test.txt");
            //创建字节输出流
            fos=new FileOutputStream("E:\\learnproject\\Iotest\\lib\\src\\main\\java\\com\\newTest.txt";
            //创建字节缓存输入流
            bis=new BufferedInputStream(fis);
            //创建字节缓存输出流
            bos=new BufferedOutputStream(fos);

            byte[] b=new byte[1024];
            int hasRead=0;
            //循环从缓存流中读取数据
            while((hasRead=bis.read(b))>0){
                //向缓存流中写入数据，读取多少写入多少
                bos.write(b,0,hasRead);
            }
        }catch (IOException e){
            e.printStackTrace();
        }finally {
            bis.close();
            bos.close();
        }
    }
}
```

可以看到使用字节缓存流读取和写入数据的方式和文件流（FileInputStream,FileOutputStream）并没有什么不同，只是把处理流套接到文件流上进行读写。缓存流的原理下节介绍。

上面代码中我们使用了缓存流和文件流，但是我们只关闭了缓存流。这个需要注意一下，当我们使用处理流套接到节点流上的使用的时候，只需要关闭最上层的处理就可以了。java会自动帮我们关闭下层的节点流。



NIO出了一个跟File差不多的类：Path，参照这个：https://mp.weixin.qq.com/s?__biz=MzU4NDQ4MzU5OA==&mid=2247483976&idx=1&sn=2296c05fc1b840a64679e2ad7794c96d&chksm=fd985429caefdd3f48e2ee6fdd7b0f6fc419df90b3de46832b484d6d1ca4e74e7837689c8146&scene=21#wechat_redirect



参照：Java面试通关手册公众号