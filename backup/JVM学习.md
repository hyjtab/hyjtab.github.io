# JVM内存结构

![Image](https://github.com/user-attachments/assets/542fa7e1-2750-4813-8030-0a871105c706)

## 虚拟机参数

-Xmx__ ：虚拟机堆内存大小

-XX:PrintStringTableStatistics ：字符串表详细信息

-XX:+PrintGCDetails：垃圾回收详细信息

-XX:StringTableSize=___: 桶个数

## 字节码

ldc 加载变量（把新的字符串符号变为字符串对象）

astore 存储到对应位置

aload 从对应位置加载

invokevirtual 调用虚函数

new 新建类对象

## 程序计数器寄存器

## 虚拟机栈

各个栈帧

## 堆空间

new出来的变量

字符串常量池

## 方法区

1. 类信息
  
2. 运行时常量池
  
3. 方法字节码
  
4. 编译后代码
  
5. 类加载器
  

## 常量池（存在于.class文件中）

就是一张为虚拟机指令提供的表，为字节码指令提供常量符号

## 运行时常量池

## StringTable

优点：相同字符串只有一个实例

字符串在用到的时候，才会加入StringTable实现上是一个hashTable，不能扩容。

intern方法，可以主动将串池中还没有的字符串放到对象中，返回串池中的对象

1.6中intern版本的返回值永远不会和原变量相同（新的字符串会复制后放入串池）

1.7以后StringTable放到了堆空间

StringTable也会进行垃圾回收

StringTable桶的个数越多，入池操作执行越快

大量数据时，最好先进行入池操作。

## 直接内存

更快，常见于NIO操作时没用于数据缓冲区

读写性能高，回收成本也较高

不受JVM内存回收管理

### NIO

主要有三个部件构成

#### bytebuffer为主的缓冲区（本地的filebuffer）

#### ServersocketChanel为主的通道

#### Selector与SelectionKey

    public class NioServer {
        private int port ;
        private Selector selector;
        private ExecutorService service = Executors.newFixedThreadPool(5);
        
        public NioServer(int port) {
            this.port = port;
        }
    
        public void init() {
            ServerSocketChannel serverSocketChannel = null;
    
            try {
                // Channel 定义
                serverSocketChannel = ServerSocketChannel.open();
                serverSocketChannel.configureBlocking(false);
                serverSocketChannel.bind(new InetSocketAddress(port));
    
                // 创建选择器
                selector = Selector.open();
    
                // 注册到 Selector ，监听 Accept 事件
                serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
    
                System.out.println("Start server");
            }catch (Exception e){
                e.printStackTrace();
            }
    
        }
    
        public void accept(SelectionKey key) {
            try{
                // ServerSocketChannel 监听到了 Accept 事件后的处理过程，从通道中获取 SocketChannel
                ServerSocketChannel serverSocketChannel = (ServerSocketChannel) key.channel();
                SocketChannel socketChannel = serverSocketChannel.accept();
                socketChannel.configureBlocking(false);
    
                // 注册客户端 Channel 的读事件，因为注册的通道对象不一样了，是收到的 Socket 对象
                socketChannel.register(selector, SelectionKey.OP_READ);
    
                System.out.println("Start to process accepted socket.");
    
                // 打印客户端地址
                String clientInfo = socketChannel.socket().getInetAddress().getHostAddress();
                int portInfo = socketChannel.socket().getPort();
                System.out.println("Receive client info "+clientInfo + ",portInfo:"+portInfo);
            }catch (Exception e) {
                e.printStackTrace();
            }
        }
    
        public void start() {
            this.init();
    
            // 轮询 Select 的事件
            while (true) {
                try {
                    int event = selector.select();
    
                    // 轮询到了 完成就绪事件，遍历并分发处理
                    if (event >0) {
                        Iterator<SelectionKey> selectionKeys = selector.selectedKeys().iterator();
    
                        while(selectionKeys.hasNext()) {
                            SelectionKey key = selectionKeys.next();
                            selectionKeys.remove();
    
                            if (key.isAcceptable()) {
                                this.accept(key);
                            } else {
                                if (!key.isReadable()) {
                                    System.out.println("Key is not read able.");
                                    continue;
                                }
    
                                // 把请求数据的通道提交给线程池处理
                                service.submit(new NioServerHandler((SocketChannel)key.channel()));
    
                                // 该 Client 请求提交给客户端后，key.cancel 可以解除监听
                                key.cancel();
                                System.out.println("Submit task and cancel.");
                            }
                        }
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    
        public static void main(String[] args) {
            NioServer server = new NioServer(8800);
            server.start();
        }
    
        /**
         * 线程池任务：接收通道对象，处理数据，而不是接收 Key
         */
        private class NioServerHandler implements Runnable {
    
            SocketChannel socketChannel;
    
            public NioServerHandler(SocketChannel socketChannel) {
                this.socketChannel = socketChannel;
            }
    
            @Override
            public void run() {
                try {
                    ByteBuffer buffer = ByteBuffer.allocate(1024);
                    socketChannel.read(buffer);
    
                    buffer.flip();
                    buffer.clear();
    
                    // 响应数据
                    ByteBuffer outBuffer = ByteBuffer.wrap("ok".getBytes());
                    socketChannel.write(outBuffer);// 将消息回送给客户端
                    System.out.println("接收到 client request :"+ new String (buffer.array()));
                    System.out.println("response finished");
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }
    

    public class NioClient {
        private static final String host = "127.0.0.1";
        private static final int port = 8800;
        private Selector selector;
    
        public static void main(String[] args){
            for (int i=0;i<1;i++) {
                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        System.out.println("Client is started listened ");
                        NioClient client = new NioClient();
                        client.connect(host, port);
                        client.listen();
                        System.out.println("finished start");
                    }
                }).start();
            }
        }
    
        public void connect(String host, int port) {
            try {
                SocketChannel sc = SocketChannel.open();
                sc.configureBlocking(false);
                this.selector = Selector.open();
    
                // 一次注册多个事件，要用 | 来进行操作，而不是执行多次 register ，否则最后一次会覆盖前面的事件的
                sc.register(selector, SelectionKey.OP_CONNECT|SelectionKey.OP_READ);
                sc.connect(new InetSocketAddress(host, port));
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    
        /**
         * 轮询就绪的事件
         */
        public void listen() {
            boolean isClose = false;
            while (!isClose) {
                try {
                    int events = selector.select();
                    if (events > 0) {
    
                        // 迭代器遍历
                        Iterator<SelectionKey> selectionKeys = selector.selectedKeys().iterator();
                        while (selectionKeys.hasNext()) {
                            SelectionKey selectionKey = selectionKeys.next();
                            selectionKeys.remove();
    
                            //连接事件：连接成功后发送数据
                            if (selectionKey.isConnectable()) {
                                SocketChannel socketChannel = (SocketChannel) selectionKey.channel();
                                if (socketChannel.isConnectionPending()) {
                                    socketChannel.finishConnect();
                                }
    
                                // 发送数据给服务器端
                                String data = "Hello this is " + Thread.currentThread().getName();
                                socketChannel.write(ByteBuffer.wrap(data.getBytes()));
                                System.out.println("send data to server "+data);
                            } else if (selectionKey.isReadable()) {
                                // 监听到响应结果，读取响应结果
                                SocketChannel sc = (SocketChannel) selectionKey.channel();
                                ByteBuffer buffer = ByteBuffer.allocate(1024);
                                sc.read(buffer);
                                buffer.flip();
                                buffer.clear();
                                System.out.println("收到服务端的数据："+new String(buffer.array()));
    
                                //  结束请求
                                sc.shutdownOutput();
                                sc.close();
                                System.out.println("sc connected {}"+sc.isConnected());
                                isClose = true;
                                break;
                            }
                        }
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    
    

### 具体回收细节

JVM将directbytebuffer引用变量回收后（成为null），随后ReferenceHandler线程检测并执行变量对应的直接内存所定义的回调函数（操作对象）。

Cleaner:(本身自己就是一个虚引用对象)

create

register

### 虚引用对象

### **入队操作**

* **谁执行的？** 入队操作是由 **Java垃圾回收器（JVM GC）** 执行的。
  
* **如何执行？** 当垃圾回收器确定某个对象不再可达（即符合垃圾回收条件）时：
  
  1. 如果这个对象有一个虚引用（`PhantomReference`）与之关联，并且这个虚引用被注册到了一个 `ReferenceQueue` 中；
  2. 垃圾回收器会在回收该对象之前，将这个虚引用加入到其关联的 `ReferenceQueue` 中。
  
  **入队的目的是通知程序，该对象已经即将被回收。**
  

* * *

### **清除操作**

* **谁执行的？** 清除操作是由 **程序开发者（手动执行）** 进行的。
  
* **如何执行？** 虚引用本身并不会自动清除或者释放对象的资源，程序员需要手动处理。通常的流程是：
  
  1. 程序通过轮询或其他方式从 `ReferenceQueue` 中取出虚引用；
  2. 执行清除操作，例如释放该对象的资源、关闭文件句柄或网络连接等；
  3. 完成后，明确地处理虚引用或者释放相关的资源。