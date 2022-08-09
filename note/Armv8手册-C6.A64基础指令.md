# D6. A64基础指令



# 5: An Introduction to the ARMv8 Instruction Sets

《Programmer's Guide for ARMv8-A》



## 5.1 ARMv8 指令集

新引入的A64指令集有以下改进：

- 统一的编码方案

- 更大的立即数范围

  A64指令集对于每种类型的指令定制不同的立即数范围：

  - 算数指令通常支持12位的立即数
  - 逻辑指令通常支持32位或64位的立即数，在其编码中有一些约束（？）
  - mov指令支持16位的立即数
  - 地址生成指令被调整为与4KB页面大小对齐的地址

- 更简单的数据类型

  A64可以很自然地处理64位有符号和无符号数据类型。

- 更长的offset

- 指针

- 使用条件结构代替IT块

- 更直观的shift和rotate操作

- 生成代码

- 定长的指令

- 对于三个参数的支持更好

### 5.1.1 区分 32-bit和64-bit A64指令

Most integer instructions in the A64 instruction set have two forms, which operate on either 
32-bit or 64-bit values within the 64-bit general-purpose register file. 

通过指令使用的寄存器名称来区分：

- If the register name starts with X, it is a 64-bit value.
- If the register name starts with W, it is a 32-bit value.



当选择32位指令形式时：

- Right shifts and rotates inject at bit 31, instead of bit 63.
- The condition flags, where set by the instruction, are computed from the lower 32 bits.
- Writes to the W register set bits [63:32] of the X register to zero.



### 5.1.2 Addressing

当单个寄存器可以存储64bit数据后，可寻址的空间就变得更大了，即支持更大的内存地址。

在寻址方面A64还有一些别的提升：

- Exclusive accesses

- 支持更大的PC相对寻址

- 非对齐内存支持

- 批量传输

- 加载/存储

  现在所有的加载/存储指令都使用容易的寻址模式。这使计算机处理char、short和int等类型的数据更加容易。

- 对齐检查

  当处于AArch64执行状态时，获取指令、借助sp加载/存储数据的操作会进行对齐检查，即对pc和sp的检查。

   这种方式比强制的PC和SP对齐检查更可取，因为PC或SP的非对齐通常表明软件错误，例如软件中不正确的地址。

  有以下几种类型的对齐检查：

  - AArch64中，无论何时当试图执行一个地址不对齐的指令时，都会产生一个与指令获取相关异常。ESR_ELx寄存器中可以获取相应的异常信息。

    非对齐的PC意味着其值的低两位非00.

    若在AArch64中处理该异常，则发生异常的地址（非对齐的PC值）将存储在FAR_ELx寄存器中。

    AARch32中，PC对齐检查属于Data Abort的一部分。

  - 当使用堆栈指针作为AArch64中的基地地址加载或存储时，堆栈指针(SP)对齐检查生成一个与数据内存访问相关的异常。

    当sp用于计算的基地址时，一个不对齐的堆栈指针是其位[3:0]不是0000。当堆栈指针被用作基址时，它必须是16字节对齐的。

    sp对齐检查只存在与AArch64当中，并且可以在每个异常等级中独立选择开关：

    - EL0 and EL1 are controlled by two separate bits in SCTLR_EL1.
    - EL2 is controlled by a bit in SCTLR_EL2.
    - EL3 is controlled by a bit in SCTLR_EL3.

- 

  



# 6: The A64 指令集

《Programmer's Guide for ARMv8-A》



## 6.3 内存访问指令

与所有之前的ARM处理器一样，ARMv8架构是一个Load/Store架构。这意味着没有数据处理指令直接对内存中的数据进行操作。数据必须首先装入寄存器，修改，然后存储到内存中。程序必须指定一个地址、要传输的数据的大小和源寄存器或目的寄存器。另外还有Load和Store指令，提供了更多的选项，如non-temporal Load/Store、Load/Store exclusive和Acquire/Release。

### 6.3.1 Load指令格式

Load指令的通用格式可以总结为：

```assembly
LDR Rt, <addr>
```

对于加载到整数寄存器，您可以选择要加载的大小。 例如，要加载小于指定寄存器值的大小，可以在LDR指令后面添加以下后缀之一：

- LDRB (8-bit, zero extended).
- LDRSB (8-bit, sign extended).
- LDRH (16-bit, zero extended).
- LDRSH (16-bit, sign extended).
- LDRSW (32-bit, sign extended).

zero extended代表用0填充多余的bit，同样sign extended代表用符号扩展。

您不需要指定对X寄存器的zero-extended，因为写入W寄存器是默认对其他位实行zero-extended。

![image-20211111163747520](https://gitee.com/lamperM/picgo_picture_bed/raw/master/img/image-20211111163747520.png)



### 6.3.2 Store指令格式

Store指令的通用格式可以总结为：

```assembly
STR Rt, <addr>
```

要存储的大小可能比寄存器小。你可以通过在STR中添加一个B或H后缀来指定它。



### 6.3.4 指定Load/Store指令的内存地址

在A64中，**地址操作数的基寄存器必须总是X寄存器**。然而，一些指令支持零扩展或符号扩展，因此32位偏移量可以由W寄存器提供。



> 偏移模式

偏移寻址模式将立即数或可修改的寄存器值添加到64位基地址寄存器以生成地址。

![image-20211111164532398](https://gitee.com/lamperM/picgo_picture_bed/raw/master/img/image-20211111164532398.png)

通常，数组下标的变换操作可以通过移位操作来实现：

![image-20211111164908347](https://gitee.com/lamperM/picgo_picture_bed/raw/master/img/image-20211111164908347.png)



> 索引模式

索引模式与偏移模式类似，但它们也更新基本寄存器。语法与A32和T32中相同，但操作集有更多限制。通常，索引模式只能支持立即数偏移。

有两种变体:pre-index模式在访问内存之前应用偏移量，post-index模式在访问内存之后应用偏移量。

![image-20211111165154101](https://gitee.com/lamperM/picgo_picture_bed/raw/master/img/image-20211111165154101.png)

C语言中也经常会用到这种方式：

![image-20211111170346534](https://gitee.com/lamperM/picgo_picture_bed/raw/master/img/image-20211111170346534.png)



> 相对寻址 (load-literal)

A64添加了另一种专门用于访问文字池的寻址模式。

文字池是被编码指令流中的数据块。池不会被执行，但是它们的数据可以使用pc相关的内存地址从周围的代码中访问。字面值池通常用于编码不适合使用mov-immediate指令的常量值。

在A32和T32中，PC可以像通用寄存器一样读取，因此可以通过将PC指定为基本寄存器来访问文字池。

在A64中，PC通常是不能访问的，但是有一个特殊的寻址模式(仅用于加载指令)来访问PC相关的地址。这种专用寻址模式的范围也比A32和T32中的PC-relative加载要大得多。

![image-20211111171338789](https://gitee.com/lamperM/picgo_picture_bed/raw/master/img/image-20211111171338789.png)

 \<label> must be 4-byte-aligned for 所有的变体。











# 

## Overview of AArch64 state  

### Registers in AArch64 state  

《Arm instruction set reference guide》

AArch64执行状态下，一下寄存器是可使用的：

- 31个64位通用寄存器：X0-X30
- 4个栈指针寄存器：SP_EL0, SP_EL1, SP_EL2, SP_EL3  
- 三个异常链接寄存器：ELR_EL1, ELR_EL2, ELR_EL3  
- 三个用于保存程序状态寄存器：SPSR_EL1, SPSR_EL2, SPSR_EL3.  
- 一个PC

除了三个程序状态寄存器之外，所有的寄存器都是64位。

大部分的A64指令既可以操作64位可以操作32位寄存器。通用寄存器的位宽通过其名字就可以得知，X代表64位，W代表32位。Wn和Xn（n从0到30）是指向同一个寄存器，只不过Wn只使用后32位，前32位将被忽略。

WZR和XZR分别是32位和64位的零值寄存器。



**AArch64中通用寄存器作为参数的分类**

函数调用时，通用寄存器可以按照作用进行如下分类：

1. 参数寄存器（X0-X7）

   x0-x7用于传递函数参数和返回值。callee开头可以保存他们，使他们能在函数内部用于保存临时变量。与AArch32相比，更多的参数寄存器保证减少堆栈的使用。

2. caller-saved临时寄存器（x9-x15）

   caller-saved的意思是如果调用者（caller）希望使用这些寄存器，必须在调用函数之前就保存他们。因为在函数中这些寄存器都会被当作临时寄存器使用。

3. callee-saved寄存器（x19-x29）

   这些寄存器是进入函数后，被保存到被调用者（callee）的栈中的。同样，函数结束之前会被恢复。

4. 特殊用途寄存器（X8, X16-X18, X29, X30）

   X8是间接结果寄存器。这用于传递间接结果的地址位置，例如，函数返回一个大的结构体。callee不需要保存。

   X16和X17是IP0和IP1，内部调用临时寄存器。callee不需要保存，一般不要修改。

   X18是平台寄存器，为平台ABIs保留使用，没有特殊的含义。callee不需要保存。

   X29是帧指针寄存器(FP)。callee需要保存和恢复它。

   X30是链接寄存器(LR)。callee需要保存和恢复它。

5. 









### 链接寄存器LR

《Arm instruction set reference guide》

在AArch64中，LR寄存器存储返回地址。如果返回地址被存在栈中，那么LR也可以当作通用寄存器使用。**LR对应与X30.**

与AArch32不同，LR和ELRs寄存器并不相同，所以他们并不能相互替代。

任何异常等级发生子函数调用，返回地址都存储在LR中。

一共有三个异常链接寄存器ELR_EL1, ELR_EL2和 ELR_EL3 ，他们与三个异常等级相对应。当异常发生时，异常处理所在异常等级对应的ELR寄存器存储着异常处理完毕后需要的返回地址。例如当异常等级从EL0切换到EL1，那么返回地址存储在ELR_EL1。

那么就必须考虑一下情况：如果某个异常要在当前等级处理，那么必须保存当前ELR寄存器的值，因为异常处理时会覆盖。

如果异常来自AArch32，那么对应的ELR的高32位被置0。



### 栈指针寄存器SP

《Arm instruction set reference guide》

在AArch64中，SP保存64位及栈指针。

SP_EL0是SP的别名，不要将SP作为通用寄存器使用。

以下情况可以将SP做为指令的操作数：

- 作为存储或加载的base register。此时在加任何偏移之前都要保证SP是4字节对齐的，否则出发栈指针对齐异常。
- 作为算数指令的源或目的寄存器，但是不能作为设置条件标志指令的目的寄存器。
- 在逻辑指令中，例如对齐。

三个异常等级都有一个独立的栈指针，SP_EL1, SP_EL2, and SP_EL3。在这三个异常等级中，你可以配置选择使用他们自己的栈指针寄存器SP_ELx，也可以选择统一使用SP_EL0。这个配置通过修改SPSel寄存器来实现。

异常等级的命名可以显示它使用的是自己的SP_ELx还是SP_EL0，后缀h代表使用自己等级下的SP，后缀t代表统一使用SP_EL0.
例如，EL0只能使用SP_EL0，所以EL0只有EL0t，而EL1就可以有EL1t和EL1h两种名称。



### 帧指针寄存器FP

（计算机组成与设计P107）

FP指向该栈帧的第一个doubleword，而栈指针SP指向栈顶。栈指针需要调整位置存放需要保存的寄存器和驻留内存的局部变量，所以在程序运行期间栈指针随时可改变。所以对于编程人员，虽然使用栈指针和少量的地址运算可以完成对变量的引用，但使用固定帧指针SP会更简单。如果在一个过程中栈内没有局部变量，编译器将可以不设置和不回复帧指针来节约时间。



### C6.1.2 Use of the PC  

A64指令集对PC寄存器的访问进行了限制。

只有以下指令能读PC的值：

- ADRADR and ADRP.  
- The Load register (literal) instruction class  
- Direct branches that use an immediate offset.  
- 无条件跳转指令：BL和BLR。他们使用PC+1（下一条指令）作为返回地址（LR）。

只有以下指令能够修改PC的值：

- 有条件/无条件转移指令和返回指令，分别执行PC=label, PC=LR来跳转。
- 异常的产生和异常返回。

### C6.1.3 Use of the stack pointer  

A64指令下只有以下情况可以使用堆栈指针SP：

- Load/store instructions use the current stack pointer as the base address:
  - 当系统启用了堆栈对齐检查时，SP必须是四字节对齐的。否则会产生SP alignment fault.
- ADD和SUB指令。
- Logical data processing instructions



### Conditional execution in AArch64 state  

《Arm instruction set reference guide》

In AArch64 state, the NZCV register holds copies of the N, Z, C, and V condition flags. The processor uses them to determine whether or not to execute conditional instructions. The NZCV register contains the flags in bits[31:28].  

条件标志所有异常等级下均可以访问，使用MRS和MSR即可。

相比A32，A64很少操作条件标志，例如：

- 很少指令可以设置条件标志
- 没有与T32 IT 指令的同等指令
- 唯一有条件执行的指令是条件分支B.cond，它在条件为false的情况下充当NOP。



### Process State  

AArch64中没有Current Program Status Register (CPSR) 寄存器，各种信息被划分为多个Process State 字段。

Process State 字段包含：

- N, Z, C, and V condition flags (NZCV).
- 当前寄存器宽度 (nRW).
- 栈指针选择 (SPSel).
- 中断禁用标志位 (DAIF).
- 当前异常等级 (EL).
- Single step process state bit (SS).
- 非法异常返回状态 (IL).  

其中可以通过MSR指令来修改的：

- N, Z, C, and V condition flags (NZCV).
- 栈指针选择 (SPSel).
- 中断禁用标志位 (DAIF).



### Saved Program Status Registers(SPSR) in AArch64 state

在AArch64中，所有的异常等级都有自己的SPSR寄存器（SPSR_ELx）。

当中断需要在AArch64的异常等级下被处理时，SPSR寄存器保存相关的处理器上下文。

例如，当异常在EL1下发生时需要在EL2中处理的情况时，使用SPSR_EL2存储EL1的处理器上下文。

SPSR存储包含以下字段信息：

- N, Z, C, and V flags.
- D, A, I, and F interrupt disable bits.
- The register width.
- The execution mode.
- The IL and SS bits.  









## C6.2  A64 base instructions（字母顺序排列）

### C6.2.45 CBZ  

指令格式：

CBZ \<Xt>, \<label>

实现方式：Compare and Branch on Zero compares the value in a register with zero,  and conditionally branches to a label at a PC-relative offset if the comparison is equal. It provides a hint that this is not a subroutine call or return. This instruction does not affect condition flags.  

### C6.2.132 LDR (immediate) 

> 指令格式：

LDR \<Xt>, [<Xn|SP>], #\<simm>

实现方式：Load Register (immediate) loads a word or doubleword from memory and writes it to a register.  

### C6.2.133 LDR (literal)  

LDR \<Xt>, \<label>  

实现方式：Load Register (literal) calculates an address from the PC value and an immediate offset, loads a word from memory, and writes it to a register. 

