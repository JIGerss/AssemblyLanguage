# [Assembly Language 汇编语言]()

## 第一章 基础知识

- 计算机将代码和数据都储存在内存中（冯诺依曼），如果不加以区分，则两者都为二进制形式，没有任何区别。
- 对于储存器和内存来说，1个字节（Byte）为一个存储单元，即一个8位二进制数 = 两个十六进制数
- CPU对储存器的一次读写分为：
  - 储存单元的地址（地址信息）
  - 器件的选择，读或写的命令（控制信息）
  - 读或写的数据（数据信息）
- 对于地址信息：
  - 总线数量 = 总线宽度
  - 2 ^ N Byte 为该类型CPU的寻址能力，N为CPU地址总线数量
    - 解释：因为1个Byte为一个存储单元，所以只需算出地址所能表达出多少个不同二进制数即可
- 主板上的RAM（如内存），ROM（如BIOS），显卡和网卡等均需要由CPU处理，并分配不同的内存地址空间。一个CPU的内存地址空间是有限的。

<img src=".\1.png" alt="1" style="zoom:50%;" />

- 计算机的两种常用结构（Harvard，Von Neumann）：<img src="D:\作品们\AssemblyNote\3.png" alt="image-20230702160048325" style="zoom:50%;" />

- 汇编语言的结构：

  ```assembly
  .STACK
  .DATA
  
  0017 90 Cnt: 	nop
  0018 B0 00 		mov al,0
  001A B8 0238 	mov ax,568
  001D 8A D6 		mov dl,dh
  001F 8B C3 		mov ax,bx
  0021 B8 0017 R 	mov ax,Cnt
  0024 8B 07 		mov ax,[bx]
  0026 8B 00 		mov ax,[bx][si]
  ```

  - .STACK/.DATA指**伪命令**（Assembler directives）
  - 0017指指令在内存中的**偏移地址**（Offset Address）

  - 90/B0/B8指对应**操作符**（Operator）**的机器码**（Machine code）
  - 00/0238/D6指**操作数**（Operand）**的机器码**
  - mov/nop指**操作符**（Operator）
  - al/ax/568/[bx]/0指**操作数**（Oper and）
  - mov al, 0/mov dl, dh指**汇编指令**（Instructions）

- 8086CPU结构：                                <img src="D:\作品们\AssemblyNote\4.png" alt="image-20230702161731652" style="zoom:75%;" />
  - BIU（Bus Interface Unit）：向内存发送地址以获取指令，并从内存中读取数据
  - EU（Execution Unit）：传递BIU获取指令的地址，并解码和执行指令。EU包括ALU和Control Circuitry

- ALU的累加器顺序：
  - 在时钟信号下降沿的时候，读取下一条指令，此时运算的结果已经产生，但还未被寄存器（Latch）保存
  - 在时钟信号上升沿的时候，将运算的结果保存到寄存器中

- 机器码（Machine Code）：A list of binary codes describing the instructions that are to be executed. This requires hand compiling which is a time consuming task.

- 汇编语言（Assembly Language）：Each binary code can be represented by a **mnemonic**（助记符）. Each mnemonic can be read as English but has a direct equivalent binary equivalent. This makes it much easier for the programmer to remember the codes.

- 汇编语言的优劣：有利于了解计算机硬件操作，并且可以运行速度非常块，且消耗较少的资源；但不适合写大型程序。



## 第二章 寄存器

- 8086CPU具有14种16位寄存器（AX，BX，CX，DX，CS，DS，SS，ES，SI，DI，SP，BP，IP，PSW），其中，AX、BX、CX和DX为通用寄存器，每个通用寄存器都可以拆分为两个8位通用寄存器使用。如AX可以拆分为AH，AL两个寄存器使用。

  - 注意：如果在代码中将通用寄存器拆分为两个寄存器使用，则这两个寄存器在如进位运算或数据存储上将没有任何关系。例子：

    ```assembly
    mov al, FFH ; 将最大1字节数赋给al
    add al, 2H  ; al加2，此时进位不会给ah，而会被CPU吃掉（这样描述很不规范）
    ```

- 在寄存器或内存中，可以用**字节**（Byte）和**字**（word）两种数据格式存储，前提是储存空间足够大。

- 在8086CPU中，物理地址（内存的真实地址）为20位二进制数，但在CPU中只有16位的总线，因此需要用段地址和偏移地址两个16位二进制数表示物理地址。

- 物理地址 = 段地址 × 16 + 偏移地址， 即段地址二进制数左移4位加上偏移地址

- 地址计算在CPU中的地址加法器中进行

- 四种取值模式：

  - Immediate addressing：直接取值，如mov ax, 1234h
  - Register addressing：寄存器直接取值，如mov ax, bx
  - Direct addressing：从地址中取值，如mov ax, Variable；mov ax, [1234]；mov ax, CS : Variable
  - Register Indirect Addressing：间接取值，如mov ax, [bx]；mov ax, \[bx][SI]

- 段的概念：

  - 内存本质上并没有分段，只是CPU将内存以分段的形式使用

  - 一个段的**最开始**单元的物理地址的16进制数总是以0结尾

  - 一个段所能包含的最大容量为64KB，即偏移地址 L(0000H ~ FFFFH) = 2 ^ 16 B = 2 ^ 6 KB = 64 KB

    

- 段寄存器

  - 段寄存器有CS，DS，SS，ES四种，用来储存对应数据的段地址
  - CS为代码段寄存器，储存代码数据在内存中的段地址。IP为指令地址寄存器，储存代码数据在内存中的偏移地址
  - 使用CS：IP的形式来表示（CS × 16 + IP）的物理地址
  - [偏移地址] 表示DS × 16 + *偏移地址*内存上的数据，其中数据可以为字也可以为字节（根据寄存器大小赋值，如mov ax，[0]则为将偏移地址为0处的字赋给ax）
  - 在内存中，**字**需要两个存储单元来存储，存储时**低位在前，高位在后，以低位所在地址为其字的物理地址**
  - 字符串的存储按原本字符顺序存储，如'1234$'则存储为31 32 33 34 24
  - **注意**：不能将一个数字赋值给段寄存器（如mov ds，0001H），需要使用其他寄存器中转（如mov ax，0001H   mov ds，ax）



- 栈
  - 由栈段寄存器SS和寄存器SP（SS：SP）表示**栈顶**的物理地址
  - <img src=".\2.png" alt="image-20230527203227137" style="zoom:50%;" />
  - 当栈为空时，SS：SP指向栈的下一位（如图中例子则为1010H，即最下方的下一个）
  - 栈存储时从内存空间的最后一位开始存储，存储字时**先将高位入栈，再将地位入栈**
  - push和pop指令可以对物理地址上的数进行直接处理



- 标志寄存器：16位寄存器用来存标志位
  - 条件标志：
    CF：（进位、借位标志）用于加减时，最高位上产生的**进位或借位**的状态，0有、1无
    AF：（辅助进位标志）用来判断第3位上是否有进位或借位
    ZF：（0标志）用于记录运算结果是否为0，ZF=1则结果为0，ZF=0则结果不为0
    SF：（符号标志）用于记录运算结果的符号位SF=0则为正，SF=1则表负
    OF：（溢出标志）用于记录带符号数加减运算时的溢出状态，0则无溢出，1则有溢出
    PF：（奇偶标志）用于记录运算结果的低8位中1的个数，若PF=0则奇数个1，1则偶数个1，一般用于奇偶校验



## 第三章 常用的指令与技巧

|  指令   |         格式         |                             含义                             | 例子                       |                             备注                             |
| :-----: | :------------------: | :----------------------------------------------------------: | -------------------------- | :----------------------------------------------------------: |
|   mov   |    mov 寄存器，值    |                       将值赋给该寄存器                       | mov ax, 0                  |          *也可以是[偏移地址]（默认指DS：偏移地址）           |
|         |  mov 寄存器，寄存器  |               将寄存器或存储单元的值赋给寄存器               | mov ax,bx                  |                                                              |
|         | mov 段寄存器，寄存器 |                   将寄存器的值赋给段寄存器                   | mov ds,ax                  |                   *不能给段寄存器直接赋值                    |
|         |  mov ax, \[bx][SI]   |               将DS : bx+SI处内存的值赋给寄存器               | mov ax,\[bx][SI]           |                                                              |
| offset  |     offset 标签      |               获得标签处的数据或代码的偏移地址               | offset str                 |  *既可以获取变量的偏移地址，也可以获取代码上标签的偏移地址   |
|   cmp   |  cmp 寄存器，寄存器  |                    将两者数值相减但不保存                    | cmp ax,bx                  |                   *前者＜后者则CF标志位为1                   |
|         |    cmp 寄存器，值    |                                                              | cmp ax,3                   |                                                              |
|   db    |     变量名 db 值     |                  声明一个字节的空间存放数据                  | var1 db 10H                |           *声明字符串时会占用连续若干个字节的空间            |
|   dw    |     变量名 dw 值     |                       声明一个字的空间                       | var2 dw 1000H              |                                                              |
|   add   |    add 寄存器，值    |         将值与寄存器中的值相加，将结果保存在寄存器中         | add ax,1H                  |                *不处理进位，进位会携带CF标志                 |
|   adc   |    adc 寄存器，值    |                将值、CF的值与寄存器中的值相加                | adc ax,1H                  |                        *会多与CF相加                         |
|   sub   |    sub 寄存器，值    |                     与add差不多，但相减                      |                            |                           *会借位                            |
| inc/dec |      inc 寄存器      |                           自加1减1                           | inc ax                     |                                                              |
|   mul   |  mul 寄存器，寄存器  |                相乘，结果保存在前一个寄存器中                | mul ax,bx                  |          *8位乘8位结果为16位；16位乘16位结果为32位           |
|         |      mul 寄存器      |           将该寄存器中的值与al相乘，结果保存在ax中           | mul bh                     |                     *因为8位乘8位为16位                      |
|   div   |        div 值        |  **ax为16位，则ax与该值相除，商保存在ax中，余数保存在dx中**  | div 10                     |                     *16位除8位结果为8位                      |
|   aaa   |         aaa          |               将前面的加法运算结果改为BCD形式                | aaa                        | *如35H与39H相加（ASCII码分别是'5','9'），则最终的结果为0，4，CF=1 |
|   and   |  and 寄存器，寄存器  |                           按位取和                           | and ax,bx                  |                                                              |
|   or    |         ...          |                           按位取或                           | ...                        |                                                              |
|   xor   |         ...          |                          按位取异或                          | ...                        |                                                              |
| shl/shr |    shl 寄存器,值     |           将寄存器中值的二进制形式左（右）移若干位           | shl ax,1                   |                     *最后一位/第一位补0                      |
| sal/sar |         ...          |                             ...                              | ...                        |                 *最后一位/第一位保持**原值**                 |
| rol/ror |         ...          |                             ...                              | ...                        |      *将第一位的值赋给**最后一位**和**CF位**，反之亦然       |
| rcl/rcr |         ...          |                             ...                              | ...                        |  *将第一位的值赋给CF位，同时CF位的值赋给最后一位，反之亦然   |
|  loop   |      loop 标签       |         循环执行标签与loop之间的代码n次，n为CX中的值         | start:          loop start |                                                              |
|   jc    |       jc 标签        |                      当CF位不为0时跳转                       | jc stop                    |               *cmp ax,bx中**ax＜bx**时则jc跳转               |
|   ja    |       ja 标签        |                      当CF≠0，ZF≠0时跳转                      | ja stop                    |                  *实质为前者大于后者时跳转                   |
|   lea   |     lea SI,标签      |               从标签处读取字符串到SI指向的内存               | lea SI,str                 |                                                              |
|  movsb  |        movsb         | 将*地址SI*处的字符串**拷贝**到*地址DI*处字符串的后面，并将结果保存在*地址DI*处 | movsb                      |            *将DS:SI处的字符串拷贝到ES:DI处字符串             |
|         |      REP movsb       |                      循环拷贝直到DX = 0                      | REP movsb                  |                                                              |
|  cmpsb  |      REPE cmpsb      |                     当每个字符相等时循环                     |                            |               *还是比较DS:SI和ES:DI处的字符串                |
|         |      RENE cmpsb      |                    当每个字符不相等时循环                    |                            |                *比较完成后可以使用je或jne跳转                |



## 第四章 特殊数据类型的运算

- CISC（Complex Instruction Set Computer）复杂指令集如x86
- RISC（Reduced Instruction Set Computer）精简指令集如ARM

- 浮点数在计算机中的表示方法：
  - 一个单精度浮点数（float）占用32位/4字节的空间
  - 一个单精度浮点数分三部分：符号（sign），指数（exponent），有效数字（significand）
  - 例子：178.625D：
    - 整数部分：178D = 10110010B，小数部分：0.625D = 0.101B
    - 178.625D = 10110010.101B = 1.0110010101 × 2 ^ 7
    - 将数字 × 2 ^ **127**，则为1.**0110010101** × 2 ^ **134**
    - 将小数部分和指数部分取出并计算指数部分的二进制形式134D = 10000110B
    - 按照 符号，指数，有效数字的顺序组合。即0,10000110,01100101010000000000000
    - 转换为16进制则为 = (0100,0011,0011,0010,1010,0000,0000,0000)b = (4332A000)h
    - 其中符号占1位，指数占8位，有效数字占23位（即8位16进制数）
  - 8087栈：<img src="D:\作品们\AssemblyNote\5.png" alt="image-20230702205740918" style="zoom:67%;" />
  
  - 8087计算浮点数：
  
    - FADD ST(3), ST(0)		将ST(0)与ST(3)的值相加并将结果存入ST(3)
    - FADD							将ST(0)与ST(1)的值相加并将结果存入ST(0)，并从栈中弹出一个元素
    - FADD ST, ST(1)			将ST(0)与ST(1)的值相加并将结果存入ST(0)
    - FSUB							相减操作，结果保存在左边
    - FSUBR						反向相减，即右-左，将结果保存在左边
    
    ```assembly
    FINIT ; Set FPU to default state
    FLDCW cntrl ; Round even, Mask Interrupts
    FLD A ; Push A onto FP stack
    FMUL ST,ST(1) ; Multiply STST result on ST
    FMUL ST,ST(1) ; Multiply STST result on ST again to get A^3
    FLD B ; Push B onto FP stack
    FSQRT ; Square root number on stack (B)
    FADD ST,ST(1) ; ADD top two numbers on stack (A^3 + √B)
    FSTSW stat ; Load FPU status into [stat]
    mov ax,stat ; Copy [stat] into ax
    and al,0BFh ; Check all 6 status bits
    jnz pass ; If any bit set then jump
    FSTP RESULT ; Copy result from stack into RESULT
  
- 字符串操作

  - 直接看例子：

    ```assembly
    assume ds:dataseg
    assume cs:codeseg
    dataseg segment
    	str1 db 'This is a test$'
    	str2 db 'this string is a placeholder bulabula$'
    dataseg ends
    
    codeseg segment
    	mov ax, ds					; DS = ES
    	mov es, ax
    	LEA SI,str1
    	LEA DI,str2
    	
    	mov cx, 15
    	REP movsb					; 重复复制字符串直到CX=0
    	
    	mov dx, offset str2
    	mov ah, 09h					; 将字符串内容输出到控制台上直到$
    	int 021h
    codeseg ends
    #输出：This is a test
    ```




## 第五章 半导体

- 基本概念
  - 导电性：1. 有自由电子（free electron） 2. 有电子空缺（holes），即带正电
  - Semiconductor band theory：即从Valence band到Conduction band所需要的能量
- 三种半导体：
  - Pure-Silicon：纯硅导电性极差，需要加热来使电子脱离成为自由电子
  - N-type Silicon：通过在硅中加入磷（Phosphorus）来增加自由电子，以增加导电性
  - P-type Silicon：通过在硅中加入硼（Boron）来增加电子空缺的数量，以增加导电性
- PN Junction Diode：因为N和P级分别有自由电子和电子空缺，所以电子只能从N级到P级，电流的方向为单向
- repel排斥
- Junction Transistors：在base中的小电流会使二极管反转，此时在集流器中的电流可以通过
- MOS FETS：当gate有电压时，电流可以通过



## 第六章 IO与中断机制

- 8086处理器不仅有20位的内存地址总线来读写内存，还有一条用来表示地址空间的控制总线（control line）

- 用法：

  ```assembly
  IN des, port		;从port中读取一个字节到des中
  IN ax, dx			;从[dx]所代表的port中读取一个字节
  OUT sou, port		;将sou写入port一个字节
  ```

- 中断机制和轮循机制：中断机制可以使低优先级的任务被高优先级的任务中断，这允许CPU首先完成最重要的任务。
- 中断发生时的步骤：
  - suspends execution of the main program（停止正在运行的主程序）
  - 将正在运行程序的状态，和寄存器数据压入栈中，等待继续被调度（Context Switch）
  - 在中断向量表中寻找代码地址，并执行
- 三种中断类型：
  - **Hardware Interrupts**（硬件中断）
    - Hardware interrupts are triggered by devices outside of the CPU to inform it of tasks to take care of. （用来通知CPU处理外部设备）
  - **Exception Interrupts**（异常中断）
    - Exceptions are generated by the CPU itself when invalid instructions are executed.（CPU在异常时产生的中断，如除以0）
  - **Software Interrupts**（软件中断）
    - Software interrupts are coded by the programmers for input/output and easy debugging.（由程序员编程产生，用于快速调试和读写IO）
- **中断向量表**（Interrupt Vector Table）通常被保存在RAM的前1024个字节中，每个中断包含一个CS和IP，共4个字节，按地址顺序先是IP然后是CS，根据**低位先入，高位后入**的原则（litttle-endian）。
- IRQ0-IRQ7为系统中断，由CPU直接管理，不能被其他设备或软件所调用。
- 针对硬件中断，8086CPU只有两个引脚INTR，NMI，有一个额外芯片**8259A**控制器连接INTR引脚并提供了8条中断线来区分中断



## 第七章 内存，缓存和硬盘

- **Static Ram（SRAM）静态随机存取存储器**：由触发器构成，只要持续通电则可以一直保存数据，不需要刷新，速度较快，但还是易挥发（volatile）
- **Dynamic Ram（DRAM）动态随机存取存储器**：由 FET 构成，需要不断刷新来保存，成本较低，存储密度较高，易挥发
- 缓存（Cache）
  - 两种缓存策略：
    - Temporal locality (locality of time): If an item is referenced, it will tend to be referenced again soon.
    - Spatial locality (locality in space): If an item is referenced, nearby items will tend to referenced soon.
  - 三种缓存：
    - **Direct Mapped Cache**：将内存整个页都存入缓存，但当处理器频繁切换内存页时降低速度
    - **Two-Way Set Associative**：使用两个独立的缓存，允许内存页来回切换
    - **Fully Associative**
  - 缓存本质上是SRAM，即速度块，但容量小，成本高。
  - DMA（Direct Memory Access）：某些设备可以直接读取内存而为CPU节省时间
- **Read Only Memory（ROM）只读存储器**：
  - 只读存储器是不挥发（volatile）的储存器，如BIOS
  - 四种不同类型的ROM：
    - **ROM**：不能写入ROM，且不能编程
    - **PROM**：可为PROM进行一次性编程，通过吹制电路进行编程
    - **EPROM**：通过在FET上留下电荷来为EPROM编程，可重复编程
    - **EEPROM**：通过电流为EEPROM编程，也可以重复编程
- 虚拟内存：在内存不够用的时候，将一部分硬盘空间临时充当为内存使用
- 硬件存储：
  - Total storage = clusters * Each cluster contains
  - 对于FAT16来说，存储一个文件需要一个clusters ，即32768字节，如果文件过小则会浪费空间
  - 根目录的集群保存在FAT的第一个集群中，其中保存了下一个集群的集群号，这个过程将会一直持续

- 现代硬件存储：
  - **Flash RAM**：闪存可以保证读写高效率的同时，断电时还不会丢失数据。
  - **Solid State Drive (SSD)**：读写块，建立在印刷电路板上，遵循SATA原则
  - Flash Memory：读写快，而且没有机械结构，不容易坏，但成本很高
  - **Field Programmable Gate Array (FPGA)**：多功能块，可以被编程以完成不同的工作，且成本较低




## 第八章 CISC，RISC和Buses

- **CISC**（Complex Instruction Set）复杂指令集：含有大量的操作符，寻址模式和指令。可以使代码编写变得容易，但会导致处理器解码指令的效率降低
- **RISC**（RISC Reduced instruction set）精简指令集：由最常用的指令组成，处理器执行速度很快，但代码编写较难
- **Pipeline**：
  - 通过获取数据，解码，执行的并行操作加快执行速度
  
  - 管道冲突：
    - 资源冲突：不同指令可能需要操作同一资源，如内容和管线
    - 程序依赖：执行哪一条指令可能需要知道另一条指令的结果
    - 数据依赖：一条指令的执行可能需要依赖另一条指令的结果
- **Parallel** **Processors**
  - **SISD**: Single Instruction, Single Data stream. Uses a single processor to interpret single instructions on single pieces of data. Made parallel using pipelining.（单个处理器处理在单个数据段上的单个指令）
  - **SIMD**: Single Instruction, Multiple Data stream. Uses a single instruction to control a number of processing elements. Vector processor.（单个处理器处理多个指令）
  - **MIMD**: Multiple Instruction, Multiple Data stream. A set of processors simultaneously execute different instructions on different sets of data.（多个处理器处理多个数据段上的指令）
- **BUS**
  - **Address** **Bus**（地址总线）：CPU通过地址总线将所寻内存地址传递给内存，仅为单向
  - **Data Bus**（数据总线）：可以从内存中读取或者向内存中传输的数据线
  - **Control Bus**（控制总线）：CPU通过控制总线控制IO读写
- Tri-State：
  - 对于数据总线来说，因为具有双向性，所以同时占用数据总线会产生严重后果
  - 使用Tri-State设备时，会使用额外的一条控制线来切断单元与数据总线的连接























































