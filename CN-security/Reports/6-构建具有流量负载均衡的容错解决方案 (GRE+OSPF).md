# 实验 № 6

**«构建具有流量负载均衡的容错解决方案 (GRE+OSPF)»**

## 引言

本场景包含一个容错解决方案的配置示例，其中总部和分支机构受保护子网之间的流量通过多个并行GRE隧道进行负载均衡，每个隧道都由「C-Terra Gateway 4.3」安全网关提供保护。

流量的GRE封装和负载均衡由位于加密网关后方可信区域内的路由器执行。
通过使用国内行业标准GOST和IPsec协议对流量进行加密和隧道传输，确保安全交互。

所有其他连接被允许，但不会通过IPsec进行保护。

在本场景中，将使用证书进行身份验证。将使用由「C-Terra CSP」公司开发的密码库作为密码提供程序。安全网关（或加密网关）为「C-Terra Gateway」版本4.3。

## 基础设施要求

### 1. 设备要求

1.1. C-Terra Gateway 设备必须已完成初始化。
1.2. 作为 host-behind-… 可以使用任何功能类似于个人计算机的设备（webterm-new, Kali Linux CLI, Linux CLI 等）。
1.3. 作为 Router1 可以使用 openwrt, MikroTik CHR 6.49.10 或 FRR。
1.4. 作为 Spoke-Router 和 Hub-Router 使用 FRR。
1.5. 使用以太网交换机作为交换机。

### 2. 网络交互要求

2.1. 必须确保测试台设备之间的IP连通性。

## 交互方案

[此处原PDF对应页面为示意图，在Markdown中通常用文字描述或链接图片代替]

## 基本工作原理

### 1. 设备部署

1.1. 在总部部署：交换机（Int_Switch1_Hub 和 Ext_Switch1_Hub）、C-Terra Gateway 加密网关（Hub1 和 Hub2）、路由器 Hub-Router 和个人计算机（host_behind_hub）。

1.2. 在分支机构部署：C-Terra Gateway 加密网关（Spoke1 和 Spoke2）、交换机（Int_switch1_spoke, Ext_switch1_spoke）、路由器 Spoke-Router 和个人计算机 host_behind_spoke。

1.3. 在非受控网段部署路由器（Router1）和个人计算机 host_behind_router。

### 2. 流量负载均衡和容错

在本场景中，从总部网络流向分支机构的流量在 Hub-Router 路由器上被封装并通过两个GRE隧道（Tunnel1 和 Tunnel2）进行负载均衡。

第一个GRE隧道将使用接口 Lo1 (10.10.100.X) 作为隧道源，使用 Spoke-Router 路由器的 Lo1 接口IP地址 (10.10.1.X) 作为隧道目标。

第二个GRE隧道将使用接口 Lo2 (10.10.100.Y) 作为隧道源，使用 Spoke-Router 路由器的 Lo2 接口IP地址 (10.10.1.Y) 作为隧道目标。

第一个隧道的GRE流量将通过加密网关 Hub1，第二个隧道的流量通过加密网关 Hub2。GRE流量通过静态路由进行路由。

类似的封装和负载均衡也在分支机构的 Spoke-Router 路由器上执行，用于流向总部的流量。

Hub-Router 和 Spoke-Router 路由器将使用动态路由协议 OSPF 交换总部和分支机构受保护子网之间的路由信息。信息交换将通过 GRE 隧道进行。

当所有测试台设备正常工作时，Hub-Router 路由器将有两条到达分支机构子网的路由（通过 GRE 隧道 Tunnel1 和 Tunnel2），且度量相同。如果其中一个加密网关发生故障，通过相应 GRE 隧道的路由将从路由表中删除，流量将转向剩余活跃的 GRE 隧道。

### 3. 互联网连接

在本场景中，使用路由器 Router1 来模拟互联网。

### 4. 安全交互参数

Hub-Router 和 Spoke-Router 路由器之间的 Tunnel1 和 Tunnel2 的 GRE 隧道流量使用 GOST 算法和 IPsec 协议的隧道模式进行保护。

受保护的连接可以由从 Hub-Router 到 Spoke-Router 的 GRE 流量发起，反之亦然。

4.1. IKE 协议参数：
   - 使用数字证书进行身份验证，签名算法 – GOST R 34.10-2012 (256位密钥)；
   - 加密算法 – GOST 28147-89 (256位密钥)；
   - 哈希函数计算算法 – GOST R 34.11-2012 TK26 (256位密钥)；
   - 共享密钥生成算法（类似 Diffie-Hellman 算法）– VKO_GOSTR3410_2012_256 (256位密钥)。

4.2. ESP 协议参数：
   - 加密和完整性保护（完整性检查）的组合算法 – ESP_GOST-4M-IMIT (256位密钥)。

## 测试台配置

| 索引 | 值                 |
| ---- | ------------------ |
| X    | 小组名单中的编号   |
| Y    | 小组名单中的编号+1 |
| Z    | 小组名单中的编号+2 |

### 配置设备 Router1

1. 在网络接口 ether1 上配置 IP 地址 - 172.16.100.1 和掩码 - 255.255.255.0。
2. 在网络接口 ether2 上配置 IP 地址 - 172.16.1.1 和掩码 - 255.255.255.0。
3. 允许 IPsec 流量通过。

### 配置设备 host_behind_hub

1. 在网络接口上配置 IP 地址 - 192.168.100.100 和掩码 - 255.255.255.0。
2. 通过 192.168.100.Z 设置默认路由。
3. 允许接收和发送 ICMP 数据包。

### 配置设备 host_behind_spoke

1. 在网络接口上配置 IP 地址 - 192.168.1.100，掩码 - 255.255.255.0。
2. 通过 192.168.1.1 设置默认路由。
3. 允许接收和发送 ICMP 数据包。

### 配置加密网关 Hub1

1. 初始化设备。
2. 生成私钥并请求加密网关证书。
3. 在 CA 上颁发加密网关证书，并将 CA 和加密网关证书导入 S-Gate 数据库。
4. 在网络接口 ether1 上配置 IP 地址 – 172.16.100.X 和掩码 255.255.255.0。
5. 在网络接口 ether0 上配置 IP 地址 – 192.168.254.X 和掩码 255.255.255.0。
6. 通过其接口 ether1 的 IP 地址配置到 Hub-Router 路由器 Loopback1 接口的路由。
   ```
   Hub1(config)#ip route 10.10.100.X 255.255.255.255 192.168.254.Z
   ```
7. 通过 Router1 设备设置默认路由。
   ```
   Hub1(config)#ip route 0.0.0.0 0.0.0.0 172.16.100.1
   ```
8. 根据之前的实践课程配置 IKE。
9. 配置 IPsec 并将创建的加密映射附加到外部接口 GigabitEthernet0/0。
   9.1. 创建访问控制列表（ACL）以定义需要保护的流量。加密网关 Hub1 将保护 Hub-Router 和 Spoke-Router 路由器之间的第一个 GRE 隧道的流量：
   ```
   Hub1(config)#ip access-list extended IPSEC_GRE_TUNNEL_1
   Hub1(config-ext-nacl)#permit gre host 10.10.100.X host 10.10.1.X
   Hub1(config-ext-nacl)#exit
   Hub1(config)#
   ```
   9.2. 本场景中的 IPsec 对等体是设备 Spoke1。

### 配置加密网关 Spoke1

加密网关 Spoke1 的配置与 Hub1 类似。所有 X 和 Z 的值与配置 Hub1 时相同。

### 配置加密网关 Hub2

加密网关 Hub2 的配置与 Hub1 类似。X 的值更改为 Y。Z 的值保持不变。

### 配置加密网关 Spoke2

加密网关 Spoke2 的配置与 Hub1 类似。X 的值更改为 Y。Z 的值保持不变。

### 配置路由器 Hub-Router

本场景使用 FRR 路由器。动态路由协议将使用 OSPF。

网络参数配置可以通过两种方案进行：

1. 使用 `interfaces.d` 文件夹进行配置
   - 在 `/etc/network/` 路径下创建 `interfaces.d` 文件夹。
   - 在 `interfaces` 文件中添加命令以包含所创建接口的配置路径：
     ```
     source /etc/network/interfaces.d/*
     ```

2. 在 `interfaces` 文件中配置
   - 所有配置按顺序添加到 `/etc/network/interfaces` 文件中。

后续配置假设使用第一种方案。

#### 配置网络参数

1. 在网络接口 ens35 上配置 IP 地址 - 192.168.100.Z 和掩码 – 255.255.255.0。
2. 在网络接口 ens34 上配置 IP 地址 - 192.168.254.Z 和掩码 – 255.255.255.0。
3. 允许 IP 流量通过。

#### 配置 Lo 接口

创建将用作 GRE 隧道源和目标的接口，需要在系统中添加两个 dummy 类型的接口。

1. 创建文件 `/etc/network/interfaces.d/lo1`，内容如下：
   ```
   auto lo1
   iface lo1 inet static
     address 10.10.100.X
     netmask 255.255.255.255
     pre-up ip link add lo1 type dummy
     post-down ip link delete lo1
   ```

2. 创建文件 `/etc/network/interfaces.d/lo2`，内容如下：
   ```
   auto lo2
   iface lo2 inet static
     address 10.10.100.Y
     netmask 255.255.255.255
     pre-up ip link add lo2 type dummy
     post-down ip link delete lo2
   ```

3. 启用接口 lo1 和 lo2：
   ```
   frr:~# ifup lo1
   frr:~# ifup lo2
   ```

4. 确保这些接口在系统中可用并已分配 IP 地址，例如使用 `ip a` 命令查看输出。

#### 配置 GRE 接口

为了将 GRE 隧道添加到系统，需要创建两个模式为 gre 的接口。

在本场景中，第一个 GRE 隧道的隧道接口地址来自子网 100.100.101.0/30，第二个 GRE 隧道来自子网 100.100.102.0/30。

1. 创建文件 `/etc/network/interfaces.d/tunnel1`，内容如下：
   ```
   auto tunnel1
   iface tunnel1 inet static
     address 100.100.101.X
     netmask 255.255.255.252
     pre-up ip tunnel add tunnel1 mode gre local 10.10.100.X remote 10.10.1.X ttl 64
     pre-up ip link set tunnel1 multicast on
     pre-up ip link set tunnel1 mtu 1400
     post-down ip link del tunnel1
   ```

2. 创建文件 `/etc/network/interfaces.d/tunnel2`，内容如下：
   ```
   auto tunnel2
   iface tunnel2 inet static
     address 100.100.102.X
     netmask 255.255.255.252
     pre-up ip tunnel add tunnel2 mode gre local 10.10.100.Y remote 10.10.1.Y ttl 64
     pre-up ip link set tunnel2 multicast on
     pre-up ip link set tunnel2 mtu 1400
     post-down ip link del tunnel2
   ```

3. 启用接口 tunnel1 和 tunnel2：
   ```
   frr:~# ifup tunnel1
   frr:~# ifup tunnel2
   ```

4. 确保这些接口在系统中可用并已分配 IP 地址，例如使用 `ip a` 命令查看输出。

#### 配置静态路由

1. 通过加密网关 Hub1 (192.168.254.X) 设置到 GRE 隧道 Tunnel1 的目标 IP 地址 (10.10.1.X) 的静态路由。
2. 通过加密网关 Hub2 (192.168.254.Y) 设置到 GRE 隧道 Tunnel2 的目标 IP 地址 (10.10.1.Y) 的静态路由。

#### 配置动态路由

##### 配置 FRR

1. 在配置文件 `/etc/frr/daemons` 中，为 `ospfd` 守护进程设置值为 `yes`：
   ```
   ospfd=yes
   ```

2. 重启 FRR 服务：
   ```
   frr:~# service frr restart
   ```

3. 确保 `ospfd` 守护进程正在运行：
   ```
   frr:~# ps -ef | grep "ospfd"
   ```

##### 配置 OSPF

1. 进入 FRR 控制台：
   ```
   frr:~# vtysh
   ```

2. 进入配置模式：
   ```
   Hub-Router# configure terminal
   ```

3. 启用 OSPF 进程：
   ```
   Hub-Router(config)# router ospf
   ```

4. 设置 router-id：
   ```
   Hub-Router(config-router)# ospf router-id 192.168.100.Z
   ```

5. 默认在所有接口上禁用发送 Hello 数据包：
   ```
   Hub-Router(config-router)# passive-interface default
   ```

6. 在接口 tunnel1 和 tunnel2 上启用发送 Hello 数据包：
   ```
   Hub-Router(config-router)# no passive-interface tunnel1
   Hub-Router(config-router)# no passive-interface tunnel2
   ```

7. 为总部子网 (192.168.100.0/24) 和 GRE 隧道子网 (100.100.101.0/30 和 100.100.102.0/30) 启用 OSPF：
   ```
   Hub-Router(config-router)# network 192.168.100.0/24 area 0
   Hub-Router(config-router)# network 100.100.101.0/30 area 0
   Hub-Router(config-router)# network 100.100.102.0/30 area 0
   ```

8. 退出 OSPF 配置模式：
   ```
   Hub-Router(config-router)# exit
   Hub-Router(config)#
   ```

9. 结束配置过程：
   ```
   Hub-Router(config)# end
   ```

10. 保存当前配置（它将写入文件 `/etc/frr/frr.conf`）：
    ```
    Hub-Router# write memory
    ```

11. 退出 FRR 控制台回到 linux 控制台：
    ```
    Hub-Router# exit
    frr:~#
    ```

### 配置路由器 Spoke-Router

Spoke-Router 路由器的配置与 Hub-Router 类似。

## 测试台功能验证

### 验证 IPsec 连接

使用加密网关的 cisco-like 控制台中的 `run sa_mgr show` 命令检查 IPsec 隧道是否存在。注意 IPsec connections 部分中的协议号，编号 47 对应 GRE 协议。

这些 IPsec 隧道应在配置 Hub-Router 和 Spoke-Router 路由器后立即出现，因为受保护连接的建立将由 GRE 流量因 OSPF 协议信息交换开始而触发。

1. 在加密网关 Hub1 上：
   ```
   Hub1#run sa_mgr show
   ISAKMP sessions: 0 initiated, 0 responded
   ISAKMP connections:
   Num Conn-id (Local Addr,Port)-(Remote Addr,Port) State Sent Rcvd
   1 1 (172.16.100.10,500)-(172.16.1.10,500) active 39952 39880
   IPsec connections:
   Num Conn-id (Local Addr,Port)-(Remote Addr,Port) Protocol Action Type Sent Rcvd
   1 1 (10.10.100.1,*)-(10.10.1.1,*) 47 ESP tunn 29608 28808
   ```

2. 在加密网关 Spoke1 上：
   ```
   Spoke1#run sa_mgr show
   ISAKMP sessions: 0 initiated, 0 responded
   ISAKMP connections:
   Num Conn-id (Local Addr,Port)-(Remote Addr,Port) State Sent Rcvd
   1 1 (172.16.1.10,500)-(172.16.100.10,500) active 39880 39952
   IPsec connections:
   Num Conn-id (Local Addr,Port)-(Remote Addr,Port) Protocol Action Type Sent Rcvd
   1 1 (10.10.1.1,*)-(10.10.100.1,*) 47 ESP tunn 28808 29608
   ```

3. 在加密网关 Hub2 上：
   ```
   Hub2#run sa_mgr show
   ISAKMP sessions: 0 initiated, 0 responded
   ISAKMP connections:
   Num Conn-id (Local Addr,Port)-(Remote Addr,Port) State Sent Revd
   1 1 (172.16.100.20,500)-(172.16.1.20,500) active 40252 40180
   IPsec connections:
   Num Conn-id (Local Addr,Port)-(Remote Addr,Port) Protocol Action Type Sent Revd
   1 1 (10.10.100.2,*)-(10.10.1.2,*) 47 ESP tunn 29248 28256
   ```

4. 在加密网关 Spoke2 上：
   ```
   Spoke2#run sa_mgr show
   ISAKMP sessions: 0 initiated, 0 responded
   ISAKMP connections:
   Num Conn-id (Local Addr,Port)-(Remote Addr,Port) State Sent Revd
   1 1 (172.16.1.20,500)-(172.16.100.20,500) active 40180 40252
   IPsec connections:
   Num Conn-id (Local Addr,Port)-(Remote Addr,Port) Protocol Action Type Sent Revd
   1 1 (10.10.1.2,*)-(10.10.100.2,*) 47 ESP tunn 28256 29248
   ```

### 验证路由是否存在

检查 Hub-Router 和 Spoke-Router 路由器上是否出现了到远程子网的路由信息：

1. 在 Hub-Router 设备上，应该出现到分支机构子网的路由：
   ```
   root@Hub-Router:~# ip r
   10.10.1.x via 192.168.254.X dev ens34
   10.10.1.Y via 192.168.254.Y dev ens34
   100.100.101.0/30 dev tunnel1 proto kernel scope link src 100.100.101.X
   100.100.102.0/30 dev tunnel2 proto kernel scope link src 100.100.102.X
   192.168.100.0/24 dev ens35 proto kernel scope link src 192.168.100.Z
   192.168.254.0/24 dev ens34 proto kernel scope link src 192.168.254.Z
   192.168.1.0/24 proto ospf metric 20
       nexthop via 100.100.101.Y dev tunnel1 weight 1
       nexthop via 100.100.102.Y dev tunnel2 weight 1
   ```

2. 在 Spoke-Router 设备上，应该出现到总部子网的路由：
   ```
   root@Spoke-Router:~# ip r
   10.10.100.x via 192.168.253.X dev ens34
   10.10.100.Y via 192.168.253.Y dev ens34
   100.100.101.0/30 dev tunnel1 proto kernel scope link src 100.100.101.Y
   100.100.102.0/30 dev tunnel2 proto kernel scope link src 100.100.102.Y
   192.168.1.0/24 dev ens35 proto kernel scope link src 192.168.1.Z
   192.168.253.0/24 dev ens34 proto kernel scope link src 192.168.253.2
   192.168.100.0/24 proto ospf metric 20
       nexthop via 100.100.101.X dev tunnel1 weight 1
       nexthop via 100.100.102.X dev tunnel2 weight 1
   ```

### 验证总部和分支机构子网之间的网络连接

从 host_behind_hub 设备检查 host_behind_spoke 设备的可达性（反之亦然），通过发送 ICMP 流量。在 host_behind_hub 设备的 linux bash 控制台中执行 ping 命令：

```
root@host_behind_hub:~# ping 192.168.1.100 -c 5
PING 192.168.1.100 (192.168.1.100) 56(84) bytes of data.
64 bytes from 192.168.1.100: icmp_seq=1 ttl=62 time=3.30 ms
64 bytes from 192.168.1.100: icmp_seq=2 ttl=62 time=2.19 ms
64 bytes from 192.168.1.100: icmp_seq=3 ttl=62 time=2.20 ms
64 bytes from 192.168.1.100: icmp_seq=4 ttl=62 time=2.13 ms
64 bytes from 192.168.1.100: icmp_seq=5 ttl=62 time=2.16 ms

--- 192.168.1.100 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 12ms
rtt min/avg/max/mdev = 2.130/2.397/3.303/0.454 ms
```

```
