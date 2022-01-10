# 深入理解计算机系统（4）

- 深度参考 Datawhale CSAPP教程

## 一、 自定义指令集 -- “Y86-64”
相对于“x86-64”指令集，Y86 做了部分精简：
- 省略了%r15 以简化指令的编码，寄存器%rsp 被定义为栈指针，其他 14 个寄存器无固定含义。
- 零标志（ZF）、符号标志（SF）和溢出标志（OF）
- 程序计数器 PC 用来存放当前正在执行的指令的地址
- 状态码 Stat 用来表示程序的执行状态

以irmovq指令   irmovq V, rB  为例
- movq 前面的两个字母分别表示了源和目的的格式，例如irmovq 的源操作数是立即数（Immediate），目的操作数是寄存器（Register）
- 该指令的指令代码为 3，指令功能为 0. 由于源操作数是立即数，因此用 0xF 进行填充:
    3 0 F rB V 

以rmmovq指令  rmmovq %rsp, 0x123456789abcd(%rdx) 为例
- 之前定义的 rrmovq，该指令的指令代码为 4，指令功能为 0：
    4 0 rsp  0x123456789abcd(%rdx)
- 指令操作数部分的寄存器%rsp 对应的寄存器编号为 0x4，基址寄存器 rdx 对应的编号为 0x2，因此该指令二进制表示中的第二个字节为 0x42：
    4 0 4 2  0x123456789abcd(%rdx)
- 偏移量占 8 个字节，因此我们需要在该偏移量前面补 0 来凑齐 8 个字节。又由于我们采用小端法存储，因此还要对偏移量进行字节反序操作：
    4 0 4 2 cd ab 89 67 45 23 01 00


## 二、 指令执行各阶段
1. 取址：处理器执行所有的指令都需要取址。在 Y86-64 指令系统中，指令的长度不是固定的，因此取址阶段需要根据指令代码判断指令是否含有寄存器指示符、是否含有常数来计算当前的指令长度。
    - 以程序计数器（PC）的值为起始地址
    - 因为Y86-64 指令集中最长的指令占 10 字节，取址操作每次从指令内存中读取 10 个字节
    - 10 字节分为两部分，第一部分占 1 字节，第二部分占 9 字节
    - 第一部分平分为两部分，分别为指令代码 icode 和指令功能 ifun ，各占4个bit
    - icode可以确定指令的合法性和长度
    - 将 PC 值加上当前的指令长度来计算内存中下一条指令的地址，用于后续的更新阶段
    - 对于剩下的 9 个字节，我们通过名为Align 的硬件单元来产生寄存器字段和常数字段

2. 译码：处理器从寄存器文件中读取数据。寄存器文件有两个读端口，可以支持同时进行两个读操作。
    - 两个读端口的地址输入为 srcA 和 srcB
    - 从寄存器文件中读出的数值通过 valA 和 valB 输出

3. 执行：指令被正式执行的阶段。在该阶段中，算术逻辑单元（ALU）主要执行三类操作：执行算术逻辑运算、计算内存引用的有效地址、针对 push 和 pop指令的运算。
    - 执行阶段的核心部件 ALU 根据指令功能 ifun 来判断要对输入的操作数进行何种运算。每次运行时，ALU 都会产生三个与条件码相关的信号——零、符号、溢出
    

4. 访存：对内存进行读写操作。
    - 该阶段硬件设计主要包含：
        读控制块，用于进行读操作。
        写控制块，用于进行写操作。
        内存地址控制块，用于产生内存地址。
        数据输入控制块，用于输入数据。
    - 最后还将根据 icode 判断出的指令有效性以及内存状况产生instr_valid 和 imem_error 信号来计算状态码
    
5. 写回：将执行结果写回到寄存器文件中。
    - 为寄存器文件系统添加 M 和 E，这两个写端口，对应的地址输入为 dstE 和dstM
    - 当执行条件传送指令 (cmov) 时，写入操作还要根据执行阶段计算出的 cnd 信号来写入寄存器文件
    - 当条件不满足条件时，以将目的寄存器设置为 0xF 来禁止写入寄存器文件
    
6. 更新：将程序计数器（PC）更新为下一条指令的地址。
    - 当前执行的指令是函数调用指令 call，此使将 PC 值设为 call 指令的常数字段。
    - 当前执行的指令是函数返回指令 ret，此使将 PC 值设为 ret 指令在访存阶段从内存中读出的返回地址。
    - 当前执行的指令是跳转指令 jxx，此使若满足跳转条件（cnd = 1），则新的 PC值等于跳转指令的常数字段；若不满足跳转条件（cnd = 0），则新的 PC 值等于当前 PC 值加上当前指令长度。




