基于 n2n 实现内网穿透，使校外电脑可以访问校内工作站，构建一个虚拟局域网环境。

本方案依赖云服务器作为 supernode，实现个人电脑与工作站之间的互联。

🌐 整体架构
你的电脑（edge）  <--->  云服务器（supernode）  <--->  工作站（edge）
🖥️ 一、云服务器部署 supernode

建议云服务器与工作站系统保持一致（推荐 Linux）

假设云服务器 IP 为：

5.34.112.8
1️⃣ 开放端口

默认使用 UDP 端口：

7654

需要：

云服务器安全组放行 UDP 7654
本机防火墙放行该端口
2️⃣ 获取源码

将项目中的 n2n_src.zip 上传至云服务器并解压：

unzip n2n_src.zip
cd n2n_v2
3️⃣ 编译
mkdir build
cd build
cmake ..
make

⚠️ 注意：请自行安装所需依赖（如 cmake、gcc、libssl-dev 等）

4️⃣ 启动 supernode
./supernode -l 1234 -k mypassword

参数说明：

-l：监听端口
-k：连接密码

建议后台运行：

nohup ./supernode -l 1234 -k mypassword &
5️⃣ 验证是否启动成功
ps aux | grep supernode

示例输出：

root   1173  0.0  0.0  ...  /usr/local/sbin/supernode -l 1234 -k mypassword
🖥️ 二、工作站配置（edge）

建议使用 Linux 系统（部分新版本如 Ubuntu 24 可能存在兼容问题）

1️⃣ 获取源码
unzip n2n_src.zip
cd n2n_v2
2️⃣ 编译
mkdir build
cd build
cmake ..
make

⚠️ 注意：请自行安装依赖

3️⃣ 启动 edge
./edge -f -d devicename -a 10.0.0.2 -c nan -u 1234 -g 1234 -k mypassword -l 5.34.112.8:1234

参数说明：

参数	含义
-d	虚拟网卡名称
-a	分配的虚拟 IP
-c	网络名称（community）
-k	密钥
-l	supernode 地址

⚠️ 不同设备的 IP 必须唯一

4️⃣ 验证 edge 是否运行
ps aux | grep edge
5️⃣ 检查虚拟网卡
ifconfig

示例：

devicename: flags=4163<UP,RUNNING>  mtu 1400
        inet 10.0.0.2  netmask 255.255.255.0

👉 出现 10.0.0.x 即表示成功加入虚拟网络

💻 三、个人电脑配置
Windows 用户

推荐使用 GUI 工具：

👉 n2n GUI

双击运行
填写与工作站一致的：
community
密码
supernode 地址
Linux 用户

同工作站步骤，运行：

./edge ...

⚠️ IP 不要重复

🔗 四、连接测试

在个人电脑上执行：

ssh username@10.0.0.2

👉 即可访问工作站

🧭 五、无管理员权限解决方案（跳板机）

在以下场景中：

❌ 无管理员权限
❌ 只能访问内网

可使用跳板机方案（Bastion Host）

💡 思路

增加一台中间设备：

位于内网
可访问外网

作为桥梁：

你的电脑 → n2n → 跳板机 → 内网服务器
🛠️ 实现步骤
1️⃣ 准备跳板机
一台闲置电脑
安装 Linux（推荐 Ubuntu）
2️⃣ 运行 edge
sudo ./edge -f -d devicename -a 10.0.0.3 -c nan -u 1234 -g 1234 -k mypassword -l 5.34.112.8:1234
3️⃣ 本地连接跳板机
ssh user@10.0.0.3
4️⃣ 建立 SSH 隧道

推荐工具：

MobaXterm
或原生 SSH
ssh -L 8888:内网IP:端口 user@10.0.0.3

访问：

localhost:8888
⚠️ 注意事项

⚠️ 跳板机必须保持在线
⚠️ IP 不可冲突
⚠️ 建议使用 SSH 密钥认证

📌 总结

本方案实现了：

✅ 校外访问校内资源
✅ 无需公网 IP
✅ 构建虚拟局域网
