# 在两台「С-Терра Шлюз」安全网关之间建立VPN隧道连接受保护子网（基于预共享密钥认证）

## 项目描述

本方案描述了如何配置总部与分支机构受保护子网之间的安全通信。通过使用俄罗斯国家标准GOST和IPsec协议进行加密和隧道传输，实现安全通信。所有其他连接被允许，但不会受到IPsec保护。本方案使用预共享密钥进行认证。

## 基础设施要求

### 设备要求
1.1 作为host-behind-...的设备可以是任何功能类似于个人计算机的设备（webterm-new, Kali Linux CLI, Linux CLI等）

### 网络要求
2.1 测试台设备之间必须确保IP连通性

## 网络拓扑
![网络拓扑](image-url)
*图1 - 网络拓扑结构*

## 工作原理

### 设备部署
1.1 总部部署：С-Терра Шлюз加密网关（Hub）和个人计算机（host_behind_hub）
1.2 分支机构部署：С-Терра Шлюз加密网关（Spoke）和个人计算机（host_behind_spoke）

## Hub设备配置

### 初始化设备
在安全网关投入使用前，管理员必须进行初始初始化 - 输入VPN产品许可证并通过生物随机数生成器进行初始化。

1. 打开Hub虚拟机的控制台
2. 使用默认密码`s-terra`以`administrator`用户登录
3. 运行初始化命令：
```bash
initialize
```
4. 按照屏幕提示完成随机数生成
5. 输入许可证信息（区分大小写）：
```
Enter product code: _
Enter customer code: 
Enter license number: 
Enter license code:
```
6. 确认信息正确性：`Y`
7. 激活预定义的安全策略：
```bash
run csconf_mgr activate
```
8. 重启设备：`reboot`

### 初始设置
1. 切换到Initial CLI控制台（默认凭证：administrator/s-terra）
2. 更改默认密码：
```bash
administrator@sterragate] change user password
```
3. 切换到Linux控制台并更改root密码：
```bash
administrator@sterragate] system
root@sterragate:# passwd
```

### 创建安全策略
1. 进入CGW CLI控制台：
```bash
administrator@sterragate] configure
```
2. 更改CGW CLI密码：
```bash
sterragate#configure terminal
sterragate(config)#username cscons password <新密码>
```
3. 设置设备名称：
```bash
sterragate(config)#hostname Hub
```
4. 设置特权模式密码：
```bash
Hub(config)#enable secret 0 russia
```
5. 配置网络接口：
```bash
Hub(config)#interface GigabitEthernet0/0
Hub(config-if)#ip address 20.20.20.11 255.255.255.0
Hub(config-if)#no shutdown
Hub(config-if)#exit
Hub(config)#interface GigabitEthernet0/1
Hub(config-if)#ip address 192.168.100.11 255.255.255.0
Hub(config-if)#no shutdown
Hub(config-if)#exit
```
6. 配置静态路由：
```bash
Hub(config)#ip route 192.168.1.0 255.255.255.0 20.20.20.12
```
7. 设置身份识别类型：
```bash
Hub(config)#crypto isakmp identity address
```
8. 配置DPD参数：
```bash
Hub(config)# crypto isakmp keepalive 1 3
Hub(config)# crypto isakmp keepalive retry-count 3
```
9. 配置IKE参数：
```bash
Hub(config)# crypto isakmp policy 1
Hub(config-isakmp)# hash gost341112-256-tc26
Hub(config-isakmp)# authentication pre-share
Hub(config-isakmp)# group vko2
Hub(config-isakmp)# exit
```
10. 设置预共享密钥：
```bash
Hub(config)#crypto isakmp key KEY address 20.20.20.12
```
11. 配置IPsec转换集：
```bash
Hub(config)# crypto ipsec transform-set GOST esp-gost28147-4m-imit
Hub(cfg-crypto-trans)#mode tunnel
Hub(cfg-crypto-trans)#exit
```
12. 创建访问控制列表：
```bash
Hub(config)#ip access-list extended LIST
Hub(config-ext-nacl)#permit ip 192.168.100.0 0.0.0.255 192.168.1.0 0.0.0.255
Hub(config-ext-nacl)#exit
```
13. 创建加密映射：
```bash
Hub(config)# crypto map CMAP 1 ipsec-isakmp
Hub(config-crypto-map)# match address LIST
Hub(config-crypto-map)# set transform-set GOST
Hub(config-crypto-map)# set peer 20.20.20.12
Hub(config-crypto-map)# exit
```
14. 应用加密映射到接口：
```bash
Hub(config)#interface GigabitEthernet0/0
Hub(config-if)#crypto map CMAP
Hub(config-if)#exit
```
15. 启用调试日志：
```bash
Hub(config)#logging trap debugging
Hub(config)#exit
```

## Spoke设备配置
Spoke设备的配置过程与Hub类似，主要配置步骤包括：

1. 设置设备名称：`hostname Spoke`
2. 配置网络接口IP地址
3. 配置静态路由到192.168.100.0/24网络
4. 设置身份识别类型
5. 配置DPD参数
6. 配置IKE参数（与Hub相同）
7. 设置相同的预共享密钥
8. 创建IPsec转换集
9. 创建访问控制列表（方向相反）
10. 创建和应用加密映射
11. 启用调试日志

## 终端设备配置
根据网络拓扑为host_behind_hub和host_behind_spoke配置IP地址和默认网关。

## 测试验证

1. 检查配置：
```bash
Hub#show run
Spoke#show run
```

2. 测试连通性：
```cmd
C:\Users\User01>ping 192.168.100.100
```

3. 验证安全连接状态：
```bash
Hub#show crypto isakmp sa
Hub#show crypto ipsec sa
```

## 技术要点
- 使用俄罗斯国家标准GOST算法进行加密
- 基于IPsec协议建立VPN隧道
- 使用预共享密钥进行设备认证
- 支持DPD（Dead Peer Detection）检测对端状态

## 安全注意事项
⚠️ 预共享密钥仅建议在测试环境中使用，生产环境应使用数字证书认证。

---

*本项目展示了在С-Терра Шлюз安全网关之间建立IPsec VPN隧道的完整配置过程，适用于企业分支机构安全互联场景。*
