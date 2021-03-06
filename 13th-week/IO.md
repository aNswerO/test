# 服务端I/O：
+ I/O：在计算机中指Input/Output

+ IOPS（Input/Output Per Second）：即每秒的输入输出量（或读写次数），是衡量磁盘性能的重要指标之一；IPOS指单位时间内系统能处理的I/O请求数量，一般以每秒处理的I/O请求（通常为读或写数据的操作请求）数量为单位

# 一次完整的I/O：
>每次I/O都要经历2个阶段：  
第一步：将数据加载到内核空间的内存（缓冲区）中，等待数据准备完成，消耗时间较长  
第二步：将数据从内核空间的内存中复制到用户空间的进程内存中，消耗时间较短
```
    一次完整的I/O是用户空间的进程数据与内核空间的内核数据的报文的完整交换，但由于内核空间与用户空间是严格隔离的，所以在数据交换过程中不能由处于用户空间的进程直接调用内核空间的内存数据，而是要先将内存空间中的内存数据copy到用户空间的进程内存中  
    简单来说，I/O就是把数据从内核空间中的内存复制到用户空间的进程内存。
```
# 磁盘I/O：
1. 磁盘I/O是指进程向内核发起系统调用，请求磁盘上的某个资源

2. 然后内核通过响应的驱动程序将目标资源加载到内核的内存中
3. 加载完成后把数据从内核空间复制到用户空间的进程内存中
# 系统I/O：
>同步和异步强调的是消息通知机制  
+ 同步（synchronous）：进程发出请求调用后，要等到内核返回响应以后才会继续下一个请求

+ 异步（asynchronous）：进程发出请求调用后，不等内核返回响应，接着处理下一个请求
>阻塞和非阻塞强调的是程序在等待调用结果（消息、返回值）时的状态
+ 阻塞（blocking）：指I/O操作需要彻底完成后才返回到用户空间，即调用结果返回前，调用者被挂起，不能处理下个请求
+ 非阻塞（unblocking）：指I/O操作被调用后立刻返回一个状态值，无需等到IO操作彻底完成，在最终的调用结果返回之前，调用者不会被挂起，可以做其他事
# 网络I/O模型：
## 同步阻塞型IO：
>apache的prefork使用的模型
```
    程序向内核发送I/O请求后一致等待内核响应，如果内核处理请求的IO操作不能立即返回，则进程将一直等待并不再接受新的请求，并由进程轮询查看IO是否完成；完成后进程将结果返回给Client，在IO没有返回期间进程不能接受其他客户的请求，而且进程自己去查看IO是否完成
```
## 同步非阻塞型IO：
```
    程序向内核发送IO请求后一直等待内核响应，如果内核处理请求的IO操作不能立即返回IO结果，进程将不再等待，而且继续处理其他请求，但是仍然需要进程隔一段时间查看一次内核IO是否完成
```
## IO多路复用型：
>apache的prefork是此模式的select，worker是此模式的poll
```
    IO multiplexing就是我们说的select，poll，epoll，也称为event driven IO。  
    select/epoll的好处就在于单个process就可以同时处理多个网络连接的IO。它的基本原理就是select，poll，epoll这个function会不断的轮询所负责的所有socket，当某个socket有数据到达了，就通知用户进程。 当用户进程调用了select，那么整个进程会被block，而同时，kernel会“监视”所有select负责的socket，当任何一个socket中的数据准备好了，select就会返回。这个时候用户进程再调用read操作，将数据从kernel拷贝到用户进程。
```
### 信号驱动型IO：
>apache的event是此模式
```
    用户进程可以通过sigaction系统调用注册一个信号处理程序，然后主程序可以继续向下执行，当有IO操作准备就绪时，由内核通知触发一个SIGIO信号处理程序执行，然后将用户进程所需要的数据从内核空间拷贝到用户空间 此模型的优势在于等待数据报到达期间进程不被阻塞。用户主程序可以继续执行，只要等待来自信号处理函数的通知。  
    程序进程向内核发送IO调用后，不用等内核响应，可以继续接受其他请求，内核收到进程请求后进行的IO若不能立即返回，就由内核等待结果，直到IO完成后内核再通知进程
```
+ 优点：线程在等待数据时没有阻塞，内核直接返回调用接收信号，不影响进程继续处理其他请求，因此可以调高资源的利用率

+ 缺点：信号I/O在大量IO操作时可能会因为信号队列溢出导致无法通知
### 异步（非阻塞）IO：
```
    程序进程向内核发送IO调用后，不用等待内核响应，可以继续接受其他请求，内核调用的IO如果不能立即返回，内核会继续处理其他事物，直到IO完成后将结果通知给内核，内核在将IO完成的结果返回给进程，期间进程可以接受新的请求，内核也可以处理新的事物，因此相互不影响，可以实现较大的同时并实现较高的IO复用
```
