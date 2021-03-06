# 深入理解计算机系统（6）

- 深度参考 Datawhale CSAPP教程

## 一、 存储器结构
存储器层次结构对应用程序的性能有者巨大：

|  数据存储 | 时钟周期  |
|  ----   | ----  |
| 寄存器   | 0 个 |
| 高速缓存  | 4 ∼ 75 个 |
| 主存     | 上百个 |
| 磁盘     | 几千万个 |


随机访问存储器（Random-Access Memory, RAM）分为两类：静态的动态的。
静态RAM（SRAM）比动态 RAM（DRAM）更快，但也更贵。
-  SRAM 常用来作为高速缓存存储器。
    - 随机访问存储器（Random-Access Memory, RAM）
    - SRAM 将每个位存储在一个双稳态（bistable）存储器单元
    - 只要有电，SRAM 存储器单元就会永远保持它的值，即使有干扰扰乱电压，当干扰消除时电路就会恢复到稳定值
-  DRAM 常用来作为主存以及图形系统的帧缓冲区。
    - 存储器单元由一个电容和一个访问晶体管组成
    - 存储器单元对干扰非常敏感，且受到干扰后永远不会恢复
    - DRAM 单元漏电时，在 10 ∼ 100 毫秒内失去电荷。但由于计算机运行的时钟周期是以纳秒来衡量的，因此相对来说 DRAM 单元电荷保持的时间还是比较长的。
    - 内存系统会周期性地通过读出后重写来刷新内存中的每一位
    - DRAM 芯片被封装在内存模块（memory module）中并插到主板的扩展槽上
    - 例如要取出内存地址 A 处的一个字，内存控制器首先将 A 转换成一个超单元地址(i, j)，并将它发送到内存模块，内存模块再将 i 和 j 广播到每个 DRAM。作为响应，每个 DRAM 输出它的 (i, j) 超单元的 8 位内容。模块中的电路收集这些输出，并把它们合并成一个 64 位字，返回给内存控制器。
    

磁盘存储
-  无电也可存储，永久存储大量的数据
-  从磁盘上读信息的时间为毫秒级
-  由盘片（platter）和可以旋转的主轴（spindle）构成。
-  5400 ∼ 15000 转每分钟（“转每分钟”也作为单位，RPM）
-  磁盘容量
    - 记录密度
    - 磁道密度
    - 面密度
-  磁盘通过读写头（read/write head）来读写存储在磁性表面的位，而读写头连接到一个传动臂（actutaor arm）一端
-  读一个磁盘扇区的数据到主存时，操作系统会将命令发送到磁盘控制器，让它读某个逻辑块号。控制器上的固件执行一个快速表查找，将一个逻辑块号翻译成一个三元组：(盘面, 磁道, 扇区) ，这个三元组唯一地表示了对应的物理扇区。随后控制器上的硬件会解释这个三元组，将读写头移动到适当的柱面，等待扇区移动到读写头下，随后将读写头感知到的位放到控制器上的一个小缓冲区中，然后将它们复制到主存中。


固态硬盘
-  固态硬盘（Solid State Disk, SSD）是一种基于闪存的存储技术。
-  一个SSD 由一个或多个闪存芯片以及闪存翻译层组成。闪存芯片取代了磁盘中机械驱动器。
-  数据在 SSD 中是以页为单位读写的。在写入数据之前，需要对一页所属的块进行擦除操作。所谓擦除，指的是将该块中所有位置为 1。而写入的操作实际上就是将部分位置为 0。


时间局部性
-  良好的时间局部性指的是被引用过一次的数据项会在不远的将来再次被多次引用
-  每一个元素都只被使用了一次，时间局部性差

空间局部性
-  良好的空间局部性指的是，如果某个数据在某个位置被引用了一次，那么在不远的将来将引用它附近的内存位置
-  步长越大，空间局部性就越差


缓存
-  位于 k 层的更快更小的存储设备作为位于 k + 1 层更大更慢的存储设备的缓存
-  当程序需要第 k + 1 层的某个数据对象 d 时，它首先在当前存储在第 k 层的一个块中查找 d。如果 d 刚好缓存在第 k 层中，那就称为缓存命中
-  如果程序在第 k 层中找不到缓存数据对象 d，那么就是缓存不命中。当缓存不命中时，就需要从第 k+1 层中取出包含 d 的那个块。同时，会将k+1层取到的块加入k层中，此时如果 k 层的缓存已经满了，那么就可能会覆盖现存的一个块
    - 缓存不命中时，如果k层是空的，称为冷缓存（cold cache)，对任何数据对象的访问都会不命中，这类不命中称为强制性不命中（compulsory miss）或冷不命中（cold miss）。
    - 发生不命中后，由放置策略（placementpolicy）决定放在k层的哪块
    - 随意放置 -  最灵活
    - 限制放置 -  可能导致冲突不命中（conflict miss），部分对象会被映射到同一个缓存块，导致缓存一直不命中
-  替换哪个块是由缓存的替换策略（replacement policy）控制的。
    - 随机替换策略会随机替换一个块
    - LRU 替换策略会选择替换最后被访问的时间距现在最远的块
  
高速缓存
-  每个存储器地址有 m 位，形成 M = 2**m 个不同的地址
-  一个高速缓存器被组织成一个有 S = 2**s 个高速缓存组的数组
-  每个组包含E个高速缓存行。每个行由一个 B = 2**b 字节的数据块组成
-  高速缓存结构可以用一个四元组 (S, E, B, m) 来描述，高速缓存的大小C 指的是所有块的大小的和，标记位和有效位不包括在内，因此 C = S × E × B
-  缓存不命中： CPU - 高速缓存 - 查找不到，不命中 - 去内存中查找  - 将查找的数据写入高速缓存 - 读取高速缓存数据 - 返回CPU
-  缓存命中： CPU - 高速缓存 - 查找到 - 返回CPU
-  直接映射高速缓存
    - 在高速缓存中查找过程： 1）组选择；2）行匹配；3）字抽取
    - 每个组只有一行
-  组相联高速缓存
    - 每个组都有多于一个的高速缓存行
    - 常见的行替换策略有最不常使用（Least-Frequently-Used, LFU）策略和最近最少使用（Least Recently-Used,LRU）策略。
-  全相联高速缓存
    - 只有一个组，不需要进行组选择
-  写问题
    - 直写（write-through），更新w后即立即将 w 写回到低一层中。这样做的缺点是每次都会引起总线流量
        - 非写分配，即直接把这个字写到低一层中        
    - 写回（write-back），即尽可能地推迟更新，直到替换算法要驱逐这个更新过的块时再写到低一层中
        - 写分配，即加载相应的低一层中的块到高速缓存中，然后更新这个高速缓存块
