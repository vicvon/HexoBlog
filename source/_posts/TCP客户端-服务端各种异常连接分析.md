---
title: TCP客户端/服务端各种异常连接分析
date: 2017-10-09 09:57:38
tags:
categories: network
---
### TCP客户端和服务端各种异常连接分析(UNP第5章)
1. accept返回前连接终止
    ```sequence
    Note left of client: connect阻塞
    client->server: SYN
    Note right of server: SYN_RCVD
    server->client: SYN+ACK
    Note left of client: connect 返回
    client->server: ACK
    Note right of server: ESTABLISHED
    client->server: RST
    Note right of server: 调用accept
    ```
    三次握手完成，调用accept前有耗时操作，此时客户端发送RST到服务端，根据不同的实现accept会有不同的返回值。

2. 服务器进程终止
    
    服务器进程终止会关闭所有文件描述符，发送FIN到客户端，客户端收到FIN后会发送ACK，这时四次挥手已经完成一半。等待客户端关闭文件描述符，发送FIN到服务端。

3. 服务器主机崩溃

    
