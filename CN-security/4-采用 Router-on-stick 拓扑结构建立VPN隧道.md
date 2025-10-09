# 基于Router-on-stick拓扑的《С-Терра》安全网关间VPN隧道构建

本场景描述了总部与分支机构受保护子网之间的安全交互配置。总部加密网关采用Router-on-a-stick拓扑。通过采用国产行业标准GOST和IPsec协议进行流量加密和隧道化，实现安全交互。

所有其他连接被允许，但不会通过IPsec进行保护。

在此方案中，将使用证书进行身份验证。S-Terra CSP 开发的加密库将用作加密提供程序。安全网关（或加密网关）将使用 S-Terra Gateway 4.3 版本。

## 基础设施要求

### 设备要求

1.1. 《С-Терра》设备必须已完成初始化  
1.2. 作为 host-behind-… 可以使用任何功能类似个人计算机的设备（webterm-new, Kali Linux CLI, Linux CLI 等）  
1.3. 作为 Router1 和 Hub1_Router 可以使用 openwrt, MikroTik CHR 6.49.10 或 FRR  

### 网络交互要求

2.1. 实验台设备之间必须确保IP连通性

## 交互方案

(./images/fig04.png)

## 总体工作逻辑

### 设备部署

1.1. 在总部部署：加密网关《С-Терра》（Hub1）、路由器（Hub1_Router）和个人计算机（host_behind_hub1）  
1.2. 在分支机构部署：加密网关《С-Терра》（Spoke1）和个人计算机（host_behind_spoke1）  
1.3. 在非受控段部署：路由器（Router1）和个人计算机（host_behind_router）  

### 互联网连接

在本场景中，使用路由器Router1模拟互联网  
如果通过ICMP协议（或"ping"）能够访问host_behind_router，则认为《С-Терра》设备的互联网连接成功  
2.1. 加密网关Hub1通过静态路由（通过路由器Hub1_Router的默认路由）连接到互联网  
2.2. 加密网关Spoke1通过静态路由（通过路由器Router1的默认路由）连接到互联网  

### 安全交互参数

总部子网（192.168.100.0/24）与分支机构子网（192.168.1.0/24）之间的所有IP流量都使用GOST算法和IPsec协议在隧道模式下进行保护

安全连接可以由从子网192.168.1.0/24到192.168.100.0/24（从分支机构到总部）的流量发起，也可以由从总部到分支机构的反向流量发起

3.1. IKE协议参数：  
 - 使用数字证书进行身份验证，签名算法 - GOST Р 34.10-2012（256位密钥）  
 - 加密算法 - GOST 28147-89（256位密钥）  
 - 哈希函数计算算法 - GOST Р 34.11-2012 ТК26（256位密钥）  
 - 通用密钥生成算法（Diffie-Hellman算法类似物） - VKO_GOSTR3410_2012_256（256位密钥）  

3.2. ESP协议参数：  
 - 加密和完整性保护组合算法 - ESP_GOST-4M-IMIT（256位密钥）  

## 实验台配置

### 表1 - 实践任务中的符号说明

| 索引 | 含义 |
|------|------|
| X | 小组名单序号 |
| Y | 小组名单序号 + 1 |

### 配置设备 Hub1_Router

```
配置网络接口ether0的IP地址 - 172.16.100.Y 和掩码 - 255.255.255.0
配置网络接口ether2的IP地址 - 100.100.100.Y 和掩码 - 255.255.255.0
配置网络接口ether1的IP地址 - 192.168.200.Y 和掩码 - 255.255.255.0
配置网络接口ether3的IP地址 - 192.168.100.Y 和掩码 - 255.255.255.0
配置路由，使需要加密的流量定向到加密网关Hub1的接口ether1的IP地址192.168.200.X
在接口ether0上配置Source NAT或伪装，使来自子网192.168.100.0/24和100.100.100.0/24的数据包被NAT到地址172.16.100.Y
在接口ether0上配置Destination NAT，使所有目标IP地址为172.16.100.Y且目标端口为500和4500的UDP数据包重定向到加密网关Hub1的接口ether0的IP地址100.100.100.X
设置通过172.16.100.X的默认路由
```

### 配置设备 Router1

```
配置网络接口ether2的IP地址 - 172.16.1.X 和掩码 - 255.255.255.0
配置网络接口ether1的IP地址 - 172.16.100.X 和掩码 - 255.255.255.0
配置网络接口ether0的IP地址 - 172.16.10.X 和掩码 - 255.255.255.0
允许IP流量通过
```

### 配置设备 host_behind_hub1

```
配置网络接口的IP地址 - 192.168.100.100, 掩码 - 255.255.255.0
设置通过192.168.100.Y的默认路由
允许接收和发送ICMP数据包
```

### 配置设备 host_behind_spoke1

```
配置网络接口的IP地址 - 192.168.1.100, 掩码 - 255.255.255.0
设置通过192.168.1.Y的默认路由
允许接收和发送ICMP数据包
```

### 配置加密网关 Hub1

```
完成设备初始化
生成私钥并请求加密网关证书
在CA上颁发加密网关证书，并将CA证书和加密网关证书导入S-Gate数据库
配置网络接口ether1的IP地址 - 192.168.200.X 和掩码 255.255.255.0
配置网络接口ether0的IP地址 - 100.100.100.X 和掩码 255.255.255.0
根据之前的实践任务配置IKE
配置IPsec并将创建的加密映射附加到外部接口GigabitEthernet0/0
```

### 配置加密网关 Spoke1

加密网关Spoke1的配置与Hub1类似。在配置IPsec对等体IP地址时，应指定路由器Hub1_Router的接口ether0的IP地址 - 172.16.100.Y

## 实验台功能验证

完成所有实验台设备配置后，需要执行功能验证

### IP连通性检查

1. 检查从加密网关Hub1和Spoke1是否可以通过ICMP访问路由器Router1。为此，从加密网关的cisco-like控制台执行ping命令

```cisco
Hub1#ping 172.16.100.Y
PING 172.16.100.Y (172.16.100.Y) 100(128) bytes of data.
108 bytes from 172.16.100.Y: icmp_seq=1 ttl=64 time=0.384 ms
108 bytes from 172.16.100.Y: icmp_seq=2 ttl=64 time=0.409 ms
108 bytes from 172.16.100.Y: icmp_seq=3 ttl=64 time=0.365 ms
108 bytes from 172.16.100.Y: icmp_seq=4 ttl=64 time=0.389 ms
108 bytes from 172.16.100.Y: icmp_seq=5 ttl=64 time=0.393 ms

--- 172.16.100.Y ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4080ms
rtt min/avg/max/mdev = 0.365/0.388/0.409/0.014 ms
```

```cisco
Spoke1#ping 172.16.1.Y
PING 172.16.1.1 (172.16.1.Y) 100(128) bytes of data.
108 bytes from 172.16.1.Y: icmp_seq=1 ttl=64 time=0.286 ms
108 bytes from 172.16.1.Y: icmp_seq=2 ttl=64 time=0.231 ms
108 bytes from 172.16.1.Y: icmp_seq=3 ttl=64 time=0.435 ms
108 bytes from 172.16.1.Y: icmp_seq=4 ttl=64 time=0.454 ms
108 bytes from 172.16.1.Y: icmp_seq=5 ttl=64 time=0.432 ms

--- 172.16.1.Y ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4067ms
rtt min/avg/max/mdev = 0.231/0.367/0.454/0.093 ms
```

可见，从Hub1和Spoke1都可以通过ICMP访问设备Router1

2. 检查从受保护设备Host_behind_spoke1和Host_behind_hub1是否可以通过ICMP访问相应的默认网关。为此，从受保护设备的linux bash控制台执行ping命令

```bash
root@Host_b_hub1:~# ping 192.168.100.X -c 5
PING 192.168.100.X (192.168.100.X) 56(84) bytes of data.
64 bytes from 192.168.100.X: icmp_seq=1 ttl=64 time=0.262 ms
64 bytes from 192.168.100.X: icmp_seq=2 ttl=64 time=0.299 ms
64 bytes from 192.168.100.X: icmp_seq=3 ttl=64 time=0.460 ms
64 bytes from 192.168.100.X: icmp_seq=4 ttl=64 time=0.287 ms
64 bytes from 192.168.100.X: icmp_seq=5 ttl=64 time=0.274 ms

--- 192.168.100.X ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 81ms
rtt min/avg/max/mdev = 0.262/0.316/0.460/0.074 ms
```

```bash
root@Host_b_spoke1:~# ping 192.168.1.X -c 5
PING 192.168.1.X (192.168.1.X) 56(84) bytes of data.
64 bytes from 192.168.1.X: icmp_seq=1 ttl=64 time=0.302 ms
64 bytes from 192.168.1.X: icmp_seq=2 ttl=64 time=0.257 ms
64 bytes from 192.168.1.X: icmp_seq=3 ttl=64 time=0.305 ms
64 bytes from 192.168.1.X: icmp_seq=4 ttl=64 time=0.354 ms
64 bytes from 192.168.1.X: icmp_seq=5 ttl=64 time=0.335 ms

--- 192.168.1.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 59ms
rtt min/avg/max/mdev = 0.257/0.310/0.354/0.038 ms
```

3. 检查从加密网关Hub1是否可以通过ICMP访问加密网关Spoke1（通过外部接口）。为此，从加密网关的cisco-like控制台执行ping命令

```cisco
Hub1#ping 172.16.1.X
PING 172.16.1.X (172.16.1.X) 100(128) bytes of data.
108 bytes from 172.16.1.X: icmp_seq=1 ttl=63 time=0.701 ms
108 bytes from 172.16.1.X: icmp_seq=2 ttl=63 time=0.806 ms
108 bytes from 172.16.1.X: icmp_seq=3 ttl=63 time=0.728 ms
108 bytes from 172.16.1.X: icmp_seq=4 ttl=63 time=0.688 ms
108 bytes from 172.16.1.X: icmp_seq=5 ttl=63 time=0.704 ms

--- 172.16.1.X ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4020ms
rtt min/avg/max/mdev = 0.688/0.725/0.806/0.048 ms
```

可见，从加密网关Hub1可以通过ICMP访问加密网关Spoke1

### PKI检查

1. 检查加密网关Hub1和Spoke1上所有证书的状态是否为Active。为此，从加密网关的cisco-like控制台执行命令 `run cert_mgr check`

```cisco
Hub1#run cert_mgr check
1 State: Active   C=RU,L=Zelenograd,O=S-Terra CSP,OU=RnD,CN=Hub1
2 State: Active   C=RU,L=Zelenograd,O=S-Terra CSP,OU=RnD,CN=S-Terra CSP Test Root CA
```

```cisco
Spoke1#run cert_mgr check
1 State: Active   C=RU,L=Zelenograd,O=S-Terra CSP,OU=RnD,CN=S-Terra CSP Test Root CA
2 State: Active   C=RU,L=Zelenograd,O=S-Terra CSP,OU=RnD,CN=Spoke1
```

```cisco
Spoke2#run cert_mgr check
1 State: Active   C=RU,L=Zelenograd,O=S-Terra CSP,OU=RnD,CN=S-Terra CSP Test Root CA
2 State: Active   C=RU,L=Zelenograd,O=S-Terra CSP,OU=RnD,CN=Spoke2
```

### 检查Hub1和Spoke1子网之间安全连接的建立

1. 通过从设备Host_behind_spoke1向Host_behind_hub1发送ICMP流量（或反向），发起加密网关Hub1和Spoke1之间的安全连接建立。为此，从受保护设备Host_behind_spoke1的linux bash控制台执行ping命令

```bash
root@Host_b_spoke1:~# ping 192.168.100.1 -c 5
PING 192.168.100.100 (192.168.100.100) 56(84) bytes of data.
64 bytes from 192.168.100.100: icmp_seq=1 ttl=62 time=402 ms
64 bytes from 192.168.100.100: icmp_seq=2 ttl=62 time=1.99 ms
64 bytes from 192.168.100.100: icmp_seq=3 ttl=62 time=1.78 ms
64 bytes from 192.168.100.100: icmp_seq=4 ttl=62 time=1.33 ms
64 bytes from 192.168.100.100: icmp_seq=5 ttl=62 time=1.35 ms

--- 192.168.100.100 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 10ms
rtt min/avg/max/mdev = 1.331/81.610/401.605/159.997 ms
```

2. 检查加密网关Hub1和Spoke1上是否已建立安全的IPsec连接。为此，从加密网关的cisco-like控制台执行命令 `run sa_mgr show`

```cisco
Hub1#run sa_mgr show
ISAKMP sessions: 0 initiated, 0 responded

ISAKMP connections:
Num Conn-id (Local Addr,Port)-(Remote Addr,Port) State Sent Rcvd
1 1 (172.16.100.2,500)-(172.16.1.2,500) active 1828 1908

IPsec connections:
Num Conn-id (Local Addr,Port)-(Remote Addr,Port) Protocol Action Type Sent Rcvd
1 1 (192.168.100.0-192.168.100.255,*)-(192.168.1.0-192.168.1.255,*) * ESP tunn 440 440
```

```cisco
Spoke1#run sa_mgr show
ISAKMP sessions: 0 initiated, 0 responded

ISAKMP connections:
Num Conn-id (Local Addr,Port)-(Remote Addr,Port) State Sent Rcvd
1 1 (172.16.1.2,500)-(172.16.100.2,500) active 1908 1828

IPsec connections:
Num Conn-id (Local Addr,Port)-(Remote Addr,Port) Protocol Action Type Sent Rcvd
1 1 (192.168.1.0-192.168.1.255,*)-(192.168.100.0-192.168.100.255,*) * ESP tunn 440 4
```
```
