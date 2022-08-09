# D5. AArch64虚拟内存架构



本章的主要内包括以下小节：

- About the Virtual Memory System Architecture (VMSA) on page D5-2674.
  关于虚拟内存的架构（VMSA）
- The VMSAv8-64 address translation system on page D5-2682.
  VMSAv8-64 的地址转换过程
- VMSAv8-64 Translation Table format descriptors on page D5-2739.
  VMSAv8-64 的页表格式
- Memory access control on page D5-2754.
  内存访问控制
- Memory region attributes on page D5-2776.
  ？？？
- Virtualization Host Extensions on page D5-2787.
  关于虚拟化主机扩展？？
- Nested virtualization on page D5-2793.
  嵌套虚拟化
- VMSAv8-64 memory aborts on page D5-2800.
  VMSAv8-64 内存出错中止？
- Translation Lookaside Buffers (TLBs) on page D5-2810.
  快表TLB相关
- TLB maintenance requirements and the TLB maintenance instructions on page D5-2816.
  维护TLB的指令
- Caches in a VMSAv8-64 implementation on page D5-2835.  
  VMSAv8-64 的Cache？



## D5.1 About the Virtual Memory System Architecture (VMSA)  



 The mapping of a VA to a PA requires either a single stage of translation, or two sequential stages of translation.

The translations are defined independently for different Exception levels and Security states.

VMSAv8-64 supports tagging of VAs，tagging 分为 address tagging 和 memory tagging:

- Address tagging 的详细描述见下方<u>5.1.4 Address tagging in AArch64 state on page D5-2676</u>  .
  Address tagging 不影响地址转换过程。
- If FEAT_MTE2 is implemented, Memory tagging as described in Chapter D6 Memory Tagging Extension.（暂不考虑）



### D5.1.3 VMSA address types and address spaces

VMSA定义了三种地址类型：VA、IPA、PA



**虚拟地址 VA：**

An address held in the PC, LR, SP, or an ELR, is a VA.  

AArch64中，VA的位数支持：

- 48 bits.
- 52 bits when FEAT_LVA is implemented and the 64KB translation granule is used.
- 52 bits when all of the following are true:
  — FEAT_LPA2 is implemented.
  — TCR_ELx.DS==1 for the translation regime controlled by that register.
  — The 4KB or 16KB translation granule is used.



地址翻译阶段支持两种VA划分的方式：

- only a single VA range  仅划分一个范围

  For a translation stage that supports a single VA range, a 48-bit VA width gives a VA
  range of 0x0000000000000000 to 0x0000FFFFFFFFFFFF.
  For a translation stage that supports a single VA range, the 52-bit VA width gives a VA
  range of 0x0000000000000000 to 0x000FFFFFFFFFFFFF.

-  two VA ranges 虚拟地址划分两个部分来使用

  ① 第一部分起始地址为 0x0

  With a maximum VA width of 48 bits this gives a VA range of
  0x0000000000000000 to 0x0000FFFFFFFFFFFF.
  With a maximum VA width of 52 bits this gives a VA range of
  0x0000000000000000 to 0x000FFFFFFFFFFFFF.

  ② 第二部分结束地址为0xFFFFFFFFFFFFFFFF

  With a maximum VA width of 48 bits this gives a VA range of
  0xFFFF000000000000 to 0xFFFFFFFFFFFFFFFF.
  With a maximum VA width of 52 bits this gives a VA range of
  0xFFF0000000000000 to 0xFFFFFFFFFFFFFFFF.  



**中间物理地址 IPA：**

IPA存在于有两个translation stage的系统中，IPA是stage1的output address，同时是stage2的input address.



**物理地址 PA：**



### D5.1.4 Address tagging in AArch64 state  

Address tagging 与 Memory tagging 不是一个东西。



VA的高8位[56, 63]用来标记下列事件：

- If the translation system is enabled, whether the address is out of range and therefore causes a Translation fault.
- If the translation system is not enabled, whether the address is out of range and therefore causes an Address size fault.
- Whether the address requires invalidation when performing a TLB invalidation instruction by address.  



VA的第55位用来选择是否启用Address tagging：

| VA[55]==0 | TCR_ELx.TBI0 determines whether address tags are used. If stage 1 translation is enabled, TTBR0_ELx holds the base address of the translation tables used to translate the address. |
| --------- | ------------------------------------------------------------ |
| VA[55]==1 | TCR_ELx.TBI1 determines whether address tags are used. If stage 1 translation is enabled, TTBR1_ELx holds the base address of the translation tables used to translate the address. |





## D5.2 The VMSAv8-64 address translation system  



### D5.2.1 About the VMSAv8-64 address translation system  

MMU管理整个地址转换的过程。

When using a VMSA, a translation regime maps a VA to a PA using one or two stages of translation.



**The AArch64 translation regimes  — 两个阶段的地址转换**

两种不同的translation regimes：

- A single stage of address translation.
  This maps an input VA to an output PA.
- Two, sequential, stages of address translation, **中间多了个IPA**:
  — Stage 1 maps an input VA to an output IPA.
  — Stage 2 maps an input IPA to an output PA.  

![image-20210925163729653](C:\Users\wl\AppData\Roaming\Typora\typora-user-images\image-20210925163729653.png) 





**About address translation and supported input address ranges  — 地址转换与VA划分范围的关系**

For a single stage of address translation,   TTBR_ELx 指明了一级页表的地址。

对于支持两个范围VA的地址转换过程： each VA range is an independent mapping from IA to OA.这就意味着需要两套页表机制来支持。

> Note: IPA不会划分两个范围，这就意味着Stage 2 没有两套页表。



一次完整的页表查找过程被称为：translation table walk  

这个过程是完全由硬件（MMU）实现的。



**The VMSAv8-64 translation table format  — 页表格式**

VMSAv8-64 最高支持4级页表。

页（translation granule）的大小可以为4KB, 16KB, or 64KB.  



输入地址格式（VA或IPA）：

— Up to 52 bits when all of the following are true:
	— FEAT_LPA2 is implemented.
	— TCR_ELx.DS==1 for the translation regime controlled by that register.
	— The 4KB or 16KB translation granule is used.
— Up to 52 bits if FEAT_LVA is implemented and the 64KB translation granule is used.
— Otherwise, up to 48 bits.

输出地址格式（IPA或OV）：

— Up to 52 bits when all of the following are true:
	— FEAT_LPA2 is implemented.
	— TCR_ELx.DS==1 for the translation regime controlled by that register.
	— The 4KB or 16KB translation granule is used.
— Up to 52 bits if FEAT_LVA is implemented and the 64KB translation granule is used.
— Otherwise, up to 48 bits.





### D5.2.3 Controlling address translation stages  



If a stage of address translation supports two VA ranges then that stage of translation provides:

- A single TCR_ELx， controls the stage of address translation.
- 两个页表基地址寄存器（TTBR）。
  **TTBR0_ELx** points to the translation tables for the address range that starts at 0x0000000000000000（用户空间）, and **TTBR1_ELx** points to the translation tables for the address range that ends at 0xFFFFFFFFFFFFFFFF（内核空间）.



**MMU操作相关的寄存器**

![image-20210925172003012](C:\Users\wl\AppData\Roaming\Typora\typora-user-images\image-20210925172003012.png)





### D5.2.4 Memory translation granule size  

translation granule size  即代表物理页的大小，其实也等于一个页表的大小。因为页表的本质还是一个物理页，只不过其内部存储的是页表项。

两个Stage的页大小可以单独设定：

<img src="C:\Users\wl\AppData\Roaming\Typora\typora-user-images\image-20210925172830232.png" alt="image-20210925172830232"  />

![image-20210925172848418](C:\Users\wl\AppData\Roaming\Typora\typora-user-images\image-20210925172848418.png)

> Note：如果仅一个Stage的情况，就使用Stage 1的设定（EL2,EL3 ）。



### D5.2.5 Translation tables and the translation process  



TODO







### D5.2.9 The effects of disabling a stage of address translation 

禁用地址转换阶段的影响。

 

**当 stage 1 地址转换被禁用时，本应该在此阶段进行的地址翻译将：**

EL1 and EL0 accesses if the HCR_EL2.DC bit is set to 1：

For the EL1&0, when EL2 is enabled, the stage 1 转换分配 the Normal Non-shareable, Inner Write-Back Read-Allocate Write-Allocate, Outer Write-Back Read-Allocate Write-Allocate 属性的内存。This applies for both instruction and data accesses.



All other accesses：

For all other accesses, when stage 1 address translation is disabled, the assigned attributes depend
on whether the access is a data access or an instruction access, as follows:

- 数据访问：The stage 1 translation assigns the Device-nGnRnE memory type.

- 指令访问：

  由 SCTLR_ELx.I 位决定 :

  - 当SCTLR_ELx.I == 0：The stage 1 translation assigns the Non-cacheable and Outer Shareable attributes.  
  - 当SCTLR_ELx.I == 1：The stage 1 translation assigns the Cacheable, Inner Write-Through Read-Allocate No Write-Allocate, Outer Write-Through Read-Allocate No Write-Allocate Outer Shareable attribute.

  

Secure accesses and Non-secure accesses  

- For accesses from the Non-secure state, the output address is to the Non-secure output address space.
- For accesses from the Secure state, the output address is to the Secure output address space.  



For this stage of translation:

• 没有内存访问权限检查，因此也不会报MMU Permission faults 

• 不保留任何内存

对齐检查 is performed, 因此会产生 Alignment faults 





## D5.4 Memory access control  

内存访问控制。



Translation Table descriptors  中的访问控制字段决定了当前状态下此PE的内存访问权限。访问没有权限的内存会引发MMU fault。

以下部分描述了内存访问控制：

关于访问权限 on page D5-2754.
关于 PSTATE.PAN on page D5-2755.
关于 PSTATE.UAO on page D5-2756.
关于 PSTATE.BTYPE on page D5-2756.
关于数据访问控制 on page D5-2758.
关于指令执行权限 on page D5-2760.
访问标志 on page D5-2765.
The dirty state on page D5-2766.
软件配置访问标志位 on page D5-2766.
硬件配置访问标志位 and dirty state on page D5-2767.
Ordering of hardware updates to the translation tables on page D5-2773.
Restriction on memory types for hardware updates on translation tables on page D5-2773.
Use of the Contiguous bit with hardware updates of the translation table entries on page D5-2774.  



### D5.4.8 The dirty state  

The dirty state 表示一个或多个内存页已经被修改。

