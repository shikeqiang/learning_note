# NIO新特性

NIO在基础的IO流上发展处新的特点，分别是：==**内存映射技术，字符及编码，非阻塞I/O和文件锁定。**==下面我们分别就这些技术做一些说明。

## 一、内存映射

这个功能主要是为了提高大文件的读写速度而设计的。内存映射文件(memory-mappedfile)能让你创建和修改那些大到无法读入内存的文件。有了内存映射文件，你就可以认为文件已经全部读进了内存，然后把它当成一个非常大的数组来访问了。将文件的一段区域映射到内存中，比传统的文件处理速度要快很多。内存映射文件它虽然最终也是要从磁盘读取数据，但是它并不需要将数据读取到OS内核缓冲区，而是直接将进程的用户私有地址空间中的一部分区域与文件对象建立起映射关系，就好像直接从内存中读、写文件一样，速度当然快了。

NIO中内存映射主要用到以下两个类：

- **java.nio.MappedByteBuffer**
- **java.nio.channels.FileChannel**

用一个内存映射读取文件和普通IO流读取文件的例子来对比：

```Java
public class MemMap {
    public static void main(String[] args) {
        try {
            RandomAccessFile file = new RandomAccessFile("c://1.pdf","rw");
            FileChannel channel = file.getChannel();
            MappedByteBuffer buffer = channel.map(FileChannel.MapMode.READ_ONLY,0,channel.size());
            ByteBuffer buffer1 = ByteBuffer.allocate(1024);
            byte[] b = new byte[1024];
            long len = file.length();
            long startTime = System.currentTimeMillis();
            //读取内存映射文件
            for(int i=0;i<file.length();i+=1024*10){
                if (len - i > 1024) {
                    buffer.get(b);
                } else {
                    buffer.get(new byte[(int)(len - i)]);
                }
            }
            long endTime = System.currentTimeMillis();
            System.out.println("使用内存映射方式读取文件总耗时： "+(endTime - startTime));
            //普通IO流方式
            long startTime1 = System.currentTimeMillis();
            while(channel.read(buffer1) > 0){
                buffer1.flip();
                buffer1.clear();
            }
            long endTime1 = System.currentTimeMillis();
            System.out.println("使用普通IO流方式读取文件总耗时： "+(endTime1 - startTime1));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

结果为：![img](https://raw.githubusercontent.com/JDawnF/learning_note/master/images//biLgFkf.png)

上面程序中调用FileChannel类的**==map方法==**进行内存映射，map主要有三个参数，第一个参数设置映射模式,现在支持3种模式：

##### 1.FileChannel.MapMode.READ_ONLY：只读缓冲区，在缓冲区中如果发生写操作则会产生ReadOnlyBufferException；

##### 2.FileChannel.MapMode.READ_WRITE：读写缓冲区，任何时刻如果通过内存映射的方式修改了文件则立刻会对磁盘上的文件执行相应的修改操作。别的进程如果也共享了同一个映射，则也会同步看到变化。而不是像标准IO那样每个进程有各自的内核缓冲区，比如JAVA代码中，没有执行 IO输出流的 flush() 或者 close() 操作，那么对文件的修改不会更新到磁盘去，除非进程运行结束；

##### 3.FileChannel.MapMode.PRIVATE ：私有缓冲区，可以进行修改，但是任何修改是缓冲区私有的，不会回到文件中。

我们注意到FileChannel类中有map方法来建立内存映射，但是**没有相应的unmap方法来卸载映射内存呢。**一旦建立映射保持有效，**直到MappedByteBuffer对象被垃圾收集。** 此外，映射缓冲区不会绑定到创建它们的通道。 关闭相关的FileChannel不会破坏映射; 只有缓冲对象本身的处理打破了映射。

**==内存映射文件的优点：==**

- 用户进程将文件数据视为内存，因此不需要发出read()或write()系统调用。
- 当用户进程触摸映射的内存空间时，将自动生成页面错误，以从磁盘引入文件数据。 如果用户修改映射的内存空间，受影响的页面将自动标记为脏，并随后刷新到磁盘以更新文件。
- 操作系统的虚拟内存子系统将执行页面的智能缓存，根据系统负载自动管理内存。
- 数据始终是页面对齐的，不需要缓冲区复制。
- 可以映射非常大的文件，而不消耗大量内存来复制数据。


下面看看复制一个120M的文件速度有多快：

```Java
public class MemMapReadWrite {
    private static int len;
    /**
     * 读文件
     * @param fileName
     * @return
     */
    public static ByteBuffer readFile(String fileName) {
        try {
            RandomAccessFile file = new RandomAccessFile(fileName, "rw");
            len = (int) file.length(); //length方法返回一个long类型的值
            FileChannel channel = file.getChannel();
            MappedByteBuffer buffer = channel.map(FileChannel.MapMode.READ_ONLY, 0, len);
            return buffer.get(new byte[(int) file.length()]);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
     * 写文件
     * @param readFileName
     * @param writeFileName
     */
    public static void writeFile(String readFileName, String writeFileName) {
        try {
            RandomAccessFile file = new RandomAccessFile(writeFileName, "rw");
            FileChannel channel = file.getChannel();
            ByteBuffer buffer = readFile(readFileName);

            MappedByteBuffer bytebuffer = channel.map(FileChannel.MapMode.READ_WRITE, 0, len);
            long startTime = System.currentTimeMillis();
            for (int i = 0; i < len; i++) {
                bytebuffer.put(i, buffer.get(i));
            }
            bytebuffer.flip();
            long endTime = System.currentTimeMillis();
            System.out.println("写文件耗时： " + (endTime - startTime));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        String readFileName = "c://1.pdf";
        String writeFileName = "c://2.pdf";
        writeFile(readFileName, writeFileName);
    }
}
```

结果为：![img](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/bMsFKdn.png)

### 二、字符及编码

说到字符和编码，我们的先说一个概念，字符编码方案：

**编码方案定义了如何把字符编码的序列表达为字节序列。**字符编码的数值不需要与编码字节相同，也不需要是一对一或一对多个的关系。原则上，把字符集编码和解码近似视为对象的序列化和反序列化。

**通常字符数据编码是用于网络传输或文件存储。编码方案不是字符集，它是映射；**但是因为它们之间的紧密联系，大部分编码都与一个独立的字符集相关联。例如，UTF-8，仅用来编码Unicode字符集。尽管如此，用一个编码方案处理多个字符集还是可能发生的。例如，EUC可以对几个亚洲语言的字符进行编码。

目前字符编码方案有US-ASCII,UTF-8,GB2312, BIG5,GBK,GB18030,UTF-16BE, UTF-16LE, UTF-16,UNICODE。其中Unicode试图把全世界所有语言的字符集统一到全面的映射之中。虽然占有一定的市场份额，但是目前其余的字符方案仍然广被采用。大**部分的操作系统在I/O与文件存储方面仍是以字节为导向的，所以无论使用何种编码，Unicode或其他编码，在字节序列和字符集编码之间仍需要进行转化。**

由==java.nio.charset==包组成的类满足了这个需求。这不是Java平台第一次处理字符集编码，但是它是最系统、最全面、以及最灵活的解决方式。 

下面我们通过一个小例子来看一下通过不同的Charset实现如何把字符翻译成字节序列：

```Java
public class CharsetTest {
    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);
        String str = input.next();
        String[] charsetNames = {"US-ASCII", "ISO-8859-1", "UTF-8", "UTF-16BE",
                "UTF-16LE", "UTF-16"};
        for (int i = 0; i < charsetNames.length; i++) {
            doEncode(Charset.forName(charsetNames[i]), str);
        }
    }
    private static void doEncode(Charset cs, String input) {
        ByteBuffer bb = cs.encode(input);
        System.out.println("Charset: " + cs.name());
        System.out.println(" Input: " + input);
        System.out.println("Encoded: ");
        for (int i = 0; bb.hasRemaining(); i++) {
            int b = bb.get();
            int ival = ((int) b) & 0xff;
            char c = (char) ival;
            // Keep tabular alignment pretty
            if (i < 10) System.out.print(" ");
            // 打印索引序列
            System.out.print(" " + i + ": ");
            // Better formatted output is coming someday...
            if (ival < 16)
                System.out.print("0");
            // 输出该字节位值的16进制形式
            System.out.print(Integer.toHexString(ival));
            // 打印出刚才我们输入的字符，如果是空格或者标准字符集中没有包含
            //该字符输出空格，否则输出该字符
            if (Character.isWhitespace(c) || Character.isISOControl(c)) {
                System.out.println("");
            } else {
                System.out.println(" (" + c + ")");
            }
        }
        System.out.println("");
    }
}
```

输出结果是：

```Java
abc
Charset: US-ASCII
 Input: abc
Encoded: 
  0: 61 (a)
  1: 62 (b)
  2: 63 (c)

Charset: ISO-8859-1
 Input: abc
Encoded: 
  0: 61 (a)
  1: 62 (b)
  2: 63 (c)

Charset: UTF-8
 Input: abc
Encoded: 
  0: 61 (a)
  1: 62 (b)
  2: 63 (c)

Charset: UTF-16BE
 Input: abc
Encoded: 
  0: 00
  1: 61 (a)
  2: 00
  3: 62 (b)
  4: 00
  5: 63 (c)

Charset: UTF-16LE
 Input: abc
Encoded: 
  0: 61 (a)
  1: 00
  2: 62 (b)
  3: 00
  4: 63 (c)
  5: 00

Charset: UTF-16
 Input: abc
Encoded: 
  0: fe (þ)
  1: ff (ÿ)
  2: 00
  3: 61 (a)
  4: 00
  5: 62 (b)
  6: 00
  7: 63 (c)

```

### 2.1 字符集编码器和解码器

字符的编码和解码是使用很频繁的，试想如果使用UTF-8字符集进行编码，但是却是用UTF-16字符集进行解码，那么这条信息对于用户来说其实是无用的。因为没人能看得懂。==在NIO中提供了两个类CharsetEncoder和CharsetDecoder来实现编码转换方案。==

**CharsetEncoder类是一个状态编码引擎**。实际上，编码器有状态意味着它们**不是线程安全的**：CharsetEncoder对象不应该在线程中共享。CharsetEncoder对象是一个状态转换引擎：字符进去，字节出来。一些编码器的调用可能需要完成转换。编码器存储在调用之间转换的状态。

字符集解码器是编码器的逆转。通过特殊的编码方案把字节编码转化成16-位Unicode字符的序列。与CharsetEncoder类似的, CharsetDecoder也是状态转换引擎。

### 三、非阻塞IO

一般来说 I/O 模型可以分为：==同步阻塞，同步非阻塞，异步阻塞，异步非阻塞==四种IO模型。

##### 同步阻塞 IO ：

在此种方式下，用户进程在发起一个 IO 操作以后，必须等待 IO 操作的完成，只有当真正完成了 IO 操作以后，用户进程才能运行。 JAVA传统的 IO 模型属于此种方式！

##### 同步非阻塞 IO:

在此种方式下，用户进程发起一个 IO 操作以后可以返回做其它事情，但是用户进程需要时不时的询问 IO 操作是否就绪，这就要求用户进程不停的去询问，从而引入不必要的 CPU 资源浪费。其中目前 **JAVA 的 NIO 就属于同步非阻塞 IO 。**

##### 异步阻塞 IO ：

此种方式下是指应用发起一个 IO 操作以后，不等待内核 IO 操作的完成，等内核完成 IO 操作以后会通知应用程序，这其实就是同步和异步最关键的区别，同步必须等待或者主动的去询问 IO 是否完成，那么为什么说是阻塞的呢？因为此时是通过 select 系统调用来完成的，而 select 函数本身的实现方式是阻塞的，而采用 select 函数有个好处就是它可以同时监听多个文件句柄，从而提高系统的并发性！

##### 异步非阻塞 IO:

在此种模式下，用户进程只需要发起一个 IO 操作然后立即返回，等 IO 操作真正的完成以后，应用程序会得到 IO 操作完成的通知，此时用户进程只需要对数据进行处理就好了，不需要进行实际的 IO 读写操作，因为 真正的 IO读取或者写入操作已经由 内核完成了。目前 Java 中还没有支持此种 IO 模型。

##### 典型的非阻塞IO模型如下：

```Java
while(true){
    data = socket.read();
    if(data!= error){
        处理数据
        break;
    }
}
```

但是对于非阻塞IO就有一个非常严重的问题，**在while循环中需要不断地去询问内核数据是否就绪，这样会导致CPU占用率非常高**，因此一般情况下很少使用while循环这种方式来读取数据。所以这就不得不说到下面这个概念–多路复用IO模型。

##### 多路复用IO模型

**在多路复用IO模型中，会有一个线程不断去轮询多个socket的状态，只有当socket真正有读写事件时，才真正调用实际的IO读写操作。**==因为在多路复用IO模型中，只需要使用一个线程就可以管理多个socket，==系统不需要建立新的进程或者线程，也不必维护这些线程和进程，并且只有在真正有socket读写事件进行时，才会使用IO资源，所以它大大减少了资源占用。

**NIO 的非阻塞 I/O 机制是围绕 选择器和 通道构建的。** Channel 类表示服务器和客户机之间的一种通信机制。Selector 类是 Channel 的多路复用器。 Selector 类将传入客户机请求多路分用并将它们分派到各自的请求处理程序。**NIO 设计背后的基石是反应器(Reactor)设计模式。**

**Reactor负责IO事件的响应，一旦有事件发生，便广播发送给相应的handler去处理**。而NIO的设计则是完全按照Reactor模式来设计的。Selector发现某个channel有数据时，会通过SelectorKey来告知，然后实现事件和handler的绑定。

在Reactor模式中，包含如下角色：

- **Reactor 将I/O事件发派给对应的Handler**
- **Acceptor 处理客户端连接请求**
- **Handlers 执行非阻塞读/写**

我们简单写一个利用了Reactor模式的NIO服务端:

```Java
public class NIOServer {
    private static final Logger LOGGER = LoggerFactory.getLogger(NIOServer.class);
    public static void main(String[] args) throws IOException {
        Selector selector = Selector.open();
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        serverSocketChannel.configureBlocking(false); // 服务器配置为非阻塞
        serverSocketChannel.bind(new InetSocketAddress(1234));
        //注册到选择器中，并设置为连接就绪
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
        while (true) {
//选择一组其相应通道准备好进行I/O操作的键。此方法执行非阻塞selection operation 。 如果以前的选择操作没有频道变为可选择，则该方法立即返回零。调用此方法将清除任何先前调用wakeup方法的效果。返回：SelectionKey，这个SelectionKey是四种状态，是int类型
            if(selector.selectNow() < 0) {
                continue;
            }
            //获取注册的channel
            Set<SelectionKey> keys = selector.selectedKeys();
            //遍历所有的key
            Iterator<SelectionKey> iterator = keys.iterator();
            while(iterator.hasNext()) {
                SelectionKey key = iterator.next();
                iterator.remove();
                //如果通道上有事件发生
                if (key.isAcceptable()) {
                    //获取该通道
                    ServerSocketChannel acceptServerSocketChannel = (ServerSocketChannel) key.channel();
                    SocketChannel socketChannel = acceptServerSocketChannel.accept();
                    socketChannel.configureBlocking(false);
                    LOGGER.info("Accept request from {}", socketChannel.getRemoteAddress());
                    //同时将SelectionKey标记为可读，以便读取。
                    SelectionKey readKey = socketChannel.register(selector, SelectionKey.OP_READ);
                    //利用SelectionKey的attache功能绑定Acceptor 如果有事情，触发Acceptor
                    //Processor对象为自定义处理请求的类
                    readKey.attach(new Processor());
                } else if (key.isReadable()) {
                    Processor processor = (Processor) key.attachment();
                    processor.process(key);
                }
            }
        }
    }
}
/**
 * Processor类中设置一个线程池来处理请求，
 * 这样就可以充分利用多线程的优势
 */
class Processor {
    private static final Logger LOGGER = LoggerFactory.getLogger(Processor.class);
    private static final ExecutorService service = Executors.newFixedThreadPool(16);

    public void process(final SelectionKey selectionKey) {
        service.submit(new Runnable() {
            @Override
            public void run() {
                ByteBuffer buffer = null;
                SocketChannel socketChannel = null;
                try {
                    buffer = ByteBuffer.allocate(1024);
                    socketChannel = (SocketChannel) selectionKey.channel();
                    int count = socketChannel.read(buffer);
                    if (count < 0) {
                        socketChannel.close();
                        selectionKey.cancel();
                        LOGGER.info("{}\t Read ended", socketChannel);
                    } else if(count == 0) {
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
                LOGGER.info("{}\t Read message {}", socketChannel, new String(buffer.array()));
            }
        });
    }
}
```

利用多路复用机制避免了线程的阻塞，提高了连接的数量。一个线程就可以管理多个socket，**只有当socket真正有读写事件发生才会占用资源来进行实际的读写操作。**虽然多线程+ 阻塞IO 达到类似的效果，但是由于在多线程 + 阻塞IO 中，每个socket对应一个线程，这样会造成很大的资源占用，并且尤其是对于长连接来说，线程的资源一直不会释放，如果后面陆续有很多连接的话，就会造成性能上的瓶颈。

另外多路复用IO为何比非阻塞IO模型的效率高是因为在非阻塞IO中，不断地询问socket状态时通过用户线程去进行的，而在多路复用IO中，**轮询每个socket状态是内核在进行的，**这个效率要比用户线程要高的多。

### 四、文件锁定

**NIO中的文件通道（FileChannel）在读写数据的时候主要使用了阻塞模式，它不能支持非阻塞模式的读写，而且FileChannel的对象是不能够直接实例化的， 他的实例只能通过getChannel()从一个打开的文件对象上边读取（RandomAccessFile、 FileInputStream、FileOutputStream），并且通过调用getChannel()方法返回一个 Channel对象去连接同一个文件，也就是针对同一个文件进行读写操作。**

文件锁的出现解决了很多Java应用程序和非Java程序之间共享文件数据的问题，在以前的JDK版本中，没有文件锁机制使得Java应用程序和其他非Java进程程序之间不能够针对同一个文件共享 数据，有可能造成很多问题，JDK1.4里面有了FileChannel，它的锁机制使得文件能够针对很多非 Java应用程序以及其他Java应用程序可见。但是Java里面 的文件锁机制主要是基于共 享锁模型，在不支持共享锁模型的操作系统上，文件锁本身也起不了作用，JDK1.4使用文件通道读写方式可以向一些文件 发送锁请求， FileChannel的 锁模型主要针对的是每一个文件，并不是每一个线程和每一个读写通道，也就是以文件为中心进行共享以及独占，也就是文件锁本身并不适合于同一个JVM的不同 线程之间。

我们简要看一下相关API：

```java
// 如果请求的锁定范围是有效的，阻塞直至获取锁
 public final FileLock lock()  
// 尝试获取锁非阻塞，立刻返回结果  
 public final FileLock tryLock()  

// 第一个参数：要锁定区域的起始位置  
// 第二个参数：要锁定区域的尺寸,  
// 第三个参数：true为共享锁，false为独占锁  
 public abstract FileLock lock (long position, long size, boolean shared)  
 public abstract FileLock tryLock (long position, long size, boolean shared) 
```

**锁定区域的范围不一定要限制在文件的size值以内，锁可以扩展从而超出文件尾。**因此，我们可以提前把待写入数据的区域锁定，我们也可以锁定一个不包含任何文件内容的区域，比如文件最后一个字节以外的区域。如果之后文件增长到达那块区域，那么你的文件锁就可以保护该区域的文件内容了。相反地，如果你锁定了文件的某一块区域，然后文件增长超出了那块区域，那么新增加 的文件内容将不会受到您的文件锁的保护。

我们写一个简单实例：

```Java
public class NIOLock {
    private static final Logger LOGGER = LoggerFactory.getLogger(NIOServer.class);
    public static void main(String[] args) throws IOException {
        FileChannel fileChannel = new RandomAccessFile("c://1.txt", "rw").getChannel();
        // 写入4个字节,wrap 将一个字节数组包装到缓冲区中。
        fileChannel.write(ByteBuffer.wrap("abcd".getBytes()));
        // 将前2个字节区域锁定（共享锁）
        FileLock lock1 = fileChannel.lock(0, 2, true);
        // 当前锁持有锁的类型（共享锁/独占锁）
        lock1.isShared();
        // IOException 不能修改只读的共享区域
        // fileChannel.write(ByteBuffer.wrap("a".getBytes()));
        // 可以修改共享锁之外的区域，从第三个字节开始写入
        fileChannel.write(ByteBuffer.wrap("ef".getBytes()), 2);
        // OverlappingFileLockException 重叠的文件锁异常
        // FileLock lock2 = fileChannel.lock(0, 3, true);
        // FileLock lock3 = fileChannel.lock(0, 3, false);
        //得到创建锁的通道
        lock1.channel();
        //锁的起始位置
        long position = lock1.position();
        //锁的范围
        long size = lock1.size();
        //判断锁是否与指定文件区域有重叠
        lock1.overlaps(position, size);
        // 记得用try/catch/finally{release()}方法释放锁
        lock1.release();
    }
}
```

具体例子，NIO socket服务端和客户端，首先是服务端，然后是客户端，先运行服务端：

```Java
public class Server {
    //标识数字/
    private int flag = 0;
    //缓冲区大小/
    private int BLOCK = 4096;
    //接受数据缓冲区/
    private ByteBuffer sendbuffer = ByteBuffer.allocate(BLOCK);
    //发送数据缓冲区/
    private ByteBuffer receivebuffer = ByteBuffer.allocate(BLOCK);
    private Selector selector;

    public static void main(String[] args) throws IOException {
        // TODO Auto-generated method stub
        int port = 7788;
        Server server = new Server(port);
        server.listen();
    }

    public Server(int port) throws IOException {
        // 打开服务器套接字通道
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        // 服务器配置为非阻塞
        serverSocketChannel.configureBlocking(false);
        // 检索与此通道关联的服务器套接字
        ServerSocket serverSocket = serverSocketChannel.socket();
        // 进行服务的绑定
        serverSocket.bind(new InetSocketAddress(port));
        // 通过open()方法找到Selector
        selector = Selector.open();
        // 注册到selector，等待连接
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
        System.out.println("Server Start----7788:");
    }

    // 监听
    private void listen() throws IOException {
        while (true) {
            // 选择一组键，并且相应的通道已经打开
            selector.select();
            // 返回此选择器的已选择键集。
            Set<SelectionKey> selectionKeys = selector.selectedKeys();
            Iterator<SelectionKey> iterator = selectionKeys.iterator();
            while (iterator.hasNext()) {
                SelectionKey selectionKey = iterator.next();
                iterator.remove();
                handleKey(selectionKey);
            }
        }
    }

    // 处理请求
    private void handleKey(SelectionKey selectionKey) throws IOException {
        // 接受请求
        ServerSocketChannel server = null;
        SocketChannel client = null;
        String receiveText;
        String sendText;
        int count = 0;
        // 测试此键的通道是否已准备好接受新的套接字连接。
        if (selectionKey.isAcceptable()) {
            // 返回为之创建此键的通道。
            server = (ServerSocketChannel) selectionKey.channel();
            // 接受到此通道套接字的连接。
            // 此方法返回的套接字通道（如果有）将处于阻塞模式。
            client = server.accept();
            // 配置为非阻塞
            client.configureBlocking(false);
            // 注册到selector，等待连接
            client.register(selector, SelectionKey.OP_READ);
        } else if (selectionKey.isReadable()) {
            // 返回为之创建此键的通道。
            client = (SocketChannel) selectionKey.channel();
            //将缓冲区清空以备下次读取
            receivebuffer.clear();
            //读取服务器发送来的数据到缓冲区中
            count = client.read(receivebuffer);
            if (count > 0) {
                receiveText = new String(receivebuffer.array(), 0, count);
                System.out.println("服务器端接受客户端数据--:" + receiveText);
                client.register(selector, SelectionKey.OP_WRITE);
            }
        } else if (selectionKey.isWritable()) {
            //将缓冲区清空以备下次写入
            sendbuffer.clear();
            // 返回为之创建此键的通道。
            client = (SocketChannel) selectionKey.channel();
            sendText = "message from server--" + flag++;
            //向缓冲区中输入数据
            sendbuffer.put(sendText.getBytes());
            //将缓冲区各标志复位,因为向里面put了数据标志被改变要想从中读取数据发向服务器,就要复位
            sendbuffer.flip();
            //输出到通道
            client.write(sendbuffer);
            System.out.println("服务器端向客户端发送数据--：" + sendText);
            client.register(selector, SelectionKey.OP_READ);
        }
    }
}
```

客户端：

```Java
public class Client {
    //标识数字/
    private static int flag = 0;
    //缓冲区大小/
    private static int BLOCK = 4096;
    //接受数据缓冲区/
    private static ByteBuffer sendbuffer = ByteBuffer.allocate(BLOCK);
    //发送数据缓冲区/
    private static ByteBuffer receivebuffer = ByteBuffer.allocate(BLOCK);
    //服务器端地址/
    private final static InetSocketAddress SERVER_ADDRESS = new InetSocketAddress(
            "localhost", 7788);
    public static void main(String[] args) throws IOException {
        // TODO Auto-generated method stub
        // 打开socket通道
        SocketChannel socketChannel = SocketChannel.open();
        // 设置为非阻塞方式
        socketChannel.configureBlocking(false);
        // 打开选择器
        Selector selector = Selector.open();
        // 注册连接服务端socket动作
        socketChannel.register(selector, SelectionKey.OP_CONNECT);
        // 连接
        socketChannel.connect(SERVER_ADDRESS);
        // 分配缓冲区大小内存
        Set<SelectionKey> selectionKeys;
        Iterator<SelectionKey> iterator;
        SelectionKey selectionKey;
        SocketChannel client;
        String receiveText;
        String sendText;
        int count = 0;

        while (true) {
            //选择一组键，其相应的通道已为 I/O 操作准备就绪。
            //此方法执行处于阻塞模式的选择操作。
            selector.select();
            //返回此选择器的已选择键集。
            selectionKeys = selector.selectedKeys();
            //System.out.println(selectionKeys.size());
            iterator = selectionKeys.iterator();
            while (iterator.hasNext()) {
                selectionKey = iterator.next();
                if (selectionKey.isConnectable()) {
                    System.out.println("client connect");
                    client = (SocketChannel) selectionKey.channel();
                    // 判断此通道上是否正在进行连接操作。
                    // 完成套接字通道的连接过程。
                    if (client.isConnectionPending()) {
                        client.finishConnect();
                        System.out.println("完成连接!");
                        sendbuffer.clear();
                        sendbuffer.put("Hello,Server".getBytes());
                        sendbuffer.flip();
                        client.write(sendbuffer);
                    }
                    client.register(selector, SelectionKey.OP_READ);
                } else if (selectionKey.isReadable()) {
                    client = (SocketChannel) selectionKey.channel();
                    //将缓冲区清空以备下次读取
                    receivebuffer.clear();
                    //读取服务器发送来的数据到缓冲区中
                    count = client.read(receivebuffer);
                    if (count > 0) {
                        receiveText = new String(receivebuffer.array(), 0, count);
                        System.out.println("客户端接受服务器端数据--:" + receiveText);
                        client.register(selector, SelectionKey.OP_WRITE);
                    }
                } else if (selectionKey.isWritable()) {
                    sendbuffer.clear();
                    client = (SocketChannel) selectionKey.channel();
                    sendText = "message from client--" + (flag++);
                    sendbuffer.put(sendText.getBytes());
                    //将缓冲区各标志复位,因为向里面put了数据标志被改变要想从中读取数据发向服务器,就要复位
                    sendbuffer.flip();
                    client.write(sendbuffer);
                    System.out.println("客户端向服务器端发送数据--：" + sendText);
                    client.register(selector, SelectionKey.OP_READ);
                }
            }
            selectionKeys.clear();
        }
    }
}
```





参照：Java面试通关手册公众号