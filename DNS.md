# 概念
## 服务器
+ 本地 :  第一个找
+ 根: 树的根节点
+ 顶级:  URL的最后面的服务器,TLD server
		**域名**：
		nTLD:tv
		gTLD:（通用顶级域名）
		基础结构域名：arpa, 反向域名解析,相关 ->[[RARP和ARP]]
+ 权限域名服务器: 范围一个区的DNSs~~我定义的~~
+ 主/辅助：
## 有关查询方式
递归：本地去迭代
迭代：host去迭代

# 首部

## 首部结构
## 默认端口:53

## 类型:

**A**: address ,是域名->ipv4 的映射
**AAAA**:adress, 是域名->ipv6的映射，A的数量是ipv4的4倍，格式的bit位数也是IPv4的4倍，长度是128，具体见[[IPv6]]
**NS**:name server, 当遇到不能解析域名ip的时候，要找谁(其他的DNS server)，域名->dns server的域名
**CNAME**: 就是起了一个别的名字，域名->别的域名

> 查询到ip地址 还得看 A/AAAA record ，对吗

