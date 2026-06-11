
# Cache

~~看到这的时候觉得 jb 太简单了，怎么还打星，没想到啊，兄弟们，一个经典的 Cache 结构图，就给我看傻了~~

# 为什么需要 Cache

CPU 的运算速度远高于主存 DRAM 的访问速度。

如果 CPU 每次取指令、读数据、写数据都直接访问主存，那么 CPU 会经常处于等待状态，整体性能会很差。

所以在 CPU 和主存之间加入 Cache：

```txt
CPU Core  <——>  Cache  <——>  主存 DRAM
```

Cache 的作用就是：

> 用一个容量较小但速度很快的存储器，保存最近可能会被 CPU 访问的数据或指令，从而减少 CPU 直接访问主存的次数。

Cache 能发挥作用，主要依赖程序访问的局部性：

```txt
时间局部性：刚访问过的数据，之后可能还会访问。
空间局部性：访问某个地址后，附近地址也可能很快被访问。
```

---

# 结构

~~上图~~  
![[Word_20260611_213646(1).pdf]]

上图展示的是典型 Cache 结构，核心是 **4 路组相联 Cache**。图中把 Cache 分成了：

```txt
Set(Index) 区域
Way 区域
Tag 区域
Data 区域
Offset
```

这里一定要注意几个概念的对应关系：

```txt
set = 组
way = 路
cache line = 某个 set 中某个 way 位置上的一项
```

也就是说：

```txt
Cache[set][way] = 一条 cache line
```

如果是 4 路组相联 Cache，那么每个 set 里面有 4 个 way：

```txt
Set 0: way0  way1  way2  way3
Set 1: way0  way1  way2  way3
Set 2: way0  way1  way2  way3
...
```

所以 **way 不是缓存行本身**。

更准确地说：

> 某个 set 和某个 way 交叉位置上的那一项，才是一条 cache line。

一条 cache line 通常可以理解为：

```txt
| valid | dirty | tag | data block |
```

其中：

```txt
valid：有效位，表示该 cache line 中的数据是否有效
dirty：脏位，表示该 cache line 是否被 CPU 修改过但还没写回
tag：标识这条 cache line 对应的是哪一个内存块
data block：真正缓存的数据
```

教材图 5.11 中也把数据区域、Tag 区域、Set(Index)、Offset 放在同一个 Cache 结构里展示，说明 Cache 不是只存 tag，而是既有用于判断身份的 tag，也有真正存数据的数据区域。

---

## 缓存寻址

CPU 执行访存指令时，整体流程可以先这样理解：

```txt
CPU 执行 load / store 指令
        ↓
产生虚拟地址 VA
        ↓
MMU / TLB 翻译得到物理地址 PA
        ↓
Cache 根据 PA 查找数据
        ↓
hit：直接返回数据
miss：向下一级 Cache / 主存取数据
```

严格说，现代 CPU 的 L1 Cache 可能会为了速度使用 VIPT 结构，也就是虚拟地址低位先参与索引，同时 TLB 做地址翻译。

但是在理解基础 Cache 结构时，可以先使用这个模型：

```txt
VA ——MMU/TLB——> PA ——> Cache
```

---

Cache 会根据自己的结构，把物理地址 PA 解释成三段：

```txt
| Tag | Set(Index) | Offset |
```

这三段不是物理地址天然自带的，而是 Cache 根据：

```txt
cache line 大小
set 数量
```

把物理地址的二进制位切出来的。

三段的作用分别是：

```txt
Tag：标识内存块，用来判断是不是目标数据
Set(Index)：选择 Cache 中的哪一个 set
Offset：在命中的 cache line 的 data block 中选择具体字节/字
```

---

## 寻址流程

Cache 查找数据的过程是：

```txt
1. CPU 给出要访问的地址，经过 MMU/TLB 后得到物理地址 PA。

2. Cache 把 PA 拆成：
   | Tag | Set(Index) | Offset |

3. 根据 Set(Index) 找到对应的 set。

4. 在这个 set 里面，同时检查所有 way。

5. 对每个 way，判断：
   valid == 1 && cache_line.tag == address_tag

6. 如果某个 way 满足条件，则 Cache hit。

7. 命中后，再根据 Offset 到该 cache line 的 data block 中取出具体数据。

8. 如果没有任何 way 命中，则 Cache miss，需要向下一级 Cache 或主存请求数据。
```

所以，命中条件不是只看 tag，也不是只看 valid，而是：

```txt
hit = valid 位为 1 && tag 匹配
```

也就是说：

```txt
Set 匹配只是说明找到了候选组；
Tag 匹配只是说明身份可能对；
Valid = 1 才说明这条 cache line 中的数据真的有效。
```

如果 tag 匹配，但是 valid = 0，那么仍然不能算命中，因为这条 cache line 里的内容无效，需要从下一级存储重新取数据。

---

![[faf40010e2abb1299caf7271a7433ed8_720.jpg]]

这个图更具体地展示了 Cortex-A57 的 L1 数据缓存结构。

图中物理地址被划分为：

```txt
| Tag | Set(Index) | Offset |
```

图上标出的位数大概是：

```txt
Tag：43 ~ 14
Set(Index)：13 ~ 6
Offset：5 ~ 0
```

因此：

```txt
Offset 有 6 位
cache line 大小 = 2^6 = 64B
```

也就是说，每条 cache line 的 data block 是 64B。

Set(Index) 是：

```txt
13 ~ 6
```

一共 8 位，因此：

```txt
set 数量 = 2^8 = 256 组
```

图中 set 编号从：

```txt
0 到 255
```

正好对应 256 个 set。

---

## 图中的一个具体例子

图中 Set 0 下面可以看到类似这样的结构：

```txt
Set 0:
    tag = 0x12344321, valid = 0, data ...
    tag = 0x0000beef, valid = 1, data ...
```

这说明在同一个 set 里，有多个 way。

访问某个物理地址时，Cache 会：

```txt
1. 用 Set(Index) 找到 Set 0
2. 在 Set 0 里面检查各个 way
3. 比较地址中的 Tag 和每个 way 里的 Tag
4. 同时检查 valid 位
```

如果某一项：

```txt
tag 匹配 && valid = 1
```

那么命中。

如果：

```txt
tag 匹配但是 valid = 0
```

也不能用，因为这条 cache line 无效，仍然需要从下一级 Cache 或主存取数据。

---

## Offset 的作用

Offset 不是用来判断 Cache 是否命中的。

Offset 的作用是：

> 在已经命中的 cache line 的 data block 里面，选择 CPU 真正需要的那几个字节。

比如 cache line 大小是 64B：

```txt
data block = byte0 byte1 byte2 ... byte63
```

如果 CPU 要读取一个 4 字节整数，且 offset = 16，那么 Cache 会从这条 cache line 里面取：

```txt
byte16 byte17 byte18 byte19
```

所以流程是：

```txt
Tag + Valid：判断是否命中
Offset：命中后取具体数据
```

---

# Cache line 到底是什么

Cache line 可以理解为 Cache 中存放数据的基本单位。

一条 cache line 不是只有数据，也不是只有 tag，而是：

```txt
| valid | dirty | tag | data block |
```

但是在计算 Cache 容量时，通常只计算 data block 的容量，不把 tag、valid、dirty 这些元数据算进去。

例如：

```txt
32KB Cache
```

一般表示它能缓存：

```txt
32KB 的数据
```

不包括 tag array 和状态位带来的额外空间。

---

# Cache line 和页表项的类比

Cache line 和页表项有一点相似：  
它们旁边都有一些状态位，比如 valid 位。

但是它们的本质不同。

页表项 PTE 主要存的是地址映射信息：

```txt
| valid | permission | physical page number |
```

它解决的是：

```txt
虚拟地址 VA -> 物理地址 PA
```

Cache line 存的是某个物理内存块的数据副本：

```txt
| valid | dirty | tag | data block |
```

它解决的是：

```txt
这个物理地址对应的数据在不在 Cache 里？
```

所以：

```txt
TLB / 页表：存地址映射
Cache：存真正的数据副本
```

---

# Cache 什么时候和内存进行数据交换

CPU 访问内存主要有两类情况：

```txt
1. 取机器指令
2. 读写普通数据
```

因此现代 CPU 中通常会有：

```txt
I-Cache：Instruction Cache，指令缓存
D-Cache：Data Cache，数据缓存
```

比如执行一行代码：

```c
int x = a[i] + 1;
```

底层可能发生：

```txt
1. CPU 从 I-Cache 取机器指令
2. 执行 load 指令，从 D-Cache 读取 a[i]
3. ALU 做加法运算
4. 如果需要保存结果，再通过 store 指令写入 D-Cache
```

---

Cache 和下一级存储交换数据的常见时机：

## 1. 读 miss

CPU 要读某个地址，但是 Cache 中没有命中。

这时 Cache 会向下一级 Cache 或主存请求该地址所在的整条 cache line。

注意，不是只取 CPU 当前需要的 4 字节，而是取一整条 cache line，例如 64B。

```txt
读 miss：
Cache 没有
    ↓
从 L2 / L3 / 主存取一整条 cache line
    ↓
填入当前 Cache
    ↓
再把 CPU 需要的数据返回
```

---

## 2. 写 miss

CPU 要写某个地址，但是 Cache 中没有这条数据。

这时根据写策略不同，处理方式也不同。

常见策略之一是 write allocate：

```txt
写 miss 时，先把对应 cache line 取入 Cache，再修改其中的数据
```

也有 no-write allocate：

```txt
写 miss 时，不取入 Cache，直接写下一级存储
```

---

## 3. dirty line 被替换

如果某个 cache line 被 CPU 修改过，但是还没有写回主存，那么它的 dirty 位为 1。

当这条 dirty line 要被替换掉时，不能直接丢弃，否则 CPU 修改过的数据就没了。

所以需要先写回下一级存储：

```txt
dirty = 1
    ↓
被替换前先写回
    ↓
再装入新的 cache line
```

---

## 4. write-through 策略

如果 Cache 使用 write-through 策略，那么 CPU 写 Cache 的同时，也会把数据写到下一级存储。

```txt
CPU 写 Cache
    ↓
同时写下一级 Cache / 主存
```

这种方式一致性更简单，但是写入开销更大。

与之相对的是 write-back：

```txt
CPU 先只写 Cache
    ↓
把 dirty 位置 1
    ↓
等 cache line 被替换时再写回
```

---

# 总结

Cache 的结构和流程一定要一起看。

Cache 不是只做地址判断，它真正存的是数据。  
但是为了确定“这个数据是不是 CPU 要的那个地址的数据”，Cache 必须保存 tag、valid、dirty 等元数据。

完整流程可以总结为：

```txt
CPU 产生虚拟地址 VA
        ↓
MMU/TLB 翻译得到物理地址 PA
        ↓
Cache 把 PA 解释成 Tag / Set(Index) / Offset
        ↓
Set(Index) 找到对应 set
        ↓
在该 set 的所有 way 中比较 tag，并检查 valid
        ↓
如果 valid = 1 且 tag 匹配，则 Cache hit
        ↓
根据 offset 从 data block 中取出数据
        ↓
返回给 CPU
```

如果 miss：

```txt
Cache miss
        ↓
向下一级 Cache / 主存请求整条 cache line
        ↓
选择一个 way 填入数据
        ↓
更新 tag、valid、dirty 等状态位
        ↓
再把 CPU 需要的数据返回
```

最关键的关系是：

```txt
set：组
way：路
cache line：某个 set + 某个 way 位置上的一项
tag：判断身份
valid：判断是否有效
dirty：判断是否修改过
data block：真正的数据
offset：在 data block 中取具体字节/字
```