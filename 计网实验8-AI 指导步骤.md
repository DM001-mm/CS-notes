你这个实验本质上**一台电脑就可以完成**，主要工具是：

**Wireshark + Windows 命令行 cmd / PowerShell**。

实验要求你做三件事：分析 IP 首部、验证 IP 分片、分析 ICMP 与 ping/tracert 原理。你的实验文档里明确要求：抓包分析 IP 首部、用 4600 字节 ping 验证分片、捕获 ICMP echo request/reply、捕获至少两种 ICMP 差错报告、分析 ping 和 tracert。

---

## 一、实验前准备

先打开 Wireshark，选择你当前正在联网的网卡。

一般是：

- 用校园网 / WiFi：选 **WLAN**
    
- 插网线：选 **Ethernet / 以太网**
    
- 不确定：看哪个网卡旁边有波形跳动，就选哪个
    

然后打开命令行：

```bash
cmd
```

或者 PowerShell 也可以。

建议你先查一下自己的 IP：

```bash
ipconfig
```

记住你的：

```text
IPv4 地址
默认网关
```

比如：

```text
IPv4 地址 . . . . . . . . . . . . : 192.168.1.23
默认网关 . . . . . . . . . . . . : 192.168.1.1
```

---

# 步骤 1：IP 协议首部格式分析

## 1. 开始抓包

Wireshark 中点击左上角蓝色鲨鱼鳍开始抓包。

然后在命令行输入：

```bash
ping www.baidu.com
```

或者：

```bash
ping 8.8.8.8
```

如果 `8.8.8.8` 不通，就用：

```bash
ping www.baidu.com
```

---

## 2. Wireshark 过滤 ICMP 包

在 Wireshark 上方过滤栏输入：

```text
icmp
```

你会看到类似：

```text
Echo (ping) request
Echo (ping) reply
```

随便点一个包，展开：

```text
Internet Protocol Version 4
```

你要截图的就是这里。

重点看这些字段：

```text
Version: 4
Header Length: 20 bytes
Total Length
Identification
Flags
Fragment Offset
Time to Live
Protocol: ICMP
Header Checksum
Source Address
Destination Address
```

---

## 3. 报告里可以这样写

> 通过 Wireshark 捕获 ping 过程中的 ICMP 数据包，并展开 IPv4 首部。可以看到 IPv4 首部包含版本号、首部长度、总长度、标识、标志、片偏移、TTL、协议字段、首部校验和、源 IP 地址和目的 IP 地址等内容。其中 Version 为 4，Header Length 通常为 20 bytes，Protocol 字段显示为 ICMP，说明该 IP 数据报承载的是 ICMP 报文。

---

# 步骤 2：验证 IP 分片过程

你的实验要求是：**使用 Ping 命令发送一个 4600 字节的数据包**。

Windows 下命令是：

```bash
ping -l 4600 www.baidu.com
```

也可以 ping 一个 IP：

```bash
ping -l 4600 8.8.8.8
```

其中：

```text
-l 4600
```

表示发送 4600 字节的数据。

---

## 1. Wireshark 过滤分片包

抓包时，过滤栏可以输入：

```text
icmp || ip.flags.mf == 1 || ip.frag_offset > 0
```

或者简单一点：

```text
ip
```

然后你找这些特征：

```text
More fragments: Set
Fragment offset: 0
Fragment offset: 185
Fragment offset: 370
...
```

同一次 ping 分出来的几个 IP 分片会有相同的：

```text
Identification
```

也就是“标识”字段一样。

---

## 2. 你应该看到什么现象？

以太网常见 MTU 是 1500 字节。

IP 首部通常是 20 字节，所以每个分片最多装：

```text
1500 - 20 = 1480 字节
```

4600 字节的 ICMP 数据再加 ICMP 首部后，整个 IP 数据报会超过 MTU，所以会被分片。

你重点截图这几个地方：

```text
Identification 相同
Flags 中 More fragments
Fragment Offset 不同
```

---

## 3. 报告里可以这样写

> 使用 `ping -l 4600 www.baidu.com` 发送较大的 ICMP 数据包。由于该数据包长度超过链路 MTU，因此在传输过程中发生 IP 分片。通过 Wireshark 可以看到多个 IPv4 分片，它们的 Identification 字段相同，表示属于同一个原始 IP 数据报；前几个分片的 More fragments 标志被置位，最后一个分片的 More fragments 标志为 0；不同分片的 Fragment Offset 不同，用于接收方重新组装数据报。

---

# 步骤 3：ICMP echo request 和 echo reply 分析

## 1. 抓包命令

继续用：

```bash
ping www.baidu.com
```

Wireshark 过滤：

```text
icmp
```

你会看到两类包：

```text
Echo (ping) request
Echo (ping) reply
```

---

## 2. 分析 request

点开 Echo request，展开：

```text
Internet Control Message Protocol
```

你会看到：

```text
Type: 8
Code: 0
Checksum
Identifier
Sequence number
```

其中：

```text
Type 8 = 回声请求
```

---

## 3. 分析 reply

点开 Echo reply，展开 ICMP 首部：

```text
Type: 0
Code: 0
Checksum
Identifier
Sequence number
```

其中：

```text
Type 0 = 回声应答
```

---

## 4. 报告里可以这样写

> ICMP echo request 报文的 Type 字段为 8，Code 字段为 0，用于询问目标主机是否可达。ICMP echo reply 报文的 Type 字段为 0，Code 字段为 0，表示目标主机对请求作出了响应。两者通常具有相同的 Identifier 和对应的 Sequence Number，用于匹配请求和应答。

---

# 步骤 4：捕获至少两种 ICMP 差错报告报文

这个是实验里最容易卡住的地方。

你可以做这两种：

## 差错报文 1：TTL 超时，Type 11

命令：

```bash
tracert www.baidu.com
```

Wireshark 过滤：

```text
icmp
```

你会看到一些：

```text
Time-to-live exceeded
```

点开 ICMP 首部，会看到：

```text
Type: 11
Code: 0
```

这就是 ICMP 超时报文。

原因是 tracert 会故意发送 TTL 很小的数据包，比如 TTL=1、TTL=2、TTL=3。路由器每转发一次 TTL 减 1，减到 0 就丢弃，并返回 ICMP 超时报文。

---

## 差错报文 2：需要分片但 DF 置位，Type 3 Code 4

命令：

```bash
ping -f -l 2000 www.baidu.com
```

解释：

```text
-f
```

表示设置 Don't Fragment，不允许分片。

```text
-l 2000
```

表示发送较大的数据包。

如果路径 MTU 小于这个数据包大小，就可能出现：

```text
Packet needs to be fragmented but DF set
```

Wireshark 里可能看到：

```text
Destination unreachable
Fragmentation needed
```

ICMP 字段一般是：

```text
Type: 3
Code: 4
```

也就是：

```text
目的不可达：需要分片但 DF 标志被置位
```

---

## 如果第二种抓不到怎么办？

可以换成下面这个尝试：

```bash
ping 10.255.255.1
```

或者：

```bash
ping 192.0.2.1
```

有时网关会返回：

```text
Destination unreachable
```

Wireshark 里对应：

```text
Type: 3
```

但是这个不一定稳定，所以优先试：

```bash
ping -f -l 2000 www.baidu.com
```

---

# 步骤 5：分析 ping 命令参数

你可以在命令行输入：

```bash
ping /?
```

常用参数写这几个就够：

```bash
ping www.baidu.com
ping -n 10 www.baidu.com
ping -l 4600 www.baidu.com
ping -t www.baidu.com
ping -f -l 2000 www.baidu.com
```

含义：

|命令|作用|
|---|---|
|`ping www.baidu.com`|默认发送 4 个 ICMP 请求|
|`ping -n 10 www.baidu.com`|发送 10 次|
|`ping -l 4600 www.baidu.com`|指定数据长度为 4600 字节|
|`ping -t www.baidu.com`|持续 ping，直到手动停止|
|`ping -f -l 2000 www.baidu.com`|设置 DF 标志并发送大包，用于测试路径 MTU|

停止 `ping -t` 用：

```bash
Ctrl + C
```

---

# 步骤 6：分析 tracert 工作原理

## 1. 执行命令

```bash
tracert www.baidu.com
```

或者：

```bash
tracert -d www.baidu.com
```

建议用：

```bash
tracert -d www.baidu.com
```

因为 `-d` 表示不解析域名，速度更快，结果更干净。

---

## 2. 你会看到类似结果

```text
  1     2 ms     1 ms     1 ms  192.168.1.1
  2     5 ms     4 ms     5 ms  ...
  3    10 ms     9 ms    11 ms  ...
```

每一行代表一跳路由器。

---

## 3. Wireshark 抓包分析

过滤：

```text
icmp
```

你重点看：

```text
TTL = 1
TTL = 2
TTL = 3
...
```

以及返回的：

```text
ICMP Time-to-live exceeded
```

---

## 4. 报告里可以这样写

> tracert 命令通过逐渐增加 IP 数据报的 TTL 值来探测从本机到目标主机之间经过的路由器。第一次发送 TTL=1 的数据包，第一个路由器将 TTL 减为 0 后丢弃该数据包，并返回 ICMP 超时报文；第二次发送 TTL=2 的数据包，第二个路由器返回 ICMP 超时报文。依次类推，tracert 就可以得到路径上各跳路由器的 IP 地址和往返时延。

---

# 你实际做实验的顺序

建议你按这个顺序做，比较顺：

```text
1. 打开 Wireshark，选择正在联网的网卡
2. 运行 ping www.baidu.com
3. 过滤 icmp，截图 IP 首部
4. 截图 Echo request 和 Echo reply 的 ICMP 首部
5. 运行 ping -l 4600 www.baidu.com
6. 过滤 ip.flags.mf == 1 || ip.frag_offset > 0，截图 IP 分片
7. 运行 tracert -d www.baidu.com
8. 截图 tracert 命令行结果
9. Wireshark 中截图 ICMP Type 11 超时报文
10. 运行 ping -f -l 2000 www.baidu.com
11. 尝试截图 ICMP Type 3 Code 4 差错报文
```

---

# 最终报告结构可以这样写

```text
一、实验目的
理解 IP 首部格式、IP 分片过程、ICMP 首部格式、ping 和 tracert 的工作原理。

二、实验环境
Windows 电脑一台
Wireshark
cmd / PowerShell
网络连接

三、实验过程与结果

1. IP 首部分析
放截图
解释 Version、Header Length、Total Length、TTL、Protocol、Source、Destination 等字段。

2. IP 分片分析
命令：ping -l 4600 www.baidu.com
放截图
解释 Identification 相同、MF 标志、Fragment Offset。

3. ICMP echo request/reply 分析
命令：ping www.baidu.com
放截图
解释 Type 8 和 Type 0。

4. ICMP 差错报告分析
差错一：tracert 产生 Type 11 TTL exceeded
差错二：ping -f -l 2000 产生 Type 3 Code 4，或者目的不可达 Type 3
放截图并解释。

5. ping 和 tracert 原理
说明 ping 使用 ICMP echo request/reply。
说明 tracert 使用 TTL 递增和 ICMP 超时报文。

四、实验总结
通过本实验，掌握了 Wireshark 抓包方法，理解了 IP 首部字段、IP 分片机制、ICMP 报文格式，以及 ping 和 tracert 的实现原理。
```

---

你现在最先做这一条就行：

```bash
ping www.baidu.com
```

然后 Wireshark 过滤：

```text
icmp
```

先把 Echo request 和 Echo reply 抓出来，后面的内容都可以在这个基础上继续做。