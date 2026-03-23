# n2n-remote-connection
通过n2n 实现内网访问服务器，实现校外电脑连接校内服务器

本方法依赖云服务器作为supernode 实现个人电脑和服务器的连接。

## 整体架构

你的电脑（edge）  <--->  云服务器（supernode）  <--->  其他设备

## 首先在购买的云服务器上安装supernode

* 建议云服务器和工作站服务器的系统一致。

1、开放云服务器端口

默认使用的UDP 端口（7654），需要在云服务器安全组放行UDP 7654, 同时防火墙放行。

a. 获取源码

把本项目下的压缩包 n2n_src.zip 上传至云服务器，解压后，进入n2n_v2.

b. 编译

```bash
mkdir build
cd build
cmake ..
make


注：自己解决依赖包的安装问题

c. 启动supernode
在 build 文件夹下有supernode 程序，启动supernode
</> Bash
./supernode -p 7654

