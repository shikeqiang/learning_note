# AIO

​	Java7中新增了AsynchronousFileChannel作为nio的一部分。AsynchronousFileChannel使得数据可以进行异步读写。

​	AIO是异步IO的缩写，虽然==NIO在网络操作中，提供了非阻塞的方法，但是NIO的IO行为还是同步的。==对于NIO来说，我们的业务线程是在IO操作准备好时，得到通知，接着就由这个线程自行进行IO操作，IO操作本身是同步的。

​	但是对AIO来说，则更加进了一步，它不是在IO准备好时再通知线程，而是在IO操作已经完成后，再给线程发出通知。因此AIO是不会阻塞的，此时==业务逻辑将变成一个回调函数==，等待IO操作完成后，由系统自动触发。

## **AIO的特点：**

1. **读完了再通知我**

2. **不会加快IO，只是在读完后进行通知**

3. **使用回调函数，进行业务处理**

有AIO的I/O操作两种具体的API：

- **Future 方式；**
- **Callback 方式。**

​	在AIO socket编程中，服务端通道是AsynchronousServerSocketChannel，这个类提供了一个==open()静态工厂，一个bind()方法用于绑定服务端IP地址（还有端口号），另外还提供了accept()用于接收用户连接请求。==在客户端使用的通道是AsynchronousSocketChannel,这个通道处理提供open静态工厂方法外，还提供了read和write方法。

==在AIO编程中，发出一个事件（accept read write等）之后要指定事件处理类（回调函数），AIO中的事件处理类是**CompletionHandler<V,A>**，这个接口定义了如下两个方法，分别在异步操作成功和失败时被回调。==

## 1.创建AsynchronousFileChannel（Creating an AsynchronousFileChannel）

AsynchronousFileChannel的创建可以通过==open()静态方法==：

```Java
Path path = Paths.get("data/test.xml");

AsynchronousFileChannel fileChannel =
    AsynchronousFileChannel.open(path, StandardOpenOption.READ);
```

**open()的第一个参数是一个Path实体，指向我们需要操作的文件。 第二个参数是操作类型。**

上述示例中我们用的是==StandardOpenOption.READ，表示以读的形式操作文件。==

## 2.读取数据（Reading Data）

​	读取AsynchronousFileChannel的数据有两种方式。每种方法都会调用AsynchronousFileChannel的一个read()接口。下面分别看一下这两种写法。

### 2.1通过Future读取数据（Reading Data Via a Future）

第一种方式是调用返回值为Future的read()方法：

```
Future<Integer> operation = fileChannel.read(buffer, 0);
```

​	这种方式中，**read()接受一个ByteBuffer作为第一个参数，数据会被读取到ByteBuffer中。第二个参数是开始读取数据的位置。**

read()方法会立刻返回，即使读操作没有完成。我们可以通过==isDone()==方法检查操作是否完成。

下面是一个略长的示例：

```Java
 AsynchronousFileChannel fileChannel = 
    AsynchronousFileChannel.open(path, StandardOpenOption.READ);

ByteBuffer buffer = ByteBuffer.allocate(1024);
long position = 0;
Future<Integer> operation = fileChannel.read(buffer, position);
while(!operation.isDone());

buffer.flip();
byte[] data = new byte[buffer.limit()];
buffer.get(data);
System.out.println(new String(data));
buffer.clear();
```

​	在这个例子中我们创建了一个AsynchronousFileChannel，然后创建一个ByteBuffer作为参数传给read。**接着我们创建了一个循环来检查是否读取完毕isDone()**。这里的循环操作比较低效，它的意思是我们需要等待读取动作完成。

一旦读取完成后，我们就可以把数据写入ByteBuffer，然后输出。

**Future.get()是同步的，**所以如果不仔细考虑使用场合，使用 Future 方式可能很容易进入完全同步的编程模式，从而使得异步操作成为一个摆设。如果这样，那么原来旧版本的 Socket API 便可以完全胜任，大可不必使用异步 I/O。

### 2.2通过CompletionHandler读取数据（Reading Data Via a CompletionHandler）

另一种方式是==调用接收CompletionHandler作为参数的read()方法==。下面是具体的使用：

```Java
//new一个CompletionHandler内部类
fileChannel.read(buffer, position, buffer, new CompletionHandler<Integer,
ByteBuffer>() {
    @Override
    public void completed(Integer result, ByteBuffer attachment) {
        System.out.println("result = " + result);
        attachment.flip();
        byte[] data = new byte[attachment.limit()];
        attachment.get(data);
        System.out.println(new String(data));
        attachment.clear();
    }
    @Override
    public void failed(Throwable exc, ByteBuffer attachment) {
    }
});
```

​	这里，一旦读取完成，将会触发CompletionHandler的completed()方法，并传入一个Integer和ByteBuffer。前面的整型表示的是读取到的**字节数大小**。第二个ByteBuffer也可以换成其他合适的对象方便数据写入。 **如果读取操作失败了，那么会触发failed()方法。**

## 3.写数据（Writing Data）

和读数据类似某些数据也有两种方式，调动不同的的write()方法，下面分别看介绍这两种方法。

### 3.1 通过Future写数据（Writing Data Via a Future）

通过AsynchronousFileChannel我们可以一步写数据

```Java
Path path = Paths.get("data/test-write.txt");
AsynchronousFileChannel fileChannel = 
    AsynchronousFileChannel.open(path, StandardOpenOption.WRITE);
ByteBuffer buffer = ByteBuffer.allocate(1024);
long position = 0;
buffer.put("test data".getBytes());
buffer.flip();
Future<Integer> operation = fileChannel.write(buffer, position);
buffer.clear();
while(!operation.isDone());
System.out.println("Write done");
```

​	首先把文件已写方式打开，接着创建一个ByteBuffer作为写入数据的目的地。再把数据进入ByteBuffer。最后检查一下是否写入完成。 需要注意的是，这里的文件必须是已经存在的，否者在尝试write数据是会抛出一个java.nio.file.NoSuchFileException.

检查一个文件是否存在可以通过下面的方法：

```
if(!Files.exists(path)){
    Files.createFile(path);
}
```

### 3.2 通过CompletionHandler写数据（Writing Data Via a CompletionHandler）

我们也可以通过CompletionHandler来写数据：

```Java
Path path = Paths.get("data/test-write.txt");
if(!Files.exists(path)){
    Files.createFile(path);
}
AsynchronousFileChannel fileChannel = 
    AsynchronousFileChannel.open(path, StandardOpenOption.WRITE);

ByteBuffer buffer = ByteBuffer.allocate(1024);
long position = 0;

buffer.put("test data".getBytes());
buffer.flip();

fileChannel.write(buffer, position, buffer, new CompletionHandler<Integer, ByteBuffer>() {
    @Override
    public void completed(Integer result, ByteBuffer attachment) {
        System.out.println("bytes written: " + result);
    }

    @Override
    public void failed(Throwable exc, ByteBuffer attachment) {
        System.out.println("Write failed");
        exc.printStackTrace();
    }
});
```

同样当数据吸入完成后completed()会被调用，如果失败了那么failed()会被调用。

## 4.NIO和AIO的区别：

**区别在于AIO是等读写过程完成后再去调用回调函数。**

==NIO是同步非阻塞的；AIO是异步非阻塞的==

由于NIO的读写过程依然在应用线程里完成，所以对于那些读写过程时间长的，NIO就不太适合。

而AIO的读写过程完成后才被通知，所以AIO能够胜任那些重量级，读写过程长的任务。

## 5.例子

### 服务端：

```Java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.AsynchronousServerSocketChannel;
import java.nio.channels.AsynchronousSocketChannel;
import java.nio.channels.CompletionHandler;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Future;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;
 
public class AIOEchoServer {
 
	public final static int PORT = 8001;
	public final static String IP = "127.0.0.1";
 
	
	private AsynchronousServerSocketChannel server = null;
	
	public AIOEchoServer(){
		try {
			//同样是利用工厂方法产生一个通道，异步通道 AsynchronousServerSocketChannel
			server = AsynchronousServerSocketChannel.open().bind(new InetSocketAddress(IP,PORT));
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
	
	//使用这个通道(server)来进行客户端的接收和处理
	public void start(){
		System.out.println("Server listen on "+PORT);
		
		//注册事件和事件完成后的处理器，这个CompletionHandler就是事件完成后的处理器
		server.accept(null,new CompletionHandler<AsynchronousSocketChannel,Object>(){
 
			final ByteBuffer buffer = ByteBuffer.allocate(1024);			
			@Override
			public void completed(AsynchronousSocketChannel result,Object attachment) {				
				System.out.println(Thread.currentThread().getName());
				Future<Integer> writeResult = null;
				
				try{
					buffer.clear();
					result.read(buffer).get(100,TimeUnit.SECONDS);					
					System.out.println("In server: "+ new String(buffer.array()));				
					//将数据写回客户端
					buffer.flip();
					writeResult = result.write(buffer);
				}catch(InterruptedException | ExecutionException | TimeoutException e){
					e.printStackTrace();
				}finally{
					server.accept(null,this);
					try {
						writeResult.get();
						result.close();
					} catch (InterruptedException | ExecutionException e) {
						e.printStackTrace();
					} catch (IOException e) {
						e.printStackTrace();
					}
				}			
			}
 
			@Override
			public void failed(Throwable exc, Object attachment) {
				System.out.println("failed:"+exc);
			}
			
		});
	}
	
	public static void main(String[] args) {
		new AIOEchoServer().start();
		while(true){
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
}
```

### 客户端：

```Java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.AsynchronousSocketChannel;
import java.nio.channels.CompletionHandler;
 
public class AIOClient {
	public static void main(String[] args) throws IOException {		
		final AsynchronousSocketChannel client = AsynchronousSocketChannel.open();	
		InetSocketAddress serverAddress = new InetSocketAddress("127.0.0.1",8001);	
		CompletionHandler<Void, ? super Object> handler = new CompletionHandler<Void,Object>(){
 
			@Override
			public void completed(Void result, Object attachment) {
				client.write(ByteBuffer.wrap("Hello".getBytes()),null, 
						new CompletionHandler<Integer,Object>(){
 
							@Override
							public void completed(Integer result,
									Object attachment) {
								final ByteBuffer buffer = ByteBuffer.allocate(1024);
								client.read(buffer,buffer,new CompletionHandler<Integer,ByteBuffer>(){
 
									@Override
									public void completed(Integer result,
											ByteBuffer attachment) {
										buffer.flip();
										System.out.println(new String(buffer.array()));
										try {
											client.close();
										} catch (IOException e) {
											e.printStackTrace();
										}
									}
 
									@Override
									public void failed(Throwable exc,
											ByteBuffer attachment) {
									}
									
								});
							} 
							@Override
							public void failed(Throwable exc, Object attachment) {
							}					
				});
			}
			@Override
			public void failed(Throwable exc, Object attachment) {
			}			
		};		
		client.connect(serverAddress, null, handler);
		try {
			Thread.sleep(1000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}
```

参照：Java面试通过手册公众号

