---
title: 网络
icon: material/earth
comments: false
---

## 命令一览

- `[socket_instance] = socket.socket([v4/v6], [TCP/UDP])`: 建立基于IPv4/IPv6的底层实现为TCP/UDP的套接字实例

    ???+ note "笔记"

        `[v4/v6]`: 

        - `socket.AF_INET`: IPv4
        - `socket.AF_INET6`: IPv6

        `[TCP/UDP]`: 

        - `socket.SOCK_STREAM`: TCP
        - `socket.SOCK_DGRAM`: UDP

- `[socket_instance].bind(('[interface_addr]', [port]))`: 绑定端口
- `[socket_instance].listen([max])`: 监听端口, `[max]`为指定等待连接的最大数量
- `[socket_instance].connet(('[host_name/ip]', [port]))`: 建立连接
- `[socket_instance].accept()`: 接受一个新的连接, 返回一个新的套接字和客户端的地址信息
- `[socket_instance].send([data])`: 发送数据
- `[socket_instance].sendto('[data]', [client_addr])`: 发送数据
- `[socket_instance].recv([max])`: 接受数据, 每次最多接受`[max]`字节
- `[socket_instance].recvform([max])`: 接受数据, 返回一个数据和客户端的地址信息
- `[socket_instace].close()`关闭连接

## 套接字

[套接字](https://zh.wikipedia.org/zh-cn/%E7%B6%B2%E8%B7%AF%E6%8F%92%E5%BA%A7)(socket)是一种应用于网络通信的编程接口, 能够让不同的设备通过网络进行数据传输. 

### 为什么使用套接字

- 对底层的抽象: TCP协议底层需要实现三次握手、数据传输、拥塞控制、流量控制、错误检测和重传等等功能. 如果直接使用TCP, 开发者需要自己实现这些复杂的功能, 极大增加了开发难度. 套接字提供了一种抽象的编程接口, 隐藏了底层TCP协议的复杂性. 开发者只需要使用套接字的API进行连接、发送和接受数据, 而不需要关注底层的实现细节. 套接字不仅支持TCP还支持UDP, 也就是说, 套接字还能够提供底层实现为UDP的接口. 
- 平台无关性: 每一个系统都有自己的套接字实现, 但它们都遵循BSD套接字标准. 尽管各个操作系统的套接字实现细节不同, 但是它们提供的API基本都是相同的, 这使得应用程序几乎可以跨平台运行. Python的`socket`模块建立在系统套接字之上, 通过这个模块, 提供对系统底层套接字接口的封装. 并且, Python的`socket`模块提供了更加统一的API, 使得你的代码几乎不用改动就能够跨平台运行.
- 双向通信: 套接字支持双向通信, 允许数据在客户端和服务端之间自由传输.
- 并发处理: 通过多线程和异步编程, 套接字可以处理多个连接, 支持并发通信.

### 套接字类型

- 数据包套接字: 数据报套接字是一种无连套接字接字, 使用用户数据报协议(UDP)传输数据. 每一个数据包都单独寻址和路由. 这导致了接收端接收到的数据可能是乱序的, 有一些数据甚至可能会在传输过程中丢失. 不过得益于数据报套接字并不需要建立并维护一个稳定的连接, 数据报套接字所占用的计算机和系统资源较小. 
- 流套接字: 连接导向式通信套接字, 使用传输控制协议(TCP)、流控制传输协议(SCTP)或者数据拥塞控制协议(DCCP)传输数据. 流套接字提供可靠并且有序的数据传输服务. 在互联网上, 流套接字通常使用TCP实现, 以便应用可以在任何使用TCP/IP协议的网络上运行. 
- 原始套接字: 原始套接字是一种网络套接字. 允许直接发送和接受IP数据包并且不需要任何传输层协议格式. 原始套接字主要用于一些协议的开发, 可以进行比较底层的操作. 

## TCP

Python中的TCP编程基于流套接字. 

### 客户端

???+ example "例子"

    ```py
    import socket

    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect(('www.google.com', 80))
    s.send(b'GET / HTTP/1.1\r\nHost: www.google.com\r\nConnection: close\r\n\r\n')
    buffer = []
    while True:
        d = s.recv(1024)
        if d:
            buffer.append(d)
        else:
            break
    data = b''.join(buffer)
    s.close()

    header, html = data.split(b'\r\n\r\n', 1) # 分离header和content
    print(header.decode('utf-8'))
    with open('sina.html', 'wb') as f:
        f.write(html)
    ```

    接收数据时, 指定`1024`字节为依次最多接收的字节数, 因此, 要在一个循环中反复接收, 知道`recv(1024)`返回空数据, 表示接收完毕, 退出循环.

### 服务端

由于服务端不像客户端那样一般情况下只需要维护一个链接, 所以服务端的处理会复杂一些. 

在服务端有两种套接字: 监听套接字和通信套接字. 监听套接字专门用于监听链接请求, 对于每个请求, 都需要生成一个新的通信套接字用于和客户端的通信. 创建通信套接字的目的是为了区分不同的客户端, 使得服务端可以通过这些独立的套接字和不同的客户端通信, 每个套接字都会维护状态信息, 如序列号, 确认号, 缓冲区等等. 并且, 对于每一个请求, 服务器会创建一个新的线程来处理和这个客户端的通信. 新线程使用新建的通信套接字进行数据传输, 而不影响其他连接的处理.

???+ example "例子"

    ```py
    import socket

    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.bind(('127.0.0.1', 9999))
    s.listen(5)
    print('Warting for connection...')

    def tcplink(sock, addr):
        print('Accept new connection from %s:%s...' % addr)
        sock.send(b'Welcome!')
        while True:
            data = sock.recv(1024)
            time.sleep(1)
            if not data or data.decode('utf-8') == 'exit':
                break
            sock.send(('Hello, %s!' % data.decode('utf-8')).encode('utf-8'))
        sock.close()
        print('Connection from %s:%s closed.' % addr)

    while True:
        sock, addr = s.accept()
        t = threading.Thread(target=tcplink, args=(sock, addr))
        t.start()
    ```

    新建的服务端线程干了这些事: 服务端线程首先向客户端发送了一条欢迎信息, 然后等待客户端数据, 收到后数据, 加上'Hello, [客户端数据]'再发送给客户端, 如果客户端发送了`exit`, 则直接关闭线程(连接).
    
## UDP

Python中的UDP编程基于数据包套接字.

### 客户端

???+ example "例子"

    ```py
    s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    for data in [b'Michael', b'Tracy', b'Sarah']:
        s.sendto(data, ('127.0.0.1', 9999))
        print(s.recv(1024).decode('utf-8'))
    s.close()
    ```

### 服务端

在UDP通信中, 由于没有连接的概念, 所以每个数据包都是独立的, 不需要新建新的套接字来维护连接状态. 这里省略了多线程处理请求, 因为这个例子比较简单, 可以做到瞬时响应.

???+ example "例子"

    ```py
    import socket

    s = socket.socket(socket.AF_INIT, socket.SOCK_DGRAM)
    s.bind(('127.0.0.1', 9999))
    print('Bind UDP on 9999...')
    while True:
        data, addr = s.recvfrom(1024)
        print('Received from %s:%s.' % addr)
        s.sendto(b'Hello, %s!' % data, addr)
    ```

[^1]: 網路插座. (2022). In 维基百科，自由的百科全书. https://zh.wikipedia.org/w/index.php?title=%E7%B6%B2%E8%B7%AF%E6%8F%92%E5%BA%A7&oldid=70060117
[^2]: TCP编程. (n.d.). Retrieved June 16, 2024, from https://www.liaoxuefeng.com/wiki/1016959663602400/1017788916649408
[^3]: UDP编程. (n.d.). Retrieved June 16, 2024, from https://www.liaoxuefeng.com/wiki/1016959663602400/1017790181885952