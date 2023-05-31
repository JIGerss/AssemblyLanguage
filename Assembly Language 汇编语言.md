# [Assembly Language 汇编语言]()

## 第一章 基础知识

- 计算机将代码和数据都储存在内存中，如果不加以区分，则两者都为二进制形式，没有任何区别。
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

- 段的概念：

  - 内存本质上并没有分段，只是CPU将内存以分段的形式使用

  - 一个段的**最开始**单元的物理地址的16进制数总是以0结尾

  - 一个段所能包含的最大容量为64KB，即偏移地址 L(0000H ~ FFFFH) = 2 ^ 16 B = 2 ^ 6 KB = 64 KB

    

- 段寄存器

  - 段寄存器有CS，DS，SS，ES四种，用来储存对应数据的段地址
  - CS为代码段寄存器，储存代码数据在内存中的段地址。IP为指令地址寄存器，储存代码数据在内存中的偏移地址
  - 使用CS：IP的形式来表示（CS × 16 + IP）的物理地址
  - [偏移地址] 表示DS × 16 + *偏移地址*内存上的数据，其中数据可以为字也可以为字节（根据寄存器大小赋值，如mov ax，[0]则为将偏移地址为0处的字赋给ax）
  - 在内存中，**字**需要两个存储单元来存储，存储时低位在前，高位在后，以低位所在地址为其字的物理地址
  - 注意：不能将一个数字赋值给段寄存器（如mov ds，0001H），需要使用其他寄存器中转（如mov ax，0001H   mov ds，ax）



- 栈
  - 由栈段寄存器SS和寄存器SP（SS：SP）表示**栈顶**的物理地址
  - <img src=".\2.png" alt="image-20230527203227137" style="zoom:50%;" />
  - 当栈为空时，SS：SP指向栈的下一位（如图中例子则为1010H）
  - 栈存储时从内存空间的最后一位开始存储，存储字时先将高位入栈，再将地位入栈
  - push和pop指令可以对物理地址上的数进行直接处理



## 第三章 常用的指令与技巧

|  指令   |         格式         |                             含义                             | 例子                       |                           备注                            |
| :-----: | :------------------: | :----------------------------------------------------------: | -------------------------- | :-------------------------------------------------------: |
|   mov   |    mov 寄存器，值    |                       将值赋给该寄存器                       | mov ax, 0                  |         *也可以是[偏移地址]（默认指DS：偏移地址）         |
|         |  mov 寄存器，寄存器  |               将寄存器或存储单元的值赋给寄存器               | mov ax,bx                  |                                                           |
|         | mov 段寄存器，寄存器 |                   将寄存器的值赋给段寄存器                   | mov ds,ax                  |                  *不能给段寄存器直接赋值                  |
| offset  |     offset 标签      |               获得标签处的数据或代码的偏移地址               | offset str                 | *既可以获取变量的偏移地址，也可以获取代码上标签的偏移地址 |
|   cmp   |  cmp 寄存器，寄存器  |                    将两者数值相减但不保存                    | cmp ax,bx                  |                 *前者＜后者则CF标志位为1                  |
|         |    cmp 寄存器，值    |                                                              | cmp ax,3                   |                                                           |
|   db    |     变量名 db 值     |                  声明一个字节的空间存放数据                  | var1 db 10H                |          *声明字符串时会占用连续若干个字节的空间          |
|   dw    |     变量名 dw 值     |                       声明一个字的空间                       | var2 dw 1000H              |                                                           |
|   add   |    add 寄存器，值    |         将值与寄存器中的值相加，将结果保存在寄存器中         | add ax,1H                  |               *不处理进位，进位会携带CF标志               |
|   adc   |    adc 寄存器，值    |                将值、CF的值与寄存器中的值相加                | adc ax,1H                  |                       *会多与CF相加                       |
|   sub   |    sub 寄存器，值    |                     与add差不多，但相减                      |                            |                          *会借位                          |
| inc/dev |      inc 寄存器      |                           自加1减1                           | inc ax                     |                                                           |
|   mul   |  mul 寄存器，寄存器  |                相乘，结果保存在前一个寄存器中                | mul ax,bx                  |         *8位乘8位结果为16位；16位乘16位结果为32位         |
|   div   |        div 值        |  **ax为16位，则ax与该值相除，商保存在ax中，余数保存在dx中**  | div 10                     |                    *16位除8位结果为8位                    |
|   and   |  and 寄存器，寄存器  |                           按位取和                           | and ax,bx                  |                                                           |
|   or    |         ...          |                           按位取或                           | ...                        |                                                           |
|   xor   |         ...          |                          按位取异或                          | ...                        |                                                           |
| shl/shr |    shl 寄存器,值     |           将寄存器中值的二进制形式左（右）移若干位           | shl ax,1                   |                    *最后一位/第一位补0                    |
| sal/sar |         ...          |                             ...                              | ...                        |                 *最后一位/第一位保持原值                  |
| rol/ror |         ...          |                             ...                              | ...                        |     *将第一位的值赋给**最后一位**和**CF位**，反之亦然     |
| rcl/rcr |         ...          |                             ...                              | ...                        | *将第一位的值赋给CF位，同时CF位的值赋给最后一位，反之亦然 |
|  loop   |      loop 标签       |         循环执行标签与loop之间的代码n次，n为CX中的值         | start:          loop start |                                                           |
|   jc    |       jc 标签        |                      当CF位不为0时跳转                       | jc stop                    |             *cmp ax,bx中**ax＜bx**时则jc跳转              |
|   ja    |       ja 标签        |                      当CF≠0，ZF≠0时跳转                      | ja stop                    |                 *实质为前者大于后者时跳转                 |
|   lea   |     lea SI,标签      |               从标签处读取字符串到SI指向的内存               | lea SI,str                 |                                                           |
|  movsb  |        movsb         | 将*地址SI*处的字符串**拷贝**到*地址DI*处字符串的后面，并将结果保存在*地址DI处 | movsb                      |           *将DS:SI处的字符串拷贝到ES:DI处字符串           |
|         |      REP movsb       |                      循环拷贝直到DX = 0                      | REP movsb                  |                                                           |
|  cmpsb  |      REPE cmpsb      |                     当每个字符相等时循环                     |                            |              *还是比较DS:SI和ES:DI处的字符串              |
|         |      RENE cmpsb      |                    当每个字符不相等时循环                    |                            |              *比较完成后可以使用je或jne跳转               |
|         |                      |                                                              |                            |                                                           |



## 第四章 特殊数据类型的运算

- 浮点数在计算机中的表示方法：
  - 一个单精度浮点数（float）占用32位/4字节的空间
  - 一个单精度浮点数分三部分：符号（sign），指数（exponent），有效数字（significand）
  - 例子：178.625D：
    - 整数部分：178D = 10110010B，小数部分：0.625D = 0.101B
    - 178.625D = 10110010.101B = 1.0110010101 × 2 ^ 7
    - 将数字 × 2 ^ 127，则为1.**0110010101** × 2 ^ **134**
    - 将小数部分和指数部分取出并计算指数部分的二进制形式134D = 10000110B
    - 按照 符号，指数，有效数字的顺序组合。即0,10000110,01100101010000000000000
    - 其中符号占1位，指数占8位，有效数字占23位
  - ```assembly
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

    





























































