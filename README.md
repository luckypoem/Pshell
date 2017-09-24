这是一个 ICMP/IP 隧道管理脚本，从服务器到本地的全部操作，都可以通过这个脚本完成，目前完美支持主流 Linux 发行版（能运行最新版本 Docker 即可）。

#### 可以用来做什么

* 内网穿透（从外网访问内网的主机，比如在家里访问学校内网的资源）
* 绕过认证（绕过一般的网络认证，比如绕过学校网络认证直接上网）
* 网络代理（又双叒叕一个翻墙姿势，比如服务端放在海外就可以翻墙了）

#### 功能

* 支持服务器自动部署并启动，服务端遇到意外可以自动重启。
* 支持本地自动部署并启动，支持 ICMP/IP 双协议隧道。
* 支持断线自动重连。
* 提供直观的监视器，可以实时查看连接状态。
* 支持指定网卡分享 socks5 代理给他人。
* 支持 socks5 转发为 http 代理。
* 支持 TCP-BBR 算法，极大提高网速（需要内核支持）。
* 密码认证。
* 自动更新脚本。

#### 待添加/修复功能

* 支持自动修复 http 代理并允许指定 http 端口。
* 自动启用负载均衡。
* TCP-BBR 算法自动启用。
* 添加 DNS Tunnel 功能。
* **proxy.list 文件最后一行不是空行会执行失败**。

```shell
$ ./Pshell.sh -h
------------------------------------------------------------------------------
   ___ ____ __  __ ____   _____ ____    ____  _          _ _ 
  |_ _/ ___|  \/  |  _ \ / /_ _|  _ \  / ___|| |__   ___| | |
   | | |   | |\/| | |_) / / | || |_) | \___ \| '_ \ / _ \ | |
   | | |___| |  | |  __/ /  | ||  __/   ___) | | | |  __/ | |
  |___\____|_|  |_|_| /_/  |___|_|     |____/|_| |_|\___|_|_|
  Email: i@zuolan.me                 Blog: https://zuolan.me
  一个隧道部署与代理管理的脚本。不加参数直接运行脚本即可连接。
------------------------------------------------------------------------------
  可选参数         -  说明
------------------------------------------------------------------------------
  -f (--fast)      -  快速模式（切换为 IP 协议隧道，速度更快，安全性降低）。
  -m (--monitor)   -  查看代理运行情况。
  -d (--driver)    -  指定网卡（enp3s0|wlp2s0|eth0|wlan0），默认全部。
  -p (--port)      -  选择本地 HTTP 代理端口（默认配置/etc/privoxy/config）。
  -k (--kill)      -  杀死 autossh 和 sshd 进程（当连接长时间中断时使用）。
  -l (--local)     -  安装本地守护容器。
  -s (--server)    -  安装服务器守护进程。
  -u (--update)    -  检测版本以及更新脚本。
  -e (--edit)      -  编辑配置列表。
  -f (--fast)      -  快速模式，网络不限速（实验功能，安全性有待考究）。
  -h (--help)      -  显示帮助信息。详细说明请阅读 README 文件。
```

## 使用说明

#### 第零步、ssh 免密码设置

在本地生成一对密钥（邮箱替换为你的邮箱）：

```shell
ssh-keygen -t rsa -b 4096 -C "i@zuolan.me"
```

把公钥（id_rsa.pub）内容复制粘贴到服务器的 `~/.ssh/authorized_keys` 文件中：

```shell
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

#### 第一步、服务器安装

执行 `sudo ./Pshell.sh --server` 即可自动安装并启动。服务器就一句话。

#### 第二步、填写本地配置文件

现在回到本地，在运行脚本连接之前需要填写配置文件，模板如下。打开 `proxy.list`，然后按照下面的模板填写你的配置。

```shell
节点名称:容器名称:容器端口:Socks5端口:服务器IP:密码:密钥
```

例如：

```shell
广州:gz:8001:10001:123.45.67.89:pass1:~/.ssh/id_rsa.gz
香港:hk:8002:10002:123.45.67.89:pass1:~/.ssh/id_rsa.hk
青岛:qd:8003:10003:123.45.67.89:pass2:~/.ssh/id_rsa.qd
东京:to:8004:10004:123.45.67.89:password:~/.ssh/id_rsa.to
```

填完就可以进行下一步了，但如果你想更详细定义脚本变量可以在脚本头部中设置（不建议）。

#### 第三步、本地电脑安装

执行 `./Pshell.sh --local` 即可自动安装并运行。

#### 第四步、直接运行

使用 `./Pshell.sh` 直接运行脚本，然后你可以使用配置文件中设置的 Socks5 端口连接到外网。设置方法和普通 Socks5 端口使用一样。（例如 Google Chrome 中的插件 SwitchyOmega。）

#### 扩展一、Socks5 转 http

有些软件不支持 Socks5 代理协议，所以提供端口转换功。

使用 `./Pshell.sh -p <port>` 可以指定其中一个 socks5 端口转换为 http 端口（转换后 http 协议代理端口为 8118）。

> 端口转换功能是保存起来的，不需要每次运行都指定它，除非你想重新指定转换的 socks5 端口。

#### 扩展二、快速模式

快速模式实际上是启用了 IP 协议的隧道，相比使用 ICMP 协议的隧道而言，IP 协议的隧道速度更快，但是安全性还有待考究。

```shell
$ ./Pshell.sh -f
```

~~不建议用这个方法来进行下载文件，因为很多网站下载文件时有自己的判定程序，不断重连会导致下载文件下载不完整。~~

> 使用自动重连过程中，所使用节点会一直处于"已经修复"状态，属于正常现象。

#### 扩展二、分享 Socks5 端口

如果你想分享代理给他人用，可以使用 `./Pshell.sh -d <enp3s0>` 参数指定网卡分享 Socks5 端口。

> 常用的网卡有 enp3s0|wlp2s0|eth0|wlan0 这些，使用 `ifconfig` 命令可以查看。

注意一点就是 Privoxy 的 8118 端口默认为仅 localhost 访问，如果需要他人访问，你还需要修改 localhost 为其他地址（例如 0.0.0.0），这样他人可以通过这个 http 端口访问外网。

#### 扩展四、重启 sshd 进程

在使用过程中可能会出现 sshd 进程崩溃的情况，这时候明明没有连接异常但死活连不上。

这个时候你可以使用 `./Pshell.sh -k` 参数来杀死崩溃 sshd 进程并手动执行 `./Pshell.sh` 重新启动 sshd 进程。

#### 最后

使用 `alias` 指定脚本为特定命令即可更加方便启动。

其他功能自己发现（其实也没什么其他功能了），在脚本中可以看到全部可选参数。