# TCP echo正常结束 #
1.
键入EOF，fgets返回一个空指针，str_cli返回。

2.
str_cli返回到客户的main函数时，调用exit终止。

3.
关闭所有打开的描述符xxfd，客户端TCP发送一个FIN给服务器端，服务器端返回一个ACK。此时，服务器端套接字处于CLOSE_WAIT状态，客户端套接字处于FIN_WAIT_2状态。

4.
服务器端TPC接收到FIN时，子进程阻塞于readline调用，readline返回0。导致str_echo函数返回服务器子进程的main。

5.
服务器子进程通过调用exit终止。

6.
子进程中打开的所有描述符关闭，服务器端发送FIN到客户端，接受客户端的ACK，客户端套接字进入TIME_WAIT状态。


#Tcp echo非正常结束（服务器端崩溃）#

1.
在客户端键入文本，由written写入内核，再由客户端TCP作为一个分节发送，客户随后阻塞于readline调用，等待回射的应答。

2.
客户TPC持续重传分节，试图从服务器上接受一个ACK。当客户TCP最后终于放弃时，给客户端进程返回一个错误。此时客户端组摄于readline调用，因此该调用返回一个错误，如果服务器崩溃，无响应，返回ETIMEOUT。如果中间服务器判定目的服务器不可达，返回一个 desstination unreachable ICMP消息，返回EHOSTUNREACH或者ENETUNREACH。

<h2>Tcp echo非正常结束（服务器进程终止）</h2>
由于此时客户端已经调用完毕written,正阻塞于readline。按照程序设计的原意，客户端进程期待收到服务器进程的答复。然而此时由于服务器进程终止，会发送一个FIN给客户TCP，客户进程却不知道已终止。当服务器TCP接收到来自于客户端的数据时，由于先前打开的套接字进程已关闭，会响应一个RST。客户进程并不能看到这个RST，在这种情况下（readline先调用，客户端TCP再接收FIN)，readline立即返回0（EOF）。此时并未预期收到EOF，因此出错信息：<a style="color:red"><big>"sever terminated prematurely"。</big></a>

当然，也存在另一种情况：<b>客户端TCP先接收到FIN，readline再调用</b>，此时readline返回一个<a style="color:red"><big>"connection reset by peer".</big></a>


<h1>I/O复用：select与poll</h1>
<h2>I/O模型</h2>
阻塞式I/O、非阻塞式I/O、I/O复用、信号驱动式I/O、异步I/O。
<h3>阻塞式I/O</h3>
默认情况下，所有套接字都是阻塞式的。<br>
<h3>非阻塞式I/O<h3>
非阻塞式套接字是在通知内核：当所请求的I/O操作非得把本进程投入睡眠才能完成时， 不要把本进程投入睡眠,而是返回一个错误。举个例子，如下图所示，
  ![img](https://github.com/WTFBoy/AnythingForSE/blob/main/1.png).<br>
  前三次调用时recvfrom没有数据返回，因此内核返回一个EWOULDBLOCK错误。第四次内核已经准备好数据报，因此recvfrom成功返回。

当一个应用进程像这样对一个非阻塞描述符循环调用recvfrom时，我们称之为轮询(polling)。往往大量耗费CPU时间。
<h3>I/O复用模型</h3>
借助I/O复用，可以借助select或Poll，阻塞在这两个调用上，而不是阻塞在真正的I/O系统调用上。即有目的的调用。
![img](https://github.com/WTFBoy/AnythingForSE/blob/main/PrimeC6-3.png)
<h3>信号驱动式I/O</h3>
让内核在数据报准备完毕后发送SIGIO信号通知我们，之后再进行recvfrom调用。
<h3>异步I/O</h3>
与信号驱动式类似，区别在于：信号驱动式时由内核通知何时启动一个I/O操作，而异步是由内核通知我们I/O操作何时完成。

#Select#
select允许进程指示内核等古代多个事件中的任何一个发生，并只在有一个或多个事件发生或经历一段时间后才唤醒它。<br>
