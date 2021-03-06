# 深入理解计算机系统（3）

- 深度参考 Datawhale CSAPP教程

## 一、 寄存器
函数 A 中调用了函数 B
A: 调用者 Caller
B: 被调用者 Callee

- 调用者保存 Caller Saved
    - A 保存寄存器 rbx 的内容（调用前）
    - A 调用函数 B
    - A 恢复寄存器 rbx 原来存储的内容

- 被调用者保存 Callee Saved
    - B 保存寄存器 rbx 的值（被调用前）
    - B 被调用 未返回
    - B 恢复寄存器 rbx 原来存储的内容
    - B 返回


寄存器 rax 目来保存函数的返回值 -- 调用者保存
寄存器 rsp 用来保存程序栈的结束位置 
还有 6 个寄存器可以用来传递函数参数 -- 均是调用者保存


## 二、 指令
大多数指令包含两部分：操作码和操作数。

movq、addq、subq -- 操作码，它决定了 CPU 执行操作的类型
%rax、%rsx -- 操作码之后的这部分是操作数，大多数指令具有一个或者多个操作数。
ret -- 返回指令，是没有操作数的。

立即数以 美元符号开头，后跟一个 C 定义的整数
寄存器带小括号表示内存引用

### mov 类指令
两个操作数： 源操作数、目的操作数
源操作数： 可以是一个立即数、一个寄存器，或者是内存引用
目的操作数：用来存放源操作数的内容，所以目的操作数要么是一个寄存器，要么是一个内存引用，目的操作数不能是一个立即数


x86-64 处理器有一条限制，就是 mov 指令的源操作数和目的操作数不能都是内存的地址。
将一个数从内存的一个位置复制到另一个位置时，需要两条 mov 指令来完成：
- 第一条指令将内存源位置的数值加载到寄存器
- 第二条指令再将该寄存器的值写入内存的目的位置

32位：
- movq 指令的源操作数是立即数时，该立即数只能是 32 位的补码表示，对该数符号位扩展后，将得到的 64 位数传送到目的位置。

64位：
- movabsq，该指令的源操作数可以是任意的 64 位立即数，需要注意的是目的操作数只能是寄存器

### 指令 leaq
功能是加载有效地址，q 表示地址的长度是四个字

指令：leaq 7(%rdx,%rdx,4),%rax
假设寄存器 rdx 内保存的数值为 x，那么有效地址的值为 7 + %rdx + %rdx * 4 = 7+ 5x。注意，对于 leaq 指令所执行的操作并不是去内存地址 (5x+7) 处读取数据，而是将有效地址 (5x+7) 这个值直接写入到目的寄存器 rax。

### 一元操作和二元操作

一元操作指令只有一个操作数
- 因此该操作数既是源操作数也是目的操作数，操作数可以是寄存器，也可以是内存地址

二元操作指令包含两个操作数
- 第一个操作数是源操作数，这个操作数可以是立即数、寄存器或者内存地址
- 第二个操作数既是源操作数也是目的操作数，这个操作数可以是寄存器或者内存地址，但不能是立即数。

addq 加法指令
subq 减法指令
incq 加一指令

左移指令有两个，分别是 SAL 和 SHL：在右边填零
SAR  算数右移，需要填符号位
SHR  逻辑右移，需要填零

对于移位量 k，可以是一个立即数，或者是放在寄存器 cl 中的数

### 条件码
ALU 除了执行算术和逻辑运算指令，还会根据该运算的结果去设置条件码寄存器。

条件码：
由CPU维护，描述了最近执行操作的属性。
某一时刻条件码寄存器中保存的是该时刻某指令的执行结果的属性
下一时刻执行了另一指令，条件码寄存器中的内容会被覆盖

CF carry flag 进位标志
ZF zero flag 零标志
SF sign flag 符号标志
OF overflow flag 溢出标志

cmp、test 指令 ：只是设置条件码寄存器，并不会更新目的寄存器的值

### 跳转指令
借助条件码实现
条件语句 x 小于 y 由指令 cmp 来实现，指令 cmp 会根据 (x-y) 的结果来设置符号标志 (SF) 和溢出标志 (OF)。

循环：条件测试指令 与跳转指令的组合实现了循环操作

switch： 跳转表来处理多重分支


## 三、 程序执行过程
程序的运行时内存分布中，栈为函数调用提供了后进先出的内存管理机制

函数 P 调用函数 Q 时，会把返回地址压入栈中，该地址指明了当函数 Q 执行结束返回时要从函数 P 的哪个位置继续执行。
这个返回地址的压栈操作是由函数调用 call 来实现的。

- 以 main 函数调用 multstore 函数为例：
    指令 call 不仅要将函数 multstore 的第一条指令的地址写入到程序指令寄存器 rip中，以此实现函数调用。同时还要将返回地址压入栈中。
    这个返回地址就是函数 multstore 调用执行完毕后，下一条指令的地址。
    当函数 multstore 执行完毕，指令 ret 从栈中将返回地址弹出，写入到程序指令寄存器 rip 中

## 四、 缓冲区溢出
- 栈随机化
    程序的栈地址非常容易预测 -- 容易被病毒攻击

- 栈破坏检测
    汇编代码中加入一种栈保护者的机制来检测缓冲区越界，就是在缓冲区与栈保存的状态值之间存储一个特殊值，这个特殊值被称作金丝雀值
    
- 限制可执行代码区域
    处理器的内存保护引入了不可执行位，将读和可执行访问模式分开了。有了这个特性，栈可以被标记为可读和可写，但是不可执行。检查页是否可执行由硬件来完成，效率上没有损失。

