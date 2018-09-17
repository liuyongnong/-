# IO和NIO #
![](https://i.imgur.com/p1cVvvA.jpg)  
磁盘操作
---
File类表示文件和目录的信息，但是不表示文件的内容  

字节操作InputStream 和 OutputStream
---


字符操作Writer,Reder
---
编码就是把字符转换为字节，而解码是把字节重新组合成字符。
#### Reader 与 Writer ####
不管是磁盘还是网络传输，最小的存储单元都是字节，而不是字符。但是在程序中操作的通常是字符形式的数据，因此需要提供对字符进行操作的方法。 

- InputStreamReader 实现从字节流解码成字符流；
- OutputStreamWriter实现字符流编码成为字节流。

NIO
---
#### 流与块 ####
通道可以进行双向移动，流只能在一个方向上移动
**通道的类型：**


- FileChannel:从文件中读写数据
- DatagramChannel:通过UDP读写网络中数据
- SocketChannel:通过TCP读写网络中数据
- ServerScoketChannel:可以监听新进来的TCP连接，对每一个新进来的链接都会去创建以恶ScoketChannel

#### 缓存区 ####
发送一个通道的所有数据都必须首先要放到缓冲区中，同样的，从通道中读取的任何数据都要先读到缓冲区中，也就是说，不会直接对通道进行读写数据，而是先经过缓冲区。

缓冲区实际上是一个数组。缓冲区提供了对数组的结构化访问，而且还可以跟进系统的读写进程
缓冲区包括以下类型：

- ByteBUffer
- CharBuffer
- ShortBuffer
- IntBuffer
- LongBuffer
- FloatBuffer
- DoubleBuffer

缓冲区状态变量
---
- capacity:最大容量
- position：当前已读的字节数
- Limit:还可以读的字节数

选择器
---
NIO通常被叫做非组赛IO，主要是因为NIo在网络通信中的非组赛特性被广泛使用

