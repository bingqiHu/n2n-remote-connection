# n2n-remote-connection
通过n2n 实现内网访问服务器，实现校外电脑连接校内工作站

本方法依赖云服务器作为supernode 实现个人电脑和工作站的连接。

## 整体架构

你的电脑（edge）  <--->  云服务器（supernode）  <--->  工作站

## 首先在购买的云服务器上安装supernode

* 建议云服务器和工作站服务器的系统一致。

假设：云服务器ip为：5.34.112.8

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
如果看到对应的服务在运行即启动成功。即出现：

```bash
root   1173  0.0  0.0   5768   312 ?        Ss    2025 447:52 /usr/local/sbin/supernode -l 1234 -k mypassword
```


## 然后在工作站服务器上安装supernode

* 建议工作站使用linux 系统，注：本人用过 Ubuntu 24版本，发现其不支持n2n 连接的UDP协议。

1. 获取源码

把本项目下的压缩包 n2n_src.zip 上传至云服务器，解压后，进入n2n_v2.

2. 编译

```bash
mkdir build
cd build
cmake ..
make
```

注：自己解决依赖包的安装问题

3. 启动edge

在 build 文件夹下有edge 程序，启动edge

```bash
用法： ./edge -f -d devicename -a 10.0.0.x -c nan -u 1234 -g 1234 -k mypassword -l 云服务器地址：端口号
./edge -f -d devicename -a 10.0.0.2 -c nan -u 1234 -g 1234 -k mypassword -l 5.34.112.8: 1234
```

其中：

      -d 设置虚拟网卡名称，可以自定义

      -a 是分配的ip, x可以自定义. 不同的设备不要使用相同的ip

      -c, -u, -g 等可以根据 ./edge -h 获取用法帮助


4.检查edge 是否正常启动。

使用ps 查看edge 进程：

```bash
ps aux | grep edge
```
如果看到对应的服务在运行即启动成功。即出现：

```bash
root 1173  0.0  0.0 5768 312 ?  Ss    2025 447:52 /usr/local/sbin/edge -f -d devicename -a 10.0.0.2 -c nan -u 1234 -g 1234 -k mypassword -l 5.34.112.8: 1234
```

5、通过ifconfig 查看是否形成新的虚拟内网ip

```bash
ifconfig
```

出现类似这样的结果：

```bash
devicename: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1400
        inet 10.0.0.2  netmask 255.255.255.0  broadcast 10.0.0.255
        inet6 fe83::9c14:deff:fe01:8001  prefixlen 64  scopeid 0x20<link>
        ether 4e:12:de:19:60:19  txqueuelen 1000  (Ethernet)
        RX packets 788788  bytes 291261911 (291.2 MB)
        RX errors 0  dropped 7  overruns 0  frame 0
        TX packets 639357  bytes 141420988 (141.4 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

即安装好了。

## 然后在个人上安装edge

1. 如果个人电脑是windows 系统，之间双击N2NGUI 界面后自行配置即可。

注： 对应配置参数值和工作站保持一致


2. 如果个人电脑是linux， 安装方法仿照工作站配置，注意ip 不要相同即可。



## 互联

个人电脑上通过执行：

```bash
ssh -p 22 username@10.0.0.2
```

即可连接。


### 无管理员权限 / 仅内网环境的解决方案（跳板机方案）

在某些场景下（如学校或公司环境）：

无法获取管理员权限（无法安装软件或创建虚拟网卡）
工作站仅能访问内网，无法直接连接外部网络

此时可以采用跳板机（Bastion Host）方案来实现间接访问。


在内网环境中准备一台“中转设备”：

位于同一内网（可访问目标资源）
同时具备外网访问能力

这台设备作为“桥梁”，连接你的本地设备与内网资源。

具体步骤：

a. 准备一台跳板机

可以是学校/公司内一台闲置电脑
性能要求不高（普通办公机即可）
建议安装 Linux 系统（如 Ubuntu）

b.在跳板机上部署 n2n 客户端（edge）

使用 n2n 的 edge 连接到你的虚拟网络：
```bash
sudo ./edge  -f -d devicename -a 10.0.0.3 -c nan -u 1234 -g 1234 -k mypassword -l 5.34.112.8: 1234
```

注意ip 不要一致

此时：

跳板机同时连接了「内网 + n2n 虚拟网络」

c. 本地设备连接同一 n2n 网络

你的个人电脑（家里）也连接到同一个 n2n 网络：

这样你就可以访问跳板机的虚拟 IP，例如：

ssh user@10.0.0.10

d. 通过 SSH 隧道访问内网资源

可以使用工具：

MobaXterm（Windows 推荐）
或原生 SSH

建立端口转发，例如：

```bash
ssh -L 8888:内网目标IP:端口 user@10.0.0.10
```

即可访问内网服务.
