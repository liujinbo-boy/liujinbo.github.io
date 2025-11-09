​		

​		在基于SIP协议的双向对讲系统开发中，NAT穿透是一个常见的技术难点。由于大多数设备都处于局域网内，直接的P2P通信会受到NAT设备的限制。我将介绍不同的NAT限制类型，并根据不同类型提供对应的解决方案



### 一、NAT的限制分类

#### 1、全锥形NAT（Full Cone NAT）

全锥形NAT是最宽松的NAT类型，其特点为：

- 双方知道对方的公网ip和端口即可P2P通信

#### 2、地址受限锥形NAT（Restricted Cone NAT）

地址受限锥形NAT在全锥形NAT基础上增加了访问限制：

- 当内网设备主动联系外部 IP 后（内网设备先主动向外部 IP 发起过通信，比如 TCP 三次握手、UDP 的首次数据发送，而非外部 IP 主动向内网设备发起连接），NAT 会自动将这个 “外部 IP” 加入一个**临时白名单**—— 后续只要是这个白名单里的 IP 发过来的数据，无论用什么端口，NAT 都会放行并转发到内网设备；反之，不在白名单里的陌生 IP，哪怕端口正确，也会被 NAT 拦截。

#### 3、端口受限锥形NAT（Port-Restricted Cone NAT）

端口受限锥形NAT进一步严格了访问条件：

- 当内网设备主动联系外部 IP 后（内网设备先主动向外部 IP 发起过通信，比如 TCP 三次握手、UDP 的首次数据发送，而非外部 IP 主动向内网设备发起连接），NAT 会自动将这个 “外部 IP和端口” 加入一个**临时白名单**—— 后续必须得是这个白名单里的 IP &端口发过来的数据，NAT 才会放行并转发到内网设备

#### 4、对称NAT（Symmetric NAT）

对称NAT是最严格的NAT类型，其特点为：

- **动态映射**：内部主机的同一IP:Port与不同外部主机通信时，会映射到不同的外部IP:Port。

- 举例：STUN协议的局限性

  在 SIP 协议中，当被叫方（callee）使用STUN服务器获取到公网 IP 和端口；随后，callee 会将这组公网IP 和端口封装在 SDP（会话描述协议）中，通过sip服务器传递给主叫方（caller）。

  但问题在于，当 caller 依据 SDP 中的 IP 和端口尝试与 callee 建立连接时，callee 的 NAT 设备会因为 “目标是 caller（而非之前通信的 SIP 服务器）” 这一差异，为此次连接分配新的公网IP 和端口。此时，caller 仍使用 SDP 中记录的旧端口发起连接，自然无法与 callee 实际的新端口匹配，最终导致媒体传输失败





### 二、用wireshark分析解决内网穿透

SIP 中 Caller 与 Callee 的通信包含**SIP 信令传输**和**媒体传输**两个独立通道：

信令传输: SIP 服务器仅在双方( Caller 与 Callee )注册阶段获取其用于 SIP 信令交互的公网 IP 和端口（该地址端口仅服务于呼叫发起、响应、挂断等信令传递）

媒体传输（如语音、视频数据流）需使用的 IP 和端口（可能是内网地址、STUN 获取的公网映射地址或 TURN 分配的中继地址），必须由 Caller 和 Callee 分别封装在 SDP（会话描述协议）中，通过 SIP 信令（如 INVITE、200 OK）经服务器互相告知，双方才能基于 SDP 中的媒体地址端口建立独立的媒体连接，且信令与媒体的 IP 端口可能不同、NAT 映射也相互独立，无法复用信令地址进行媒体传输。



#### 1.SIP 协议在不同网络环境下的核心地址传递逻辑

1. 场景一：Caller 与 Callee 处于**同一公网下的局域网**（内网互通）

- **场景前提**：双方在同一内网（如同一家庭 / 公司路由器下），无需跨 NAT 通信。
- 核心操作
  1. Caller 和 Callee 无需依赖 STUN/TURN 服务器，直接使用自身的**内网 IP 地址（Host Address）** 生成 SDP。
  2. 双方通过 SIP 服务器传递包含 “内网 IP + 媒体端口” 的 SDP。
- **通信逻辑**：Caller 直接用 SDP 中的内网地址与 Callee 建立连接，无需 NAT 转换，媒体流可直接互通。

2. 场景二：Caller 与 Callee 处于**不同局域网**（跨 NAT），且双方均为**非对称型 NAT**（如全锥形、端口受限锥形）

- **场景前提**：双方需跨 NAT 通信，但 NAT 支持 “同一内网地址对不同外部目标，复用同一公网映射”（非对称 NAT 特性）。
- 核心操作
  1. Caller 和 Callee 分别向 STUN 服务器发送请求，获取各自**NAT 分配的公网 IP 和端口（Server Reflexive Address）**。
  2. 双方将 “公网 IP + 媒体端口” 封装到 SDP 中，通过 SIP 服务器互相告知。
- **通信逻辑**：Caller 用 SDP 中的 Callee 公网地址发起连接，Callee 的 NAT 会识别该请求并匹配已有的公网映射，媒体流可直接穿透 NAT。

3. 场景三：Caller 与 Callee 处于**不同局域网**（跨 NAT），且至少一方为**对称型 NAT**

- **场景前提**：对称型 NAT 会为 “不同外部目标” 分配不同公网映射，STUN 获取的公网地址无法复用，直接连接会失败。
- 核心操作
  1. Caller 和 Callee 分别向 TURN 服务器发起认证，通过后获取 TURN 分配的**中继 IP 和端口（Relay Address）**。
  2. 双方将 “TURN 中继 IP + 媒体端口” 封装到 SDP 中，通过 SIP 服务器互相告知。
- **通信逻辑**：Caller 和 Callee 的媒体流不再直接传输，而是先发送到 TURN 服务器，由 TURN 服务器中继转发给对方，绕开对称型 NAT 的映射限制。



#### 2.SDP的抓包分析

![img](./blog/sip/pic/2.jpg)



Connetion Information (c): 表示媒体传输的IP

Media Description,name and address (m): 红色划线部分是媒体传输端口



#### 3.STUN的抓包分析

**STUN 协议的核心功能**：获取内网设备的公网ip和端口，识别内网设备的NAT限制类型

![1](./blog/sip/pic/1.png)

```
Binding Request是 STUN 最基础的请求类型，用于获取 NAT 映射的公网 IP 和端口。
Binding Success Response由STUN服务器返回，携带客户端的公网 IP 和端口（通过MAPPED-ADDRESS属性）。
```

![image-20251103143743370](./blog/sip/pic/image-20251103143743370.png)



#### 4.TURN（中继服务器）的抓包分析

TURN（Traversal Using Relays around NAT，通过中继绕过 NAT）是一种专门用于解决复杂 NAT 环境（尤其是对称 NAT）下媒体流（如 RTP）双向传输的技术，是 NAT 穿透方案中 “中继转发” 的核心实现。它不像 STUN 那样仅获取公网映射地址，而是直接作为 “中间转发节点”，让通信双方的流量通过 TURN 服务器中转，从而彻底避开 NAT 的限制。

简单说：对称 NAT 的 “严格绑定” 让直接通信成为不可能，而 TURN 服务器通过 “中间转发”，把 “陌生的跨 NAT 通信” 变成 “熟悉的与服务器通信”，从而绕过限制

1.设备（10.68.5.197）注册TURN服务器（91.98.207.60）的认证交互流程

```
source			destination 	info

//UDP 地址分配请求，请求服务器分配中继地址
10.68.5.197	    91.98.207.60	Allocate Request UDP 

//服务器拒绝初始请求，返回 401 未认证错误
91.98.207.60	10.68.5.197	   	Allocate Error Response error-code: 401 (Unauthenticated) Unauthorized with nonce realm: 91.98.207.60

//客户端补充认证信息(user,passwords,realm)后重试
10.68.5.197	    91.98.207.60	Allocate Request UDP user: ahmed realm: 91.98.207.60 with nonce

/*认证通过，服务器返回 分配成功响应
 *XOR-RELAYED-ADDRESS  拿到的中继地址ip端口
 *XOR-MAPPED-ADDRESS   客户端的公网ip端口，即服务器感知到的客户端外网地址
 */
91.98.207.60	10.68.5.197	   	Allocate Success Response XOR-RELAYED-ADDRESS: 91.98.207.60:50009 XOR-MAPPED-ADDRESS: 223.104.77.3:39056 lifetime: 600

/*客户端向服务器发起 通信权限申请：请求允许自身通过已分配的中继地址，与目标的中继ip通信
 *XOR-PEER-ADDRESS  目标中继ip端口
 */
10.68.5.197	   91.98.207.60		CreatePermission Request XOR-PEER-ADDRESS: 91.98.207.60:18419 user: ahmed realm: 91.98.207.60 with nonce

//服务器验证通过，返回 权限创建成功响应
91.98.207.60	10.68.5.197		CreatePermission Success Response

/*客户端发送数据
 *XOR-PEER-ADDRESS  目标中继ip  发送的对象是谁（91.98.207.60:18418）
 */
10.68.5.197	    91.98.207.60    Send Indication XOR-PEER-ADDRESS: 91.98.207.60:18418

/*客户端接收数据
 *XOR-PEER-ADDRESS  目标中继ip  谁（91.98.207.60:18418）发送的
 */
91.98.207.60	10.68.5.197		Data Indication XOR-PEER-ADDRESS: 91.98.207.60:18418
```



#### 5.ICE的介绍

ICE 协议的核心作用：自动检测 NAT 类型，优先尝试 STUN 获取的映射建立直接连接；若失败（如对称 NAT 场景），自动切换到 TURN 中继模式，确保媒体流能正常传输。

1. **候选地址收集**：

   收集双方的所有可达地址（候选地址），包括：

   - **Host Candidate**：本地网卡的内网地址（如`192.168.1.2`）。
   - **Server Reflexive Candidate**：通过 STUN 获取的公网 NAT 映射地址（如`203.0.113.5:12345`）。
   - **Relay Candidate**：通过 TURN 获取的中继服务器地址（如`198.51.100.1:5000`）。

2. **连通性检查**：

   对所有候选地址对（Caller 候选 + Callee 候选）发起 STUN 探测，验证哪些地址对可以直接通信。

3. **路径选择**：

   基于连通性结果和优先级，选择最优路径（优先直连，失败则用中继），确保媒体流稳定传输。



ICE 协议的使用需要双方在SDP 中携带ice相关的字段如a=candidate，a=ice-controlling等

