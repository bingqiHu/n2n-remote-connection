# n2n-remote-connection
通过n2n 实现内网访问服务器，实现校外电脑连接校内工作站

本方法依赖云服务器作为supernode 实现个人电脑和工作站的连接。

## 整体架构

你的电脑（edge）  <--->  云服务器（supernode）  <--->  工作站

## 首先在购买的云服务器上安装supernode

* 建议云服务器和工作站服务器的系统一致。

1、开放云服务器端口

默认使用的UDP 端口（7654），需要在云服务器安全组放行UDP 7654, 同时防火墙放行。

2. 获取源码

把本项目下的压缩包 n2n_src.zip 上传至云服务器，解压后，进入n2n_v2.

3. 编译

```bash
mkdir build
cd build
cmake ..
make
```

注：自己解决依赖包的安装问题

4. 启动supernode

在 build 文件夹下有supernode 程序，启动supernode

```bash
./supernode -l 1234 -k mypassword
```

其中-l 后接端口号，-k 后接连接密码

可以将这个程序在后台一直运行即可，云服务器一般不会无故停止。


5、检查supernode 是否正常启动。

使用ps 查看supernode 进程：

```bash
ps aux | grep supernode
```
如果看到对应的服务在运行即启动成功。出现：

```bash
root   1173  0.0  0.0   5768   312 ?        Ss    2025 447:52 /usr/local/sbin/supernode -l 1234 -k mypassword
```


## 然后在工作站服务器上安装supernode


