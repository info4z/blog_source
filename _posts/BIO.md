---
title: BIO阻塞式网络编程
date: 2020-07-03
categories:
- 高性能编程
tags: 
- 高并发网络编程
- NIO网络编程
---

> BIO : Blocking IO, 阻塞式IO



## 一 : 简单 C/S 程序

服务端示例

```java
public class BIOServer {

    public static void main(String[] args) throws Exception {
        ServerSocket serverSocket = new ServerSocket(8080);
        System.out.println("服务器启动成功");
        while (!serverSocket.isClosed()) {
            Socket request = serverSocket.accept();// 阻塞
            System.out.println("收到新连接 : " + request.toString());
            try {
                // 接收数据、打印
                InputStream inputStream = request.getInputStream(); // net + i/o
                BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream, "utf-8"));
                String msg;
                while ((msg = reader.readLine()) != null) { // 没有数据，阻塞
                    if (msg.length() == 0) {
                        break;
                    }
                    System.out.println(msg);
                }
                System.out.println("收到数据,来自："+ request.toString());
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                try {
                    request.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        serverSocket.close();
    }
}
```

客户端示例

```java
public class BIOClient {
    private static Charset charset = Charset.forName("UTF-8");

    public static void main(String[] args) throws Exception {
        Socket s = new Socket("localhost", 8080);
        OutputStream out = s.getOutputStream();

        Scanner scanner = new Scanner(System.in);
        System.out.println("请输入：");
        String msg = scanner.nextLine();
        out.write(msg.getBytes(charset)); // 阻塞，写完成
        scanner.close();
        s.close();
    }
}
```



## 二 : Http 协议

代码示例(和浏览器交互)

```java
// 使用多线程技术
public class BIOServer1 {
    private static ExecutorService threadPool = Executors.newCachedThreadPool();

    public static void main(String[] args) throws Exception {
        ServerSocket serverSocket = new ServerSocket(8080);
        System.out.println("tomcat 服务器启动成功");
        while (!serverSocket.isClosed()) {
            Socket request = serverSocket.accept();
            System.out.println("收到新连接 : " + request.toString());
            threadPool.execute(() -> {
                try {
                    // 接收数据、打印
                    InputStream inputStream = request.getInputStream();
                    BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream, "utf-8"));
                    String msg;
                    while ((msg = reader.readLine()) != null) { // 阻塞
                        if (msg.length() == 0) {
                            break;
                        }
                        System.out.println(msg);
                    }
                    System.out.println("收到数据,来自："+ request.toString());
                } catch (IOException e) {
                    e.printStackTrace();
                } finally {
                    try {
                        request.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            });
        }
        serverSocket.close();
    }
}
```



请求数据包解析

```shell
# 第一部分: 请求行,请求类型,资源路径以及HTTP版本
GET /servlet-demo-1.0.0/index HTTP/1.1
# 第二部分: 请求头部,紧接着请求行(即第一行)之后的部分,用来说明服务器要使用的附加信息
Cache-Control: max-age=O
Accept: text/html
Accept-Language: zh-Hans-CN,zh-Hans;q=0.5
Upgrade-Insecure-Requests: 1
User-Agent: Chrome/64.0.3282.140
Accept-Encoding: gzip, deflate
Host: 127.0.0.1:8080
Connection: Keep-Alive
# 第三部分: 空行,请求头部后面的空行是必须的请求头部和数据主体之间必须有换行

# 第四部分: 请求数据也叫主体,可以添加任意的数据(没有则不显示)
```

响应数据包解析 

```shell
# 第一部分: 状态行。HTTP版本、状态码、状态消息
HTTP/1.1 200 OK
# 第二部分: 响应报头部,紧接着请求行（即第一行)之后的部分,用来说明服务器要使用的附加信息
Content-Length: 11
# 第三部分:空行,头部后面的空行是必须的头部和数据主体之间必须有换行

# 第四部分:响应正文。可以添加任意的数据。例如“Hello World”
Hello World
```

响应状态码

| 状态码 | 类型       | 描述                                                         |
| ------ | ---------- | ------------------------------------------------------------ |
| 1xx    | 临时响应   | 表示临时响应并需要请求者继续执行操作的状态代码               |
| 2xx    | 成功       | 表示成功处理了请求的状态代码                                 |
| 3xx    | 重定向     | 表示要完成请求,需要进一步操作; 通常这些状态代码用来重定向    |
| 4xx    | 请求错误   | 这些状态代码表示请求可能出错,妨碍了服务器的处理              |
| 5xx    | 服务器错误 | 这些状态码表示服务器在尝试处理请求时发生内部错误; 这些错误可能是服务器本身的错误, 而不是请求出错 |

## 三 : 服务端升级版 

代码示例(和浏览器交互, 返回 Http 内容)

```java
public class BIOServer2 {

    private static ExecutorService threadPool = Executors.newCachedThreadPool();

    public static void main(String[] args) throws Exception {
        ServerSocket serverSocket = new ServerSocket(8080);
        System.out.println("服务器启动成功");
        while (!serverSocket.isClosed()) {
            Socket request = serverSocket.accept();
            System.out.println("收到新连接 : " + request.toString());
            threadPool.execute(() -> {
                try {
                    // 接收数据、打印
                    InputStream inputStream = request.getInputStream();
                    BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream, "utf-8"));
                    String msg;
                    while ((msg = reader.readLine()) != null) {
                        if (msg.length() == 0) {
                            break;
                        }
                        System.out.println(msg);
                    }

                    System.out.println("收到数据,来自："+ request.toString());
                    // 响应结果 200
                    OutputStream outputStream = request.getOutputStream();
                    outputStream.write("HTTP/1.1 200 OK\r\n".getBytes());
                    outputStream.write("Content-Length: 11\r\n\r\n".getBytes());
                    outputStream.write("Hello World".getBytes());
                    outputStream.flush();
                } catch (IOException e) {
                    e.printStackTrace();
                } finally {
                    try {
                        request.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            });
        }
        serverSocket.close();
    }
}
```

## 四 : BIO - 阻塞IO的含义

**阻塞(blocking) IO** : 资源不可用时, IO 请求一致阻塞, 直到反馈结果(有数据或超时)

**非阻塞(non-blocking) IO** : 资源不可用时, IO 请求离开返回, 返回数据标识资源不可用

**同步(synchronous) IO** : 应用阻塞在发送或接受数据的状态, 直到数据成功传输或返回失败

**异步(asynchronous) IO** : 应用发送或接受数据后立刻返回, 实际处理时异步执行的

**总结** : 阻塞和非阻塞是<u>获取资源</u>的方式, 同步和异步是程序如何<u>处理资源</u>的逻辑设计; 

- 代码中使用的 API: ServerSocket#accept, InputStream#read 都是阻塞的 API
- 操作系统地层 API 中, 默认 Socket 操作都是 Blocking 型, send/recv 等接口都是阻塞的

**带来的问题** : 阻塞导致在处理网络 I/O 时, 一个线程只能处理一个网络连接