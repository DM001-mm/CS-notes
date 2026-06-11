**tcp**,**80**,无连接 无状态
# 版本迭代

## HTTP/1.0
![[assets/4725d74d1e0026f0b75a86e71ad3c2c0_720.jpg]]

## HTTP/1.1
![[assets/fccffadc34c3753cc3e05cc29fd75813_720.jpg]]
## HTTP/2.0
![[assets/890b1124a5a0bef95bf360c41323c94b_720.jpg]]
# 优化

 为了减少通信量，降低时延，采用 代理服务器 ，也叫 web cache

# 报文解读
![[assets/c58ef0ac9c12a6c748d9691e35846f04_720.jpg]]
这和ip数据报首部有很大的不同，点在于没有明确标明长度，有空格，原因就是 ascii 码串长度不确定

![[assets/Pasted image 20260523035414.png]]
**请求行**
+ get 方法 （命令）
+ url 就是那个相对地址
+ 版本号 http/1.1

**首部行**
+ 希望连接 继续连

+ 如果可以，就升级成 https

+ user-agent:能看到浏览器的信息，window nt 操作系统， 架构， app... 向下兼容，（mozi...也是），chrome 内核，Edge浏览器
+ ![[assets/Pasted image 20260523040041.png]]
+ refer：这个地址 从哪来 嗯，就是他上一级的url
+ 发来的是 html的gzip，然后由本地解析，（降低发送的数据量，提高加载速度）
+ 语言
+ cookie
[[你今天吃cookie 了吗]]
> 下面有蓝色下划线的是 wireshark 搞出来的，先不管他

至于 实体主体：
request 一般没有
reply 一般有

这是reply


![[assets/1bfc4f5e5a28efe1cdffaa6d8ff032d8_720.jpg]]