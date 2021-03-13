<h1>ps netstat tcpdump</h1>

1. ps

列出系统中运行的进程，包括进程id、命令、CPU使用量、内存使用量。

· ps -a :列出所有运行/激活的进程。

· ps  -aux：显示所有包含其他使用者的行程。

2. netstat

   显示网络状态。

   netstat -a ：显示所有进程网络状态。

   netstat-t : 显示tcp传输协议的连线状态。

   netstat -n: 直接使用IP地址，而不通过服务器。

   netstat -l:显示监控中的进程状态（listening）

3. tcpdump

   倾倒（即显示)网络传输数据。必须root

   -a:将网络和广播地址转换为名称

   





# TCP echo正常结束 #

1. 键入EOF，fgets返回一个空指针，str_cli返回。

2. str_cli返回到客户的main函数时，调用exit终止。

3. 关闭所有打开的描述符xxfd，客户端TCP发送一个FIN给服务器端，服务器端返回一个ACK。此时，服务器端套接字处于CLOSE_WAIT状态，客户端套接字处于FIN_WAIT_2状态。

4. 服务器端TPC接收到FIN时，子进程阻塞于readline调用，readline返回0。导致str_echo函数返回服务器子进程的main。

5. 服务器子进程通过调用exit终止。

6. 子进程中打开的所有描述符关闭，服务器端发送FIN到客户端，接受客户端的ACK，客户端套接字进入TIME_WAIT状态。

#Tcp echo非正常结束（服务器端崩溃）#

1. 在客户端键入文本，由written写入内核，再由客户端TCP作为一个分节发送，客户随后阻塞于readline调用，等待回射的应答。

2. 客户TPC持续重传分节，试图从服务器上接受一个ACK。当客户TCP最后终于放弃时，给客户端进程返回一个错误。此时客户端阻塞于readline调用，因此该调用返回一个错误，如果服务器崩溃，无响应，返回ETIMEOUT。如果中间服务器判定目的服务器不可达，返回一个 desstination unreachable ICMP消息，返回EHOSTUNREACH或者ENETUNREACH。

<h2>Tcp echo非正常结束（服务器进程终止）</h2>
由于此时客户端已经调用完毕written,正阻塞于readline。按照程序设计的原意，客户端进程期待收到服务器进程的答复。然而此时由于服务器进程终止，会发送一个FIN给客户TCP，客户进程却不知道已终止。当服务器TCP接收到来自于客户端的数据时，由于先前打开的套接字进程已关闭，会响应一个RST。客户进程并不能看到这个RST，在这种情况下（readline先调用，客户端TCP再接收FIN)，readline立即返回0（EOF）。此时并未预期收到EOF，因此出错信息：<a style="color:red"><big>"sever terminated prematurely"。</big></a>

当然，也存在另一种情况：<b>客户端TCP先接收到FIN，readline再调用</b>，此时readline返回一个<a style="color:red"><big>"connection reset by peer".</big></a>


<h1>I/O复用：select与poll</h1>
<h2>I/O模型</h2>
阻塞式I/O、非阻塞式I/O、I/O复用、信号驱动式I/O、异步I/O。
<h3>阻塞式I/O</h3>
默认情况下，所有套接字都是阻塞式的。<br>
<h3>非阻塞式I/O<h3>
非阻塞式套接字是在通知内核：当所请求的I/O操作非得把本进程投入睡眠才能完成时， 不要把本进程投入睡眠,而是返回一个错误。举个例子，如下图所示
。<br>
![img](https://github.com/WTFBoy/AnythingForSE/blob/whatever/1.png?raw=true)
前三次调用时recvfrom没有数据返回，因此内核返回一个EWOULDBLOCK错误。第四次内核已经准备好数据报，因此recvfrom成功返回。


当一个应用进程像这样对一个非阻塞描述符循环调用recvfrom时，我们称之为轮询(polling)。往往大量耗费CPU时间。
<h3>I/O复用模型</h3>
借助I/O复用，可以借助select或Poll，阻塞在这两个调用上，而不是阻塞在真正的I/O系统调用上。即有目的的调用。

![img](https://github.com/WTFBoy/AnythingForSE/blob/whatever/PrimeC6-3.png?raw=true)
<h3>信号驱动式I/O</h3>
让内核在数据报准备完毕后发送SIGIO信号通知我们，之后再进行recvfrom调用。
<h3>异步I/O</h3>
与信号驱动式类似，区别在于：信号驱动式时由内核通知何时启动一个I/O操作，而异步是由内核通知我们I/O操作何时完成。

#Select#
select允许进程指示内核等古代多个事件中的任何一个发生，并只在有一个或多个事件发生或经历一段时间后才唤醒它。<br>


<h1 style="color:green">Raw Socket</h1>
<big><b>原始套接字提供普通的TCP、UDP套接字所不提供的以下3种能力：

1. 进程可以读写ICMPv4、IGMPv4与ICMPv6等分组。
2. 进程可以读写内核不处理其协议字段的IPv4数据报。
3. 进程可以使用IP_HDRINCL套接字选项自行构造IPv4首部。

<h3>Raw Socket的创建</h3>
int sockfd;
sockfd = socket(AF _ INET,SOCK _ RAW , *protocol*); 需要Root权限。

开启IP_HDRINCL套接字选项：
const int on =1;<br>
if (setsockopt(sockfd, IPPROTO _ IP, IP _ HDRINCL, &on, sizeof(on)<0) *出错处理*

与IP\_HDRINCL绑定的level：IPPROTO_IP

与IP_TTL绑定的level:IPPROTO_IP

:iphone:  ​ :horse:  :horse_racing:



#Ping 与 traceroute#

1. Ping

ping程序往某个IP地址发送一个ICMP回射请求，该节点则以一个ICMP回射应答响应。IPv4与IPv6都支持这两种ICMP消息。 ICMP规则要求在回射应答中返回来自回射请求的标识符、序列号以及任何可选数据。在回射请求中存放时间戳可以让我们在收到回射应答时计算TTL。

2. Traceroute

traceroute程序使用IPv4的TTL字段或者IPv6的跳限字段以及两种ICMP消息，一开始向目的地发送一个TTL或者跳限为1的UDP数据报。这个数据报导致第一跳路由器返回一个ICMP “time ecxeeded in transmit"传输中超时。接着每递增一次TTL发送一个UDP数据报，逐步确定下一跳路由器。当到达目的地时，目的主机返回一个ICMP"port unreachable"端口不可达。

#部分函数原型Prototype#

1. socket:<br>
    `#include<sys/socket.h>`<br>
    `int socket(int family, int type, int protocol);`

2. bind:<br>
    `#include<sys/socket.h>`<br>
    `int bind(int sockfd, const struct sockaddr *myaddr, socklen_t addrlen);`

3. connect:<br>
    `#include<sys/socket.h>`<br>
    `int connect(int sockfd, const struct sockaddr *myaddr, socklen_t addrlen);`

4. listen:<br>
    ` #include<sys/socket.h>`<br>
    `int listen(int sockfd, int backlog);`

5. accept:<br>

  ```c
  #include<sys/socket.h
  
  int accept(int sockfd, struct sockaddr *myaddr,socklen_t *addrlen);
  /*
  	这里的addrlen 是一个value-result参数，在调用时作为一个value告诉内核结构的大小，在返回时，结构大小作为一个result，告知进程内核在该结构中存储的信息大小。因为调用的时候会修改这个参数的值，所以它既作为输入参数也作为输出参数(具体调用没细看，觉得不好理解所以这样子说）。
  
  */
  ```

  6. close:

  ```c
  #include<sys/socket.h>
  int close(int sockfd);
  ```

  7. getsockname/getpeername:

  ```c
  #include<sys/socket.h>
  int getsockname(int sockfd, struct sockaddr *localaddr, socklen_t *addrlen);
  
  int getpeername(int sockfd, struct sockaddr *peeraddr, socklen_t *addrlen);
  ```

  8. fork 与 exec

     ``` c
     #include<sys/socket.h>
     pid_t fork(void);
     exec函数是6个exec函数的统称。
     ```

     

9. wait与waitpid

   ```c
   #include<sys/wai.h>
   pid_t wait(int *statloc);//statloc指针返回子进程终止状态。

   pid_t waitpid(pid_t pid, int *statloc, int options);//最常用的options是WNOHANG  waitnohang?大概QAQ
   
   //处理多个进程 sig_child
   #include"unp.h"
   void sig_child(int signo)
   {
       pid_t pid;
       int stat;
       while((pid=waitpid(-1,&stat,WNOHANG))>0)
           printf("child %d terminated\n",pid);
       return;
   }
   ```
   
   10. select
   
       ```c
       #include<sys/select.h>
       #include<sys/time.h>
        int select(int maxfdpl, fd_set *readset, fd_set *writeset, fd_set exceptest, const struct timeval *timeout);
       ```
   
   11. setsockopt与getsockopt
   
       ```c
       #include<sys/socket.h>
       int getsockopt(int sockfd, int level, int optname, void *optval, socklen_t *optlen);
       
       int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t *optlen);
       ```
   
       12. recvfrom 与sendto
   
           ```c
           #include<sys/socket.h>
           ssize_t recvfrom(int sockfd, void *buff, size_t nbytes, int flags, struct sockaddr *from, socklen_t *addrlen);
           
           ssize_t sento(int sockfd, void *buff, size_t nbytes, int flags, struct sockaddr *to, socklen_t addrlen);
           ```
   
           13. Raw Socket
           
               ```c
               #include<sys/socket.h>
               int sock(int AF_INET,int Sock_Raw, int protocol);
               ```
           
               





#TCP与UDP并发服务器设计#

1.TCP并发服务器设计<br>创建listenfd，bind绑定端口，用listen函数将套接字转化为监听套接字，然后使用accept函数阻塞进程，当客户端连接请求到来时，用connfd接收accept函数的返回值，然后创建子进程，用pid判断当前是父进程还是子进程，如果是父进程就关闭connfd，如果是子进程就关闭listenfd然后由子进程对该客户服务。

2.UDP并发服务器设计

服务器启动后，等待下一个客户的到来。当一个客户到来时，记下其IP和port，并fork一个子进程，新建立一个socket，bind一个随机端口，然后connect建立与这个客户的连接，在子进程中处理客户的请求，父进程继续循环等待下一个客户的到来。

#大小端判定#

```c
#include"unp.h"

int  main(int argc, char **argv)
{
	union
	{
	  short a;
	  char c[sizeof(short)];
	}un;
	un.s= 0x0102;
	printf("%s:",CPU_VENDOR_OS);
	if (un.c[0]==1&&un.c[1]==2)
	{
	printf("big-endian\n");
	else if (un.c[0]==2&&un.c[1]==1)
	printf("little-endian\n");
	else
	printf("Unknown\n");
	}
	else printf("sizeof(short)= %d\n),sizeof(short));
	exit(0);
}
```



<h2>Tcp srv cli程序</h2>

```c
//srv
#include"unp.h"
 int main(int agrc, char**argv)
 {
     int listenfd, connfd;
     pid_t =childpid;
     socklen_t clilen;
     struct sockaddr_in cliaddr, servaddr;
     void sig_child(int);
     
     listenfd=Socket(AF_INET, SOCK_STREAM, 0);
     
     bzero(&servaddr, sizeof(servaddr));
     servaddr.sin_family=AF_INET;
     servaddr.sin_addr.s_addr= htonl(INADDR_ANY);
     servaddr.sin_port=htons(SERV_PORT);
     
     Bind(listenfd, (SA*)&servaddr, sizeof(servaddr));
     
     Listen(listenfd, LISTENQ);
     
     Signal(SIGCHLD,sig_child);
     
     for(;;)
     {
         clilen=sizeof(cliaddr);
         if((connfd=accept(listenfd,(SA*)&cliaddr,&clilen))<0)
         {
             if(errno==EINTR)
                 continue;
             else 
                 err_sys("accept error\n");
         }
         if((childpid=Fork())==0)
         {
             Close(listenfd);
             str_echo(connfd);
             exit(0);
         }
         Colse(connfd);
	 }
   
 }
```



```c
//建立多个连接的cli
#include"unp.h"

int main(int argc, char**argv)
{
    int i, sockfd[5];//5个连接
    struct sockaddr_in servaddr;
    
    if(argc!=2)
        err_quit("usage:tcpcli<IPaddress\n");
    for(i=0;i<5;i++)
    {
        sockfd[i]=Socket(AF_INET,SOCK_STREAM,0);
        
        bzero(&servaddr,sizeof(servaddr));
        servaddr.sin_family=AF_INET;
        servaddr.sin_port=htons(SERV_PORT);
        Inet_pton(AF_INET,argv[1],&servaddr.sin_addr);
        
        Connect(sockfd[i],(sa*)servaddr, sizeof(servaddr));
    }
    str_cli(stdin, sockfd[0]);
    exit(0);
}
```

```c

//tcp echo 函数
#include"unp.h"
    void str_echo(int sockfd)
    {
        ssize_t n;
        char buf[MAXLINE];
        again:
        while((n=read(sockfd,buf,MAXLINE))>0)
            Writen(sockfd,buf, n);
        if(n<0&&errno==EINTR)
            goto again;
        else if (n<0)
			err_sys("str_echo:read error");        
    }

```

```c
//str_cli
#include"unp.h"
void str_cli(FILE *fp, int sockfd){
    char sendline[MAXLINE],recvline[MAXLINE];
    while(Fgets(sendline, MAXLINE,fp)!=NULL)
    {
        Writen(sockfd,sendline,strlen(sendline));
        
        if(Readline(sockfd,recvline,MAXLINE)==0)
        	err_quit("str_cli:server terminated prematurely");
        Fputs(recvline, stdout);
	}
}
```

<h1>UDP srv cli</h1>

```c
//srv
#include"unp.h"
int main(int argc, char **argv)
{
	int sockfd;
    struct sockaddr_in servaddr, cliaddr;
    
    sockfd=sock(AF_INET,SOCK_DGRAM,0);
    
    bzero(&servaddr, sizeof(servaddr));
    servaddr.sin_family=AF_INET;
    servaddr.sin_addr.s_addr=htonl(INADDR_ANY);
    servaddr.sin_pot=htons(SERV_PORT);/
}
```



```c
//cli
#include"unp.h"
int main(int argc, char **argv)
{
	int sockfd;
    struct sockaddr_in servaddr;
    if(argc!=2)
    {
        err_quit("usage:udpcli<IPaddress>");
    }
    bzero(&servaddr,sizeof(servaddr));
    servaddr.sin_family=AF_INET;
    servaddr.sin_port=htonl(SERV_PORT);
    Inet-pton(AF_INET,argc[1],&servaddr.sin_addr);
    sockfd=sock(AF_INET,SOCK_DGRAM,0);
    dg_cli(stdin,sockfd,(SA*)&servaddr,sizeof(servaddr));
    exit(0);
}
```



```c
//dg_echo
#inclue"unp.h"
void dg_echo(int sockfd, SA* pcliaddr,socklen_t clilen)
{
   int n;
   socklen_t len;
    char mesg[MAXLINE];
    
    for(;;)
    {
        len=clilen;
        n=Recvfrom(sockfd,mesg,MAXLINE,0,pcliaddr,&len);
        
        Sendto(sockfd,mesg,n,0,pcliaddr,len);
    }
}
```

