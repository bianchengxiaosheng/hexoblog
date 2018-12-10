title: C++ Socket
author: gwl
tags: 
- C++
- Socket
categories: 
- C++
date: 2018-11-26 13:30:00
---
### C++Socket
#### 套接字的概念及分类
在网络中，要全局的标识一个参与通信的进程，需要三元组：协议，IP地址以及端口号。要描述两个应用进程之间的端到端的通信关联需要五元组：协议，信源主机IP，信源应用进程端口，信宿主机IP，信宿应用进程端口。为了实现两个应用进程的通信连接，提出了套接字的概念。套接字可以理解为通信连接的一端，将两个套接字连接在一起，可以实现不同进程之间的通信。 
针对不同的通信需求，TCP/IP中提供了三种不同的套接字： 
- 1.流套接字（SOCK_STREAM） 
流套接字用于面向连接，可靠的数据传输服务。它之所以能实现可靠的数据服务，是因为它使用了传输控制协议–TCP。适合传输大量的数据，但是不支持广播和组播。 
- 2.数据报套接字（SOCK_DGRAM） 
数据报套接字提供了一种无连接的服务，通信双方不需要建立任何显示连接，数据可以发送到指定的套接字。数据报套接字使用UDP进行数据传输，支持广播和组播方式。 
- 3.原始套接字（SOCK_RAW） 
原始套接字与标准套接字（上面两个）区别在于：原始套接字可以读写内核没有处理的IP数据报，流套接字只能读写TCP数据报，数据报套接字只能读写UDP数据报。原始套接字的主要目的是避开TCP/IP的处理机制，被传送的数据报可以直接传送给需要他的程序。主要用于编写自定义地层协议的应用程序。
#### WinSock函数

##### 1.WSAStartup函数：
功能：用于初始化WinSock，即检查系统中是否有Windows Sockets的实现库。 
格式：int WSAStartup(WORD wVersionRequest, LPWSADATA lpWSAData)。 
参数：wVersionRequest使用WinSock的最低版本号，lpWSAData是WSADATA指针。 
返回值：函数成功调用返回0，失败时返回非0。 
说明：此函数是应用程序调用的第一个WinSock函数，只有在该函数调用成功后才能调用其他WinSock函数。

##### 2.socket函数：

功能：为应用程序创建套接字。 
格式：SOCKET socket(int af, int type, int protocol)。 
参数：af-套接字使用的协议地址族，如果使用TCP或者UDP，只能使用AF_INET；type-套接字协议类型，如SOCK_STREAM、SOCK_DGRAM；protocol-套接字使用的特定协议，如果不希望特别指定协议类型，则设置为0。 
返回值：函数成功调用后返回一个新的套接字，是一个无符号的整型数据；失败时返回INVALID_SOCKET。 
说明：应用程序在使用套接字通信之前，必须拥有一个套接字。

##### 3.bind函数： 
功能：实现套接字与主机本地IP地址和端口号的绑定。 
格式：int bind(SOCKET s, const struct sockaddr *name, int namelen)。 
参数：s-将要绑定的套接字；name-与指定协议有关的地址结构指针；namelen-name参数的长度。 
返回值：函数成功时返回0；失败时返回SOCKET_ERROR。

##### 4.listen函数： 
功能：设定套接字为监听状态，准备接收由客户机进程发出的连接请求。 
格式：int listen(SOCKET s, int backlog)。 
参数：s-已绑定地址，但还未建立连接的套接字标识符；backlog-指定正在等待连接的最大队列长度。 
返回值：函数成功时返回0；失败时返回SOCKET_ERROR。 
说明：仅适用于面向连接的套接字，且用于服务器进程。

##### 5.connect函数： 
功能：提出与服务器建立连接的请求，如果服务器进程接受请求，则服务器进程与客户机进城之间便建立了一条通信连接。 
格式：int connect(SOCKET s, const struct sockaddr FAR *name, int namelen )。 
参数：s-欲要建立连接的套接字；name-指向通信对方的套接字地址结构指针，表示s欲与其建立连接；namelen-name参数的长度。 
返回值：函数成功时返回0；失败时返回SOCKET_ERROR。 
说明：在客户机进程调用该方法请求建立连接时，将激活建立连接的3次握手，以此来建立一条与服务器进程的TCP连接。如果该函数调用之前没有绑定地址，系统自动绑定本地地址到此套接字。

##### 6.accept函数： 
功能：接受客户机进程调用connect函数发出的连接请求。 
格式：SOCKET accept(SOCKET s, struct sockaddr FAR *addr, int FAR *addrlen)。 
参数：s-处于侦听状态的套接字；addr-指向一个用来存放发出连接请求的客户机进程IP地址信息的地址结构指针；addrlen-addr的长度。 
返回值：调用成功返回一个新的套接字，这个套接字对应于已接受的那个客户机进程的连接，失败时返回INVALID_SOCKET。 
说明：用于面向连接的服务器进程，在IP协议族中只适用于TCP服务器端。

##### 7.shutdown函数： 
功能：关闭套接字读写通道，即停止套接字接受传递的功能。 
格式：int shutdown(SOCKET s, int how)。 
参数：s-套接字；how-描述停止哪些操作。 
how-0：不再接收消息； 
how-1：不再允许发送消息； 
how-2：既不接收消息，也不再发送消息。 
返回值：函数成功时返回0；失败时返回SOCKET_ERROR。 
说明：只是停止套接字的功能，并没有关闭套接字，套接字的资源还没释放。

##### 8.closesocket函数： 
功能：关闭套接字，释放与套接字关联的所有资源。 
格式：int closesocket(SOCKET s)。 
参数：s-将要关闭的套接字。 
返回值：函数成功时返回0；失败时返回SOCKET_ERROR。 
说明：当套接字s的数据缓冲队列中还有未发出的数据时，如果套接字设定为SO_DONTLINGER，则等待数据缓冲队列中的数据继续传输完毕关闭该套接字；如果套接字设定为SO_LINGER，则分以下两种情况： 
（1）Timeout设为0，套接字马上关闭，数据缓冲队列中数据丢失。 
（2）Timeout不为0，等待数据传输完毕或者Timeout为0时关闭套接字。

##### 9.WSACleanup函数： 
功能：终止使用WinSock，释放为应用程序分配的相关资源。 
格式：int WSACleanup()。 
参数：无。 
返回值：调用成功时返回0，失败时返回非0。 
说明：该函数是任何一个基于WinSock应用程序在最后必须调用的函数，终止了所有Windows Sockets在所有线程上的操作。

##### 10.recv函数： 
功能：在已建立连接的套接字上接收数据。 
格式：int recv(SOCKET s, char *buf, int len, int flags)。 
参数：s-已建立连接的套接字；buf-存放接收到的数据的缓冲区指针；len-buf的长度；flags-调用方式： 
（1）0：接收的是正常数据，无特殊行为。 
（2）MSG_PEEK：系统缓冲区数据复制到提供的接收缓冲区，但是系统缓冲区内容并没有删除。 
（3）MSG_OOB：表示处理带外数据。 
返回值：接收成功时返回接收到的数据长度，连接结束时返回0，连接失败时返回SOCKET_ERROR。

##### 11.send函数： 
功能：在已建立连接的套接字上发送数据. 
格式：int send(SOCKET s, const char *buf, int len, int flags)。 
参数：参数：s-已建立连接的套接字；buf-存放将要发送的数据的缓冲区指针；len-发送缓冲区中的字符数；flags-控制数据传输方式： 
（1）0：接收的是正常数据，无特殊行为。 
（2）MSG_DONTROUTE：表示目标主机就在本地网络中，无需路由选择。 
（3）MSG_OOB：表示处理带外数据。 
返回值：发送成功时返回发送的数据长度，连接结束时返回0，连接失败时返回SOCKET_ERROR。

套接字编程流程
![](/images/c++/socket/c-s.jpg)
![](/images/c++/socket/c-c.jpg)
#### socket缓冲区
每一个socket在被创建之后，系统都会给它分配两个缓冲区，即输入缓冲区和输出缓冲区。 
这里写图片描述
![](/images/c++/socket/socketbuffer.jpg)
send函数并不是直接将数据传输到网络中，而是负责将数据写入输出缓冲区，数据从输出缓冲区发送到目标主机是由TCP协议完成的。数据写入到输出缓冲区之后，send函数就可以返回了，数据是否发送出去，是否发送成功，何时到达目标主机，都不由它负责了，而是由协议负责。

recv函数也是一样的，它并不是直接从网络中获取数据，而是从输入缓冲区中读取数据。

输入输出缓冲区，系统会为每个socket都单独分配，并且是在socket创建的时候自动生成的。一般来说，默认的输入输出缓冲区大小为8K。套接字关闭的时候，输出缓冲区的数据不会丢失，会由协议发送到另一方；而输入缓冲区的数据则会丢失。

#### socket数据发送与接收问题
数据的发送和接收是独立的，并不是发送方执行一次send，接收方就执行以此recv。recv函数不管发送几次，都会从输入缓冲区尽可能多的获取数据。如果发送方发送了多次信息，接收方没来得及进行recv，则数据堆积在输入缓冲区中，取数据的时候会都取出来。换句话说，recv并不能判断数据包的结束位置。

send函数： 
在数据进行发送的时候，需要先检查输出缓冲区的可用空间大小，如果可用空间大小小于要发送的数据长度，则send会被阻塞，直到缓冲区中的数据被发送到目标主机，有了足够的空间之后，send函数才会将数据写入输出缓冲区。

TCP协议正在将数据发送到网络上的时候，输出缓冲区会被锁定（生产者消费者问题），不允许写入，send函数会被阻塞，直到数据发送完，输出缓冲区解锁，此时send才能将数据写入到输出缓冲区。

要写入的数据大于输出缓冲区的最大长度的时候，要分多次写入，直到所有数据都被写到缓冲区之后，send函数才会返回。

recv函数： 
函数先检查输入缓冲区，如果输入缓冲区中有数据，读取出缓冲区中的数据，否则的话，recv函数会被阻塞，等待网络上传来数据。如果读取的数据长度小于输出缓冲区中的数据长度，没法一次性将所有数据读出来，需要多次执行recv函数，才能将数据读取完毕。
#### 循环发送和接收
防止send或者 recv不完整，这样你想发一个 
几MB直接调用下面方法就okay，不会少发~
```c++
bool SendAll(SOCKET &sock, char*buffer, int size)
{
    while (size>0)
    {
        int SendSize= send(sock, buffer, size, 0);
        if(SOCKET_ERROR==SendSize)
            return false;
        size = size - SendSize;//用于循环发送且退出功能
        buffer+=SendSize;//用于计算已发buffer的偏移量
    }
    return true;
}

bool RecvAll(SOCKET &sock, char*buffer, int size)
{
    while (size>0)//剩余部分大于0
    {
        int RecvSize= recv(sock, buffer, size, 0);
        if(SOCKET_ERROR==RecvSize)
            return false;
        size = size - RecvSize;
        buffer+=RecvSize;
    }
    return true;
}
```


