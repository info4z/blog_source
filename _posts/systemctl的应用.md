---
title: systemctl的应用
date: 2023-02-24
excerpt: systemctl是一个systemd工具, 主要负责控制systemd系统和服务管理器
categories: 服务器
tags: shell
---



## 一 : 概述

systemctl 是 linux 系统继 init.d 之后的一个 systemd 工具, 主要负责控制 systemd 系统和管理系统服务。

systemd 即为 system daemon(系统守护进程), 是 linux 下的一种 init 软件。

Systemd 可以管理所有系统资源, 将**系统资源**划分为12类; 将每个系统资源称为一个 **Unit**; 12类包括 : Service、Target、Device等, 其中 `.service` 是最常见的单元文件。

以 nginx.service 文件为例

```shell
[Unit]
Description=The nginx HTTP and reverse proxy server
After=network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
# Nginx will fail to start if /run/nginx.pid already exists but has the wrong
# SELinux context. This might happen when running `nginx -t` from the cmdline.
# https://bugzilla.redhat.com/show_bug.cgi?id=1268621
ExecStartPre=/usr/bin/rm -f /run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/usr/sbin/nginx -s reload
KillSignal=SIGQUIT
TimeoutStopSec=5
KillMode=process
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

Unit 是 Systemd 管理系统资源的基本单位。使用一个 Unit File 作为 Unit 的单元文件, Systemd 通过单元文件控制 Unit 的启动。



## 二 : 常用命令

systemctl 可以管理服务, 也就是说可以启动服务, 查看服务状态, 停止服务, 重启服务等等

```shell
# 显示状态单元
systemctl status xxx.service  
# 启动服务单元
systemctl start xxx.service
# 停止服务单元
systemctl stop xxx.service
# 重启服务单元
systemctl restart xxx.service
# 设置开机自启
systemctl enable xxx.service
# 取消开机自启
systemctl disable xxx.service
# Reload systemd manager configuration(重新载入systemd),扫描新的或有变动的单元
systemctl daemon-reload
```



## 三 : service文件

有时我们将自定义程序注册为systemd service(服务进程管理), 交由系统管理, 可以方便启动停止, 也可以实现服务异常退出重启, 开机自启动, 减少自定义程序服务管理的时间消耗。

服务分为系统服务和用户服务, 系统服务开机不登陆就能运行, 常用于开机自启。

* **系统服务目录 :** `/usr/lib/systemd/system/`
* **用户服务目录 :** `/usr/lib/systemd/user/`

每一个服务以 .service 结尾, 一般会分为3部分: [Unit]、[Service] 和 [Install]

### (一) Unit

所有 Unit 文件通用, 该部分主要是对这个服务的说明, 以及配置与其他服务的关系

| 参数            | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| **Description** | 一段描述这个Unit文件的文字, 通常只是简短的一句话             |
| Documentation   | 指定服务单元的文档, 可以是一个或多个文档的URL路径            |
| Requires        | 依赖的Unit列表,  该Unit会在目标服务单元启动的同时被启动, 如果有任意一个启动失败, 目标单元也会被终止 |
| **After**       | 当列出的所有单元启动完成后, 才会启动目标服务单元             |
| Before          | 在启动指定的每个单元之前, 都会首先确保目标服务单元已经运行, 与After相反 |
| Wants           | 目标Unit启动的同时, 启动列出的每个Unit, 不会关注启动是否成功 |
| Conflicts       | 与目标单元有冲突的单元, 如果列出单元中有已经在运行的, 那么目标单元就不能启动 |
| OnFailure       | 当目标单元启动失败时, 就自动启动列出的每个单元               |
| PartOf          | 该参数仅作用于单元的停止或重启; 当停止或重启这里列出的某个单元时,  也会同时停止或重启目标单元 |
| @               | 动态启动方式配置, @之后加载具体配置文件的名字                |

重要说明 : 

1. `Before=`, `After=` 是配置服务间的启动顺序, 比如一个 a.service 包含了一行 `Before=b.service`, 那么当他们同时启动时, b.service 会等待 a.service 启动完成后才启动; 注意这个设置和 `Requires=` 的相互独立的, 同时包含 `After=` 和 `Requires=` 也是常见的; 此选项可以指定多次, 然后按顺序全部启动。
2. `PartOf=`, 这个依赖是单向的,  该单元自身的停止或重启并不影响这里列出的单元; 如果 a.service 中包含了 `PartOf=b.service` , 那么这个依赖关系将在 b.service 的属性列表中显示为 `ConsistsOf=a.service`; 也就是说, 不能直接设置 `ConsistsOf=` 依赖。
3. `@`, 这个参数不是写在配置文件内容中的, 而是写在配置文件名字上的, 例如我们启动openvpn服务会编写 `openvpn@.service` 文件, 客户端的配置文件为 `/etc/openvpn/client.conf`, 服务端配置文件为 `/etc/openvpn/server.conf`, 对应启动时则应该使用 `openvpn@client.service` 和 `openvpn@server.service`, 同时在内容中使用 `%i` 对 `@` 之后的内容进行接收。

### (二) Service

Service 段是服务(Service)类型的 Unit 文件(后缀为 .service)特有的, 用于定义服务的具体管理和执行动作

| 参数             | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| **Type**         | 设置进程的启动类型, 必须设为 simple, exec, forking, oneshot, dbus, notify, idle 之一, 默认的 simple 类型可以适应于绝大多数的场景, 因此一般可以忽略这个参数的配置; 而如果服务程序启动后会通过 fork 系统调用创建子进程, 然后关闭应用程序本身进程的情况, 则应该将 Type 的值设置为 forking, 否则 systemd 将不会跟踪子进程的行为, 而认为服务已经退出 |
| Environment      | 为服务添加环境变量                                           |
| EnvironmentFile  | 指定加载一个包含服务所需的环境参数的文件, 文件中的每一行都是一个环境变量的定义 |
| **ExecStart**    | 这个参数是几乎每个 .service 文件都会有的, 指定服务启动的主要命令, 在每个配置文件中只能使用一次; **需要使用绝对路径** |
| ExecStartPre     | 指定在启动执行 ExecStart 的命令前的准备工作, 可以有多个, 所有命令会按照文件中书写的顺序依次被执行 |
| ExecStartPost    | 指定在启动执行 ExecStart 的命令后的收尾工作, 也可以有多个    |
| **ExecStop**     | 停止服务所需要执行的主要命令; **需要使用绝对路径**           |
| ExecStopPost     | 指定在 ExecStop 命令执行后的收尾工作, 也可以有多个           |
| **ExecReload**   | 重新加载服务文件所需执行的主要命令; **需要使用绝对路径**     |
| **Restart**      | 指定在什么情况下需要重启服务进程                             |
| RestartSec       | 如果服务需要被重启, 这个参数的值为服务被重启前的等待秒数。注意, 该重启等待时间只针对上面Restart的参数值起作用时的重启才有效, 比如说: 因Unit段配置的关系或者人为使用 `systemctl restart` 命令导致该服务重启时, 该参数无效, 会马上重启 |
| Nice             | 服务的进程优先级, 值越小优先级越高, 默认为0; -20为最高优先级, 19为最低优先级。 |
| WorkingDirectory | 指定服务的工作目录。                                         |
| RootDirectory    | 指定服务进程的根目录( / 目录), 如果配置了这个参数后, 服务将无法访问指定目录以外的任何文件。 |
| **User**         | 指定运行服务的用户, 会影响服务对本地文件系统的访问权限。可使用root |
| Group            | 指定运行服务的用户组, 会影响服务对本地文件系统的访问权限。   |
| PrivateTmp       | 是否给服务分配独立的临时空间(true/false)                     |

**Restart** 的取值分别表示了在哪些情况下, 服务会被重新启动

| 参数        | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| no          | **默认值**, 退出后不会重启                                   |
| **always**  | 除了用 systemctl stop 或等价的服务停止操作命令, 其他情况都可以重启 |
| on-success  | 只有正常退出时(退出状态码为0), 才会重启                      |
| on-failure  | 非正常退出时(退出状态码非0), 包括被信号终止和超时, 才会重启(守护进程, 推荐值) |
| on-abnormal | 只有被信号终止和超时, 才会重启(对于允许发生错误退出的服务, 推荐值) |
| on-abort    | 只有在收到没有捕捉到的信号终止时, 才会重启                   |
| on-watchdog | 超时退出, 才会重启                                           |

具体情况如下

| 参数        | 正常退出 | 退出码不为0 | 进程被强制杀死 | systemd 操作 | 看门狗超时 |
| ----------- | -------- | ----------- | -------------- | ------------ | ---------- |
| no          |          |             |                |              |            |
| always      | 重启     | 重启        | 重启           | 重启         | 重启       |
| on-success  | 重启     |             |                |              |            |
| on-failure  |          | 重启        | 重启           | 重启         | 重启       |
| on-abnormal |          |             | 重启           | 重启         | 重启       |
| on-abort    |          |             | 重启           |              |            |
| on-watchdog |          |             |                |              | 重启       |

### (三) Install

Install段是服务的安装信息, 它不在 systemd 的运行期间使用, 只在使用 `systemctl enable` 和 `systemctl disable` 命令启用/禁用服务时有用, 所有 Unit 文件通用, 用来定义如何启动以及是否开机启动

| 参数         | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| **WantedBy** | 它的值是一个或多个target, 执行enable命令时, 符号链接会放入 `/etc/systemd/system` 目录下以 `target 名 + .wants` 后缀构成的子目录中。`WantedBy=multi-user.target` 表明当系统以多用户方式（默认的运行级别）启动时, 这个服务需要被自动运行。当然还需要 **systemctl enable** 激活这个服务以后自动运行才会生效 |
| RequiredBy   | 依赖当前服务的模块。它的值是一个或多个 target, 执行enable命令时, 符号链接会放入/etc/systemd/system目录下以 target 名 + .required后缀构成的子目录中 |
| Alias        | 当前 Unit 可用于启动的别名                                   |
| Also         | 当前 Unit 被 enable/disable 时, 会被同时操作的其他 Unit      |



