# 参考内容
* [IBM: NIO 入门](https://www.ibm.com/developerworks/cn/education/java/j-nio/j-nio.html)
* [Java NIO Tutorial](http://tutorials.jenkov.com/java-nio/index.html)
* [Java NIO 浅析](https://tech.meituan.com/nio.html)
* [IBM: 深入分析 Java I/O 的工作机制](https://www.ibm.com/developerworks/cn/java/j-lo-javaio/index.html)
* [IBM: 深入分析 Java 中的中文编码问题](https://www.ibm.com/developerworks/cn/java/j-lo-chinesecoding/index.html)
* [IBM: Java 序列化的高级认识](https://www.ibm.com/developerworks/cn/java/j-lo-serial/index.html)
* [NIO 与传统 IO 的区别](http://blog.csdn.net/shimiso/article/details/24990499)
* [Decorator Design Pattern](http://stg-tud.github.io/sedc/Lecture/ws13-14/5.3-Decorator.html#mode=document)
* [Socket Multicast](http://labojava.blogspot.com/2012/12/socket-multicast.html)

# 概览
Java 的 I/O 大概可以分成以下几类：
* 磁盘操作：File
* 字节操作：InputStream 和 OutputStream
* 字符操作：Reader 和 Writer
* 对象操作：Serializable
* 网络操作：Socket
* 新的输入/输出：NIO

# 磁盘操作
File 类可以用于表示文件和目录的信息，但是他不表示文件的内容。   
递归列出一个目录下所有文件：

    public static void listAllFiles (File dir) {
      if (dir == null || !dir.exists()) { return; }
      if (dir.isFile()) {
        System.out.println(dir.getName());
        return;
      }
      for (File file : dir.listFiles()) {
        listAllFiles(file);
      }
    }
1.7 开始可以使用 Paths 和 Files 代替 File


# 字节操作
## 实现文件复制

    public static void copyFile(String src, String dist) throws IOException {
        FileInputStream in = new FileInputStream(src);
        FileOutputStream out = new FileOutputStream(dist);

        byte[] buffer = new byte[20 * 1024];
        int cnt;

        // read() 最多读取 buffer.length 个字节
        // 返回的是实际读取的个数
        // 返回 -1 的时候表示读到 eof，即文件尾
        while ((cnt = in.read(buffer, 0, buffer.length)) != -1) {
            out.write(buffer, 0, cnt);
        }

        in.close();
        out.close();
    }

## 装饰者模式
Java I/O 使用了装饰者模式实现：
* InputStream 是抽象插件
* FileInputStream 是 InputStream 的子类，属于具体组件，提供字节流的输入操作
* FilterInputStream 属于抽象装饰者，装饰者用于装饰组件，为组件提供额外的功能。如 BufferedInputStream 为 FileInputStream 提供缓存功能。

![](assets/JavaIO-04871c05.png)

实例化一个具有缓存功能的字节流对象时，只需要在 FileInputStream 对象上再套一层 BufferedInputStream 即可

    FileInputStream fileInputStream = new FileInputStream(filePath);
    BufferedInputStream BufferedInputStream = new BufferedInputStream(fileInputStream);

DataInputStream 装饰者提供了更多数据类型进行输入操作，例如 int、double 等类型


# 字符操作
## 编码与解码
编码就是把字符转换为字节，而解码是吧字节重新组合成字符     
如果编码和解码过程使用不同编码方式就会出现乱码   
* GBK 中，中文字符占两字节，英文字符占一字节
* UTF-8 中，中文字符占三个字节，英文字符占一个字节
* UTF-16be 中，中文字符和英文字符都占两个字节
UTF-16be 中的 be 是指 Big Endian，也就是大端。也有 UTF-16le，le值得是 Little Endian，也就是小端。   
Java 的内存编码使用两字节编码 UTF-16be，这不是指 Java 只支持一种编码方式，而是指 char 这种类型使用 UTF-16be 进行编码， char 类型占 16 位，两个字节

## String 编码方式
String 可以看成一个字符序列，可以制定一个编码方式将它编码为字节序列，也可以制定一个编码方式将一个字节序列解码为 String

    String str1 = "中文";
    byte[] bytes = str1.getBytes("UTF-8");
    String str2 = new String(bytes, "UTF-8");
    System.out.println(str2);

在调用无参数 getBytes() 方法时，默认编码不是 UTF-16be。双字节编码的好处是可以使用一个 char 存储中文和英文，而将 String 转换成 byte[] 不需要，所以不在需要双字节编码。 getBytes() 默认编码与平台相关，一般为 UTF-8。

    byte[] bytes = str1.getBytes();


## Reader 和 Writer
不管是磁盘还是网络传输，最小的存储单元都是字节，而不是字符。但是程序中操作的通常都是字符形式的数据，因此需要提供对字符进行操作的方法。   
* InputStreamReader 实现从字节流解码成字符流
* OutPutStreamWriter 实现字符流编码成为字节流


## 实现逐行输出文本文件的内容

public static void readFileContent(String filePath) throws IOException {
  FileReader fileReader = new FileReader(filePath);
  BufferedReader bufferedReader = new BufferedReader(fileReader);

  String line;
  while ((line = bufferedReader.readLine()) != null) {
    System.out.println(line);
  }

  bufferedReader.close();
}


# 对象操作
## 序列化
序列化就是将一个对象转换成字节序列，方便存储和传输。
* 序列化：ObjectOutPutStream.writeObject();
* 反序列化：ObjectInputStream.readObject();

不会对静态变量进行序列化，因为序列化只是保存对象的状态，静态变量属于类的状态。

## Serializable
序列化的类实现需要 Serializable 接口，它是一个标准，没有任何方法实现

public static void main(String[] args) throws IOException, ClassNotFoundException {
    A a1 = new A(123, "abc");
    String objectFile = "file/a1";

    ObjectOutPutStream objectOutPutStream = new ObjectOutPutStream(new FileOutputStream(objectFile));
    objectOutPutStream.wirteObject(a1);
    objectOutPutStream.close();

    ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream(objectFile));
    A a2 = (A) objectInputStream.readObject();
    objectInputStream.close);
    System.out.println(a2);
}

private static class A implements Serializable {
  private int x;
  private String y;

  A(int x, String y) {
    this.x = x;
    this.y = y;
  }

  @Override
  public String toString() {
    return "x = " + x + " y = " + y;
  }
}

## transient
transient 关键字可以使一些属性不会被序列化。   
ArrayList 中存储数据的数组 elementData 是用 transient 修饰的，因为这个数组是动态扩展的，并不是所有的空间都被使用，因此不需要所有内容都被序列化。


# 网络操作
Java 中的网络支持：
* InetAddress：用于表示网络上的硬件资源，既 IP 地址；
* URL：统一资源定位符
* Sockets：使用 TCP 协议实现网络通信
* Datagram：使用 UDP 协议实现网络通信

## InetAddress
没有公有的构造函数，只能通过静态方法来创建实例。

    InetAddress.getName(String host);
    InetAddress.getByAddress(byte[] address);

## URL
可以直接从 URL 中读取字节流数据。

    public static void main (String[] args) throws IOException {

      URL url = new URL("http://www.baidu.com");

      /* 字节流 */
      InputStream is = url.openStream();

      /*  字符流  */
      InputStreamReader isr = new InputStreamReader(is,"UTF-8");

      /*  提供缓存  */
      BufferedReader br = new bufferedReader(isr);

      String line;
      while ((line = br.readLine()) != null) {
        System.out.println(line);
      }

      br.close();
    }

## Sockets
* ServerSocket：服务器端类
* Socket：客户端类
* 服务器和客户端通过 InputStream 和 OutputStream 进行输入输出

![](assets/JavaIO-bc2782b1.png)

## Datagram
* DatagramSocket : 通信类
* DatagramPacket ：数据包类


# NIO
1.4 中引入，弥补原来 I/O 的不足，提供高速，面向块的 I/O。

## 流与块
I/O 与 NIO 的区别是数据打包和传输的方式，I/O 以流的方式处理数据，而 NIO 以块的方式处理数据。   
面向流的 I/O 一次处理一个字节数据；一个输入流产生一个字节数据，一个输出流消费一个字节数据。为流式数据创建过滤器非常容易，链接几个过滤器，以便每个过滤器只负责复杂处理机制的一部分，不利的一面是，面向流的 I/O 通常很慢。   
面向块的 I/O 一次处理一个数据块，按快处理数据比按流处理数据要快得多，但是面向块的 I/O 缺少一些面向流的 I/O 所具有的优雅和简单。   

## 通道与缓冲区
### 1. 通道
通道 Channel 是对原 I/O 包中的流的模拟，可以通过它读取和写入数据。    
通道与流的不同在于流只能在一个方向上移动 (一个流必须是 InputStream 或 OutputStream 的子类)，而通道是双向的，可以用于读、写或者同时读写。   
通过包括以下类型：
* FileChannel：从文件中读写数据
* DatagramChannel：通过 UDP 读写网络中数据
* SocketChannel：通过 TCP 读写网络中数据
* ServerSocketChannel：可以监听新进来的 TCP 链接，对每一个新进来的链接都会创建一个 SocketChannel

### 2. 缓冲区
发送给一个通道的所有数据都必须首先放到缓冲区中，同样的，从同道中人读取的任何数据都要首先读到缓冲区中。也就是说，不会直接对通道进行读写数据，而是首先经过缓冲区。    
缓冲区实质上是一个数组，它提供了对数据的结构化访问，还可以跟踪系统的读/写进程。    
缓冲区包括以下类型：
* ByteBuffer
* CharBuffer
* ShortBuffer
* IntBuffer
* LongBuffer
* FloatBuffer
* DoubleBuffer


## 缓冲区状态变量
* capacity：最大容量
* position：当前已经读写的字节数
* limit：还可以读写的字节数

状态变量的改变过程举例：
①. 新建一个 8 字节的缓冲区，此时 position = 0，limit = capacity = 8

![](assets/JavaIO-6f42e5b8.png)

②. 从输入通道读取 5 个字节的数据写入缓冲区，此时 position = 5，limit 不变

![](assets/JavaIO-a2714295.png)

③. 将缓冲区的数据写到输出通道之前，需要先调用 flip() 方法，将 limit 设置为 当前 position， 然后将 position = 0

![](assets/JavaIO-b6b97c4b.png)

④. 从缓冲区中取 4 个字输出到内存中，此时 position = 4

![](assets/JavaIO-f2a2eac1.png)

⑤. 最后需要调用 clear() 方法清空缓冲区，复原 position 和 limit

![](assets/JavaIO-3d0ebf1b.png)


## 文件 NIO 实例
下面是 NIO 快速复制文件实例：

    public static void fastCopy(String src, String dist) throws IOException {
      /*  获得源文件的输入字节流 */
      FileInputStream fin = new FileInputStream(src);
      /*  获取输入字节流的文件通道  */
      FileChannel fcin = fin.getChannel();
      /*  获取目标文件的输出字节流  */
      FileOutputStream fout = new FileOutputStream(dist);
      /*  获取输出字节流的文件通道  */
      FileChannel fcout = fout.getChannel();
      /*  为缓冲区分配 1024 个字节  */
      ByteBuffer buffer = ByteBuffer.allocateDirect(1024);

      while (true) {
        /*  从输入通道中读取数据到缓冲区  */
        int r = fcin.read(buffer);
        /*  read() 返回 -1 表示 EOF  */
        if (r == 1) { break;  }
        /*  切换读写  */
        buffer.flip();
        /*  把缓冲区内容写入输出文件中  */
        fcout.write(buffer);
        /*  清空缓存区  */
        buffer.clear();
      }
    }

## 选择器
NIO 常常被叫非阻塞 IO，主要是因为 NIO 在网络通信中的非阻塞特性被广泛使用。   
NIO 实现了 IO 多路复用中的 Reactor 模型，一个线程 Thread 使用一个选择器 Selector 通过轮询的方式去监听多个通道 Channel 上的事件，从而让一个线程可以处理多个事件。    
通过配置监听的通道 Channel 为非阻塞，那么当 Channel 上的 IO事件未到达时，就不会进入阻塞状态一直等待，而是继续轮询其他 Channel，找到 IO 事件已经到达的 Channel 执行。   
使用一个线程处理多个事件，对于 IO 密集型的应用有很好的性能。    
应该注意的是，只有套接字 Channel 才能配置为非阻塞，而 FileChannel 不能。   

![](assets/JavaIO-6304fda8.png)

### 1. 创建选择器
> Selector selector = Selector.open();

### 2. 将通道注册到选择器上

    ServerSocketChannel ssChannel = ServerSocketChannel.open();
    ssChannel.configureBlocking(false);
    ssChannel.register(selector, SelectionKey.OP_ACCEPT);

通道必须配置为非阻塞模式，否则使用选择器就没有任何意义了。   
将通道注册到选择器上还要指定具体事件，主要有以下几类：
* SelectionKey.OP_CONNECT
* SelectionKey.OP_ACCEPT
* SelectionKey.OP_READ
* SelectionKey.OP_WRITE


    public static final int OP_READ = 1 << 0;
    public static final int OP_WRITE = 1 << 2;
    public static final int OP_CONNECT = 1 << 3;
    public static final int OP_ACCEPT = 1 << 4;

可以看出每个事件可以被当成一个位域，从而组成事件集争叔。例如：

    int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;

### 3. 监听事件
    int num = selector.select();

使用 select() 来监听到达的事件，它会一直阻塞到有一个事件到达.

### 4. 获取到达的事件
    Set<SelevtionKey> keys = selectors.selectedKeys();
    Iterator<SelectionKey> keyIterator = keys.iterator();
    while (keyIterator.hasNext()) {
      SelectionKey key = keyIterator.next();
      if (key.isAcceptable()) {
        //..
      }
      else {
        //..
      }
      keyiteraotr.remove();
    }


### 5. 时间循环
因为一次 select() 调用不能处理完所有的事件，并且服务器端有可能需要一直监听事件，因此服务器端处理事件的代码一般会放在一个死循环内。    

    while (true) {
      int num = selector.select();
      Set<SelectionKey> keys = selector.selectedKeys();
      Iterator<SelectionKey> keyIterator = keys.iterator();
      while (keyIterator.hasNext()) {
        SelectionKey key = keyIterator.next();
        if (key.isAcceptable()) {
          //..
        }
        else {
          //..
        }
        keyIterator.remove();
      }
    }


## 套接字 NIO 实例
    public Class NIOServer {
      public static void main(String[] args) throws IOException {
        Selector selector = Selector.open();
        ServerSocketChannel ssChannel = ServerSocketChannel.open();
        ssChannel.configureBlocking(false);
        ssChannel.register(selector, SelectionKey.OP_ACCEPT);

        ServerSocket serverSocket = ssChannel.socket();
        InetSocketAddress address = new InetSocketAddress("127.0.0.1", 8888);
        serverSocket.bind(address);

        while (true) {
          selector.select();
          Set<SelectionKey> keys = selector.selectedKeys();
          Iterator<SelectionKey> keyIterator = keys.iterator();

          while (keyIterator.hasNext()) {
            SelectionKey key = keyIterator.next();
            if (key.isAcceptable()) {
              ServerSocketChannel ssChannel1 = (ServerSocketChannel) key.channel();
              //为每个新连接创建一个 SocketChannel
              SocketChannel sChannel = ssChannel.accept();
              sChannel.configureBlocking(false);
              //这个新连接主要用于客户端读取数据
              sChannel.register(selector, SelectionKey.OP_READ);
            }
            else if (key.isReadable()) {
              SocketChannel sChannel = (SocketChannel) key.channel();
              System.out.println(readDataFromSocketChannel(scChannel));
              sChannel.close();
            }
            keyIterator.remove();
          }
        }
      }

      private static String readDataFromSocketChannel(SocketChannel sChannel) throws IOException {
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        StringBuilder data = new StringBuilder();
        while (true) {
          buffer.clear();
          int n = sChannel.read(buffer);
          if (n == -1) {
            break;
          }
          buffer.flip();
          int limit = buffer.limit();
          char[] dst = new char[limit];
          for (int i = 0; i < limit; i++) {
            dst[i] = (char) buffer.get(i);
          }
          data.append(dst);
          buffer.clear();
        }
        return data.toString();
      }
    }
.

    public class NIOClient {
      public static void main(String[] args) throws IOException {
        Socket socket = new Socket("127.0.0.1", 8888);
        OutputStream out = socket.getOutputStream();
        String s = "hello world";
        out.write(s.getBytes());
        out.close();
      }
    }

## 内存映射文件
内存映射文件 I/O 是一种读和写文件数据的方法，它可以比常规的基于流或者基于通道的 I/O 快得多。   
向内存映射文件写入可能是危险的，只是改变数组的单个元素这样的简单操作，可能会直接修改磁盘上的文件。修改数据与将数据保存到磁盘是没有分开的。   
下面代码行将文件的前 1024 个字节映射到内存中，map() 方法返回一个 MappedByteBuffer ，它是 ByteBuffer 的子类。因此，可以像使用任何其他 ByteBuffer 一样都是用新映射的缓冲区，操作系统会在需要时负责执行映射。    

    MappedByteBuffer mbb = fc.map(FileChannel.MapMode.READ_WRITE, 0, 1024);

## 对比
NIO 与普通 I/O 的区别主要是以下：
* NIO 是非阻塞的
* NIO 面向块， I/O 面向流
